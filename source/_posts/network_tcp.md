---
title: 传输层
categories: 计算机网络
mathjax: true
---



## TCP长连接和短链接

https://zhuanlan.zhihu.com/p/598303451?utm_id=0





## 面试题

> TCP为什么是三次握手?

三次握手的核心是: Client和Server都要确认自己的发送/接收是正常的.

* 第一次握手:
  * Client什么都确认不了.
  * Server可以确认: Client发送正常, Server接收正常.
* 第二次握手:
  * Client可以确认: Client发送正常, Client接收正常, Server发送正常, Server接收正常.
  * Server可以确认: Client发送正常, Server接收正常.
* 第三次握手: 全部确认.