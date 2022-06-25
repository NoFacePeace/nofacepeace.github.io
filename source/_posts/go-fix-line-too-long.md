---
title: go fix line too long
date: 2022-06-14 16:03:25
tags:
---


## 常见的行太长类型

- 参数过多
- 字符串太长
- 判断条件太多
- 链式调用太长

```go
package main

import (
	"fmt"

	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

func main() {
	// argument too long
	arg1 := "argument one"
	arg2 := "argument two"
	arg3 := "argument three"
	arg4 := "argument four"
	arg5 := "argument five"
	arg6 := "argument six"
	arg7 := "argument seven"
	fmt.Println(arg1, arg2, arg3, arg4, arg5, arg6, arg7)
	// fix
	fmt.Println(
		arg1,
		arg2,
		arg3,
		arg4,
		arg5,
		arg6,
		arg7,
	)

	// string too long
	sql := "SELECT id, name, info FROM t_user user LEFT JOIN t_info info ON user.id = info.user_id where name != ''"
	fmt.Println(sql)
	// fix
	sql = `
SELECT
  id,
  name,
  info
FROM
  t_user user
  LEFT JOIN t_info info ON user.id = info.user_id
where
  name != ''
  and info != ''
`
	fmt.Println(sql)
	// or fix
	sql = "SELECT id, name, info FROM t_user user " +
		"LEFT JOIN t_info info ON user.id = info.user_id " +
		"where name != '' and info != ''"
	fmt.Println(sql)

	// if condition too long
	if arg1 == arg2 && arg2 == arg3 && arg3 == arg4 && arg4 == arg5 && arg5 == arg6 && arg6 == arg7 {
		fmt.Println("pass")
	}
	// fix
	if arg1 == arg2 &&
		arg2 == arg3 &&
		arg3 == arg4 &&
		arg4 == arg5 &&
		arg5 == arg6 &&
		arg6 == arg7 {
		fmt.Println("pass")
	}

	// function chain call too long
	db, err := gorm.Open(mysql.Open(""), &gorm.Config{})
	if err != nil {
		fmt.Println(err)
		return
	}
	db.Table("table").Select("id", "name", "info").Where("name != ? and info != ?", "", "").Find(nil)
	// fix
	db.Table("table").
		Select("id", "name", "info").
		Where("name != ? and info != ?", "", "").
		Find(nil)
}

```

```bash
go run main.go
argument one argument two argument three argument four argument five argument six argument seven
argument one argument two argument three argument four argument five argument six argument seven
SELECT id, name, info FROM t_user user LEFT JOIN t_info info ON user.id = info.user_id where name != ''

SELECT
  id,
  name,
  info
FROM
  t_user user
  LEFT JOIN t_info info ON user.id = info.user_id
where
  name != ''
  and info != ''

SELECT id, name, info FROM t_user user LEFT JOIN t_info info ON user.id = info.user_id where name != '' and info != ''

2021/10/08 19:21:11 /Users/holtchen/Desktop/go/main.go:69
[error] failed to initialize database, got error dial tcp 127.0.0.1:3306: connect: connection refused
dial tcp 127.0.0.1:3306: connect: connection refused
```


