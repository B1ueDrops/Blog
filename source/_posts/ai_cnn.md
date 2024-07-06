---
title: CNN的基本原理
categories: 深度学习
mathjax: true
---



## CNN的基本结构

* CNN的输入一般结构是: `(B, C, H, W)`, 也就是Batch个数, 通道数, 高度和宽度.

* 一个CNN可以分为如下几个结构:
  * 可重复的单元: 可以在CNN中重复多次.
    * 卷积层
    * BatchNorm层.
    * 激活函数.
    * Pooling层.
  * Dropout层.
  * 全连接层.

### 卷积层

* 定义: `nn.Conv2d(in_features, out_features, kernel_size, stride, padding)`.
  * `in_features`: 输入的通道数, 也就是`C`.
  * `out_features`: 输出的通道数, 也就是卷积核的数量, 对应输出张量中的`C'`是多少.
  * `kernel_size`: 是一个tuple/整数, 表示卷积核的`(height, width)`.
  * `stride`: 也是一个tuple/整数, 表示卷积核在高度和宽度上的步长.
    * 注意, 卷积核的移动是先移动一个维度, 移动到结束之后再从另一个维度移动一次, 并不是同时移动两个维度.
  * `padding`: 也是一个tuple/整数, 表示在高度/宽度上填充像素的层数, 有不同的padding模式, 默认是0填充.
  
* 卷积层的计算过程:
  
  * 首先, 每一个卷积核中的具体数值, 并不需要操心, 梯度下降会自动学, 可以假设卷积核中的数值是设计好的.
  * 之后, 对于每一个Batch, 每一个卷积核都会在`(C, H, W)`上进行卷积操作, 得到feature map.
    * 卷积核的尺寸是`(C, kernel_size)`, 每一个通道独立操作, 最后得到的结果相加作为feature map的值.
  
* 卷积层的输出: `(N, C', H', W')`, 最终输出的feature map中的数值代表每一种特征的显著程度.

  * `C'`就是`out_features`的数值.
  * `H'`的计算公式如下(`W'`遵循同样的模式)

  $$
  H' = \lfloor \frac{H + 2 \times padding - (kernel\ size - 1) - 1}{stride} + 1\rfloor
  $$
  
  
  
  