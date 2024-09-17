---
title: 模型量化技术
categories: AI技术
mathjax: true
---

[toc]

## 模型量化的概念

* 模型量化本质上是一个**离散化**算法, 它可以将模型数据从`float`类型映射到`int8/uint8`类型.
  * 可以对模型的`Weight`进行量化.
  * 可以对模型的输入输出进行离散化(输入输出叫做`Activation`).



> 为什么模型的`Weight`和`Activation`可以进行离散化?

* 因为`Weight`和`Activation`在某一个范围内的分布较集中, 并且较离散.





* **模型量化的Trade off:**
  * 优势: 收益较大, 例如大模型.
    * 尺寸变小.
    * 推理速度变快.
    * 运算的Throughput变高: CPU/GPU有整数运算加速指令.
    * 功耗变低.
  * 缺点: 准确率降低.



## 模型量化算法

* 算法思路: 将集合中的某个浮点数, 距离集合中浮点数的最大值与最小值的距离, 映射到整数范围.
* 缺陷:
  * 如果数据分布较集中, 那么众多浮点数可能会映射到同一个整数, 精度影响大.


<font color=red>**因此, 在模型量化之前, 要充分已知原始数据的分布情况, 并且让量化的最大值喝最小值贴近分布集中的区域.**</font>

### 数据校准

* 数据校准(Calibration) 会对要量化的数据(`Weight`和`Activation`) 的分布进行统计, 然后画出分布图.
  * 这个分布图用来确定量化的最大值和最小值, 这里面也有一个Tradeoff:
    * 量化范围越大, 则因为`round`操作, 范围内的更多不一样的浮点数会被映射到同一个整数, 也就是Rounding Error增多.
    * 量化范围越小, 则不在最大值和最小值范围内的浮点数都会被`clip`为边界值, 也就是Clip Error增多.

### 量化粒度

* Per Tensor: 每个Tensor之间的量化独立.

* Per Channel: 对Tensor内部, Channel之间的量化也是独立的.

* 注意: Activation一般不采用Per Channel的量化, 因为计算过后无法dequantize.

### 量化分类

* 均匀量化 (scale quant): 量化是线性映射, 适用于分布数据均匀, 没有太多异常值的模型.

  * 非均匀量化: 一般用于大模型 (数据分布不均匀).

* 对称量化 (sym quant): 浮点数的0会映射到整数的0.

  * 非对称量化:

    * 假设确定的上界为$T$, 那么量化算法可以表示为:
      $$
      W_i = \frac{i_{max}}{T} \times W_{f} + z
      $$

      * 其中`z`是zero point, 也就是浮点数的0.0映射到整数的哪里.



### 量化的等价性




### 一些细节

* 对于`Weight`量化, 一般对称量化, 对于`Activation`, 一般用非对称量化:

  * 证明: 假设Weight和Activation都使用非对称量化, 在进行反量化的时候:
    $$
    S_{W}(W_{int8} - z_{W}) S_{x}(x_{int8} - z_{x}) = \\
    S_{W}S_{x}W_{int8}x_{int8} - S_{w}z_{W}S_{x}x_{int8} - S_{W}S_{x}z_{x}W_{int8} + S_{W}z_{w}S_{x}z_{x}
    $$

    * 其中, $S_x, z_{w}, S_{x}, z_{x}$都是可以被提前算出来的.
    * $W_{int8}x_{int8}$是对称量化的结果.
    * 但是第二项依赖于$x_{int8}$, 也就是量化后推理的输入, 如果要进行反量化, 那么我就需要保存这个结果, 并且需要额外的计算.
    * 因此, 可以选择让$z_{W} = 0$, 让第二项消失, 反量化的时候就可以少算一项, 而且可以不用保留任何推理输入.

* 量化后的整数类型有两种选择, 分别是`int8`和`uint8`.
  * 还有`ReLU`的模型会考虑到`uint8`.
  * 或者需要看CPU/GPU是否对`int8 * uint8`有特殊支持.



## Pytorch模型量化



### 量化分类

* Pytorch中, 根据什么时候开始量化, 可以分为以下类型:
  * Post Training Quantization (PTQ): 在训练结束之后再开始量化, 模型训练时不知道有量化这回事.
  * Quantization-Aware Training (QAT): 模型训练时, 会考虑到自己的输入/输出是`int`类型, 并进行特殊优化.
* 根据Pytorch的执行模式, 量化也可以分为三种模式:
  * Eager Mode Quantization.
  * FX Graph Mode Quantization.
  * Pytorch 2 Export Quantization.

### 量化后端

可以通过`torch.backends.quantized.supported_engines`查看. 一般来说分为三种引擎:

* `fbgemm`: `x86`机器上的引擎.
* `qnnpack`: `ARM`机器上的引擎, 可适配移动端.
* `TensorRT/cuDNN`: GPU上的引擎.



### FX Graph Mode Quantization

* 首先, 需要检测模型是否满足`symbolic traceable`, 只有满足条件才能继续.

  ```python
  import torch
  
  # 调用这个函数, 如果没有异常, 就是symbolic traceable
  traced_model = torch.fx.symbolic_trace(model)
  ```

* 





```python
import copy

import torch
import torch.nn as nn
import torch.nn.init as init
from torch.quantization import quantize_fx

if __name__ == "__main__":

	# Define Simple Model
	net = nn.Sequential(
		nn.Linear(784, 256),
		nn.ReLU(),
		nn.Linear(256, 10)
	)

	# Init parameter
	for layer in net:
		if isinstance(layer, nn.Linear):
			init.xavier_uniform_(layer.weight)
			if layer.bias is not None:
				init.zeros_(layer.bias)

	# Quantize
	net.eval()
	# 用于prepare_fx中确定输入tensor的shape
	example_input = torch.randn(1, 784)
	torch.backends.quantized.engine = 'qnnpack'
	qconfig_dict = {"": torch.quantization.default_dynamic_qconfig}
  
  # 输入example_input的原因是: Pytorch需要构建FX Graph
	prepared_net = quantize_fx.prepare_fx(net, qconfig_dict, example_input)
	quantized_net = quantize_fx.convert_fx(prepared_net)

	# Inference
	quantized_net.eval()
	input_data = torch.randn(1, 784)
	with torch.no_grad():
		output = net(input_data)
		q_output = quantized_net(input_data)
		print(output)
		print(q_output)

```







### 量化方式选择标准

> Static Quant vs. Dynamic Quant ?

* 这个主要看模型数据的特性:
  * Dynamic Quant适合数据具有控制流特性的模型, 例如语言, NLP模型及其对应的算子(`nn.RNN`, `nn.LSTM`等).
  * Static Quant适合数据不具有控制流特性, 但是分布较稳定的模型, 例如图片, CV类模型(`nn.Conv`).
    * 这个也解释了为什么`nn.Conv`不能用Dynamic Quant.



