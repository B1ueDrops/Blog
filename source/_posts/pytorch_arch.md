---
title: Pytorch的整体架构
categories: AI加速
mathjax: true
---



## Pytorch的组成部分

* Pytorch主要由三个大部分组成:
  * **前端**: 由Python编写, 方便构建模型的原型, 并且测试模型性能.
  * **后端**: 张量计算的引擎, 例如CPU上的GEMM Kernel或者GPU的cuDNN等, 一般是C++编写.
  * **执行模式**: 根据科研需求和生产部署需求, 分别有Eager Mode (动态图模式)和Graph Model (静态图模式).
    * Eager Mode: 算子一个一个执行, 不构建图, 一般用于科研.
    * Graph Mode: 将张量计算先转换为一个DAG, 然后对DAG进行特定优化, 一般用于模型部署.
    * Pytorch原生支持Eager Mode, 用户体验极好, 但是对Graph Mode做了很多尝试, 例如FX Graph.



> Pytorch的源码架构?

* 在Pytorch的源码中, 有两个部分:
  * `torch`: 对应Pytorch的前端部分, 由Python编写.
  * `aten`: 全称是A Tensor Library, 由C++编写, 包含了各种Kernel和硬件平台的算子实现.
* 用`pip install torch`时, `aten`部分会用动态链接库的形式下载, 然后在`torch`的`__init__.py`中进行链接.
  * Pytorch后端的链接库`libtorch.so`也可以自己使用, 不借助任何Pytorch前端.
