---
title: goroutines&channels
date: 2018-09-12 16:48:30
tags: Goroutines/Channels
---



高并发是GO语言最大的特性，谈及并发一定要提到Go语言中的Goroutine和Channel了：

Goroutine是<u>并发的执行单元</u>。

Channel是Goroutine之间的<u>通信机制</u>。



goroutine在go中被称为线程，那么线程，进程，协程的差别在哪里呢？

进程：程序的一个实例

线程：程序的一个执行流

协程：轻量级的线程，在切换内核态时可控切换时机，并且切换的代价小



channel分为<u>无缓存channel</u>（同步channel）和<u>带缓存channel</u>：

无缓存channel：例如

![](../images/goroutine&channel.png)

g2接收数据**happends before**唤醒g1<u>(happends before:并非时间上，而是保证在此之前已经完成)</u>

基于无缓存Channels的发送和接收操作将导致两个goroutine做一次同步操作，因此称为同步channel。串联的channel行程pipeline(管道)。

带缓存channel：带缓存的Channel内部持有一个元素队列。队列的最大容量是在调用make函数创建channel时  通过第二个参数指定的。。下面的语句创建了一个可以持有三个字符串元素的带缓存Channel。  图8.2是ch变量对应的channel的图形表示形式。

![](../images/channel.png)

向缓存Channel的发送操作就是向内部缓存队列的尾部插入元素，接收操作则是从队列的头部  删除元素。如果内部缓存队列是满的，那么发送操作将阻塞直到因另一个goroutine执行接收  操作而释放了新的队列空间。相反，如果channel是空的，接收操作将阻塞直到有另一个  goroutine执行发送操作而向队列插入元素。



Go语言中的并发程序模型分为以下两种：

1）**顺序通信进程**（  communicating sequential processes ，缩写为CSP）。在CSP中，程序是一组中间没有共享状态的平行运行的处理过程，它们之间使用管道进行通信和控制同步。

2）**多线程共享内存**（传统模型）。

而传统的多线程共享内存就会涉及到**数据竞争（Data Race）**的问题，如何避免数据竞争是并发中需要解决的头号问题，以下有三种避免数据竞争的方法：

1）不要去写变量

2）避免从多个goroutine访问变量

3）允许从多个goroutine访问变量，但同一时刻最多只有一个goroutine在访问变量

如若在多goroutine之间已经存在共享变量，那么在使用到共享变量的时候，需要一些限定或是锁，来确保变量安全：

1）限定缓存为1的chan,保证每次只有一个goroutine访问

2）sync.Mutex()互斥锁

3）sync.RWMutex读写锁

另外Go语言中，还有竞争检测器，使用 go run /test  -race flag 可在调试中检测是否有数据竞争存在。