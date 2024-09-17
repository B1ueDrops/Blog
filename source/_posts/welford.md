---
title: Welford算法计算方差
categories: AI技术
mathjax: true
---



* 假设有一堆数据$x_i, i\in [1, n]$, 现在要计算方差$\sigma_x^2$.

### Two-pass的方差计算

* 这种是最朴素的方差计算, 根据公式: $\sigma_x^2 = \frac{1}{n} \sum_{i=1}^{n} (x_i - \bar{x})^2$.
  * 首先, 遍历全部$x_i$, 计算得到$\bar{x}$.
  * 之后, 再遍历一次$x_i$, 得到$\sigma_x^2$.
* 缺点: 遍历两次, 访存次数增多, 计算密度降低.



### One-pass的方差计算

* 之所以要遍历两次$x_i$, 是因为Two-pass计算中, 每一项都依赖$\bar{x}$.
* 根据公式的等价变换: $\sigma_x^2 = \frac{1}{n}\sum_{i=1}^{n}x_i^2 - \bar{x}^2$.
* 这样, 只需要遍历一次, 在遍历的时候维护$x_i$的和, 最后减去一次$\bar{x}^2$即可.

* 缺点: 如果数据方差较小, 但是数据本身数值较大, 那么$\frac{1}{n}\sum_{i=1}^{n}x_i^2$和$\bar{x}^2$本身较大, 相减数值不稳定, 甚至可能出现负数.



### Welford算法

* 思路: 通过计算前$n-1$个样本的$\sum_{i=1}^{n-1} (x_i - \bar{x}_{n-1})^2$, 来得到$\sum_{i=1}^n (x_i - \bar{x}_{n})^2$

* Welford算法公式推导:

$$
\sum_{i=1}^n (x_i - \bar{x}_{n})^2 - \sum_{i=1}^{n-1} (x_i - \bar{x}_{n-1})^2 = \\
(x_n - \bar{x}_n)^2 + \sum_{i=1}^{n-1}(x_i - \bar{x}_n)^2 - \sum_{i=1}^{n-1}(x_i - \bar{x}_{n-1})^2 = \\
(x_n - \bar{x}_n)^2 + \sum_{i=1}^{n-1}[(x_i - \bar{x}_n)^2 - (x_i - \bar{x}_{n-1})^2] = \\
(x_n - \bar{x}_n)^2 + \sum_{i=1}^{n-1}(x_i - \bar{x}_n + x_i - \bar{x}_{n-1})(x_i - \bar{x}_n - x_i + \bar{x}_{n-1}) = \\
(x_n - \bar{x}_n)^2 + (\bar{x}_n - x_n)(\bar{x}_{n-1} - \bar{x}_n) = \\
(x_n - \bar{x}_n)(x_n - \bar{x}_n - \bar{x}_{n-1} + \bar{x}_n) = \\
(x_n - \bar{x}_n)(x_n - \bar{x}_{n-1})
$$

* 因此, 需要在遍历过程中, 维护$\bar{x}_n$和$\bar{x}_{n-1}$, 由$\bar{x}_{n-1}$得到$\bar{x}_{n}$的公式如下:
  $$
  \bar{x}_n = \bar{x}_{n-1} + \frac{1}{n} (x_n - \bar{x}_{n-1})
  $$

* 因此, 在遍历的时候, 维护三个值`s, old_mean, mean`

  * `s`负责$(x_i - \bar{x}_i)(x_i - \bar{x}_{i-1})$的和, 每一层循环加一次.
  * `old_mean`负责维护$\bar{x}_{i-1}$.
  * `mean`负责维护$\bar{x}_i$.

* 优点: 减少了访存次数, 提高了稳定性.
