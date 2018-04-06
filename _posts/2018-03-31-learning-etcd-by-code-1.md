---
title: etcd源码学习（一）：启动流程
layout: page
categories: Kubernetes
tags: [Distribution, Etcd, Kubernetes]
---

## 源码结构
***【TODO:源代码层次结构】***

## etcd
etcd的入口main函数定义在etcdmain/main.go中，在其中主要工作如下：
- 检测平台支持情况：checkSupportArch()
- 分析rpc相关的参数
- 启动etcd以及按照rpc设置启动proxy

然后调用etcdmain/etcd.go中的startEtcdOrProxyV2()，其主要就是解析config的工作，etcd是自身定义了一个较为复杂的config struct：
```
type config struct {
	ec           embed.Config
	cp           configProxy
	cf           configFlags
	configFile   string
	printVersion bool
	ignored      []string
}
```
通过上面各config成员实现非常丰富的configure功能。后续专门分析其config实现，以及可借鉴之处。

完成解析后，则调用startEtcd()和startProxy()分别启动etcd主逻辑，以及proxy。

startEtcd调用embed/etcd.go中的StartEtcd，其主要流程是：
!["etcd_server_start"](/assets/etcd/etcd_server_start.png){:class="img-responsive"}
至此etcd就启动了，其中有两个非常重要的功能，
- startPeerListeners()，创建peerListeners
- startClientListeners()，创建clientListeners
- e.servePeers()，启动peerListeners协程处理请求
- e.serveClients()，启动clientListeners协程处理请求
- e.Server.Start()，再提供服务前做一些必要的初始化，最终会在一个goroutine中运行raft相关操作```s.run()```
- s.run()，raft相关的操作

## PeerListeners
peerListener定义如下：
```
type peerListener struct {
	net.Listener
	serve func() error
	close func(context.Context) error
}
```
- net.Listener，调用Go基础库监听请求
- serve，接收到请求后具体的处理逻辑
- close，监听关闭的逻辑

startPeerListeners()定义在embed/etcd.go中，主要逻辑：
- 如果有设置，验证peer cert
- 根据cfg.LPUrls个数，申请peerListener对象
- 通过rafthttp.NewListener()设置每个peerListener的net.Listener对象

### rafthttp
etcd的分布式协议采用的是raft，它实现的非常高效，如果自己的项目中需要在Go语言中利用raft协议，可以直接借用。对于raft协议，可以参考[]()。
***【TODO：etcd里面rafthttp的实现】***


## ClientListeners
clientListener定义如下：
```
type serveCtx struct {
	l        net.Listener
	addr     string
	secure   bool
	insecure bool

	ctx    context.Context
	cancel context.CancelFunc

	userHandlers    map[string]http.Handler
	serviceRegister func(*grpc.Server)
	serversC        chan *servers
}
```

它的启动逻辑比peerListener要简单，就是根据config配置的接口启动相应的net.listener。
- 如果fd数量有限制，调用transport.LimitListener()
- 如果是tcp协议，调用transport.NewKeepAliveListener()设置链接
- 根据config，设置userHandlers、registerPprof、registerTrace

这样就等着client请求一些操作了。

## etcdctl
***【TODO：各中命令实现】***
