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

## 并发通信

## channel

## 同步

## 其他
