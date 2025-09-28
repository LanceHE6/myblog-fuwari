---
title: Go语言中new()和make()的区别
description: Go语言中new()和make()的区别
#cover: /cover/cover2.png
tags:
 - Go 
#sticky: 999
published: 2025-01-11
---

# Go语言中new()和make()的区别

## new()

* new()用于创建引用类型以外的数据类型，比如：**基础数据类型**(int, float, string, bool...)、**结构体**。
* new()函数会为数据类型分配内存空间，并返回一个指向已分配内存的指针。
* new()创建的变量是指定类型的零值.

```go
package new_make

import (
	"fmt"
	"testing"
)

type Person struct {
	Name string
	Age  int
}

func TestNewAndMake(t *testing.T) {
    
    // 创建一个int类型的指针
	a := new(int)
	fmt.Println(*a) //  输出 0
    
	// 创建一个Person类型的指针
	p := new(Person)
	p.Age = 1
	p.Name = "test"
	fmt.Println(p) //   输出 &{test 1}
}

```

##  make()

* make()函数只用于**创建slice、map和channel**，并且返回一个有初始值(非零值)的**引用类型**。
* make()适用于创建切片、映射(map)和通道等引用类型的变量。
* make(T, size, cap) 
  * T: 返回值的类型
  * size: 返回值的初始长度
  * cap: 返回值的容量
* make()在创建切片时可以省略size参数，此时size默认为0。
* make()在创建映射时可以省略size和cap参数，此时size和cap参数默认为0。
* make()在创建通道时无size,cap参数,只有**缓冲容量**。

```go
package new_make

import (
	"fmt"
	"testing"
)

func TestNewAndMake(t *testing.T){
    // 创建一个长度为3，容量为10的切片
    arr := make([]int, 3, 10)
	fmt.Println(arr) //   输出 [0 0 0]
	arr = append(arr, 1)
	fmt.Println(arr) //    输出 [0 0 0 1]
	
    // 创建容量为3的映射
	m := make(map[int]string, 3)
	fmt.Println(m) //   输出 map[]
	m[1] = "test"
	fmt.Println(m) //   输出 map[1:test]
	
    // 创建一个缓冲大小为3的通道
	c := make(chan int, 3)
	fmt.Println(c) //   输出 0xc00004020
	c <- 1
	fmt.Println(<-c) //   输出 1
		    				    	
}
```

##  new()和make()的区别

* new() 用于创建任意类型的变量，而 make() 仅用于创建引用类型的变量。
* new() 返回的是**指针**，而 make() 返回的是初始化后的**值**。
* new() 创建的变量是零值，make() 创建的变量是根据类型进行初始化。
