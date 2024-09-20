---
title: 理解程序运行的时间
categories: AI技术
mathjax: true
---



[toc]



## wallclock time

* wallclock time (real time) 指的是一段程序从开始到结束的时间, 可理解为程序开始时, 开始计时, 到程序完全结束后停止计时, 最终统计的时间.
* wallclock time是在用户视角下的耗时, 如果越长, 用户体验越差.



## cpu time

* cpu time指的是一段程序所产生的所有进程, 在系统所有CPU中占用时间的加和.
* 首先, 一个进程在一个CPU上的时间包括:
  * user time: 进程执行用户态代码的时间.
  * sys time: 进程执行内核态代码的时间.
* 如果系统是多核, 那么程序可能会用多个进程执行, 假设每个进程$i$的执行时间是$t_i$, 那么cpu time就是$\sum_i t_i$.

* 用wallclock time和cpu time了解任务的情况:
  * `wallclock time = cpu time`: 程序是单线程运行.
  * `wallclock time < cpu time`: 程序是CPU密集型, 用并行计算可以带来收益.
    * `cpu time`和`wallclock time`之间的差距就是并行可以带来的收益.
  * `wallclock time > cpu time`: 程序是I/O密集型, 并行计算不能带来收益.