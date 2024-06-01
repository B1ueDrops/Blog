---
title: 缓存一致性
categories: 体系结构
mathjax: true
---



[TOC]



## 缓存一致性问题

一般来说, 一个CPU有一个L1缓存.

假设CPU0访问了内存地址A, 那么内存地址A对应的数据就会被放到CPU0的L1缓存中.

此后, CPU0可能修改内存地址A的数据, 这时候, 由于L1缓存的Write-Back策略, 修改可能同步不到内存.

这时候, 如果CPU1想要访问内存地址A, 它应该从CPU0的L1缓存中读数据呢? 还是要从内存中读数据呢?

* 这两个地方的数据很可能不一致.

总结来说, 我的数据可能在:

* 多个CPU的缓存.
* 内存.

中有多次的备份.

我有很多的观察者, 例如CPU, DMA, GPU等, 我的目的就是要在这些观察者眼中, 保持这些副本的一致性.

这种一致性会在硬件上实现, 这种硬件单元叫做SCU (Snoop Control Unit), CPU会通过总线连接到SCU.



## MESI协议



### Cache的结构

Cache的结构可以看作一个二维数组, 一行是一个Set, 一列是一个Way, 其中的一个元素叫做一个Cache Line.

为了实现缓存一致性, 我需要给每一个Cache Line添加若干个状态:

* M (Modified): 
  * 数据有效.
  * 数据已经被修改, 但是没被同步到内存中.
  * 数据只存在于当前CPU的Cache中.
* E (Exclusive):
  * 数据有效.
  * 数据没有被修改, 并且和内存一致.
  * 数据只存在于当前CPU的Cache中.
* S (shared):
  * 数据有效.
  * 数据没有被修改, 并且和内存一致.
  * 数据存在于多个CPU的Cache中.
* I(Invalid):
  * 数据无效.

在实际实现中, MESI的状态可以用Valid位, Dirty位和Shard位来表示, 表示如下:

| 状态 | Valid | Dirty | Shared |
| ---- | ----- | ----- | ------ |
| M    | 1     | 1     | 0      |
| E    | 1     | 0     | 0      |
| S    | 1     | 0     | 1      |
| I    | 0     | 0     | 0      |



### 状态转移

下面针对某一个CPU的某一个Cache Line而言.



#### 初始状态为I

假设这个Cache Line的初始状态是I:

* 如果CPU0发起了读请求, 正好读到这个Cache Line.
  * CPU0会把这个读请求广播到总线, 其他CPU会找到这个请求.
  * 然后其他CPU会检查自己的Cache中是否有这个数据:
    * 如果有(假设有这个数据的CPU叫CPU1), 并且状态为S, 那么它会把这个数据通过总线发送到原来的CPU上, 原来CPU的Cache Line的状态就会变成S.
    * 如果CPU1有, 并且状态为E, 那么它会把数据通过总线传到CPU0, 并且CPU0上Cache Line状态从I变成S, CPU1上Cache Line状态从E变到S.
    * 如果CPU1有, 并且状态是M, 那么它会把数据通过总线传到CPU0, 然后再把数据修改同步到内存, 并且CPU0上Cache Line从I变成S, CPU1上Cache Line从M变成S.
    * 如果所有CPU都没有(状态都是I), 那么CPU0就会从内存中把数据读取到L1缓存, 并且状态从I变成E.
* 如果CPU0发起了写请求, 正好要写到这个Cache Line.
  * CPU0会把这个写请求广播到总线, 其他CPU会找到这个请求.
  * 然后其他CPU会检查自己的Cache中是否有这个数据:
    * 如果有, 并且状态是S, 那么CPU1会把数据发送到总线上, 并且设置Cache Line的状态是I.
    * 如果有, 并且状态是E, 那么CPU1会把数据发送到总线上, 并且设置Cache Line的状态是I.
    * 如果有, 并且状态是M, 那么CPU1会把数据发送到总线上, 并且设置Cache Line的状态是I, 并且把这个Cache Line的修改同步到内存中.
    * 如果所有CPU都没有, 那么CPU0就会从内存中把数据读取到L1 Cache, 然后进行写操作, 状态从I变成M.
* 如果CPU0从总线上收到了读写请求, 那么它的状态不变(还是I), 会给总线回应一个信号, 告诉别的CPU自己没有这个Cache Line.



#### 初始状态为M

* 如果初始状态是M, 那么说明CPU的L1 Cache中是有最新的数据的, 对于本地的读/写请求, 状态M不会改变.
* 当这个CPU收到一个总线读信号, 它会把这个Cache Line的数据发送到总线上, 并且把这个Cache Line的修改同步到内存中, Cache Line的状态从M到S.
* 当CPU收到一个总线写信号, 具体操作和收到读信号一样, 只是Cache Line的状态需要从M到I.



#### 初始状态为S

* 对于本地的读操作, Cache Line的状态不变.
* 对于本地的写操作, Cache Line的状态从S变成M, 然后将这个写操作广播到总线上, 其他的CPU收到这个信号之后, 就会检查自己的Cache中是否有这个Cache Line, 如果有, 那么状态设置成I.
* 对于总线读操作, Cache Line状态不变, Cache Line的数据会被广播到总线上.
* 对于总线写操作, 将自身的Cache Line状态变为I.



#### 初始状态为E

* 对于本地的读操作, Cache Line的状态不变.
* 对于本地写操作, Cache Line的状态从E变成M.
* 对于总线读操作: Cache Line状态变为S, 对应的Cache Line数据会被放到总线上.
* 对于总线写操作: Cache Line的状态变成I, 对应的Cache Line会被放到总线上.



## 缓存伪共享

