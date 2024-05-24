---
title: 信号与系统
categories: 信号处理
mathjax: true
---



## 信号分类/常见信号

* 连续信号: $x(t)$: 其中$t$是连续的时间.
* 离散信号: $x[n]$: 其中$n$是采样点.
* 周期信号:
  * 连续: $x(t) = x(t + mT)$, 其中$m \in Z$, $T$是最小正周期.
  * 离散: $x[n] = x[n + mN]$.
* 奇信号:
  * $x(t) = -x(-t)$.
  * $x[n] = -x[-n]$.
* 偶信号:
  * $x(t) = x(-t)$.
  * $x[n] = x[-n]$.

> 任何一个信号都可以拆分成奇信号和偶信号拼接的形式, 并且拆法唯一.

$$
x(t) = \frac{x(t) + x(-t)}{2} + \frac{x(t) - x(-t)}{2}
$$

* 信号的能量: $E = \int_{t_0}^{t_1}|x(t)^2|dt$

* 信号的功率: $P = \frac{1}{t_1 - t_0}E$.

* 单位阶跃信号: 
  $$
  u(t) = \begin{cases}
  1 & x > 0 \\
  0 & x < 0
  \end{cases}
  $$

  * $u(t)$在$t = 0$处没有定义, 可以等于任何值.

* 冲激信号:
  $$
  \delta(t) = \begin{cases}
  +\infty & x = 0\\
  0 & x \neq 0
  \end{cases}
  $$

  * $\delta(t) = \frac{du(t)}{dt}$.
  * $\int_{-\infty}^{\infty} \delta(t)dt = 1$.

* 抽样函数:
  $$
  Sa(t) = \begin{cases}
  \frac{sin(t)}{t} & t \neq 0 \\
  1 & t = 0
  \end{cases}
  $$

  * $\int_{-\infty}^{\infty}Sa(t)dt = \pi$.
  * $\int_{0}^{\infty}Sa(t)dt = \frac{\pi}{2}$.
  * $Sa(t)$​是一个偶函数.



> 证明$\int_{0}^{\infty}Sa(t)dt = \frac{\pi}{2}$

* 令$I(a) = \int_{0}^{\infty}Sa(t)e^{-at} dt$.

* 然后用$sin(t) = \frac{e^{jt} - e^{-jt}}{2j}$带入积分, 可以求得:

  $I(a) = -arctan(a) + \frac{\pi}{2}$.

* $I(0) = \int_{0}^{\infty}Sa(t)dt = \frac{\pi}{2} = \frac{\pi}{2}$.



## 信号的自变量变换

* 假设信号是$x(t)$, 那么信号$x(at + b)$应该如何确定?
  * 变为标准形式: $x(a(t + \frac{b}{a}))$
  * 如果$a < 0$, 上下反转.
  * 如果$|a| > 0$, 压缩, 如果$|a| < 0$拉伸.
  * 如果$\frac{b}{a} > 0$, 向左平移, 否则向右平移.
* 任何信号可以表示成冲激信号与原信号的卷积:
  * $x(t) = \int_{-\infty}^{\infty}x(u)\delta(t-u)du$.
  * $x[n] = \sum_{k=-\infty}^{\infty}x[k]\delta[n-k]$.



## 常见的系统

### 线性系统

