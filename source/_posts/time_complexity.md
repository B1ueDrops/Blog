---
title: 时间复杂度的一些知识
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



## 主定理

* 主定理(Master's Theorem) 用来分析递归算法的时间复杂度.

* 假设一个问题的规模为$n$, 求解这个问题的时间复杂度是$T(n)$.

  * 如果递归地, 将这个问题分为$k$个子问题, 每个子问题的规模是$\frac{n}{m}$, 解决子问题后, 将问题合并的时间复杂度是$f(n)$, 假设这里$f(n) = n^d$.

  * 那么就有:
    $$
    T(n) = kT(\frac{n}{m}) + n^d
    $$

* 主定理的推导: 在递归树中, 合并层的节点的cost不同:

  * 合并第0层 (第0层指的是最顶层): 最顶层只有一个节点,$T(n)$, 从下一层的所有节点合并到这个节点的cost是 $n^d$.
  * 合并第1层: $k(\frac{n}{m})^d$
  * 合并第2层: $k^2(\frac{n}{m^2})^d$
  * 合并第$i$层: $k^i (\frac{n}{m^i})^d$, 其中$i = log_mn$.

* 现在, 将各个层的cost相加:
  $$
  T(n) = n^d + k(\frac{n}{m})^d + k^2(\frac{n}{m^2})^d + ... + k^i (\frac{n}{m^i})^d = \\
  n^d[(\frac{k}{m^d})^0 + (\frac{k}{m^d})^1 + (\frac{k}{m^d})^2 + ... + (\frac{k}{m^d})^i]
  $$

  * 在这种情况下, 需要根据$\frac{k}{m^d}$和1的关系进行分类讨论, 下面是结论:

$$
T(n) = \begin{cases}
O(n^d), &\ log_mk < d \\
O(n^dlog_mn), &\ log_mk = d \\
O(n^{log_mk}), &\ log_mk > d
\end{cases}
$$

