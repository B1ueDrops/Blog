---
title: 描述旋转的几种方式
categories: 控制基础
mathjax: true
---



[TOC]



## 欧拉角

* 欧拉角(Euler angle)用线性变换的方式用来描述物体的位姿, 分为静态欧拉角和动态欧拉角.



### 静态欧拉角

* 描述方法:
  * $x, y, z$​分别为三个旋转轴, 逆时针为正方向,  刚体旋转时, 三个旋转轴保持不动.
  * 一个三维空间的旋转由四个自由度来描述, 这四个自由度, 分别是绕$x, y, z$三个旋转轴的旋转的角度, 以及旋转顺序.
    * 注意: 这里刚体的旋转是外旋, 也就是绕固定不动的旋转轴进行旋转, 第二次旋转对第一次旋转没有任何影响.
* 旋转矩阵:

$$
x(\theta) = \begin{pmatrix}
1 & 0 & 0 \\
0 & cos\theta & -sin\theta \\
0 & sin\theta & cos\theta
\end{pmatrix}
\\
\\
y(\theta) = \begin{pmatrix}
cos\theta & 0 & sin\theta\\
0 & 1 & 0\\
-sin\theta & 0 & cos\theta \\
\end{pmatrix}
\\
\\
z(\theta) = \begin{pmatrix}
cos\theta & -sin\theta & 0\\
sin\theta & cos\theta & 0 \\
0 & 0 & 1
\end{pmatrix}
$$





### 动态欧拉角

* 描述方法:
  * $x, y, z$三个旋转轴不再固定, 而是会随着刚体的旋转进行移动.
  * 刚体的旋转可以拆分成按照$x, y, z$​轴进行旋转的角度, 以及旋转顺序这四个自由度. 
    * 注意: 这里刚体的旋转是自旋, 每一次旋转都会在上一次的旋转结果上进行.
  
* 旋转矩阵:
  
  * 在外旋中, 每一次线性变换都是在原空间(基向量不变)的情况下进行的, 矩阵直接左乘即可.
  
  * 在自旋中, 每一次线性变换都是在上一个变换的基础上进行的(基向量改变了), 矩阵乘的顺序需要相反.
  
    * 例如, 假设向量是$v$, 如果是分别绕$x, y, z$外旋$\alpha, \beta, \gamma$, 那么变换矩阵就是$z(\gamma)y(\beta)x(\alpha)$.
    * 如果绕$x, y, z$自旋$\alpha, \beta, \gamma$, 那么变换矩阵就是$x(\alpha)y(\beta)z(\gamma)$​.
  
    * 推导: 
      * 首先绕$x$自旋$\alpha$, 对应的旋转矩阵为$x(\alpha)$.
      * 之后, 需要在新的基向量的基础上进行下一次旋转, 因此旋转矩阵是$[x(\alpha)y(\alpha)x^{-1}(\alpha)]x(\alpha) = x(\alpha)y(\alpha)$, 矩阵乘的顺序相反.





### 万向锁

* 假设用动态欧拉角去描述物体的旋转, 旋转的顺序和角度分别是$x(\alpha), y(\beta), z(\gamma)$.
* 首先, 我先绕$x$自旋$\alpha$, 等价于乘上矩阵$x(\alpha)$.
* 之后, 我再绕$y$自旋$\beta$, 等价于乘上矩阵$y(\beta)$.
  * 这时, 如果$\beta = \frac{\pi}{2}$, 尝试计算$A = x(\alpha)y(\beta)$

$$
A = x(\alpha)y(\frac{\pi}{2}) = \begin{pmatrix}
1 & 0 & 0 \\
0 & cos\alpha & -sin\alpha \\
0 & sin\alpha & cos\alpha
\end{pmatrix}
\begin{pmatrix}
0 & 0 & 1\\
0 & 1 & 0\\
-1 & 0 & 0
\end{pmatrix} = \begin{pmatrix}
0 & 0 & 1\\
sin\alpha & cos\alpha & 0\\
-cos\alpha & sin\alpha & 0
\end{pmatrix}
$$

* 然后, 计算$A^{-1}$:

$$
A^{-1} = \begin{pmatrix}
0 & sin\alpha & -cos\alpha\\
0 & cos\alpha & sin\alpha \\
1 & 0 & 0
\end{pmatrix}
$$

* 然后计算$Az(\gamma)A^{-1}$, 可以得到:

$$
Az(\gamma)A^{-1} = \begin{pmatrix}
1 & 0 & 0\\
0 & cos\gamma & -sin\gamma \\
0 & sin\gamma & cos\gamma
\end{pmatrix} = x(\gamma)
$$

也就是说, 当我经历了$x(\alpha)$和$y(\frac{\pi}{2})$两次自旋之后, 我在这个基础上, 再做任何关于$z$轴的自旋$z(\gamma)$, 都等价于在$x(\alpha)$这个变换之前, 先做$x(\gamma)$.

那么, 对于我这一个变换, 我就有两种等价的方式实现:

* $x(\alpha)y(\frac{\pi}{2})z(\gamma)$
* $x(\gamma + \alpha)y(\frac{\pi}{2})$

因此, 如果$\beta = \frac{\pi}{2}$, 我无论想让这个物体怎么绕$z$轴自旋, 它都会等价成绕$x$轴的自旋, 就少了一个方向的自由度, 也就是万向锁.
