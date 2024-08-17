---
title: Softmax的相关问题
categories: AI加速
mathjax: true
---



## Softmax的原理

* Softmax的作用是将一个$n \times 1$的向量转成一个$n \times 1$的概率向量, 概率向量中的每一个元素都是原始元素的概率分布.
* 假设$x = (x_1, x_2, ..., x_n)$

$$
Softmax(x) = \frac{e^{x_i - max(x_i)}}{\sum_{i=1}^n e^{x_i - max(x_i)}}
$$

* 分子分母同时除以$e^{max(x_i)}$是为了防止$e^{x_i}$数值过大, 超过`float32`的表示范围.