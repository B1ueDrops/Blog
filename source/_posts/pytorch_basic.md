---
title: Pytorch构建模型的基础知识
categories: AI加速
mathjax: true
---



## 计算后端

* 在训练脚本开头, 一般用这么一句话指定计算后端设备:

  ```python
  # 0代表Nvidia GPU的id
  device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
  ```

* 之后, 对于模型和数据, 都需要:
  * 在Tensor喂到模型前, 需要`x = x.to(device)`.
  * 模型需要`model = model.to(device)`.

## Tensor预处理

> 获取Tensor的维度信息:

* `output.shape`: 返回元组.
* `output.size()`: 返回`torch.Size`对象.

一般两者可以互换使用, `torch.Size`兼容元组操作.



> 生成随机Tensor:

* 生成特定维度的随机Tensor:
  * `torch.randn(a, b, c, d)`: 用于生成一个维度是`(a, b, c, d)`的, 服从$N(0, 1)$的张量.
  * `torch.rand(a, b, c, d)`: 生成的数据服从`[0, 1)`的均匀分布.



> 将Feature map转为全连接层的输入:

* `x.view(x.size(0), -1)`:
  * 例如`x`是一个维度为`(B, C, H, W)`的Tensor, 那么经过这个操作可以得到`(B, C * H * W)`的Tensor.



> 增加/删除一个新维度:

* `x = x.unsqueeze(0)`: 在第0维度前面加上一个维度.
  * 假设Tensor维度是`(3, 4)`, 经过`unsqueeze(0)`变成`(1, 3, 4)`.
  * `unsqueeze(-1)`在最后一个位置增加新维度.
* `x = x.squeeze()`: 删除所有大小是1的维度.



> 维度交换:

`x = x.permute(0, 2, 1, 3)`: 将原来维度编号是`(0, 1, 2, 3)`的Tensor变成`(0, 2, 1, 3)`.



## 模型定义

* 实现`__init__`和`forward`函数即可:

```python
import torch.nn as nn

# infer: x = SimpleModel(x)
class SimpleModel(nn.Module):
  
  def __init__(self):
    super(SimpleModel, self).__init__()
    #...
   
 	def forward(self):
    #...
```

* 一般将网络的每个Block分布式定义, 然后把子模块进行拼接.



#### `nn.Linear`

* API的全名是: `nn.Linear(in_features, out_features, bias=True)`

  * 首先, 输入的Tensor的维度必须是`(batch_size, in_features)`形式.

  * 输入`(batch_size, in_features)`的Tensor, 输出`(batch_size, out_features)`的Tensor.

* `nn.Linear`代表的是一种线性变换:
  
  * 这种方法可以将一个特定空间的向量, 映射到高维度/低维度的另一个隐空间.
  
  * 这个模块的参数, 也就是$W, b$描述的是: 原空间的基向量映射到隐空间中的坐标.
  
    
  

#### `nn.Conv2d`

* 定义: `nn.Conv2d(in_features, out_features, kernel_size, stride, padding)`.
  * `in_features`: 输入的通道数, 也就是`C`.
  * `out_features`: 输出的通道数, 也就是**卷积核的数量**, 对应输出张量中的`C'`是多少.
  * `kernel_size`: 是一个tuple/整数, 表示卷积核的`(height, width)`.
  * `stride`: 也是一个tuple/整数, 表示卷积核在高度和宽度上的步长.
    * 注意, 卷积核的移动是先移动一个维度, 移动到结束之后再从另一个维度移动一次, 并不是同时移动两个维度.
  * `padding`: 也是一个tuple/整数, 表示在高度/宽度上填充像素的层数, 有不同的padding模式, 默认是0填充.
  
* 输入输出: `(N, C, H, W) -> (N, C', H', W')`
  
  
  
  $$
  H' = \lfloor \frac{H + 2 \times padding - (kernel\ height - 1) - 1}{stride\ height} + 1\rfloor\\
  W' = \lfloor \frac{W + 2 \times padding - (kernel\ width - 1) - 1}{stride\ width} + 1\rfloor\\
  $$

> 为什么卷积操作可以提取到特征?

* 傅立叶变换的本质是希尔伯特空间的变换, 基向量是正弦余弦函数.
* 正弦余弦函数在时域中是全局的, 可以通过一个特殊函数, 让正弦余弦函数只在某个窗口内有值, 其余窗口衰减, 这样就可以通过希尔伯特空间变换, 得到局部特征的特征值.

* 因此, 对于卷积操作:
  * 卷积核与图片对应相乘, 可以理解为时域信号与基向量(窗口化)做内积.
  * 不同位置的卷积核, 本质上就是不同的基向量.



#### `nn.BatchNorm2d`

* **Internal Covariate Shift**

  * 在机器学习系统中, 有一个重要的假设就是: 训练集和测试集是独立同分布的.
  * 但是, 在神经网络中, 每一层接收的输入分布都是会变化的, 每一层神经网络就需要调整参数适应这些变化, 导致神经网络收敛缓慢, 这个现象叫Internal Covariate Shift.
  * 通过在每一层网络前面加上一层BatchNorm, 可以将数据拉回$N(0, 1)$的分布, 从而加速收敛, 并且可以提高模型的泛化性.

* **BatchNorm公式:** 对于一个Batch的数据$X$:
  $$
  \hat{X} = \frac{X - \mu}{\sqrt{\sigma^2 + \epsilon}} \\
  Y = \gamma \hat{X} + \beta
  $$

  * 其中, $\epsilon$是为了防止方差过小, 导致除0.
  * $\gamma, \beta$是两个可学习的参数, 解释为: 在进行分布规范化之后, 模型自己决定是否要恢复特征的原始分布.

* `nn.BatchNorm2d(num_features)`:

  * `num_features`就是`(N, C, H, W)`中的`C`.
  * 输入输出的Tensor的维度都是`(N, C, H, W)`.

* BatchNorm的位置: 一般是`Conv2d -> BatchNorm2d -> ReLU`.



#### 激活函数

* 激活函数的作用:
  * 首先, 神经网络内部的变换很多都是线性的, 如`nn.Linear, nn.Conv2d`等.
  * 如果线性的layer之间堆叠, 其实等价为一个layer, 都是线性.
  * 因此, 如果要区分不同的layer, 需要引入非线性的因素进行隔离, 这就是激活函数.
* 激活函数通常使用`ReLU`.
  * API: `nn.ReLU(inplace=False)`.



#### `F.softmax`

* Softmax的作用是将一个$n \times 1$的向量转成一个$n \times 1$的概率向量, 概率向量中的元素属于$[0, 1)$, 并且加和是1.
* 假设$x = (x_1, x_2, ..., x_n)$

$$
Softmax(x) = \frac{e^{x_i - max(x_i)}}{\sum_{i=1}^n e^{x_i - max(x_i)}}
$$

* 分子分母同时除以$e^{max(x_i)}$是为了防止$e^{x_i}$数值过大, 超过`float32`的表示范围.

* API: `torch.nn.functional.softmax(x, dim=None)`

  * `dim`就是$n$维向量中的$n$.

  



## 训练评估指标

* **TP (True Positive)**: 模型预测对了, 本身是目标类, 预测也是目标类.
* **TN (True Negative)**: 模型预测对了, 本身不是目标类, 预测不是目标类.
* **FP (False Positive)**: 模型预测错了, 本身不是目标类, 但是预测为目标类的数量.
* **FN (False Negative)**: 模型预测错了, 本身是目标类, 但是预测为不是目标类..
* **召回率 (Recall):** `TP / (TP + FN)`
  * 召回率是所有目标类样本中, 模型有多大比例能识别出来.
* **准确率 (Accuracy): **`(TP + TN) / (TP + TN + FP + FN)`.
* **F1 Score**: : `2 * (Recall * Accuracy) / (Recall + Accuracy)`.



## 现成的机器学习训练/测试脚本

