---
title: Go语言快速入门（二）：面向对象
layout: page
categories: Go
tags: [Go, Kubernets]
---

摘录一段《Go语言编程》中的介绍：
```
Go语言的主要设计者之一 布 派 (Rob Pike) 经说过，如果只能选择一个Go语言的特 性  到其他语言中，他会选择接口。

接口在Go语言有着 关重要的地位。如果说goroutine和channel 是支 起Go语言的并发模型 的基 ，让Go语言在如 集群化与多核化的时代成为一道极为  的风景，那么接口是Go语言 整个类型系统的基 ，让Go语言在基础编程哲学的  上达到前所未有的高度。

Go语言在编程哲学上是变革派，而不是改良派。这不是因为Go语言有goroutine和channel， 而更重要的是因为Go语言的类型系统，更是因为Go语言的接口。Go语言的编程哲学因为有接口 而趋近完 。
```

Go面向对象的核心就是接口，interface。
