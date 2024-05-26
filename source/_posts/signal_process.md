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
  * 如果$a < 0$, 左右反转.
  * 如果$|a| > 0$, 压缩, 如果$|a| < 0$拉伸.
  * 如果$\frac{b}{a} > 0$, 向左平移, 否则向右平移.
* 任何信号可以表示成冲激信号与原信号的卷积:
  * $x(t) = \int_{-\infty}^{\infty}x(u)\delta(t-u)du$.
  * $x[n] = \sum_{k=-\infty}^{\infty}x[k]\delta[n-k]$.



## 系统的性质

### 线性系统

* 线性系统的定义: 假设一个系统$x(t)\rightarrow y(t)$, 如果满足:

  * 齐次性: $\forall a \in R, ax(t) \rightarrow ay(t)$.
  * 叠加性: $x_1(t) + x_2(t) \rightarrow y_1(t) + y_2(t)$.

  那么就是线性系统.

* 线性系统的判据: 假设系统是$y(t) = f(x(t))$

  * 每一项都必须是$x$.
  * 每一项的$x$都必须是一次.



### 时不变系统

* 时不变系统的定义: 假设一个系统$x(t) \rightarrow y(t)$, 如果满足:

  * $\forall t_0 \in R, x(t - t_0) \rightarrow y(t - t_0)$.

  那么就是时不变系统.

* 时不变系统的判据:

  * $t$只能在$x$的括号里.
  * $t$只能是$t$, 不能是$-t, 2t, -2t, t^2...$.



## 卷积公式

这里的卷积公式是针对线性时不变系统(Linear Time invarient System, LTI).

对于一个LTI系统, 只要知道了一组信号$x(t)$和$y(t)$之间的对应关系, 就可以知道整个系统的对应关系.

### 离散卷积公式

* 离散系统的单位脉冲序列定义为:
  $$
  \delta[n] = \begin{cases}
  1 & n = 0\\
  0 & n \neq 0
  \end{cases}
  $$

* 离散系统的单位脉冲响应定义为: $h[n]$, $\delta[n] \rightarrow h[n]$, 也就是这个系统对于单位脉冲序列的输出.

* 对于一个LTI系统, $h[n]$就是这个系统的唯一标识, 系统的表达式是: $y[n] = x[n] *h[n] = \sum_{k=-\infty}^{+\infty}x[k]h[n-k]$.



> 离散卷积公式的证明

* 首先, 有$\delta[k] \rightarrow h[k]$​.
* 然后, 根据时不变系统定义: $\delta[n - k] \rightarrow h[n - k]$.
* 然后, 根据线性系统齐次性: $x[k]\delta[n-k] \rightarrow x[k]h[n-k]$.
* 然后, 再根据线性系统叠加性: $\sum_{k=-\infty}^{+\infty}x[k]\delta[n-k] \rightarrow \sum_{k=-\infty}^{+\infty}x[k]h[n-k]$​
* 即: $x[n] \rightarrow y[n]$.



* 离散卷积的计算:
  * 首先求系统的$h[n]$.
  * 然后, 如果已知$x[k]$, 求$y[k]$:
    * 首先把$h[n]$左右翻转.
    * 然后向右移动$k$个单位, 然后从后向前相乘相加到$k$.



### 连续卷积公式

* 假设$x(t) \rightarrow y(t)$是LTI系统.
* **单位脉冲信号的原始形态 **: 任意一个连续函数$x(t)$, 都能通过很多个无限窄的方波来逼近, 这个无限窄的阶梯函数就是$\delta(t)$.
  * 方波函数的原始形态是$\delta_{\Delta}(t)$, 宽度是$\Delta$, 高度是$\frac{1}{\Delta}$.
  * $\lim\limits_{\Delta\rightarrow0} \delta_{\Delta}(t) = \delta(t)$​.
* 对于一个连续函数$x(t)$, 可以通过方波 的形式来划分$x_{\Delta}(t) = \sum_{k=-\infty}^{\infty}x(k\Delta)\delta_{\Delta}(t-k\Delta) \Delta$.
  * 对于任意一个$x_{\Delta}(t)$, 我都可以通过平移$\delta_{\Delta}(t)$来进行构造.
  * $\lim\limits_{\Delta\rightarrow 0}x_{\Delta}(t) = x(t)$.
* 假设$\delta_{\Delta}(t)\rightarrow h_{\Delta}(t)$, 那么根据LTI的性质:
  * $\lim\limits_{\Delta \rightarrow 0}\delta_{\Delta}(t) \rightarrow \lim\limits_{\Delta \rightarrow 0}h_{\Delta}(t)$.
  * 假设极限$\lim\limits_{\Delta \rightarrow 0}h_{\Delta}(t)$​​存在.



> 连续卷积公式推导

* 首先: $\delta_{\Delta}(t)\rightarrow h_{\Delta}(t)$
* 根据时不变性: $\delta_{\Delta}(t - k\Delta)\rightarrow h_{\Delta}(t - k\Delta)$.
* 根据齐次性: $x_{\Delta}(t)\delta_{\Delta}(t - k\Delta)\Delta \rightarrow x_{\Delta}(t)h_{\Delta}(t - k\Delta) \Delta$​
* 根据叠加性: $\sum_{k=-\infty}^{\infty}x_{\Delta}(t)\delta_{\Delta}(t - k\Delta)\Delta \rightarrow \sum_{k=-\infty}^{\infty}x_{\Delta}(t)h_{\Delta}(t - k\Delta)\Delta$​
  * 即$x_{\Delta}(t) \rightarrow \sum_{k=-\infty}^{\infty}x_{\Delta}(t)h_{\Delta}(t - k\Delta)\Delta$

* 然后, $x(t) = \lim\limits_{\Delta\rightarrow 0} x_{\Delta}(t) \rightarrow \lim\limits_{\Delta\rightarrow 0}\sum_{k=-\infty}^{\infty}x_{\Delta}(t)h_{\Delta}(t - k\Delta)\Delta = \int_{-\infty}^{\infty}x(\tau)h(t-\tau)d\tau = y(t)$​.
* 那么卷积公式就是: $y(t) = x(t) * h(t) = \int_{-\infty}^{\infty}x(\tau)h(t-\tau)d\tau$

其中$h(t)$就是系统的单位脉冲响应, 可以唯一标识一个连续的LTI系统.
