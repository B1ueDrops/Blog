---
title: 卡尔曼滤波数学推导
categories: 信号处理
mathjax: true
---





## 问题描述

* 根据状态空间方程, 可以得到一个系统的状态$X_k$的精确值: $X_k = AX_{k-1} + Bu_{k} + \omega$​.
  * 其中, $u_k$是系统输入, $\omega$是噪声, 噪声没有办法精确估计, $\omega \sim N(0, Q)$, 其中$Q = E[\omega\omega^{k}]$是$w$的协方差矩阵.
* 测量值$Z_k = HX_k + v$, 其中:
  * $H$取决于传感器.
  * $v$表示测量误差, $v \sim N(0, R)$.

* 在实际过程中, 可以得到以下两个数值:
  * $X_k$的先验估计$\hat{X_k}$, 这个是用$\hat{X_k} = AX_{k-1} + Bu_k$计算得到的.
  * 测量值$Z_k$, 根据测量值, 也可以得到$X_k$的测量值$\hat{X_k} = H^{-1}Z_k$.
* 现在, 已经有了关于$X_k$的先验估计值和测量值, 需要把这两个值进行融合, 来让$X_k$的波动最小, 从而获得更精确的$X_k$的后验估计值, 这个过程就叫做卡尔曼滤波.



## 推导过程

* 假设$X_k$是一个$n$维向量, $X_k = (x_1, x_2, ..., x_n)^T$
* 假设符号$\bar{X_k}$就表示卡尔曼滤波计算得到的后验估计值.
* 后验估计值:

$$
\bar{X_k} = (I_n - G_k)\hat{X_k} + G_kH^{-1}Z_k = \hat{X_k} + G_k(H^{-1}Z_k - \hat{X_k})
$$

* 令$G_k = K_kH$, 其中$K_k$叫做卡尔曼增益.

$$
\bar{X_k} = \hat{X_k} + K_{k}(Z_k - H\hat{X_k})
$$

* 定义误差$e_k = X_k - \bar{X_k}$, 这个误差同样服从高斯分布$e_k \sim N(0, P_k)$, 其中$P_k$是协方差矩阵, 首先计算$e_k$的表达式:

$$
e_k = X_k - \bar{X_k} = X_k - \hat{X_k} - K_kZ_k + K_kH\hat{X_k}
$$

* 然后, 将$Z_k = HX_k + v$代入, 可以得到 (其中, $\hat{e_k}$叫做先验误差,  $\hat{e_k} = X_k - \hat{X_k}$)

$$
e_k = X_k - \hat{X_k} - K_kHX_k - K_kv + K_kH\hat{X_k} = (X_k - \hat{X_k}) - K_kH(X_k - \hat{X_k}) - K_kv \\
=(I - K_kH)(X_k - \hat{X_k}) - K_kv = (I - K_kH)\hat{e_k} - K_kv
$$

* 然后, 计算$e_ke_k^{T}$的表达式:

$$
e_ke_k^{T} = [(I - K_kH)\hat{e_k} - K_k v][\hat{e_k}^T(I - H^TK_k^T) - v^TK_k^T] = \\
(I - K_kH)\hat{e_k}\hat{e_k}^T(I - H^TK_k^T) - (I - K_kH)\hat{e_k}v^{T}K_k^{T} - K_kv\hat{e_k}^T(I - H^TK_k^T) + K_kvv^TK_k^T
$$

* 然后需要计算误差的协方差矩阵$P_k = e_ke_k^{T}$
  * $E[(I - K_kH)\hat{e_k}v^{T}K_k^{T}] = (I - K_kH)E[\hat{e_k}v^{T}]K_k^T$.
    * 然后, 由于$\hat{e_k}$和$v^T$独立, 可以得到上面的表达式等于: $(I - K_kH)E[\hat{e_k}]E[v^{T}]K_k^T$​ 
    * 又由于$E[\hat{e_k}] = E[v^{T}] = 0$, 这个表达式最终等于0.
  * 同理, $E[K_kv\hat{e_k}^T(I - H^TK_k^T)]$最终也是0.
  * $E[(I - K_kH)\hat{e_k}\hat{e_k}^T(I - H^TK_k^T)] = (I - K_kH)\hat{P_k}(I - H^TK_k^T)$, 因为$\hat{P_k}= E[\hat{e_k}\hat{e_k}^T]$.
  * $E[K_kvv^TK_k^T] = K_k RK_k^T$.
  * 那么, $P_k$最终的结果就是:

$$
P_k = (I - K_kH)\hat{P_k}(I - H^TK_k^T) + K_k RK_k^T = \\
\hat{P_k} - \hat{P_k}H^TK_k^{T} - K_kH\hat{P_k} + K_kH\hat{P_k}H^TK_k^{T} + K_{k}RK_k^{T}
$$

* 然后, 总体的优化目标是让$tr({P_k})$最小, 需要让$\frac{tr({P_k})}{K_k} = 0$.

$$
tr(P_k) = tr(\hat{P_k}) - 2tr(K_kH\hat{P_k}) + tr(K_kH\hat{P_k}H^TK_k^{T}) + tr(K_kRK_k^T)
$$

* 然后计算$\frac{tr(P_k)}{K_k}$:

$$
\frac{tr(P_k)}{K_k} = 0 - 2(H\hat{P_k})^T + 2K_kH\hat{P_k}H^T + 2K_kR = 0
$$

* 然后, 根据协方差矩阵的性质$\hat{P_k} = \hat{P_k}^T$, 可以求出:

$$
K_k = \frac{\hat{P_k}H^T}{H\hat{P_k}H^T + R}
$$

* 现在需要计算$\hat{P_k}$, $\hat{P_k} = E[\hat{e_k}\hat{e_k}^T]$.
* 其中:

$$
\hat{e_k} = X_k - \hat{X_k} = AX_{k-1} + Bu_{k} + \omega - A\hat{X}_{k-1} - Bu_k = A(X_{k-1} - \hat{X}_{k-1}) + \omega = A\hat{e}_{k-1} + \omega
$$

* 然后, 计算$\hat{e}_k\hat{e}_k^T$:

$$
(A\hat{e}_{k-1} + \omega)(\hat{e}_{k-1}^TA^T + \omega^T) = A\hat{e}_{k-1}\hat{e}_{k-1}^TA^T + A\hat{e}_{k-1}\omega^T + \omega\hat{e}_{k-1}A^T + \omega\omega^T
$$

* 其中, $\hat{e}_{k-1}$和$\omega$相互独立, 并且它们的期望都是0, 因此, $\hat{P_k}$可以写成:

$$
\hat{P_k} = AE[\hat{e}_{k-1}\hat{e_{k-1}}^T]A^T + Q = A\hat{P}_{k-1}A^T + Q
$$

* 在这里, 对于$\hat{P}_{k-1}$, 由于我们会修正误差的协方差矩阵, 这里可以改成$\bar{P}_{k-1}$, 因此, $\hat{P}_k$的表达式是:
  $$
  \hat{P_k} = A\bar{P}_k A^T + Q
  $$
  
* 而将$\hat{P}_k$修正为$\bar{P}_k$, 可以用:

$$
\bar{P}_k = \hat{P_k} - \hat{P_k}H^TK_k^{T} - K_kH\hat{P_k} + K_k(H\hat{P_k}H^T + R)K_k^{T}
$$

* 将$K_k = \frac{\hat{P_k}H^T}{H\hat{P_k}H^T + R}$带入, 可以得到:

$$
\bar{P_k} = \hat{P_k} - \hat{P_k}H^TK_k^{T} - K_kH\hat{P_k} + \hat{P}_kH^TK_k^{T} = (I - K_kH)\hat{P}_k
$$



## 计算步骤

* 第一步, 需要先设定初始值$\hat{X}_0$以及$\bar{P}_0$.
* 第二步, 计算先验估计$\hat{X}_k$, 以及先验误差估计$\hat{P}_k$
  * $\hat{X}_k = A\hat{X}_{k-1} + Bu_{k}$​.
  * $\hat{P}_k = A\bar{P}_{k-1}A^T + Q$
* 第三步: 计算卡尔曼增益$K_k$:
  * $K_k = \frac{\bar{P_k}H^T}{H\bar{P_k}H^T + R}$

* 第四步, 根据$K_k$和测量值$Z_k$, 求出后验估计$\bar{X_k}$以及$\bar{P_k}$ 
  * $\bar{X_k} = \hat{X_k} + K_{k}(Z_k - H\hat{X_k})$
  * $\bar{P}_k = (I - K_kH)\hat{P}_k$



## 扩展卡尔曼滤波

* 对于一个非线性系统, 它的方程是: $\omega$这里需要变成$\omega_{k-1}$, 因为正态分布经过非线性系统后, 不再成为正态分布.

$$
X_k = f(X_{k-1}, u_{k}, \omega_{k-1}) \\
Z_k = h(X_k, v_{k-1})
$$

* 如果要对这个系统进行卡尔曼滤波, 需要把这个系统进行线性化, 这样的卡尔曼滤波叫做扩展卡尔曼滤波(EKF).
* 当计算$X_k$的值时, 可以将函数$f$在$(\bar{X}_{k-1}, u_k, 0)$处进行线性化.
  * 其中$A$是$f$关于$(X_{k-1}, u_k, \omega_{k-1})$的雅可比矩阵在$(\bar{X}_{k-1}, u_k, 0)$上的值 (假设$\omega_{k-1} = 0$).
  * $B$是$f$关于$(X_{k-1}, u_k, \omega_{k-1})$的雅可比矩阵在$(\bar{X}_{k-1}, u_k, 0)$上的值.

$$
X_k = f(\bar{X}_{k-1}, u_k, 0) + A(X_k - \bar{X}_{k-1}) + B\omega_{k-1}
$$

* 同理, 将$h$在$X_k = f(\bar{X}_{k-1}, u_k, 0)$​上线性化:
  * 其中$H$是$h$关于$({X_k}, v_{k-1})$的雅可比矩阵在$(\bar{X}_{k}, 0)$上的值.
  * 其中$J$是$h$关于$({X_k}, v_{k-1})$的雅可比矩阵在$(\bar{X}_{k}, 0)$上的值.

$$
Z_k = h(\bar{X}_k, 0) + H(X_k - \bar{X_k}) + Jv_{k-1}
$$

