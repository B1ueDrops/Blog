---
title: 概率论与数理统计
categories: AI
mathjax: true
---



# 一元概率论



## 数学期望

* 数学期望衡量均值.

* 离散型随机变量的数学期望:
  * 如果$P(X = x_k) = p_k$
  * 并且$\sum_{k=1}^{\infty} x_kp_k$绝对收敛.
  * 那么离散型随机变量$X$的期望存在,   值是$\sum_{k=1}^{\infty} x_kp_k$.
* 连续型随机变量的数学期望:
  * 假设随机变量$X$的概率密度函数$f(x)$.
  * 并且$\int_{-\infty}^{+\infty}xf(x)dx$ 绝对收敛.
  * 那么$E[X]$存在, 值就是$\int_{-\infty}^{+\infty}xf(x)dx$​
* 数学期望的性质:
  * $E[cX] = cE[X]$, $c$是常数.
  * $E[X \pm Y] = E[X] \pm E[Y]$.



## 方差

* 方差衡量数据波动.
* 方差: $D[X] = E[(X - E[X])^2]$​.
  * 离散型: $D[X] = \sum_{k} (x_k - E[X])^2p_k$​
  * 连续型: $D[X] = \int_{-\infty}^{+\infty}(x - E[X])^2f(x)dx$​
  * 计算式: $D[X] = E[X^2] - (E[X])^2$
* 方差的量纲和原变量的量纲不同, 用标准差.

* 方差的性质:
  * $D[c] = 0$.
  * $D[X + c] = D[x]$.
  * $D[cX] = c^2D[X]$.
  * $D[cX + b] = c^2D[X]$.
  * 如果$X, Y$独立, 那么$D[X \pm Y] = D[X] + D[Y]$.

## 协方差/相关系数

* 协方差和相关系数都衡量两个随机变量$X, Y$之间的相关性.
* 协方差(Covariance):
  * 定义: $Cov(X, Y) = E[(X - E[X])(Y - E[Y])]$.
    * 另一种表达式: $Cov(X, Y) = E[XY] - E[X]E[Y]$​.
  * 协方差用来衡量随机变量之间的相关性:
    * 协方差大于0, 正相关, 数值越大, 相关性越强.
    * 协方差等于0, 独立.
    * 协方差小于0, 负相关.

* 相关系数:
  * $\rho(X, Y) = \frac{Cov(X, Y)}{\sqrt{D[X]} \sqrt{D[Y]}}$​
  * $\rho(X, Y) \in [-1, 1]$.
    * 如果$\rho(X, Y) = 1/-1$, 那么$X, Y$是完全线性相关.
    * 如果是0, 则独立.

# 多元概率论



## 随机向量

* 随机向量$X = (X_1, X_2, X_3, ..., X_m)^T$
  * 其中$X$可以看成一个特征集合, 而$X_i$​就是其中的一个特征.
  * 如果有$n$个样本点, 那么就可以形成一个$n \times m$​的矩阵, 这就是数据集. 



## 数学期望

* 随机向量$X$的数学期望 (这个数学期望也是个向量):

  * $E[X] = (E[X_1], E[X_2], ..., E[X_m])^T$​​​
* 数学期望的性质:
  * 设$a = (a_1, a_2, ..., a_m)$, 那么$E[aX] = aE[X]$



## 方差

* 随机向量$X$的方差:
  * $D[X] = (D[X_1], D[X_2], ..., D[X_m])^T$
* 方差的性质:
  * 设$a = (a_1, a_2, ..., a_m)$, 那么$D[aX] = aa^TD[X]$.

## 协方差矩阵

* 随机向量$X$的协方差矩阵$Cov(X)$, 协方差矩阵的维度是$m \times m$

  * 定义$\sigma_{ij} = Cov(X_i, X_j)$

  * 协方差矩阵就是:
    $$
    Cov(X) = \begin{pmatrix} 
    \sigma_{11} & \sigma_{12} &.. & \sigma_{1m} \\ 
    \sigma_{21} & \sigma_{22} &.. & \sigma_{2m} \\
    ..\\
    \sigma_{m1} & \sigma_{m2} &.. & \sigma_{mm} \\
    \end{pmatrix}
    $$
    



## 相关系数矩阵

* 随机变量$X$的相关系数矩阵$\rho(X)$和协方差矩阵维度相同, 都是$m \times m$.

  * 相关系数矩阵: $\rho_{ij} = \frac{\sigma{ij}}{\sqrt{D[X_i]} \sqrt{D[X_j]}}$

    



## 典型相关分析

* 典型相关分析可以衡量两个随机向量$X$和$Y$之间的相关关系.
* 核心: 对于随机变量$X = (X_1, X_2, ..., X_p)$和$Y = (Y_1, Y_2, ..., Y_q)$:
  * 我找一组线性组合, 使得$U = aX$, $V = bY$, 其中$a = (a_1, a_2, ..., a_p), b = (b_1, b_2, ..., b_q)$.
  * 然后, 在$D[U] = 1$, 并且$D[V] = 1$的前提下, 使得$\rho(U, V)$的值最大, 其中$U, V$​叫做第一典型相关变量.

推导过程:

* 首先计算$D[U]$和$D[V]$:

$$
D[U] = D[aX] = aa^TD[X] = 1 \\
D[V] = bb^TD[Y] = 1
$$



* 