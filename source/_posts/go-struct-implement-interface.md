---
title: go struct implement interface
date: 2022-06-14 16:05:36
tags:
---

| 编译结果 | 结构体实现接口	 | 结构体指针实现接口 |
| :---: | :---: | :---: |
| 结构体初始化变量	 | 通过 | 不通过 |
| 结构体指针初始化变量 | 通过 | 通过 |

```go
package main

import "fmt"

// Ducker 鸭子类型
type Ducker interface {
	Quack()
}

// Cat cat
type Cat struct{}

// Quack cat quack
func (c Cat) Quack() {
	fmt.Println("cat quack")
}

// Dog dog
type Dog struct{}

// Quack dog duack
func (d *Dog) Quack() {
	fmt.Println("dog quack")
}

func main() {
	// 第一种情况，结构体初始化变量，结构体实现接口
	var c1 Ducker = Cat{}
	c1.Quack()

	// 第二种情况，结构体指针初始化变量，结构体实现接口
	var c2 Ducker = &Cat{}
	c2.Quack()

	// 第三种情况，结构体初始化变量，结构体指针实现接口
	// compiler
	// cannot use (Dog literal) (value of type Dog) as Ducker value in variable declaration:
	// missing method Quack (Quack has pointer receiver)
	// var d1 Ducker = Dog{}
	// d1.Quack()

	// 第四种情况，结构体指针初始化变量，结构体指针实现接口
	var d2 Ducker = &Dog{}
	d2.Quack()
}

```

- 结构体指针初始化变量，结构体实现接口时，能够通过指针隐式地获取到指向的结构体
- 结构体初始化变量，结构体指针实现接口时，变量缺少指针指向它

## 参考

- [Go 语言接口原理](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/#%E7%BB%93%E6%9E%84%E4%BD%93%E7%B1%BB%E5%9E%8B)