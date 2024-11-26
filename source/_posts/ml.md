---
title: 基础机器学习的一些内容
categories: AI-HPC
mathjax: true
---



## Basic Question

1. 监督学习:

* 给定一个矩阵$X = [x_1, x_2, ..., x_m] \in R^{n\times m}$, 和一个结果矩阵$Y \in R^{n \times 1}$
  * $n$是样本的个数.
  * $m$是feature的个数.
* 用这个矩阵$X$作为训练集, 得到一个拟合函数$f$, 输入新的样本$x_i \in R^{1 \times m}$, 经过$y = f(x)$, 能够得到正确的结果$y \in R^{1\times 1}$.

2. 无监督学习:



## Feature Extraction & Visualization

* 基本问题: 给定的特征可能有信息之间的重叠, 需要通过算法将特征进行正交化.

### PCA

* PCA的基本思路是: 



#### UMAP

* UMAP的全称是: Uniform Manifold Approximation and Projection for dimension reduction.



## Algorithm Design



### 线性模型

* 前提: 数据是线性可分的.

#### 线性回归

* 模型: $f(x_i) = \omega^Tx_i + b$.
  * 其中$\omega \in R^{m\times 1}$, 表示每个特征的权重, $b \in R^{1 \times 1}$表示bias.
  * $\omega$可以用最小二乘法求出.



#### Fisher线性判别

* Fisher线性判别的核心思路是: 通过最大化类间散度, 最小化类内散度.
* 模型:
  * 假设$x_i \in R^{1 \times m}$是第$i$个样本的特征向量, $\mu_{y_i} \in R^{1 \times m}$是类别$y_i$的所有样本特征的均值.
  * 类内散度矩阵: $S_W = \sum_{i=1}^n(x_i - \mu_{y_i})(x_i - \mu_{y_i})^T$
  * 类间散度矩阵: $S_B = \sum_{c=1}^C n_c(\mu_c - \mu)(\mu_c - \mu)^T$.
    * 其中$C$是类总数, $n_c$是第$c$类的样本数, $\mu_c$是第$c$类的均值向量, $\mu$是所有样本的均值向量.
  * 计算最优投影方向: $S_{W}^{-1}S_BW = \lambda W$, 然后找到前$C - 1$个最大特征值对应的特征向量组成$W \in R^{m \times C-1}$.
  * 对于每一个样本样本$x_i$, 计算它的投影: $Z_i = x_iW$.
    * 然后对于每一个类, 计算投影的均值: $\mu_c = \frac{1}{n_c}\sum_{i=1}^c Z_i$.
  * 然后对于新输入的样本, 投影得到$Z$之后, 计算$Z$与每一个$\mu_c$之间的差距, 差距最小的类就是最好的结果.



#### 感知机

* 模型: $f(x_i) = sgn(\omega^Tx_i + b)$, 其中$sgn$在数据大于等于0时输出1, 否则输出0.



### 逻辑回归

* 模型: $f(x_i) = \frac{1}{1 + e^{-w^Tx_i}}$.



#### MLP



#### SVM



#### Bayes Algorithm



#### 决策树与随机森林



#### Hidden Markov Models





### K-means算法



### Hopfield NN



## Training



### 损失函数

* MSE: Mean Squared Error, 假设$\hat{Y} = f(X)$.

  * $MSE = \frac{1}{n}(Y - \hat{Y})^T(Y - \hat{Y})$.
* Cross-Entropy Loss:



### 正则化



### 梯度更新



#### SGD

#### Adam











## Evaluation





### K-fold验证

* 训练时的评估一般用准确率衡量, 并且使用K-fold验证.

  * K-fold验证: 把数据集分成K个互不相交, 但是数量差不多的子集, 每次在K-1个子集上训练, 在留出的子集上验证. 经过K次之后, 取这K次的平均准确率作为验证准确率.

    * LOOCV (Leave-One-Out Cross Validation): 把每个样本都作为验证集一次. 假设样本数量是n, 相当于n-fold验证.



### 准确率指标

* 最终的评估有如下指标可以选择:
  * 准确率.
  * 混淆矩阵 (confusion matrix): 
    * 第`i`行和第`j`列表示: 一个样本实际的label是`i`, 你的模型预测成`j`的数量.

* 如果是二分类模型, 还有如下指标:
  * Sensitivity(灵敏度): 所有正类样本中, 模型预测为正类所占比例.
  * Specificity(特异度): 所有负类样本中, 模型预测为负类所占比例.
  * ROC曲线: 一般来说, 模型会输出一个样本是正类的概率$p$.
    * 通过调整不同的$p$, 会得到不同的Sensitivity (也叫TPR (True Positive Rate)), 也可以得到不同的FPR (所有负类样本中, 模型识别成正类的比例).
    * 那么将不同的$(FPR, TPR)$做为点, 画出的曲线就是ROC曲线.
    * ROC曲线越靠近左上角, 表示模型性能越好.

  * AUC (曲线下面积): ROC曲线和横轴围成的面积, 越靠近1, 模型性能越好, 如果是0.5, 表示模型没有区分能力.

