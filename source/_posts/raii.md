---
title: RAII的概念
categories: 编程语言
---

## RAII的概念理解

* 计算机中, 有很多资源, 例如内存, 文件描述符, socket, 互斥锁等, 如果要使用这些资源, 就必须遵循以下三个步骤:
  * 申请资源.
  * 使用资源.
  * 释放资源.
* 如果没有及时释放资源, 那么就会出现资源泄漏.

* 但是, 如果你每次都手动释放资源, 代码就会变的臃肿不堪, 难以维护, 并且很可能程序出现异常时, 没有执行到释放资源的语句, 这时, 就需要RAII的机制.
* **RAII (Resource Acquisition Is Initialization)**:
  * 抽象来说, RAII就是在全局提供一种机制, 统一管理资源的申请/释放.
  * 具体来说, 以C++为例:
    * C++中, 系统的资源不具有自动释放的功能, 但是C++的类具有自动调用析构函数的功能.
    * 如果把资源的管理放在一个类中, 释放资源的代码放到析构函数中, 就可以实现RAII. 例如, 智能指针.