---
title: go error
date: 2022-06-14 16:01:34
tags:
---


## template.go
```go
// License

// Package doc 包内容描述
//
// Heading 标题一
//
// 描述一
//
// Heading 标题二
//
// 描述二
//
// Code Blocks
//
// 代码块
// 	fmt.Printf("Hello, World")
//
// Links
//
// 链接
//
// https://www.google.com
package doc

import "fmt"

// ConstOne 常量一
const ConstOne = "const one"

// ConstTwo 常量二
const ConstTwo = "const two"

// Const 集合
const (
	ConstThree = "const three"
	ConstFour  = "const four"
)

// VarOne 全局变量一
var VarOne = "var one"

// VarTwo 全局变量二
var VarTwo = "var two"

// Var 变量集合
var (
	VarThree = "var three"
	VarFour  = "var four"
)

// Namer 获取名字
type Namer interface {
	GetName() string
}

// FunctionOne 函数一
func FunctionOne() {
	fmt.Println("function one")
}

// FunctionTwo 函数二
func FunctionTwo() {
	fmt.Println("function two")
}

// StructOne 结构体一
type StructOne struct{}

// NewStructOne 创建结构体一实例
func NewStructOne() *StructOne {
	return &StructOne{}
}

// GetName 返回结构体名称
func (StructOne) GetName() string {
	return "StructOne"
}

// StructTwo 结构体二
type StructTwo struct{}

```

## example_test.go
```go
package doc

import "fmt"

// Example 包示例一
func Example_one() {
	fmt.Println("example one")

	// Output:
	// example one
}

// Example 包示例二
func Example_two() {
	fmt.Println("example two")

	// Output:
	// example two
}

// ExampleNamer 接口示例
func ExampleNamer() {
	fmt.Println("namer")
}

// ExampleStructOne 结构体一示例
func ExampleStructOne() {
	fmt.Println("const one")
}

// ExampleNewStructOne 函数示例
func ExampleNewStructOne() {
	fmt.Println("new struct one")
}

// ExampleStructOne_GetName 方法示例
func ExampleStructOne_GetName() {
	fmt.Println("get name")
}

```

## 参考
https://elliotchance.medium.com/godoc-tips-tricks-cda6571549b
