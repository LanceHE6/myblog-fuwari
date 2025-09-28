---
title: Golang基础知识
description: Golang基础知识整理
#cover: /cover/cover2.png
tags:
 - Go 
#sticky: 999
published: 2024-11-17
---

# Golang基础知识
 
## 进程,线程,协程

* 进程: **进程**是**系统资源**(地址空间,文件句柄...)分配的基本单位.
* 线程: **线程**是**CPU调度**的基本单位.
    - 所有线程共享相同的虚拟空间地址,可以访问相同的代码段,数据段和堆栈段.
    - 所有线程可以访问所属进程的全局变量和静态变量.
* 协程: **用户态**的线程,由用户程序创建,删除,协程切换**不需要**切换内核态.

### 线程和协程的区别

* 1.线程是操作系统的概念，而协程是程序级的概念。线程由**操作系统调度**执行，每个线程都有自己的**执行上下文**，包
括程序计数器、寄存器等。而协程由**程序自身控制**。
* 2.多个线程之间通过**切换执行**的方式实现**并发**。线程切换时需要保存和恢复上下文，涉及到上下文切换的开销。而协
程切换时不需要操作系统的介入，只需要保存和恢复自身的上下文，切换开销较小。
* 3.**线程是抢占式的并发**，即操作系统可以随时剥夺一个线程的执行权。而协程是合作式的并发，_协程的执行权由程序
自身决定_，只有当协程主动让出执行权时，其他协程才会得到执行机会。

### 线程之间的通信方式

* 1.匿名管道
* 2.命名管道
* 3.消息队列
* 4.共享内存
* 5.信号
* 6.信号量
* 7.socket

## 同步和异步

同步和异步是描述计算机程序中任务执行方式的两个重要概念，它们主要的区别在于任务执行的顺序和时间点：

1. **同步（Synchronous）**：
  - 同步操作指在发起任务后，调用方会等待这个任务完成，然后再继续执行后续操作。执行方式是顺序的，一个任务完成后才进行下一个任务。
  - 特点是代码执行简单直观，容易理解和管理，但可能会导致性能瓶颈和响应延迟，尤其在等待I/O操作或网络请求等耗时任务时。
  - 在前端开发中，同步操作可能会阻塞用户界面，影响用户体验。

2. **异步（Asynchronous）**：
  - 异步操作指在发起任务后，调用方不会等待这个任务完成，而是继续执行后续操作。任务的完成会在将来的某个时间点通过回调函数、事件、Promise等机制来处理。
  - 特点是不阻塞当前执行流程，可以提高效率和响应速度，尤其在I/O密集型或高并发的应用场景中。但也可能导致代码复杂度增加，需要更多的控制逻辑来处理异步流程和状态。
  - 在前端开发中，异步操作允许页面保持响应，提升用户体验，例如使用`setTimeout`、`setInterval`和`ajax`等。

总结来说就是，**同步是按顺序执行任务**，简单但可能效率低；**异步是同时执行多个任务**，复杂但效率高。根据不同的应用场景和性能要求来决定使用同步还是异步方式。

## GMP调度和CSP模型

### CSP模型
CSP 是 Communicating Sequential Process 的简称，中文可以叫做通信顺序进程，是一种**并发编程模型**，是一个很强大的并发数据模型，是上个世纪七十年代提出的，用于描述两个独立的并发实体通过共享的通讯 channel(管道)进行通信的并发模型。
相对于Actor模型，CSP中channel是第一类对象，它不关注发送消息的实体，而关注与发送消息时使用的channel。

严格来说，CSP 是一门形式语言（类似于 ℷ calculus），用于描述并发系统中的互动模式，也因此成为一众面向并发的编程语言的理论源头，并衍生出了 Occam/Limbo/Golang…

而具体到编程语言，如 Golang，其实只用到了 CSP 的很小一部分，即理论中的 Process/Channel（对应到语言中的 goroutine/channel）：这两个并发原语之间没有从属关系， Process 可以订阅任意个 Channel，Channel 也并不关心是哪个 Process 在利用它进行通信；Process 围绕 Channel 进行读写，形成一套有序阻塞和可预测的并发模型。

### 什么是GMP

* G: goroutine, go的协程,每个`go`关键字都会创建一个协程.
* P: process, 包含运行Go代码所需要的必要资源，用来**调度G和M之间的关联关系**，其数量可以通过`GOMAXPROCS`来设置，默认为核心数
* M: machine, 工作线程, 数量对应真是的CPU数.

线程想运行任务就得获取 P，从 P 的本地队列获取 G，当 P 的本地队列为空时，M 也会尝试从全局队列或其他 P 的本地队列获取 G。M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。

### Goroutine调度策略
* 1.队列轮转：P会周期性的将G调度到M中执行，执行一段时间后，保存上下文，将G放到队列尾部，然后从队列中再取出一个G进行调度，P还会周期性的查看全局队列是否有G等待调度到M中执行
* 2.系统调用：当G0即将进入系统调用时，M0将释放P，进而某个空闲的M1获取P，继续执行P队列中剩下的G。M1的来源有可能是M的缓存池，也可能是新建的。
* 3.当G0系统调用结束后，如果有空闲的P，则获取一个P，继续执行G0。如果没有，则将G0放入全局队列，等待被其他的P调度。然后M0将进入缓存池睡眠。

### Goroutine的切换时机
* 1.select阻塞
* 2.io阻塞
* 3.channel阻塞
* 4.等待锁
* 5.程序调用
* 6.程序员显示编码操作??

## context

### Context结构原理
Context（上下文）是Golang应用开发常用的并发控制技术 ，它可以控制一组呈树状结构的goroutine，每个goroutine拥有相同的上下文。Context 是**并发安全**的，主要是用于控制多个协程之间的协作、取消操作。

```go
type Context interface {
  Deadline() (deadline time.Time, ok bool)
  Done() <-chan struct{}
  Err() error
  Value(key interface{}) interface{}
}
```

「Deadline」 方法：可以获取设置的截止时间，返回值 deadline 是截止时间，到了这个时间，Context 会自动发起取消请求，返回值 ok 表示是否设置了截止时间。
「Done」 方法：返回一个只读的 channel ，类型为 struct{}。如果这个 chan 可以读取，说明已经发出了取消信号，可以做清理操作，然后退出协程，释放资源。
「Err」 方法：返回Context 被取消的原因。
「Value」 方法：获取 Context 上绑定的值，是一个键值对，通过 key 来获取对应的值。

几个实现context接口的对象：

context.Background()和context.TODO()相似，返回的context一般作为根对象存在，其不可以退出，也不能携带值。要具体地使用context的功能，需要派生出新的context。
context.WithCancel()函数返回一个子context并且有cancel退出方法。子context在两种情况下会退出，一种情况是调用cancel，另一种情况是当参数中的父context退出时，该context及其关联的context都退出。
context.WithTimeout函数指定超时时间，当超时发生后，子context将退出。因此子context的退出有3种时机，一种是父context退出；一种是超时退出；一种是主动调用cancel函数退出。
context.WithDeadline()与context.WithTimeout()返回的函数类似，不过其参数指定的是最后到期的时间。
context.WithValue()函数返回待key-value的子context

### Context原理
context在很大程度上利用了通道在close时会通知所有监听它的协程这一特性来实现。每一个派生出的子协程都会创建一个新的退出通道，组织好context之间的关系即可实现继承链上退出的传递。

context使用场景：

* 1.RPC调用
* 2.PipeLine
* 3.超时请求
* 4.HTTP服务器的request互相传递数据

## 竞态、内存逃逸

* 1.资源竞争，就是在程序中，同一块内存同时被多个 goroutine 访问。我们使用 go build、go run、go test 命令时，添加 -race 标识可以检查代码中是否存在资源竞争。

解决这个问题，我们可以给资源进行加锁，让其在同一时刻只能被一个协程来操作。

```go
sync.Mutex
sync.RWMutex
```

* 2.一般来说，局部变量会在函数返回后被销毁，因此被返回的引用就成为了"无所指"的引用，程序会进入未知状态。

但这在Go中是安全的，Go编译器将会对每个局部变量进行逃逸分析。如果发现局部变量的作用域超出该函数，则不会将内存分配在栈上，而是分配在**堆**上，因为他们不在栈区，即使释放函数，其内容也不会受影响。

```go
func add(x, y int) *int {
  res := x + y
  return &res
}
```

这个例子中，函数`add`局部变量`res`发生了逃逸。`res`作为返回值，在`main`函数中继续使用，因此`res`指向的内存不能够分配在栈上，随着函数结束而回收，只能分配在堆上。

编译时可以借助选项 `-gcflags=-m`，查看变量逃逸的情况

## new和make的区别

`var`声明值类型的变量时，系统会默认为他分配内存空间，并赋该类型的**零值**
如果是**指针类型**或者**引用类型**的变量，系统不会为它分配内存，默认是`nil`。

* 1.`make` 仅用来分配及初始化类型为 `slice`、`map`、`chan` 的数据。
* 2.`new` 可分配任意类型的数据，根据传入的类型申请一块内存，返回指向这块内存的指针，即类型 `*Type`。
* 3.`make` 返回**引用**，即 `Type`，`new` 分配的空间被清零， `make` 分配空间后，会进行**初始化**。
* 4.`make`函数返回的是`slice`、`map`、`chan`类型本身
* 5.`new`函数返回一个指向该类型内存地址的指针

## slice的实现原理

_`slice`不是线程安全的_
切片是基于**数组**实现的，底层是数组，可以理解为对底层数组的抽象

```go
type slice struct{
  array unsafe.Pointer
  len int
  cap int
}
```

`slice`占24个字节
`array`：指向底层数组的指针，占用8个字节
`len`:切片的长度，占用8个字节
`cap`：切片的容量，cap总是大于等于len，占用8个字节

初始化`slice`调用的是`runtime.makeslice`，`makeslice`函数的工作主要就是计算`slice`所需内存大小，然后调用`mallocgc`进行内存的分配

_所需内存的大小=切片中元素大小*切片的容量_

## slice和array的区别

* 1.长度不同
  * 数组的初始化必须指定长度且长度是固定的
  * 切片的长度是不固定的,可以追加元素且追加元素时可能触发扩容

* 2.函数传参不同
  * 数组是值类型,在作为函数参数时会将该数组的值**拷贝**给另一个数组,传递的是一份深拷贝,在函数中对数组进行操作不会影响原数组
  * 切片是引用类型,在作为函数参数时(或一个切片赋值给另一个切片)会只会将切片的`len`,`cap`拷贝出去,底层共用一个数组,不会占用额外空间,所以函数对数组的操作会影响到原数组

* 3.计算长度的方式不同
  * 数组计算长度需要遍历,时间复杂度为`O(n)`
  * 切片包含字段`len`,可直接获得切片长度,时间复杂度为`O(1)`

## map的实现原理

Go中的`map`是一个指针，占用8个字节，指向`hmap`结构体，`map`底层是基于 **哈希表+链地址法(链表结构作为桶)** 存储的。

### map的特点

* 1.键不能重复
* 2.键必须可哈希,如`int/bool/string/float/array`
* 3.无序

### map底层结构
```go
// A header for a Go map.
type hmap struct {
  count int //代表哈希表中的元素个数，调用len(map)时，返回的就是该字段值。
  flags uint8 //状态标志是否处于正在写入的状态等
  B uint8 //buckets(桶)的对数 如果B=5，则buckets数组的长度=2^B=32，意味着有32个桶
  noverflow uint16 //溢出桶的数量
  hash0 uint32  //生成hash的随机数种子
  buckets unsafe.Pointer  //指向buckets数组的指针，数组大小为2^B，如果元素个数为0，它为nil.
  oldbuckets unsafe.Pointer //如果发生扩容，oldbuckets是指向老的buckets数组的指针，老的buckets数组大小是新的buckets的1/2;非扩容状态下，它为ni1.
  nevacuate uintptr //表示扩容进度。小于此地址的buckets代表已搬迁完成。
  extra *mapextra //存储溢出桶，这个字段是为了优化GC扫描面设计的
}
```
源码包中`src/runtime/map.go`定义了`hmap`的数据结构
`hmap`包含若干个结构为`bmap`的数组，每个`bmap`底层都采用链表结构，`bmap`通常叫其`bucket`,也就是我们常说的**桶**,一个桶最多装8个**key**
这些`key`之所以会落入同一个桶，是因为它们经过哈希计算后，哈希结果的低`B`位是相同的

### map的初始化过程

* 1.创建一个`hmap`结构体对象
* 2.生成一个哈希因子hash0并赋值到`hmap`对象中（用于后续为key创建哈希值）
* 3.根据`hint=10`，并根据算法规则来创建`B`，此时的`B`为1
* 4.根据`B`去创建桶(`bmap`对象)并存放在`bucket`数组中。当前的`bmap`的数量为2
  * B<4时，根据B创建桶的个数的规则为：`2^B`（标准桶）
  * B>=4时，根据B创建桶的个数的规则为：`2^B+2^(B-4)` （标准桶+溢出桶）

### Go的map为什么是无序的

使用`range`多次遍历map时输出的`key`和`value`的顺序可能不同。这是Go语言的设计者们有意为之，旨在提示开发者们，Go底层实现并不保证map遍历顺序稳定，请大家不要依赖range遍历结果顺序

主要原因有2点:

* 1.`map`在遍历时，并不是从固定的0号`bucket`开始遍历的，每次遍历，都会从一个随机值序号的`bucket`， 再从其中随机的`cell`开始遍历
* 2.`map`遍历时，是按序遍历`bucket`，同时按需遍历`bucket`中和其`overflow bucket`中的cell。但是`map`在扩容后，会发生`key`的搬迁，这造成原来落在一个`bucket`中的`Key`,搬迁后，有可能会落到其他`bucket`中了，从这个角度看，遍历`map`的结果就不可能是按照原来的顺序了

### map是如何查找的

* 1.写保护检测:先检查`map`的`flags`标志位是否为1, 如果是则表明有其他协程正在写入该`map`继而导致`panic`,这也说明`map`**不是线程安全的**
* 2.计算hash值:将`key`经过hash函数得到哈希值,不同类型的`key`有不同的hash函数
* 3.找到hash值对应的`bucket`
  * bucket定位：哈希值的低B个bit位，用来定位`key`所存放的`bucket`
    如果当前正在扩容中，并且定位到的旧`bucket`数据还未完成迁移，则用旧的`bucket`（扩容前的`bucket`）
    ```go
    hash:=t.hasher(key, uintprt(h.hash0))
    m=bucketMask(h.B)
    b:=(*bmap)(add(h.buckets,(hash&m)*uintptr(t.bucketsize))
    if c:=h.oldbucket;c!=nil{
      if !h.sameSizeGrow(){
        m>>=1
      }
      oldb:=(*bmap)(add(c,(hash&m)*uintptr(t.bucketsize)))
      if !evacuated(oldb){
        b=oldb
      }
    }
    ```
* 4.遍历`bucket`查找
* 5.返回`key`对应的指针

### 负载因子

负载因子(load factor)，用于衡量当前哈希表中空间占用率的核心指标，也就是每个bucket桶存储的平均元素个数。
_负载因子=哈希表存储的元素个数/桶个数_

Go官方发现:
装载因子越大，填入的元素越多，空间利用率就越高，但发生哈希冲突的几率就变大。反之，装载因子越小，填入的元素越少，冲突发生的几率减小，但空间浪费也会变得更多，而且还会提高扩容操作的次数
根据这份测试结果和讨论，Go官方取了一个相对适中的值，把Go中的 map的负载因子硬编码为6.5，这就是6.5的选择缘由。
这意味着在Go语言中，当map存储的元素个数大于或等于6.5*桶个数时，就会触发扩容行为。

### map扩容

扩容条件:

* 条件1：超过负载 map元素个数>6.5*桶个数
```go
func overLoadFactor(count int, B uint8) bool{
  return count > bucketCnt && uintptr(count)>loadFactor*bucketShift(B)
}
```
其中

`bucketCnt=8`，一个桶可以装的最大元素个数
`loadFactor=6.5`，负载因子，平均每个桶的元素个数
`bucketShift(8)`, 桶的个数
  
* 条件2：溢出桶太多(桶中存放的元素超出了最大的存储数量,则需要将超出的元素存放进另一个桶中,则这个桶就叫做溢出桶)
当桶总数<2^15时，如果溢出桶总数>=桶总数，则认为溢出桶过多
当桶总数>=2^15时，直接与2^15比较，当溢出桶总数>=2^15时，即认为溢出桶太多了

扩容机制:
* 1.双倍扩容：针对条件1，新建一个`buckets`数组，新的`buckets`大小是原来的`2`倍，然后旧的`buckets`数据 搬迁到新的`buckets`
* 2.等量扩容：针对条件2，并不扩大容量，`buckets`数量维持不变，重新做一遍类似双倍扩容的搬迁操作， 把松散的键值对重新排列一次，使得同一个`bucket`中的`key`排列地更紧密，节省空间，提高`buckets`利用 率，进而保证更快的存取。该方法我们称之为等量扩容。

## Golang中对nil的Slice和空Slice的处理是一致的吗?
首先Go的JSON 标准库对 nil slice 和 空 slice 的处理是不一致的。
1.`slice := make([]int,0)`：slice不为nil，但是slice没有值(即为`[]`)，slice的底层的空间是空的。
2.`var slice []int` ：slice的值是nil，可用于需要返回slice的函数，当函数出现异常的时候，保证函数依然会有nil的返回值。

## chanel实现原理

Go中的`channel`是一个循环队列，遵循**先进先出**的原则，负责**协程**之间的**通信**(Go语言提倡不要通过共享内存
来通信，而要通过通信来实现内存共享，CSP(CommunicatingSequential Process)并发模型，就是通过
goroutine和channel来实现的)

### 使用场景

* 1.停止信号监听
* 2.定时任务
* 3.生产方和消费方解耦
* 4.控制并发数

### channel是并发安全的

通道的发送和接收操作是原子的，即一个完整的发送或接收操作是一个原子操作，不会被其他`goroutine`中断。

当一个`goroutine`向`channel`发送数据时，如果`channel`已满，则发送操作会被**阻塞**，直到有其他`goroutine`从该`channel`中接收数据后释放
空间，发送操作才能继续执行。在这种情况下，`channel`内部会获取一个锁，保证只有一个`goroutine`能够往其中写入数据。

同样地，当一个`goroutine`从`channel`中接收数据时，如果`channel`为空，则接收操作会被**阻塞**，直到有其他`goroutine`向该`channel`中发送
数据后才能继续执行。在这种情况下，`channel`内部也会获取一个锁，保证只有一个`goroutine`能够从其中读取数据。

因此可以通过`channel`的阻塞实现主线程等待协程的效果
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	result := make(chan int)
	params := make(chan int)
	go inputParams(params)
	go add(params, result)
	fmt.Println(<-result) // 未计算出结果之前主线程将一直阻塞在这
}

func inputParams(params chan int) {
	fmt.Println("等待输入")
	var a, b int
	fmt.Scanln(&a, &b)
	params <- a
	params <- b

}

func add(params chan int, result chan int) {
	a := <-params
	b := <-params
	// 等待计算
	fmt.Println("等待计算")
	time.Sleep(time.Second * 2)
	result <- a + b
}

```

## Channel是同步的还是异步的？
`channel`是异步进行的, `channel`存在3种状态：

1.`nil`，未初始化的状态，只进行了声明，或者手动赋值为`nil`
  * 发送: 阻塞
  * 接收: 阻塞
  * 关闭: panic

2.`active`，正常的`channel`，可读或者可写
  * 发送: 依是否有值以及是否有缓存而发送成功或阻塞
  * 接收: 依是否有值以及是否有缓存而接受成功或阻塞
  * 关闭: 成功

3.`closed`，已关闭，千万不要误认为关闭`channel`后，`channel`的值是`nil`，对已关闭`channel`读写都会`panic`

## Channel死锁场景
* 1.非缓存`channel`只写不读
* 2.非缓存`channel`读在写后面
* 3.缓存`channel`写入超过缓冲区数量
* 4.空读
* 5.多个协程相互等待

## 悲观锁和乐观锁

* 悲观锁：它假设最坏的情况，认为数据冲突很可能发生。因此，_在数据被读取时悲观锁就会对数据进行加锁_，以防止其他事务对数据的修改，直到事务完成。这适用于写操作频繁、冲突多的场景，典型实现是使用数据库锁机制。

* 乐观锁：与悲观锁相反，它假设冲突很少发生。在数据读取时不会加锁，而是在提交更新时检查在读取和更新之间数据是否被改变，通常使用版本号或时间戳来检测。这适用于读操作多、写操作少的场景，能够减少锁的使用，提高系统吞吐量。实现乐观锁常见的方法是 CAS（比较并替换）操作。

## 原子操作

原子操作是计算机科学中的一个重要概念，指的是在执行过程中**不可分割**的操作。具体来说，_原子操作在执行完毕之前不会被任何其他操作打断_，具有以下特性：

1. **不可分割性**：原子操作执行过程中不能被中断，确保了操作的**完整性**。
2. **线程安全**：原子操作避免了多个线程同时对同一数据进行操作时可能产生的数据竞争和不一致性问题。

原子操作在多线程编程中非常有用，用于保证变量操作的安全性。例如，在多线程环境中，递增一个计数器通常需要通过原子操作来确保其线程安全。

实现原子操作的方式包括：
1. **锁机制**：通过锁保护临界区代码来保证操作的原子性。
2. **特定CPU指令**：如x86架构的CMPXCHG指令，可以在硬件层面实现原子性。
3. **内存屏障**：如在ARM架构中通过内存屏障指令实现原子操作。

### 原子操作和锁的区别

* 1.原子操作由底层硬件支持，而锁是基于原子操作+信号量完成的。若实现相同的功能，前者通常会更有效率
* 2.原子操作是单个指令的互斥操作；互斥锁/读写锁是一种数据结构，可以完成临界区（多个指令）的互斥操作，扩大原子操作的范围
* 3.原子操作是无锁操作，属于乐观锁；说起锁的时候，一般属于悲观锁
* 4.原子操作存在于各个指令/语言层级，比如*机器指令层级的原子操作"，““汇编指令层级的原子操作”，“Go语言层级的原子操作”等。
* 5.锁也存在于各个指令/语言层级中，比如“机器指令层级的锁”，“汇编指令层级的锁“Go语言层级的锁“等* 

## Goroutine的实现原理

Goroutine可以理解为一种Go语言的协程 **（轻量级线程）**，是Go支持高并发的基础，属于**用户态**的线程，由`Goruntime`管理而不是操作系统。

```go
type g struct {
  goid int64 //唯一的goroutine的ID
  sched gobuf //goroutine切换时，用于保存g的上下文
  stack stack //栈
  gopc //pc of go statement that created this goroutine
  startpc uintptr  //pc of goroutine function
  ...
}
type gobuf struct {
   sp uintptr //栈指针位置
   pc uintptr //运行到的程序位置
   g guintptr //指向goroutine
   ret uintptr //保存系统调用的返回值
   ...
}
type stack struct {
  lo uintptr //栈的下界内存地址
  hi uintptr //栈的上界内存地址
}
```

* 1.**创建**：`go`关键字会调用底层函数`runtime.newproc()`创建一个`goroutine`，调用该函数之后，`goroutine`会被设置成`runnable`状态
  - 创建好的这个`goroutine`会新建一个自己的栈空间，同时在`G`的`sched`中维护栈地址与程序计数器这些信息。
  - 每个`G`在被创建之后，都会被优先放入到本地队列中，如果本地队列已经满了，就会被放入到全局队列中。

* 2.**运行**：`goroutine`本身只是一个数据结构，真正让`goroutine`运行起来的是调度器。Go实现了一个用户态的调度器（GMP模型），这个调度器充分利用现代计算机的多核特性，同时让多个`goroutine`运行，同时`goroutine`设计的很轻量级，调度和上下文切换的代价都比较小。

* 3.**阻塞**：`channel`的读写操作、等待锁、等待网络数据、系统调用等都有可能发生阻塞，会调用底层函数`runtime. gopark()`，会让出CPU时间片，让调度器安排其它等待的任务运行，并在下次某个时候从该位置恢复执行。
  当调用该函数之后，`goroutine`会被设置成`waiting`状态。

* 4.**唤醒**：处于`waiting`状态的`goroutine`，在调用`runtime.goready()`函数之后会被唤醒，唤醒的`goroutine`会被重新放到**M**对应的上下文**P**对应的`runqueue`中，等待被调度。
  当调用该函数之后，`goroutine`会被设置成`runnable`状态

* 5.**退出**：当`goroutine`执行完成后，会调用底层函数`runtime.Goexit()`，当调用该函数之后，`goroutine`会被设置成 dead 状态

### Goroutine的泄露

泄露原因:
* `Goroutine`内进行`channel`/`mutex`等读写操作被一直阻塞。
* `Goroutine`内的业务逻辑进入死循环，资源一直无法释放。
* `Goroutine`内的业务逻辑进入长时间等待，有不断新增的`Goroutine`进入等待

### 怎么查看Goroutine的数量？怎么限制Goroutine的数量？

在开发过程中，如果不对`goroutine`加以控制而进行滥用的话，可能会导致服务整体崩溃。比如耗尽系统资源导致程序崩溃,或者CPU使用率过高导致系统忙不过来。

1.在`Golang`中,`GOMAXPROCS`中控制的是未被阻塞的所有`Goroutine`,可以被 `Multiplex` 到多少个线程上运行,通过`GOMAXPROCS`可以查看`Goroutine`的数量。
2.使用**通道**。每次执行的`go`之前向通道写入值，直到通道满的时候就阻塞了

### Goroutine和线程的区别？

* 1.一个线程可以有多个协程
* 2.线程、进程都是同步机制，而协程是异步
* 3.协程可以保留上一次调用时的状态，当过程重入时，相当于进入了上一次的调用状态
* 4.协程是需要线程来承载运行的，所以协程并不能取代线程，「线程是被分割的CPU资源，协程是组织好的代码流程」

## Go中两个Nil可能不相等吗？
Go中两个Nil可能不相等。

接口(interface) 是对非接口值(例如指针，struct等)的封装，内部实现包含 2 个字段，类型 T 和 值 V。一个接口等于 nil，当且仅当 T 和 V 处于 unset 状态（T=nil，V is unset）。

**两个接口值**比较时，会先比较 T，再比较 V。接口值与非接口值比较时，会先将非接口值尝试转换为接口值，再比较

## go 打印时 %v %+v %#v 的区别？

* `%v` 只输出所有的值；
* `%+v` 先输出字段名字，再输出该字段的值；
* `%#v` 先输出结构体名字值，再输出结构体（字段名字+字段的值）；
```go
package main
import "fmt"

type student struct {
  id   int32
  name string
}

func main() {
  a := &student{id: 1, name: "Hycer"}
  
  fmt.Printf("a=%v \n", a) // a=&{1 Hycer}
  fmt.Printf("a=%+v \n", a) // a=&{id:1 name:Hycer}
  fmt.Printf("a=%#v \n", a) // a=&main.student{id:1, name:"Hycer"}
}
```

## 什么是 rune 类型？
Go语言的字符有以下两种：

* 1.`uint8` 类型，或者叫 `byte` 型，代表了 `ASCII` 码的一个字符。
* 2.`rune` 类型，代表一个 `UTF-8` 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 `rune` 类型。**`rune` 类型等价于 `int32` 类型**。

## golang值接收者和指针接收者的区别
golang函数与方法的区别是，方法有一个接收者。

如果方法的接收者是**指针类型**，无论调用者是对象还是对象指针，修改的都是对象本身，会影响调用者

如果方法的接收者是**值类型**，无论调用者是对象还是对象指针，修改的都是对象的副本，不影响调用者

## recover

在Go语言中，`recover` 是一个内建函数，用于从panic中恢复并返回一个错误值。其主要作用和特点如下：

1. **恢复panic状态**：`recover` 可以捕获并恢复由 `panic` 引起的程序异常状态，防止程序崩溃退出。
2. **只能在defer函数中使用**：`recover` 只能在延迟函数（由 `defer` 修饰的函数）中调用，否则编译将不通过。
3. **返回错误值**：如果程序发生panic，`recover` 可以从panic中恢复，并返回panic值；如果没有发生panic或者`recover` 不在defer函数中调用，它会返回 `nil`。
4. **阻止程序终止**：通过适当使用 `recover`，可以阻止panic导致的程序终止，使程序能够继续运行。
5. **资源清理和善后处理**：使用 `recover` 可以在程序崩溃前做一些资源清理和善后处理工作，例如关闭数据库连接、释放资源等。

总之，`recover` 提供了一种机制在程序发生panic时进行优雅地恢复，避免程序崩溃退出，对于处理运行时错误和异常情况非常重要。正确使用 `defer`、`panic` 和 `recover` 可以让程序更加健壮和稳定。

## select
Go语言中的 `select` 语句是一种控制结构，它用于处理多个`channel`（通道）操作。`select` 语句允许在多个发送或接收操作上等待，并且当其中一个操作准备就绪时执行对应的代码块，类似于一个多路复用的开关。

以下是 `select` 语句的基本用法和特点：

1. **多路复用机制**：`select` 允许通过一个协程同时处理多个IO请求（Channel 读写事件）。

2. **使用场景**：只能用于channel操作，即只能包含发送或接收channel的case。

3. **基本语法**：
   ```go
   select {
   case <-channel1:
       // 当channel1可以接收数据时执行这里的代码
   case channel2 <- x:
       // 当channel2可以发送数据时执行这里的代码
   default:
       // 如果没有channel操作可以执行，则执行default分支的代码（如果没有default分支，则select会阻塞）
   }
   ```

4. **非阻塞行为**：如果没有channel操作可以立即执行，`select`将阻塞，直到有操作可以执行，或者有超时（如果用在for循环中）。

5. **随机性**：如果有多个case都准备就绪，`select`会随机选择一个case来执行，而不是顺序执行。

6. **效率**：相比于简单地使用for循环遍历通道，使用 `select` 语句可以更加高效地管理多个通道。

7. **控制并发**：`select` 语句是实现高效并发控制的一种机制，常用于Go协程间的消息传递和同步。

## String和[]byte的区别
string类型本质也是一个结构体，定义如下：

```go
type stringStruct struct {
  str unsafe.Pointer
  len int
}
```
`string`类型底层是一个`byte`类型的数组，`stringStruct`和`slice`还是很相似的，`str`指针指向的是`byte`数组的首地址，`len`代表的就是数组长度。

`string`和`byte`的区别：

`string`类型为什么还要在数组的基础上再进行一次封装呢？
这是因为在Go语言中`string`类型被设计为不可变的，不仅是在Go语言，其他语言中`string`类型也是被设计为不可变的，这样的好处就是：在并发场景下，我们可以在不加锁的控制下，多次使用同一字符串，在保证高效共享的情况下而不用担心安全问题。

`string`类型虽然是不能更改的，但是可以被替换，因为`stringStruct`中的`str`指针是可以改变的，只是指针指向的内容是不可以改变的。

## HTTP和RPC对比

* RPC（Remote Produce Call）：远程过程调用，
* HTTP：网络传输协议

**相同点：**

* 1.都是基于TCP协议的应用层协议
* 2.都可以实现远程调用，服务调用服务

**不同点：**

* 1.RPC主要用于在不同的进程或计算机之间进行函数调用和数据交换。HTTP主要用于数据传输和通信。
* 2.RPC协议通常采用二进制协议和高效的序列化方式，而HTTP通常采用文本协议和基于ASCII码的编码方式，数据传输效率较低
* 3.RPC通常需要使用专门的IDL文件来定义服务和消息类型，生成服务端和客户端的代码。而HTTP没有这个限制，可以使用套接字进行通信

## golang中指针的作用

* 1.传递大对象
* 2.修改函数外部变量
* 3.动态分配内存
* 4.函数返回指针
