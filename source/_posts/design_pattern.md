---
title: 设计模式的实际操作
categories: 后端技术
---



## 系统的结构

* 一个系统有三个部分, 分别是Input, Output, 以及Executor.
  * 任何一个复杂的系统, 都可以拆分成这样一个递归的Input, Executor, Output的结构.
  * 这三个部分都可能可变, 可以采用不同的策略提高系统稳定性.



## Input/Output可变

* 如果Input/Output可变:
  * 定制一个Input/Output抽象接口, 接口中提供足量函数供后面的Executor调用.
  * 不同的Input/Output实现这个接口, 实现时注意:
    * 所有的成员变量需要在**构造函数内部**提供默认值 (签名处不用定义成员变量), 如果要修改成员变量, 在接口中暴露`getter/setter`方法.
      * 因为Input/Output的参数不一定有经常需要改变的需求.
  * 在Executor中, 将Input/Output作为成员变量, 成员变量类型定义为抽象接口类型, 并且只使用接口中的方法完成任务.



## Executor可变

* Executor中的成员变量需要在**构造函数签名**处提供默认值, 因为很可能需要对不同配置的Executor进行实验.

* Executor可变可以分为不同类型:
  * Executor中某一个功能可变: **策略模式**
    * 为功能提供接口, 然后不同功能类实现接口, 然后用组合方式整合到Executor中.
  * Executor中工作流程可变: **模板方法模式**
    * 提供一个抽象类, 子方法留接口, 模板方法规定顺序.
    * 不同类实现抽象类中的子方法接口.
    * 模板方法对象用组合方式整合到Executor中.