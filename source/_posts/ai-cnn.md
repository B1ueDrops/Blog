---
title: CNN的一些基础知识
categories: AI
---



## 为什么有CNN

如果用全连接的神经网络处理图像, 会有如下几个问题:

* 图像的输入维度很多, 全连接会导致参数过多.
* 图像中的物体都具有局部不变的特征., 全连接很难捕捉.
* 模拟人脑, 人脑在接收刺激后, 只有特定区域的神经元会响应刺激, 而不是所有神经元.



## 卷积

如果是一维, 假设卷积核$w$的长度为$m$, 那么卷积的表达式就是: $y[i] = \sum_{k=0}^mw[k]x[i+m-1-k]$, 记做$y = w * x$.

* 直观理解就是拿一个长度为$m$的滑动窗口, 和原始信号的逆序做一个点乘.

二维卷积也类似, 只是卷积核$w$也是二维, 相当于拿一个二维的滑动窗口在矩阵中扫描, 然后计算卷积.

> 卷积是逆序操作, 如果是正序操作, 就叫做互相关(Cross-Correlation), 很多卷积实际上是互相关.



## Stride和Padding

Stride指的是卷积核在移动时的步长.

* Stride的作用是为了让输入数据能够成倍成倍地缩小尺寸.

Padding指的是在数据周围加一圈0的操作, 例如padding = 1的话:

* 如果是一维, 那么在开头和结尾都会补0.
* 如果是二维, 那么矩阵的一圈都会补0.



## 多通道卷积

假设我输入的数据的维度是$(N, M, C)$, 分别是行数, 列数和通道数.

假设卷积核的维度是$(n, m, C, k)$​​​.

* 卷积核的通道数要和输入的通道数一致.

* $k$是卷积核的个数.

假设Stride是$S$, Padding是$P$, 那么经过卷积操作得到的数据的维度就是:
$$
(\lfloor \frac{N + 2P - n}{S} \rfloor+1, \lfloor \frac{M + 2P - m}{S} \rfloor+1, k)
$$


## 卷积层Conv

卷积层的作用是提取局部特征, 不同的卷积核就是不同的特征提取器.

卷积能充分利用图像的局部信息, 所以常用来处理图像.

卷积层可以表示为: $Z^p = W^p * X + b^p$, 然后用非线性的激活函数输出得到输出的特征$Y^p = f(Z^p)$.

* $p$表示卷积核的ID.
* 激活函数一般用ReLU.

卷积层的参数比全连接要少很多, 参数只有卷积核$w$和参数$b$.



## 池化层Pooling

如果你嫌特征太多了, 想减少一点特征的数量, 那么有两种方法:

* 增加Stride.
* 用Pooling层.

> 一般来说, 更好地方法还是增加Stride, Pooling层用的越来越少.

Pooling层计算过程和卷积类似, 也是用一个滑动窗口, 只不过, Pooling是选择滑动窗口中的最大值/滑动窗口所有值的平均值作为结果, 分别叫Max pooling或者Average pooling.



## 典型的CNN结构

典型的CNN结构一般是:

* 卷积层 + ReLU提取若干次特征.
* 加一个Pooling层.
* 最后接一个全连接层用Softmax输出.






