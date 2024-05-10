---
title: 傅立叶级数与变换
categories: 信号处理
mathjax: true
---



## 三角函数的正交性

* **三角函数系**是一个集合: $\{sin(0x), cos(0x), sin(x), cos(x), sin(2x), cos(2x), ..., sin(nx), cos(nx)\}$.

* 三角函数系具有正交性:

  * 从三角函数系中, 任取两个**不同**的函数, 有: 
    $$
    \int_{-\pi}^{\pi}sin(nx)cos(mx) = 0, n \neq m \\
    \int_{-\pi}^{\pi}sin(nx)sin(mx) = 0, n \neq m \\
    \int_{-\pi}^{\pi}cos(nx)cos(mx) = 0, n \neq m \\
    $$
    



## 周期是$2\pi$的函数傅立叶级数

* 假设函数$f(x)$是周期为$2\pi$的函数, 那么$f(x)$可以写成:
  $$
  f(x) = \frac{a_0}{2} + \sum_{n=1}^{\infin}(a_ncosnx + b_nsinnx)
  $$
  

* 求$\int_{-\pi}^{\pi}f(x)dx$, 可以得到$a_0 = \frac{1}{\pi}\int_{-\pi}^{\pi}f(x)dx$
* 求$\int_{-\pi}^{\pi}f(x)cosmx\ dx$, 利用三角函数正交性, 可以得到$a_n = \frac{1}{\pi}\int_{-\pi}^{\pi}f(x)cosnx\ dx$
* 求$\int_{-\pi}^{\pi}f(x)sinmx\ dx$, 可以得到$a_n = \frac{1}{\pi}\int_{-\pi}^{\pi}f(x)sinnx\ dx$



## 周期是$2L$的函数傅立叶级数

* 如果函数$f(x)$的周期是$2\pi$, 那么$f(\frac{\pi}{L}x)$的周期就是$2L$​.
* 假设$g(x) = f(\frac{\pi}{L}x)$, $f(x) = \frac{a_0}{2} + \sum_{n=1}^{\infin}(a_ncosnx + b_nsinnx)$​, 那么$g(x) = \frac{a^{'}_0}{2} + \sum_{n=1}^{\infin}(a^{'}_ncos\frac{n\pi}{L}x + b^{'}_nsin\frac{n\pi}{L}x)$.
  * 其中, $a^{'}_0 = \frac{1}{\pi}f(\frac{\pi}{L}x)d(\frac{\pi}{L}x) = \frac{1}{L} \int_{-L}^{L}g(x)dx$
  * $a^{'}_n = \frac{1}{L}\int_{-L}^{L}g(x)cos\frac{n\pi}{L}x\ dx$
  * $b^{'}_{n} = \frac{1}{L}\int_{-L}^{L}g(x)sin\frac{n\pi}{L}x\ dx$

由于在工程中, 时间没有负数, $\int_{-L}^{L}$一般写成$\int_{0}^{T}$, 其中$T = 2L$, 然后定义$\omega = \frac{2\pi}{T}$, 那么:
$$
a_0 = \frac{2}{T}\int_{0}^{T}g(x)dx \\
a_n = \frac{2}{T}\int_{0}^{T}g(x)cosn\omega x\ dx \\
b_n = \frac{2}{T}\int_{0}^{T}g(x)sinn\omega x\ dx \\
$$
这个就是周期是$T$的函数$g(x)$的傅立叶变换, 当$T \rightarrow \infin$​时, 就是一般的函数.

$g(x) = \frac{a_0}{2} + \sum_{n=1}^{\infin}(a_ncosn\omega x + b_nsinn\omega x)$



## 傅立叶级数的复数形式

* 根据欧拉公式$e^{ix} = cosx + isinx$, 可以得到:
  * $cosx = \frac{1}{2}(e^{ix} + e^{-ix})$
  * $sinx = -\frac{1}{2}i(e^{ix} - e^{-ix})$
* 将$cosx$和$sinx$的表达式带入 傅立叶级数的表达式:
  * $g(x) = \frac{a_0}{2} + \sum_{n=1}^{+\infin} \frac{a_n - ib_n}{2}e^{in\omega x} + \sum_{n=1}^{+\infin} \frac{a_n + ib_n}{2} e^{-in\omega x}$
  * 将最后一项的$n$变成$-n$, 可以得到: $g(x) = \sum_{n=0}^0 \frac{a_n - ib_n}{2}e^{in\omega x}+ \sum_{n=1}^{+\infin} \frac{a_n - ib_n}{2}e^{in\omega x} + \sum_{-\infin}^{-1}\frac{a_{-n} + ib_{-n}}{2}e^{in\omega x}$.
  * 因此, $g(x) = C_n e^{in\omega x}$, 其中:
    * $n = -1, -2, ...$时, $C_n = \frac{a_{-n} + ib_{-n}}{2}$.
    * 当$n = 0, 1, 2, $时, $C_n = \frac{a_n - ib_n}{2}$.
* 将$a_n, b_n$的表达式带入$C_n$, 可以发现:$C_n = \frac{1}{T}f_{0}^{T}f(x)e^{-in\omega x}\ dx$.

那么, 对于任何一个周期函数$f(t)$, 假设它的周期是$T$, 它可以写成傅立叶级数:
$$
f(t) = \sum_{n=-\infin}^{\infin} C_n e^{in\omega t}\\
C_n = \frac{1}{T}\int_{0}^T f(t)e^{-in\omega t}dt
$$


## 傅立叶变换

* 根据周期函数傅立叶级数的结果, 真正决定一个函数的是$C_n$
  * $C_n$是一个函数, 给定一个$n$, 就会对应一个复数, 其中给定一个$n$就等价于给定一个$n\omega_0$, 这个函数就是频谱.
* $f(t) = \sum_{n=-\infin}^{\infin} \frac{\Delta \omega}{2\pi} \int_{-\frac{T}{2}}^{\frac{T}{2}}f(t)dt\ e^{in \omega_0 t}$​​
  * 其中, $\Delta \omega$就是频谱两个离散的$\omega$之间的距离, 当周期趋于正无穷时, $\Delta \omega$趋于0, $\omega_0$就相当于一个基频率.
* 当$T \rightarrow \infin$​时,
  * $\sum_{n=-\infin}^{\infin}\Delta \omega = \int_{-\infin}^{\infin}d\omega$​
  * $n\omega_0$可以用一个连续的$\omega$来代替.
  *  $f(t) = \frac{1}{2\pi}\int_{-\infin}^{\infin}\int_{-\infin}^{\infin}f(t) e^{in\omega_0 t}dt\ d\omega$​
* 那么傅立叶变换就是:

$$
F(\omega) = \int_{-\infin}^{\infin}f(t) e^{i\omega t}dt \\
f(t) = \frac{1}{2\pi} \int_{-\infin}^{\infin}F(\omega)d\omega
$$

