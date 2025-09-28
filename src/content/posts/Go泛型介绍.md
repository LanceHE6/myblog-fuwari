---
title: Go泛型介绍
description: Go泛型介绍
#cover: /cover/cover2.png
tags:
 - Go 
 - 泛型
sticky: 900
published: 2025-01-11
---

# Go泛型介绍

## go语言泛型

自 `1.18` 版本开始，Go 语言正式支持泛型，为开发者带来了更强大、更灵活的编程能力.
泛型是一种编程语言的特性，它允许我们编写能够处理多种类型的代码，而不是只针对特定类型编写的代码。
泛型主要基于类型参数实现，通过在函数、接口或数据结构定义中添加类型参数，我们可以实现通用的代码逻辑，使其适用于各种数据类型。

## 函数泛型

例如我们可以利用泛型编写一个通用的交换函数`swap`

```go
func Swap[T any](a, b *T) {
	*a, *b = *b, *a
}
```

函数名后 `T` 是一个类型参数，`any` 表示 T 可以是任何类型
我们也可以约束它的类型参数,比如我只接收`int`和`string`类型的参数

```go
func Swap2[T int | string](a, b *T) {
	*a, *b = *b, *a
}
```
这样如果传入的参数非`int`和`string`类型就会报错:

`Cannot use [] int as the type interface{ int | string } Type does not implement constraint interface{ int | string } because type is not included in type set (int, string)`

假如我们要约束的类型很多,我们还可以使用**接口**将这些类型单独封装
```go
type Swapable interface {
	~int | string
}

func Swap2[T Swapable | string](a, b *T) {
	*a, *b = *b, *a
}
```
其中`~int`的意思**也包括基于`int`类型的其他派生类型**

## 结构体泛型

可以利用泛型创建通用的列表、映射、堆栈等数据结构，它们可以存储任意类型的值。例如，我们可以定义一个通用的栈结构体：

```go
package generic

import "fmt"

type Stack[T any] struct {
	data []T
}

func NewStack[T any]() *Stack[T] {
	return &Stack[T]{}
}

func (s *Stack[T]) Push(item T) {
	s.data = append(s.data, item)
}

func (s *Stack[T]) Pop() T {
	if len(s.data) == 0 {
		panic("stack is empty")
	}
	item := s.data[len(s.data)-1]
	s.data = s.data[:len(s.data)-1]
	return item
}
func (s *Stack[T]) String() string {
	return fmt.Sprintf("%v", s.data)
}
```

使用示例:

```go
package generic

import "testing"

func TestStack(t *testing.T) {
	s := NewStack[int]()
	s.Push(1)
	s.Push(2)
	t.Log(s.String())
	t.Log(s.Pop())
}
```

