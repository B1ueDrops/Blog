---
title: Pytorch构建模型的基础知识
categories: AI技术
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

## Tensor预处理与计算



### `torch`自带的函数

* 获取维度: `x.shape`
* 生成$N(0, 1)$, 形状是`(a, b, c, d)`的随机张量: `torch.randn(a, b, c, d)`
* 删除大小是1的维度: `x = x.squeeze()`
* 在下标为`n`的维度前面加上大小是1的维度: `x = x.unsqueeze(n)`

### `rearrange`

* `rearrange`用于调整Tensor的维度顺序, 或者组合/拆分某些维度:

  ```python
  from einops import rearrange
  ```

* 下面假设张量`x`的维度是`(b, c, h, w)`:

  * 交换维度:

    ```python
    x = rearrange(x, 'b c h w -> b h w c')
    ```

  * 组合维度:

    ```python
    x = rearrange(x, 'b c h w -> b c (h w)')
    ```

  * 拆分维度:

    ```python
    x = rearrange(x, 'b e (h n) -> b h e n', h = xxx)
    ```

* 如果要将`rearrange`作为模型的一个层:

  ```python
  from einops.layers.torch import Rearrange
  
  Rearrange('b e h w -> b (h w) e') # layer
  ```

  

### `torch.einsum`

* `einsum`的计算模型: 例如`C = einsum('ik, kj -> ij', A, B)`:

  * 翻译成Python的循环是:

    ```python
    for i in range(A.size(0)):
      for j in range(B.size(1)):
        
        total = 0
        for k in range(A.size(1)):
          total += A[i][k] * B[k][j]
        
        C[i][j] = total
    ```

  * 理解方法:

    * 首先, 看左侧比右侧多了哪些下标, 在这里是`k`.
    * 然后, 想象有一个指针, 在`k`这个维度从上向下扫描, 其他下标固定, 两个矩阵的元素对应相乘求和.
    * 之后, 将得到的值累加到固定的下标, 在这里是`C[i][j]`.









## 模型定义

* 实现`__init__`和`forward`函数即可:

```python
import torch.nn as nn

# infer: x = SimpleModel(x)
class SimpleModel(nn.Module):
  
  def __init__(self):
    super().__init__()
    #...
   
 	def forward(self):
    #...
```

* 一般将网络的每个Block分布式定义, 然后把子模块进行拼接.



#### `nn.Parameter`





#### `nn.Linear`

* API的全名是: `nn.Linear(in_features, out_features, bias=True)`

  * 首先, 输入的Tensor的维度必须是`(batch_size, *, in_features)`形式, `*`表示可以有中间维度, 但是不会被处理.

  * 输入`(batch_size, *, in_features)`的Tensor, 输出`(batch_size, *, out_features)`的Tensor.

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

> 对`BatchNorm`的理解

* 图片经过`nn.Conv2d`之后, 会得到`(N, C, H, W)`的一个张量, 其中每一个`C`都对应着一个feature map.
  * `BatchNorm`的过程, 就是固定一个`C`, 然后将`(N, H, W)`的所有数据变成$N(0, 1)$的分布.

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



#### `nn.LayerNorm`

> 对`LayerNorm`的理解

* 语言数据经过几层`MultiHeadAttention`之后, 会得到`(N, n, e)`的一个张量, 其中`(n, e)`就是Embedding Matrix的维度.
* `LayerNorm`是固定`N`, 然后把对应的`(e, n)`数据变成$N(0, 1)$的分布.



* `nn.LayerNorm(normalized_shape)`
  * 假设输入的张量尺寸是`(N, C, H, W)`, 那么`normalized_shape`就是`(C, H, W)`.



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

  

#### 残差连接

* 梯度消失问题: 当神经网络层数过多时, 反向传播会导致梯度变小, 神经网络会收敛缓慢.

* 解决方案: 残差连接 (输出和输入拼接, 作为下一个block的输入)

  ```python
  class ResidualAdd(nn.Module):
  	def __init__(self, fn):
  		super().__init__()
  		self.fn = fn
  
  	def forward(self, x, **kwargs):
  		res = x
  		x = self.fn(x, **kwargs)
  		x += res
  		return x
  ```

* 更加本质的理解: 

  * 如果一个模型套上了残差连接模块, 那么就相当于这个模型学习的是相对原输入$x$的增量$\Delta x$.
  * $x + \Delta x$就是残差模型的输出.
  

## 训练评估指标

* **TP (True Positive)**: 模型预测对了, 本身是目标类, 预测也是目标类.
* **TN (True Negative)**: 模型预测对了, 本身不是目标类, 预测不是目标类.
* **FP (False Positive)**: 模型预测错了, 本身不是目标类, 但是预测为目标类的数量.
* **FN (False Negative)**: 模型预测错了, 本身是目标类, 但是预测为不是目标类..
* **召回率 (Recall):** `TP / (TP + FN)`
  * 召回率是所有目标类样本中, 模型有多大比例能识别出来.
  * 在医院中, 召回率又叫灵敏度(Sensitivity), 表示某种疾病所有阳性的人群中, 医院有多少比例能检测出来.
    * 还有一个指标叫特异性(Specificity), 表示所有阴性的人群中, 医院判断是阴性的比例.
* **准确率 (Accuracy): **`(TP + TN) / (TP + TN + FP + FN)`.
* **F1 Score**: : `2 * (Recall * Accuracy) / (Recall + Accuracy)`.



