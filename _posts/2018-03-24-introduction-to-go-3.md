---
title: Go语言快速入门（三）：并发编程
layout: page
categories: Go
tags: [Go, Kubernets]
---

躁动起来，Go自所以成为Go，就是因为它的主要特性并发的关键字是“go”。通过一个关键字go就可以让一个函数以并发的形式运行起来，而不是调用一系列pthread接口，还不知道怎么搞。

为什么要并发编程，这就不多说了。多进程，多线程，协程，异步IO，都是为了解决程序的运行效率，为了充分利用现在体系结构中的并行特性。关于硬件上的并行，在文章[浅谈硬件体系结构中的并行](https://leonlee110.github.io/go/2018/03/24/parallel-in-hardware-architecture)中再介绍介绍（当前还未完成）。

Go实现并发非常简单，在函数调用起来使用go关键字即可。
```
go a()
```
这样函数a就会以并发的基本单元goroutine来运行。是不是很简单。

但我们运行如下程序，会发现一个问题，好像函数Add没有打印出需要的信息啊，怎么回事？
```
package main
import "fmt"
func Add(x, y int) {
  z := x + y
  fmt.Println(z)
}

func main() {
  for i := 0; i < 10; i++ {
    go Add(i, i)
  }
}
```
这是因为Go程序从main package并执行main()函数开始，当main()函数返回时，程序退出，并不会等其他goroutine(非主goroutine)结束。那怎么办？

## 1. 并发通信
并发是一个大问题，不管什么语言，不管什么系统，甚至硬件架构都在极力考虑怎样提高并发，提高效率，这样让任务跑都更快。并发编程都难度就在于协调，怎样交流，交换信息，这就是并发通信。

了解多线程或者多进程的同学都知道要线程同步或者进程间通信是编程高效率代码都关键，而一些不好的同步机制也往往造成了整个系统的性能瓶颈。比如在IO系统中往往会遇到锁开销大的问题，而无锁的设计也往往成为一些高效率系统的要点，而在以往的编程语言环境下要实现无锁或者开销较小的同步机制有往往非常难。因此，涉及到大量同步的系统往往系统很受影响。

总结起来，在工程上，主要有两种并发通信的机制：共享内存和消息机制。

共享内存指的是对需要同步的内容保持直接的引用，因此不同进程或者线程访问这个内容时，往往也配合锁等机制，这就大大降低了并发的效率。而消息传递是每个进程或者线程维护自己的变量或者信息，然后通过消息机制同步信息。Go是后者，一句话，不要通过共享内存来通信，而是通过通信来共享内存。

## 2. channel
channel在语言级别提供了goroutine间通信的方式。Go语言中的goroutine是协程，也就是说channel通信实际上是个进程间的通信方式，channel传递对象的过程和调用函数时的参数传递行为比较一致。如果是跨进程通信，则可通过网络的方式来进行。
```
package main

import "fmt"

func Count(ch chan int) {
  ch <- 1
  fmt.Println("Counting")
}

func main() {
  chs := make([]chan int, 10)
  for i := 0; i < 10; i++ {
      chs[i] = make(chan int, 1)
      go Count(chs[i])
  }

  for _, ch := range(chs) {
    <-ch
  }
}
```
上面这段代码就很好的演示了channel怎样使用：
- channel是类型相关的，所以在声明或者通过make初始化的时候，都需要在chan关键字后跟上通信的数据类型，比如代码中的int
- 通过<-符号来表述写入或者读取channel内容，如果channel变量在左侧表示写入，如果在右侧则表示读取
- 当读取channel内容时，如果channel当前为空，调用会阻塞；反之依然，写入的时候如果里面有内容，也会阻塞

当有很多channel需要读取时，由于channel为空，读操作会阻塞，所以不能一个个读取，效率太低了。有Linux C开发经验的人可能都想到了select。Go语言原生支持了select关键字。

select的使用和switch语法类似：
```
select {
  case <- chan1:
    xxxx
  case chan2 <- 1:
    xxxx
  default:
    xxxx
}
```
如果对应chan读取到数据或者写入数据成功，则执行相关操作。

前面看到初始化channel的方法是```make(chan int, 1)```，其中第二参数指的是缓冲区大小，这样在持续有大量数据传输的场景下，数据就会被写入到缓存区中而不会丢失。

除了缓冲机制，由于会可能导致阻塞，一般在实现相关方法的时候需要考虑超时，不然出现一个channel长期没有数据写入，则会卡死整个程序，这是在设计上不允许的。Go没有直接提供超时机制，但是借助前面介绍的select能很容易的实现超时机制。
```
// 定义一个channel用于传输超时信息
time_out = make(chan bool, 1)

// 定义个goroutine运行超时等待，当超时则将前面定义的channel写入true
go func() {
  time.Sleep(1e9)
  time_out <- true
}

select {
  case <- ch: // 在下面那个case满足之前，读取到ch的数据，则没有超时
  case <- time_out: // 当ch中一直没有写入数据，而time_out又被写入了true，则执行超时逻辑
}
```

另一个关于channel的点是，channel是Go原生支持的类型，因此它也可以作为channel传递的类型，

前面定义的channel都是可以读，可以写的，当然我们也可以在某些地方限制channel的读写权限，特别在作为一些函数参数时，这样就限制了函数对这个channel不恰当的操作。
```
func Parse(ch <-chan int) {
  for value := range ch {
      fmt.Println("Parsing value", value)
  }
}
```
比如这个函数就不能对ch进行写入操作。```chan <- int```类似这样的定义则限制了对channel读取。

## 3. 其他
- 虽然前面提到channel是goroutine间主要的同步机制，同时Go也提供了其他语言类似的锁机制：sync.Mutex和sync.RWMutex。就是普通锁和读写锁。
- Once.Do(function)，通过这种方法会保证从全局的角度来说只运行function一次。
- 多核并行化，现在Go还不能非常好地自动利用多核。我们可以通过```NumCPU()```获取当前平台核心数，然后通过```runtime.GOMAXPROCS(16)```设置并行核心数
- Gosched()函数会使goroutine主动让出时间片

至此，Go语言的语法特性就基本介绍完，是不是非常少，非常简单？

更多的学习则是通过学习库和一些开源项目，并多在自己的项目中实践来提高了。

其他Go语言入门文章：

[Go语言快速入门（一）：基础篇](https://leonlee110.github.io/go/2018/03/21/introduction-to-go-1)

[Go语言快速入门（二）：面向对象](https://leonlee110.github.io/go/2018/03/23/introduction-to-go-2)
