---
title: go unit test db
date: 2022-06-14 16:04:31
tags:
---

```go
// main.go
package main

import (
	"fmt"
	"log"

	"github.com/pkg/errors"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

// Settle 结算系统接口
type Settle interface {
	GetSettleItem(id int) (*SettleItem, error)
	GetSettleRule(id int) (*SettleRule, error)
}

// settle 结算系统接口实现
type settle struct {
	db *gorm.DB
}

// SettleItem 结算项
type SettleItem struct {
	ID   int    `gorm:"column:id"`
	Name string `gorm:"column:name"`
}

// TableName 结算项表名
func (SettleItem) TableName() string {
	return "t_settle_item"
}

// SettleRule 结算规则
type SettleRule struct {
	ItemID int     `gorm:"column:item_id"`
	Month  string  `gorm:"column:month"`
	Price  float64 `gorm:"column:price"`
}

// TableName 结算规则表面
func (SettleRule) TableName() string {
	return "t_settle_rule"
}

// NewSettle 创建实例
func NewSettle(db *gorm.DB) Settle {
	return &settle{db: db}
}

// GetSettleItem 根据 ID 获取结算项
func (s *settle) GetSettleItem(id int) (*SettleItem, error) {
	var i SettleItem
	if err := s.db.First(&i, id).Error; err != nil {
		return nil, errors.Wrap(err, "get settle item error")
	}
	return &i, nil
}

// GetSettleRule 根据结算项 ID 获取结算规则
func (s *settle) GetSettleRule(id int) (*SettleRule, error) {
	var r SettleRule
	if err := s.db.First(&r, "item_id = ?", id).Error; err != nil {
		return nil, errors.Wrap(err, "get settle rule error")

	}
	return &r, nil
}

func main() {
	dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal(err)
	}
	s := NewSettle(db)
	item, err := s.GetSettleItem(1)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(item)
}

```
```go
// main_test.go
package main

import (
	"io/ioutil"
	"log"
	"os"
	"testing"

	"github.com/pkg/errors"
	. "github.com/smartystreets/goconvey/convey"
	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
)

const dbFile = "./test.db"

var db *gorm.DB

// TestMain 单测初始化
func TestMain(m *testing.M) {
	if err := InitSQLite(); err != nil {
		log.Fatal(err)
	}
	if err := ExecSQLite("test.sql"); err != nil {
		log.Fatal(err)
	}
	code := m.Run()
	if err := DelSQLite(); err != nil {
		log.Fatal(err)
	}
	os.Exit(code)
}

// InitSQLite init sqlite
func InitSQLite() error {
	if err := DelSQLite(); err != nil {
		return err
	}
	var err error
	db, err = gorm.Open(sqlite.Open(dbFile), &gorm.Config{})
	if err != nil {
		return errors.Wrap(err, "init sqlite error")
	}
	if err := db.AutoMigrate(&SettleItem{}); err != nil {
		return errors.Wrap(err, "init sqlite error")
	}
	if err := db.AutoMigrate(&SettleRule{}); err != nil {
		return errors.Wrap(err, "init sqlite error")
	}
	return nil
}

// ExecSQLite 根据 sql 文件执行 sql 语句
func ExecSQLite(sqlFile string) error {
	data, err := ioutil.ReadFile(sqlFile)
	if err != nil {
		return errors.Wrap(err, "exec sqlite error")
	}
	s := string(data)
	if err := db.Exec(s).Error; err != nil {
		return errors.Wrap(err, "exec sqlite error")
	}
	return nil
}

// DelSQLite 删除 db 文件
func DelSQLite() error {
	if _, err := os.Stat(dbFile); os.IsNotExist(err) {
		return nil
	}
	err := os.Remove(dbFile)
	if err != nil {
		return errors.Wrap(err, "del sqlite error")
	}
	return nil
}

func TestNewSettle(t *testing.T) {
	Convey("NewSettle", t, func() {
		obj := NewSettle(db)
		So(obj, ShouldNotBeNil)
	})
}

func Test_settle_GetSettleItem(t *testing.T) {
	Convey("GetSettleItem", t, func() {
		obj := NewSettle(db)
		i, err := obj.GetSettleItem(1)
		So(err, ShouldBeNil)
		So(i.Name, ShouldEqual, "1")
	})
	Convey("GetSettleItem not found", t, func() {
		obj := NewSettle(db)
		_, err := obj.GetSettleItem(2)
		So(errors.Is(err, gorm.ErrRecordNotFound), ShouldEqual, true)
	})
}

func Test_settle_GetSettleRule(t *testing.T) {
	Convey("GetSettleRule", t, func() {
		obj := NewSettle(db)
		i, err := obj.GetSettleRule(1)
		So(err, ShouldBeNil)
		So(i.Price, ShouldEqual, 1)
	})
	Convey("GetSettleRule not found", t, func() {
		obj := NewSettle(db)
		_, err := obj.GetSettleRule(2)
		So(errors.Is(err, gorm.ErrRecordNotFound), ShouldEqual, true)
	})
}

```
```sql
INSERT INTO `t_settle_item` (`id`, `name`) VALUES (1, '1');
INSERT INTO `t_settle_rule` (`item_id`, `month`, `price`) VALUES (1, '202109', 1);
```