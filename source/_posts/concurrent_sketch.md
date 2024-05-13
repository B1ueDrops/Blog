---
title: 并发编程的概述
categories: UNIX编程
---



## 多线程模型

* 最基础的模型: 单个CPU, 多个线程, 线程通过上下文切换的方式在CPU上执行.

* 多个CPU的情况:
  * 多个CPU提供了并行度.
  * CPU硬件级别需要实现Cache Coherence, Memory Consistency Model, 操作系统级别需要封装, 提供互斥锁, 信号量等并发编程模型.

线程的任务分为两种:

* CPU Task: 很快.
* I/O Task: 包括内存I/O, 网络I/O等等, 相对于CPU Task极慢.

当线程执行I/O Task时, 如果I/O未完成, 那么线程就会被阻塞, 直到I/O完成, 这种情况比较低效, 优化方法是使用<font color=red>非阻塞I/O和I/O多路复用</font>.

* 对于很多I/O Task, 不用创建很多线程处理, 而是使用少量线程.
* 非阻塞I/O就是当I/O未完成, 线程不会被阻塞, 而是可以去执行其他任务.
* 当I/O Task完成后, 会通知线程处理.



## 协程模型

协程的出现是为了减少CPU线程上下文切换时的开销.

协程是比线程更细粒度的调度单位, 协程的切换不需要切换到内核态, 可以在用户态实现轻量级的任务调度.



## async/await模型

借助非阻塞I/O和I/O多路复用的思想, 以及协程模型, 可以实现在单线程基础上的异步编程模型, 最典型的是async/await编程模型.

在async/await模型中, 只有一个线程, 线程中有一个部分叫做`EventLoop`, 负责调度不同的Task.

对于用户而言, 一般需要定义一个协程函数(Coroutine Function), 里面写入业务逻辑.

调用协程函数, 可以得到一个Future, 此时这个Future并不会被执行.

之后, 通过`await`一个Future:

* 可以将这个Future变成Task, 注册到Event Loop中.
* 同时建立一个Task之间的依赖关系, 当前线程执行的Task需要等待`await`的Task完成之后才能继续执行.
* 将控制权转交给Event Loop, 等待下一步调度.
