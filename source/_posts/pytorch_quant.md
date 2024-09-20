---
title: 模型量化的理论和实战
categories: AI技术
mathjax: true
---



## Pytorch模型量化

### 量化后端

* 首先, 需要明确模型需要跑在什么架构上, 然后指定后端:

  ```python
  # x86架构
  torch.backends.quantized.engine = 'x86'
  # ARM架构
  torch.backends.quantized.engine = 'qnnpack'
  ```

### 量化分类

* 量化大体上可以分为两类:

  * Post Training Quant: 训练结束后, 将模型进行量化.
  * Quant-Aware Training: 量化参数会作为可学习参数, 让模型进行训练.

* Pytorch中, 有两种执行模式, 分别是Eager Mode和FX Graph Mode, 两种执行模式下量化的实现方式不同.

  * 如果需要用FX Graph Mode, 模型需要满足`symbolic traceable`, 判定条件是下面的代码不会出现异常:

    ```python
    import torch
    traced_model = torch.fx.symbolic_trace(model)
    ```

    * `einops`中的`Rearrange, rearrange`操作不满足`symbolic traceable`, 可以用`torch`的原生操作`permute`或者`view`代替.
    * 一般来说, 不含控制流的模型都可以满足`symbolic traceable`, 例如CV类模型.

### Observer

* Observer是一种算法, 用于根据观测到的浮点数据调整量化参数.

  * pytorch中是: `torch.ao.quantization.observer`.

* Observer是一个类, 其中常用的参数是:

  * `dtype`: 量化的数据类型, 默认是`torch.quint8`.

  * `quant_min`: 指定量化后数据的最小值, 默认是`None`, 表示使用0.

  * `quant_max`: 指定量化后数据的最大值, 默认是`None`, 表示使用255 (`torch.quint8`的最大值).

  * `qscheme`: 用来指定量化的类型和粒度, 常用的有如下几个:

    * `torch.per_tensor_affine`
    * `torch.per_channel_affine`
    * `torch.per_tensor_symmetric`
    * `torch.per_channel_symmetric`

  * `reduce_range`: 是一个`bool`值, 如果是`True`, 那么会在原来的量化范围下进行缩减, 具体看下表:

    | 类型                   | `quant_min` | `quant_max` |
    | ---------------------- | ----------- | ----------- |
    | `qint8`                | -128        | 127         |
    | `qint8 reduced range`  | -64         | 63          |
    | `quint8`               | 0           | 255         |
    | `quint8 reduced range` | 0           | 127         |

    

* Activation常用的Observer:

  * Activation一般使用`torch.quint8`类型.
  * Activation一般采用`torch.per_tensor_affine`量化.
  * `HistogramObserver`: 会采用一个直方图记录Activation的分布情况, 并且根据分布情况计算出最合适的`scale`和`zero point`.

* Weights常用的Observer:

  * Weights一般使用`torch.int8`类型.
  * Activation一般采用`torch.per_channel_symmetric`量化.
  * Weights可以直接采用封装好的Observer: `torch.ao.quantization.default_fused_per_channel_wt_fake_quant`来调试.




### FakeQuantize

* `FakeQuantize`是一个模块, 用来模拟量化和反量化的操作:

  * 这个模块在: `torch.ao.quantization.fake_quantize`中.

  * 这个模块的运算是:

    ```txt
    x_out = ( 
    	clamp( 
    		round( x / scale + zero_point ), quant_min, quant_max
    	) - zero_point
    ) * scale
    ```

* `FakeQuantize`也是一个类, 最常用的参数是`observer`, 用来指定需要用到的`Observer`类.

### QConfig/QConfigMapping

* `QConfig`用来配置Activation和Weights所用到的FakeQuantize类.

  * `QConfig`在`torch.ao.quantization`中.
  * `QConfig`有`weight`和`activation`两个参数, 用来指定对应的FakeQuantize类.

* `QConfigMapping`用来将模型不同模块映射到不同的`QConfig`.

  ```python
  # 对模型所有模块用一个QConfig
  qconfig_mapping = QConfigMapping().set_global(global_qconfig)
  ```


### Post Training Quant

#### FX Graph Mode

```python
import torch
from tqdm import tqdm
from torchinfo import summary
from torch.ao.quantization import QConfig, QConfigMapping
from torch.ao.quantization.fake_quantize import FakeQuantize
from torch.ao.quantization.observer import HistogramObserver
from torch.ao.quantization.quantize_fx import prepare_fx, convert_fx
from torch.profiler import profile, record_function, ProfilerActivity


def calibration(dataset, model):
	logger.info('FX static calibration...')
	with torch.inference_mode():
    # 选取一些典型的input, 放到这里进行推理
		for i in tqdm(range(20)):
			data, target = dataset.signal_list[i], dataset.target_list[i]
			data = data.unsqueeze(0)
			output = model(data)

# 测试模型性能
def evaluate_performance(model):
	torch.set_num_threads(1)
	example_input = torch.randn(1, 8, 500)
	with profile(activities=[ProfilerActivity.CPU], record_shapes=True) as prof:
		with record_function("model_inference"):
			model(example_input)
  # 函数会打印模型的CPU time
	print(prof.key_averages().table(sort_by="cpu_time_total", row_limit=10))


# 指定量化的后端
torch.backends.quantized.engine = 'qnnpack'

# Activation的量化配置
act_fake_quant = FakeQuantize.with_args(observer=HistogramObserver.with_args(
	reduce_range=False,
	dtype=torch.quint8, quant_min=0, quant_max=255,
	qscheme=torch.per_tensor_affine,
))

# Weight的量化配置
wt_fake_quant = torch.ao.quantization.default_fused_per_channel_wt_fake_quant

# 构建QConfigMapping
global_qconfig = QConfig(
	activation=act_fake_quant, weight=wt_fake_quant
)
qconfig_mapping = QConfigMapping().set_global(global_qconfig)

example_input = torch.randn(1, 8, 500)
fake_quant_model = prepare_fx(model, qconfig_mapping, (example_input,))

dataset = Benchmark()

# Calibration确定量化参数
calibration(dataset, fake_quant_model)

# 将Fake Quantized模型转换为真实的int8模型
quant_model = convert_fx(fake_quant_model)
torch.save(quant_model.state_dict(), 'model/fx_static_model.pth')

logger.info('Evaluating PTQ FX Mode Static Quant...')

# 测试量化后模型的准确率和推理时间
evaluate_performance(quant_model)
evaluate_accuracy(quant_model, dataset)

```



#### Eager Mode



### Quant-Aware Training

#### FX Graph Mode

```python
import torch
from tqdm import tqdm
from torchinfo import summary
from torch.ao.quantization import QConfig, QConfigMapping
from torch.ao.quantization.quantize_fx import prepare_fx, convert_fx, prepare_qat_fx
from torch.profiler import profile, record_function, ProfilerActivity
from torch.ao.quantization.observer import HistogramObserver, PerChannelMinMaxObserver
from torch.ao.quantization._learnable_fake_quantize import _LearnableFakeQuantize as LearnableFakeQuantize


# 设置后端
torch.backends.quantized.engine = 'qnnpack'
# 利用
act_learnable = lambda range: LearnableFakeQuantize.with_args(
		observer=HistogramObserver,
		quant_min=0,
		quant_max=255,
		dtype=torch.quint8,
		qscheme=torch.per_tensor_affine,
		scale=range / 255.0,
		zero_point=0.0,
		use_grad_scaling=True,
)
# Weight中如果有Conv2d, 那么这个模块就有channel, 用per channel
wt_learnable = lambda channels: LearnableFakeQuantize.with_args(
		observer=PerChannelMinMaxObserver,
		quant_min=-128,
		quant_max=127,
		dtype=torch.qint8,
		qscheme=torch.per_channel_symmetric,
		scale=0.1,
		zero_point=0.0,
		use_grad_scaling=True,
		channel_len=channels,
)

# 全局的一些量化配置
act_fake_quant = LearnableFakeQuantize.with_args(observer=HistogramObserver.with_args(
	reduce_range=False,
	dtype=torch.quint8, quant_min=0, quant_max=255,
	qscheme=torch.per_tensor_affine,
))
wt_fake_quant = torch.ao.quantization.default_fused_per_channel_wt_fake_quant
global_qconfig = QConfig(
	activation=act_fake_quant, weight=wt_fake_quant
)
qconfig_mapping = QConfigMapping().set_global(global_qconfig)

# 给卷积模块设置activation和weight的LearnableFakeQuantize
for name, module in model.named_modules():
	if hasattr(module, 'out_channels'):
		qconfig = torch.quantization.QConfig(
			activation=act_learnable(range=2),
			weight=wt_learnable(channels=module.out_channels)
		)
		qconfig_mapping.set_module_name(name, qconfig)

example_input = torch.randn(1, 8, 500)
qat_model = prepare_qat_fx(model, qconfig_mapping, (example_input,))

# Training on qat_model
trainer = Trainer(qat_model)
qat_model = trainer.start_train()

# Convert it to true model
quant_model = convert_fx(qat_model)
logger.info('Saving Quant Model...')
torch.save(quant_model.state_dict(), 'model/qat_model.pth')

dataset = Benchmark()
logger.info('Evaluating QAT Quant...')
evaluate_performance(quant_model)
evaluate_accuracy(quant_model, dataset)
```





#### Eager Mode

