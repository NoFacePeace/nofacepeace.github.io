---
title: Golang 优雅地使用函数类型实现接口
date: 2022-05-11 14:32:08
tags:
  - Golang
category: 后端
---

> Golang 支持将函数定义为类型，类型又能实现接口，这样就可以让函数类型实现接口，这样有什么好处呢？我们可以从一个实际的场景来体会一下这种用法。

<!-- more -->

## 场景

假设我们需要一个基于 Web 实现的打招呼功能。
当有请求访问时，如果请求要求皮卡丘出来接待，它会回复：皮卡皮卡；
如果请求要求妙蛙种子出来接待，它会回复：种子种子；
如果请求没有任何要求，那么默认回复：Hello World！

## 实现

1. 先起一个 Web 服务

```go
func main() {
  http.Handle("/", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello, World!")
  }))
  log.Fatal(http.ListenAndServe("localhost:8080", nil))
}
```

1. 定义一个说话的接口

```go
// Sayer 定义说话的接口
type Sayer interface {
	Say() string
}
```

1. 定义皮卡丘的结构体，并实现说话接口

```go
// Pikachu 皮卡丘
type Pikachu struct{}

// Say 皮卡丘说话
func (p Pikachu) Say() string {
	return "皮卡 皮卡!"
}
```

1. 定义妙蛙种子的结构体，并实现说话接口

```go
// Bulbasaur 妙蛙种子
type Bulbasaur struct{}

// Say 妙蛙种子说话
func (b Bulbasaur) Say() string {
	return "种子 种子!"
}
```

1. 实现一个函数，用来默认响应请求

```go
// SayFunc 说话的函数
type SayFunc func() string

// Say SayFunc 实现 Sayer 接口
func (s SayFunc) Say() string {
	return s()
}

// DefaultSay 默认说话
func DefaultSay() string {
	return "Hello, World!"
}
```

1. 定义一个 map，基于表驱动的方式响应请求

```go
m := map[string]Sayer{
			"pikachu":   Pikachu{},
			"bulbasaur": Bulbasaur{},
			"default":   SayFunc(DefaultSay),
		}
```

1. 实现处理请求的 handler

```go
http.Handle("/greet", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		query := r.URL.Query()
		who, ok := query["who"]
		if !ok {
			fmt.Fprintf(w, "say: %s", m["default"].Say())
			return
		}
		if _, ok := m[who[0]]; !ok {
			fmt.Fprintf(w, "say: %s", m["default"].Say())
			return
		}
		fmt.Fprintf(w, "%s say: %s", who[0], m[who[0]].Say())
	}))
```

1. 完整代码

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.Handle("/", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, World!")
	}))

	m := map[string]Sayer{
		"pikachu":   Pikachu{},
		"bulbasaur": Bulbasaur{},
		"default":   SayFunc(DefaultSay),
	}
	http.Handle("/greet", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		query := r.URL.Query()
		who, ok := query["who"]
		if !ok {
			fmt.Fprintf(w, "say: %s", m["default"].Say())
			return
		}
		if _, ok := m[who[0]]; !ok {
			fmt.Fprintf(w, "say: %s", m["default"].Say())
			return
		}
		fmt.Fprintf(w, "%s say: %s", who[0], m[who[0]].Say())
	}))

	log.Fatal(http.ListenAndServe("localhost:8080", nil))
}

// Sayer 定义说话的接口
type Sayer interface {
	Say() string
}

// SayFunc 说话的函数
type SayFunc func() string

// Say SayFunc 实现 Sayer 接口
func (s SayFunc) Say() string {
	return s()
}

// DefaultSay 默认说话
func DefaultSay() string {
	return "Hello, World!"
}

// Pikachu 皮卡丘
type Pikachu struct{}

// Say 皮卡丘说话
func (p Pikachu) Say() string {
	return "皮卡 皮卡!"
}

// Bulbasaur 妙蛙种子
type Bulbasaur struct{}

// Say 妙蛙种子说话
func (b Bulbasaur) Say() string {
	return "种子 种子!"
}
```

## 优点

从上述场景分析，我们定义了 Sayer 的接口，对于不同的招待对象，我们都得先定义结构体，然后用这个结构体实现 Sayer 接口。
例如皮卡丘和妙蛙种子，先定义 struct 再实现 Say 方法。
然而，有时候我们并不需要具体的对象，或者我们不在乎对象是谁，只需要有函数能满足处理即可，但是接口必须有类型才能实现，所以我们的代码可能写成这样

```go
type Default1 struct{}
func (d Default1) Say() string {
  return "Hello, World!"
}

type Default2 struct{}
func (d Default2) Say() string {
  return "Hello, World!"
}
m := map[string]Sayer{
		"default1":   Default1{},
    "default2":   Default2{},
}
```

如果我们把函数先定义为类型，然后函数实现接口，那么后续的函数实现就不需要再定义结构体了，只需要在使用的时候做下类型转换，例如

```go
// SayFunc 说话的函数
type SayFunc func() string

// Say SayFunc 实现 Sayer 接口
func (s SayFunc) Say() string {
	return s()
}

func Default1Say() string {
  return "Hello, World!"
}

func Default2Say() string {
  return "Hello, World!"
}

m := map[string]Sayer{
		"default1":   SayFunc(DefaultSay1),
    "default2":   SayFunc(DefaultSay2),
}
```

## 经典实现

Golang 标准库提供 net/http 包里关于 Handler 的实现是比较经典的实现

http.Handle 函数需要传入两个参数，第一个参数 pattern 是请求路由，第二个参数是一个 interface，需要我们提供一个实现 Handler 接口的值。
假设我们需要实现一个计数器，我们可以这样实现

1. 先定义计数器类型
2. 实现 Handler 接口
3. 作为参数传给 http.Handle

但是，大多数情况下，我们只想要为每个请求路由提供一个处理函数，而不是先定义一个类型，所幸的是 net/http 里面定义一个函数类型 HandlerFunc，它实现了 ServeHTTP 接口，并在实现里面回调自己，所以我们可以直接实现处理函数，然后作为参数传递给 http.Handle
的时候，用 HandlerFunc 类型转换一下即可
