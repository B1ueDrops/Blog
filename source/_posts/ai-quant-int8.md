---
title: AI模型的量化部署
categories: AI加速
mathjax: true
---

[toc]

## 模型量化

* 模型量化本质上是一个**离散化**算法, 它可以将模型数据从`float`类型映射到`int8/uint8`类型.
  * 可以对模型的`Weight`进行量化.
  * 可以对模型的输入输出进行离散化(输入输出叫做`Activation`).



> 为什么模型的`Weight`和`Activation`可以进行离散化?

* 因为`Weight`和`Activation`在某一个范围内的分布较集中, 并且较离散.



* **模型量化的TradeOff:**
  * 优势: 收益较大, 例如大模型.
    * 模型尺寸小, 磁盘, 内存占用变低.
    * 模型推理速度快, CPU/GPU有加速的整数运算指令可利用.
    * 设备功耗较低.
  * 缺点: 模型精度降低, 不过一般在可以接受范围.



## 模型量化设计思路

* 假设$W_{f}$代表浮点数的数据, 需要将其转化为由少量`bit`表示的整数数据$W_i$:
  $$
  W_i = round[\frac{W_f - min(W_f)}{max(W_{f}) - min(W_{f})} \times (i_{max} - i_{min}) + i_{min}]
  $$

  * $max(W_f)$表示所有浮点数数据中的最大值.
  * $i_{max}$表示整数类型的最大值.



> 这种量化方法的缺陷?

* $max(W_f) - min(W_f)$可能极大 (离群值).
* $W_f - min(W_f)$可能极小.
* 假设所有数据均分布在$min(W_f)$附近, 那么几乎所有的数据都会映射到$i_{min}$.

<font color=red>**因此, 在模型量化之前, 要充分已知原始数据的分布情况, 并且要根据数据分布情况选择量化方法, 使得量化收益最大.**</font>



## 模型量化流程



### 数据校准

* 数据校准(Calibration) 会对要量化的数据(`Weight`和`Activation`) 的分布进行统计, 然后画出分布图.
  * 这个分布图用来确定量化的范围, 这里面也有一个Tradeoff:
    * 量化范围越大, 则因为`round`操作, 范围内的更多不一样的浮点数会被映射到同一个整数, 也就是Rounding Error增多.
    * 量化范围越小, 则不在上界范围内的浮点数都会被`clip`为边界值, 也就是Clip Error增多.

### 量化粒度

* Per Tensor: 每个Tensor之间的量化独立.

* Per Channel: 对Tensor内部, Channel之间的量化也是独立的.

* 注意: Activation一般不采用Per Channel的量化, 因为计算过后无法dequantize.

### 量化算法

* 假设确定的上界为$T$, 那么量化算法可以表示为:
  $$
  W_i = \frac{i_{max}}{T} \times W_{f} + z
  $$

  * 其中`z`是zero point, 也就是浮点数的0.0映射到整数的哪里.

* 这种量化算法叫做均匀量化(scale quant), 也叫线性量化, 适用于量化数据分布较均匀, 没有太多异常值(outlier)的模型.

  * 大模型一般数据分布不均匀, 会采用非均匀量化, 例如`LLM.int8()`.



### 一些细节

* 对于`Weight`量化, 一般设置`z = 0`, 对于`Activation`, 一般`z`不等于0.

  * 证明: 假设Weight和Activation都使用asym quant, 在进行dequantize的时候:
    $$
    S_{W}(W_{int8} - z_{W}) S_{x}(x_{int8} - z_{x}) = \\
    S_{W}S_{x}W_{int8}x_{int8} - S_{w}z_{W}S_{x}x_{int8} - S_{W}S_{x}z_{x}W_{int8} + S_{W}z_{w}S_{x}z_{x}
    $$

    * 其中, $S_x, z_{w}, S_{x}, z_{x}$都是可以被提前算出来的.
    * $W_{int8}x_{int8}$是sym quant的结果.
    * 但是第二项依赖于$x_{int8}$, 也就是量化后推理的输入, 如果要进行dequantize, 那么我就需要保存这个结果, 并且需要额外的计算.
    * 因此, 可以选择让$z_{W} = 0$, 让第二项消失, dequantize的时候就可以少算一项, 而且可以不用保留任何推理输入.

* 最后映射到的类型有两种选择, 分别是`int8`和`uint8`.
  * 还有`ReLU`的模型会考虑到`uint8`.
  * 或者需要看CPU/GPU是否对`int8 * uint8`有特殊支持.



