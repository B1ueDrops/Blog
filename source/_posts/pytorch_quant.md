---
title: AI模型的量化
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



## Pytorch模型量化

> 官方文档: https://pytorch.org/docs/stable/quantization.html
>
> 官方实例代码: https://pytorch.org/blog/quantization-in-practice/

### 量化分类

* 根据Activation的量化方式, Pytorch的量化可以分为以下类型:
  * Post Training Quantization (PTQ): 在训练结束之后再开始量化:
    * **Weights Only**: 不对Activation进行量化.
    * **Dynamic Quantization**: Activation在模型进行推理时, 动态确定量化参数, 读取时是以浮点数类型进行.
    * **Static Quantization**: Activation在模型进行推理时提前量化好, 需要进行Calibration确定量化参数.
  * Quantization-Aware Training (QAT): 在模型训练时, 考虑上量化参数, 用来提高精度.
* 根据Pytorch的执行模式, 量化也可以分为三种模式:
  * Eager Mode Quantization.
  * FX Graph Mode Quantization.
  * Pytorch 2 Export Quantization.





### 量化后端

进行量化之前, 要明确Pytorch的后端是哪个, 可以通过`torch.backends.quantized.supported_engines`查看. 一般来说分为三种引擎:

* `fbgemm`: `x86`机器上的引擎.
* `qnnpack`: `ARM`机器上的引擎, 可适配移动端.
* `TensorRT/cuDNN`: GPU上的引擎.

### Eager Mode Quantization

* 由于在Eager Mode中, 算子是一步一步执行的, 没有构建计算图, 因此, 量化模块和反量化模块需要作为一个算子手动添加到模块内部.



#### Dynamic Quant

* 代码实例:

  ```python
  import torch
  import torch.nn as nn
  import torch.nn.init as init
  
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
    # 这里要根据你的架构设置
  	torch.backends.quantized.engine = 'qnnpack'
    
    # qconfig_spec表示参与量化的layer
    # inplace=False表示不修改原模型
  	quantized_net = torch.quantization.quantize_dynamic(
  		model=net, qconfig_spec={nn.Linear}, dtype=torch.qint8, inplace=False
  	)
  
  	# Inference
  	quantized_net.eval()
  	input_data = torch.randn(1, 784)
  	with torch.no_grad():
  		output = net(input_data)
  		q_output = quantized_net(input_data)
  		print(output)
  		print(q_output)
  
  ```



#### Static Quant



### FX Graph Mode Quantization

#### Dynamic Quant

* 实例代码: 注意, `prepare_fx`中, 需要提供`example_input`来帮助生成FX Graph.

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
  
  

#### Static Quant



### 量化方式选择标准

> Static Quant vs. Dynamic Quant ?

* 这个主要看模型数据的特性:
  * Dynamic Quant适合数据具有控制流特性的模型, 例如语言, NLP模型及其对应的算子(`nn.RNN`, `nn.LSTM`等).
  * Static Quant适合数据不具有控制流特性, 但是分布较稳定的模型, 例如图片, CV类模型(`nn.Conv`).
    * 这个也解释了为什么`nn.Conv`不能用Dynamic Quant.



