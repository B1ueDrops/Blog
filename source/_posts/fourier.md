---
title: 傅立叶级数分解
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
    



## 傅立叶变换的总结

* 傅立叶变换: $F(\omega) = \int_{-\infin}^{\infin} f(t)e^{-i\omega t}dt$.
* 逆变换: $f(t) = \frac{1}{2\pi} F(\omega)e^{i\omega t} d\omega$



## Python中的傅立叶变换

