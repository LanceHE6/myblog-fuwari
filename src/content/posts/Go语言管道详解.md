---
title: Go语言管道详解
description: Go语言管道详解
#cover: /cover/cover2.png
tags:
 - Go 
#sticky: 999
published: 2025-01-11
---

# Go语言管道详解

管道(channel)是go语言中的一种核心引用类型,如同字面意思管道,通过它**并发核心单元**就可以发送或者接收数据进行通讯.

同映射和切片一样,管道需要先创建才能使用

```go
ch := make(chan int)
```

使用make初始化Channel时可以设置容量(缓存):

```go
ch := make(chan int, 100)
```

容量(capacity)代表Channel容纳的最多的元素的数量，代表Channel的缓存的大小。

* 如果没有设置容量，或者容量设置为0, 说明Channel没有缓存，只有sender和receiver都准备好了后它们的通讯才会发生。

* 如果设置了缓存，就有可能不发生阻塞， 只有缓存满了后 send才会阻塞， 而只有缓存空了后receive才会阻塞。一个nil channel不会通信。

可以通过内建的close方法可以关闭Channel。

你可以在多个goroutine从/往 一个channel 中 receive/send 数据, 不必考虑额外的同步措施。

Channel可以作为一个先入先出(FIFO)的队列，接收的数据和发送的数据的顺序是一致的。

## channel的类型

channel分为3种类型

```go
ch1 := make(chan int)       
ch2 := make(chan<- int)     
ch3 := make(<-chan int)     
```

* ch1:普通的双向管道,可以发送和接收类型T的数据
* ch2:只发送管道,只能向管道发送类型T的数据,不能从管道读取数据
* ch3:只接收(读取)管道,只能从管道读取类型T的数据,不能向管道发送数据

## channel操作符 `<-`

### 向channel中发送数据

通信操作符 `<-` 的箭头指示数据流向，箭头指向哪里，数据就流向哪里

```go
ch := make(chan int, 3)
ch <- 1 // 向ch管道中添加数据1
```

### 从channel中读取(接收)数据

```go
ch := make(chan int, 3)
ch <- 1
ch <- 2
data := <- ch
<- ch
```
上述操作中,我们创建了一个缓存容量为3的管道并向其发送了数据`1`和`2`

然后创建变量`data`接收了管道中的数据`1`

如果读取数据时没有拿变量接收,管道则会直接丢弃该数据

## channel各种状态对应的操作结果

- 正常的 channel，可读、可写
- nil 的 channel，表示未初始化的状态，只进行了声明，未向其发送数据,或者手动赋值为 nil
- 已经 closed 的 channel，表示已经 close 关闭了，千万不要误认为关闭 channel 后，channel 的值是 nil

|   操作    | 正常channel | nil channel | closed channel |
| :-------: | :---------: | :---------: | :------------: |
|   ch <-   | 成功或阻塞  |    panic    |     panic      |
|   <-ch    | 成功或阻塞  |    panic    |    读取零值    |
| close(ch) |    成功     |    panic    |     panic      |


## 应用示例
```go
func outputFromCh(ch chan int) {
	for {
		time.Sleep(time.Second * 2)
		fmt.Println("output: ", <-ch)
	}
}

func main() {
	ch := make(chan int)
	go outputFromCh(ch)
	for {
		var data int
		fmt.Scan(&data)
		ch <- data
	}
}
```
这段Go程序实现了一个简单的从控制台接收整数输入并通过通道发送给另一个goroutine进行处理的流程。下面是详细的程序流程解释：

**1.创建通道：**
使用 `make(chan int)` 创建一个可以传递整数的通道，并将其存储在变量 `ch` 中。

**2.启动 `outputFromCh` goroutine：**

- 使用 `go outputFromCh(ch)` 启动一个新的goroutine来执行 `outputFromCh` 函数，并传入通道 `ch` 作为参数。这意味着 `outputFromCh` 函数将在后台并发执行。

在 `outputFromCh` 函数中，程序进入一个无限循环：
- 每次循环开始前，先调用 `time.Sleep(time.Second * 2)` 使goroutine暂停2秒钟。
- 然后执行 `<-ch` 从通道 `ch` 中接收一个整数。由于通道是阻塞类型的，如果通道中没有数据，goroutine会在这里阻塞**直到数据可读**。
- 接收到整数后，打印出该整数。

**3.主函数中的无限循环：**
- 同时，在 `main` 函数中，程序也进入了一个无限循环：
- 使用 `fmt.Scan(&data)` 从标准输入读取一个整数。用户输入的整数将被存储在变量 `data` 中。
- 将读入的整数 `data` 发送到通道 `ch` 中

程序会无限期地运行下去。`main` 函数中的循环会不断地从用户那里接收输入，并将输入发送到通道 `ch`。

`outputFromCh` 函数中的循环会每隔2秒从通道 `ch` 读取一个整数并打印它。
