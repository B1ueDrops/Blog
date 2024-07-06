---
title: 理解Transformer
categories: 深度学习
mathjax: true
---



## Task

给定一句话, 让你预测一句话的最后一个词会是什么?

## Word2Vec

* 首先, 将这句话划分成不同的token, 例如一句话可能分为不同单词, 不同单词就是token.
* 然后, 将这些token通过**Embedding**机制转换成一个多维度的向量.
  * 如果两个词意思相近, 那么这两个词的vector在高维空间中的位置相近, 换句话说, 点积越大.
  * 也就是说, 向量的空间位置承载了单词的语义信息.



### Embedding

* 首先, 你需要有一个pre-trained矩阵, 你可以根据每一个token, 从这个矩阵中获取这个token应该被转换成什么样的vector.
  * 这个矩阵相当于一个词表, 你需要让你的模型知道有什么词.
  * 这个矩阵叫做**Embedding Matrix**, 可以记作$W_E$​​.
  * 这个$W_E$是可学习的.
* 这个$W_E$, 就引入了两个衡量LLM的重要参数:
  * $W_E$的列数, 代表模型能够处理的token的数量, 叫做**Context Size**.
  * $W_E$​的行数, 代表了word vector的维度数.



## Self-Attention



### Attention Matrix

* 经过Word2Vec之后, **这个vector只能表示这个token的含义**, 但是这个token的准确语义与上下文的token有关, 因此需要提供一种机制, 来根据上下文的token去调整这个vector.

* 假设$W_E = [\vec{E_1}, \vec{E_2}, \vec{E_3},\vec{E_4}]$.

* 对于每一个$\vec{E_i}$, 它会主动发起一个“询问”, 询问前面的token是否有关于描述自己的信息.

  * 这个“询问”用数学表示就是用一个矩阵$W_Q$, 经过$\vec{Q_i} = W_Q\vec{E_i}$, 得到$\vec{E_i}$的询问向量$\vec{Q_i}$​.
  * $W_Q$的列数和vector的维度数目一样, 行数是一个比较小的值.
  * 一般来说, $\vec{Q_i}$的维度会比$\vec{E_i}$要小很多.

* 同样, 对于每一个$\vec{E_i}$, 他也会主动提供自己的回答信息.

  * 这个回答也有一个矩阵叫$W_K$, 和$W_Q$一样的维度, 经过$\vec{K_i} = W_K\vec{E_i}$, 可以得到回答向量$\vec{K_i}$.

* 这样, 对于每一个$\vec{Q_i}$和每一个$\vec{K_j}$, 都可以计算一个点积$\vec{Q_i}\cdot\vec{K_j}$, 这样会形成一个矩阵:
  $$
  \begin{bmatrix}
  \vec{Q_1}\cdot\vec{K_1} & \vec{Q_2}\cdot\vec{K_1} & \vec{Q_3}\cdot\vec{K_1} & \vec{Q_4}\cdot\vec{K_1} \\
  \vec{Q_1}\cdot\vec{K_2} & \vec{Q_2}\cdot\vec{K_2} & \vec{Q_3}\cdot\vec{K_2} & \vec{Q_4}\cdot\vec{K_2} \\
  \vec{Q_1}\cdot\vec{K_3} & \vec{Q_2}\cdot\vec{K_3} & \vec{Q_3}\cdot\vec{K_3} & \vec{Q_4}\cdot\vec{K_3} \\
  \vec{Q_1}\cdot\vec{K_4} & \vec{Q_2}\cdot\vec{K_4} & \vec{Q_3}\cdot\vec{K_4} & \vec{Q_4}\cdot\vec{K_4} \\
  \end{bmatrix}
  $$
  
* 对于一个$\vec{Q_i}$来说, 为了避免后面的token对其语义产生影响, 需要用$\vec{K_j}, j \leqslant i$, 来进行点乘, 其余的值需要设置成$-\infin$(softmax中, $-\infin$可以被映射成0), 这个机制叫做**Masking**.

$$
\begin{bmatrix}
\vec{Q_1}\cdot\vec{K_1} & \vec{Q_2}\cdot\vec{K_1} & \vec{Q_3}\cdot\vec{K_1} & \vec{Q_4}\cdot\vec{K_1} \\
-\infin & \vec{Q_2}\cdot\vec{K_2} & \vec{Q_3}\cdot\vec{K_2} & \vec{Q_4}\cdot\vec{K_2} \\
-\infin & -\infin & \vec{Q_3}\cdot\vec{K_3} & \vec{Q_4}\cdot\vec{K_3} \\
-\infin & -\infin & -\infin & \vec{Q_4}\cdot\vec{K_4} \\
\end{bmatrix}
$$

* 之后, 对这个矩阵的每一列使用Softmax求一下概率分布, 最终输出的矩阵叫做**Attention Matrix**.



### Update Word Vector

* 首先, 对于每一个$\vec{E_i}$, 准备一个矩阵$W_V$, 然后得到$\vec{V_i} = W_V \vec{E_i}$.

  * 其中, $W_V$的行数和列数都等于词向量的维度数.

* 然后, 假设你要更新向量$\vec{E_i}$, 那么在Attention Matrix的第$i$列, 把第$i$列的每一个值当作一个权重, 和$\vec{V_i}$进行加权平均, 最终得到$\Delta \vec{E_i}$, 然后更新$\vec{E_i}$, 用数学语言来描述就是:
  $$
  \Delta \vec{E_i} = (\vec{Q_i}\cdot\vec{K_1})\vec{V_1} + (\vec{Q_i}\cdot\vec{K_2})\vec{V_2} + ... + (\vec{Q_i}\cdot\vec{K_4})\vec{V_4}\\
  \vec{E_i} = \vec{E_i} + \Delta \vec{E_i}
  $$
  



---

