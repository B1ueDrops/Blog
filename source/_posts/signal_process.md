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

  * $u(t)$在$t = 0$​处没有定义, 可以等于任何值.
  * 区间$[a, b]$上的, 高度为1的方波可以表示为: $u(t - a) - u(t - b)$.
  
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



### 因果系统

* 因果系统定义: 输出$y(t)$在输入$x(t)$之后发生.

* 因果系统的判据: $x$括号中的数恒小于$y$括号中的数.



### 无记忆系统

* 无记忆系统定义: 一个系统无记忆, 是指$y(t)$的值仅仅依赖于$x(t)$的值.
* 无记忆系统一定是因果系统.



### 可逆系统

* 定义: $x(t)$能够唯一写成$y(t)$的表达形式.



### 稳定系统

* 定义: 对于一个系统$x(t) \rightarrow y(t)$, 如果$x(t)$有界能够推出$y(t)$有界, 那么这个系统就是稳定系统.
* 有界的定义: $\forall t \in R, \exist M, |x(t)| < M$, 那么$x(t)$就是有界.

## LTI系统

* LTI系统的全称是线性时不变系统(Linear Time invarient System, LTI).



### 稳定性

* LTI系统稳定的充要条件是: $\int_{-\infty}^{\infty}|h(t)|dt < +\infty$​.
  * 或者是$\sum_{k=-\infty}^{\infty}|h[k]| < +\infty$​



### 因果性

* LTI系统是因果系统的充要条件是: 当$t < 0$时, $h(t) = 0$​.
  * 如果$t < 0$时, $h(t)$有一些离散的值, 这个定理依然成立 (勒贝格积分).

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

那么离散卷积公式就是: $x[n] * h[n] = \sum_{k=-\infty}^{\infty}x[k]h[n-k]$.

* 离散卷积的计算:
  * 首先求系统的$h[n]$.
  * 然后, 如果已知$x[k]$, 求$y[k]$:
    * 首先把$h[n]$左右翻转.
    * 然后向右移动$k$个单位, 然后从后向前相乘相加到$k$​.

> 离散卷积的信号长度

* 假设$x_1[n]$有$N_1$个元素, $x_2[n]$有$N_2$个元素, 那么$x_1[n] * x_2[n]$有$N_1+ N_2 - 1$​个元素.
* 假设$x_1[n]$在$[a_1, b_1]$有值, $x_2[n]$在$[a_2, b_2]$有值, 那么$x_1[n] * x_2[n]$在$[a_1 + a_2, b_1 + b_2]$有值.



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





## 冲激信号的性质



### 性质1

> 证明: $\int_{-\infty}^{\infty}\delta(t)dt = 1$

* $\int_{-\infty}^{\infty}\delta(t)dt = \int_{-\infty}^{\infty}[\lim\limits_{\Delta\rightarrow 0}\delta_{\Delta}(t)]dt = \lim\limits_{\Delta\rightarrow 0}\int_{-\infty}^{\infty}\delta_{\Delta}(t)dt = 1$.



### 性质2

> 证明: $\int_{-\infty}^{\infty}x(t)\delta(t)dt = x(0)$

$$
\int_{-\infty}^{\infty}x(t)\delta(t)dt = \int_{-\infty}^{\infty}x(t)\lim\limits_{\Delta\rightarrow0}\delta_{\Delta}(t)dt = \lim\limits_{\Delta\rightarrow0}\int_{-\infty}^{\infty}x(t)\delta_{\Delta}(t)dt = \lim\limits_{\Delta\rightarrow0}\int_{0}^{\Delta}x(t)\delta_{\Delta}(t)dt
$$

$$
\lim\limits_{\Delta\rightarrow0}\int_{0}^{\Delta}x(t)\delta_{\Delta}(t)dt = \lim\limits_{\Delta\rightarrow0}\frac{1}{\Delta}\int_{0}^{\Delta}x(t)dt
$$

* 根据积分中值定理, 可以得到:

$$
\lim\limits_{\Delta\rightarrow0}\frac{1}{\Delta}\int_{0}^{\Delta}x(t)dt = \lim\limits_{\Delta\rightarrow0}\frac{1}{\Delta}x(\epsilon) \Delta = \lim\limits_{\Delta\rightarrow0}x(\epsilon), \epsilon \in (0, \Delta)
$$

* 当$\Delta \rightarrow 0$时, 最终的结果就是$x(0)$.



### 勒贝格定义

* 勒贝格定义: 两个函数$f_1(t)$和$f_2(t)$相等是指, 对于任意函数$y(t)$, 都有:
  $$
  \int_{-\infty}^{\infty}y(t)f_1(t)dt = \int_{-\infty}^{\infty}y(t)f_2(t)dt
  $$

* 勒贝格定义说明, 如果两个函数只在有限个点的函数值不相等, 那么它们仍然可以是相等函数.

* 假设两个系统的单位脉冲响应分别是$h_1(t)$和$h_2(t)$, 如果在勒贝格定义下, $h_1(t) = h_2(t)$, 那么就有$x(t) * h_1(t) = x(t) * h_2(t)$.
* 如果要证明一个$f(t)是$$\delta(t)$, 只需要证明: 对于任意函数$y(t)$, 都有$\int_{-\infty}^{\infty}y(t)f(t)dt = y(0)$​.



> 证明: 假设$f(t) = \lim\limits_{\omega \rightarrow \infty}\frac{sin\omega t}{\pi t} = \delta(t)$

* 引理: 如果$x(t)$不是无限震荡函数, 那么有:

$$
\lim\limits_{\omega \rightarrow +\infty}\int_{-\infty}^{\infty}x(t)cos(\omega t)dt = 0 \\
\lim\limits_{\omega \rightarrow +\infty}\int_{-\infty}^{\infty}x(t)sin(\omega t)dt = 0
$$

* 引理的证明: 假设$x(t)$​可导, 那么:

  * 分部积分法: $\int udv = uv - \int vdu$

  $$
  \int_{-\infty}^{\infty}x(t)cos(\omega t)dt = \frac{1}{\omega}\int_{-\infty}^{\infty}x(t)dsin(\omega t) = \frac{1}{\omega}x(t)sin(\omega t) - \frac{1}{\omega}\int_{-\infty}^{\infty}sin(\omega t)x^{'}(t)dt
  $$

  * 当$\omega \rightarrow +\infty$​时, 这个式子等于0 (因为`-`两边都是有界变量乘无穷大的形式).

* 下面回到原定理的证明:

$$
\int_{-\infty}^{\infty} y(t)f(t)dt = \int_{-\infty}^{\infty}y(t)\lim\limits_{\omega \rightarrow \infty}\frac{sin\omega t}{\pi t} dt = \lim\limits_{\omega \rightarrow \infty} \int_{-\infty}^{\infty}\frac{y(t)}{\pi t}  sin\omega t dt
$$

* 对于这个式子, 可以写成:
  $$
  \lim\limits_{\omega \rightarrow \infty} \int_{-\infty}^{\infty}\frac{y(t) - y(0)}{\pi t}sin\omega t dt + y(0) \lim\limits_{\omega \rightarrow \infty} \int_{-\infty}^{\infty} \frac{sin\omega t}{\pi t} dt
  $$

  * 左边, 由于$\frac{y(t) - y(0)}{\pi t}$不是无限震荡函数(在0处的极限可以通过洛必达法则求出), 根据引理, 等于0.
  * 右边, 根据$\int_{-\infty}^{\infty} \frac{sin\omega t}{\pi t} dt = 1$(采样函数的性质), 等于$y(0)$.

* 因此, 证明成功.

### 性质3

> 证明: $x(t)\delta(t) = x(0)\delta(t)$

* 根据勒贝格定义, 证明左边等于右边只需要证明左边的函数和右边的函数相等.

左边:
$$
\int_{-\infty}^{\infty}y(t)[x(t)\delta(t)]dt = y(0)x(0)
$$
右边:
$$
\int_{-\infty}^{\infty}y(t)[x(0)\delta(t)]dt = x(0)\int_{-\infty}^{\infty}y(t)[\delta(t)]dt = x(0)y(0)
$$
因此两边相等.

根据这个式子可以得到: $x(t)\delta(t - t_0) = x(t_0)\delta(t - t_0)$



### 性质4

*  $\delta(f(t)) = \sum_{\forall f(t_0) = 0} \frac{1}{|f^{'}(t_0)|}\delta(t - t_0)$, $t_0$是$f(t)$的零点.



## 卷积的性质

* 交换律: $x(t) * h(t) = h(t) * x(t)$.
* 结合律: $[x(t) * h_1(t)] * h_2(t) = x(t) * [h_1(t) * h_2(t)]$​.
* 分配律: $x(t) * [h_1(t) + h_2(t)] = x(t) * h_1(t) + x(t) * h_2(t)$​​.
* 平移: $x(t + t_0) * h(t - t_0) = x(t) * h(t)$​​.
  * 理解: 左边界不变, 右边界不变, 面积不变.

* 卷积的导数: $\frac{d[x(t) * h(t)]}{dt} = x(t) * \frac{dh(t)}{dt} = \frac{dx(t)}{dt} * h(t)$​
  * 证明可以用微分是LTI和$\delta^{'}(t)$来证明.




## 常见的卷积

### 积分



* 积分器: $\int_{-\infty}^{t}x(\tau)d\tau = x(t) * u(t)$​
  * 积分是一个LTI系统, 其中$h(t) = u(t)$.

### 微分



* 微分器: $\frac{dx(t)}{dt} = x(t) * \delta^{'}(t)$
  * 微分也是一个LTI系统, 其中$h(t) = \delta^{'}(t)$​.

* 其中$\delta^{'}(t) = \frac{d\delta(t)}{dt}$.

> 证明: $\int_{-\infty}^{\infty} y(t)\delta^{'}(t)dt = -y^{'}(0)$

$$
\int_{-\infty}^{\infty} y(t)\delta^{'}(t)dt = \int_{-\infty}^{\infty} y(t)d\delta(t) = y(t)\delta(t) |_{-\infty}^{\infty} - \int_{-\infty}^{\infty} y^{'}(t)\delta(t)dt = -y^{'}(0)
$$





## 函数的正交分解

