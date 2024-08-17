---
title: 时间复杂度
categories: 算法
mathjax: true
---

## 数据范围推导时间复杂度

用C++解题需要控制在$10^7$到$10^8$左右.


* 数据规模是30, 时间复杂度可以是指数.
* 数据规模是$10^2$, 是$O(n^3)$.
* 数据规模是$10^3$, 是$O(n^2)$.
* 数据规模是$10^5$, 是$O(nlogn)$.
* 数据规模是$10^6$或$10^7$, 是$O(n)$.
* 数据规模到$10^9$, 是$O(\sqrt n)$
* 数据规模到$10^{18}$, 是$O(logn)$.