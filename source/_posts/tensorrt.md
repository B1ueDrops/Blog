---
title: TensorRT推理加速教程
categories: AI-HPC
mathjax: true
---

## 编译运行过程

* 首先需要引入如下三个头文件：

  ```cpp
  #include "NvInfer.h"
  #include "NvOnnxParser.h"
  #include "cuda_runtime.h"
  
  // 智能指针
  #include <memory>
  ```

* 编译的命令如下：注意把`include`和`lib`都要添加全：

  ```cpp
  g++ test.cpp \
                  -L/cuda-12.4/lib64 \
                  -I/cuda-12.4/include \
                  -L/TensorRT-10.7.0.23/lib \
                  -I/TensorRT-10.7.0.23/include \
                  -lcudart -lnvinfer -lnvonnxparser \
  ```

* TensorRT的大部分组件在`nvinfer1`这个namespace下面.



## TensorRT推理过程

> 第一步：继承`ILogger`， 创建`TRTLogger`对象：

* TensorRT的大部分组件在`nvinfer1`这个namespace下面。

    ```cpp
    inline const char *severity_string(nvinfer1::ILogger::Severity t) {
        switch (t) {
            case nvinfer1::ILogger::Severity::kINTERNAL_ERROR: return "INTERNAL_ERROR";
            case nvinfer1::ILogger::Severity::kERROR: return "ERROR";
            case nvinfer1::ILogger::Severity::kWARNING: return "WARNING";
            case nvinfer1::ILogger::Severity::kINFO: return "INFO";
            case nvinfer1::ILogger::Severity::kVERBOSE: return "VERBOSE";
            default: return "UNKNOWN";
        }
    }
    
    class TRTLogger : public nvinfer1::ILogger {
    public:
        void log(Severity severity, nvinfer1::AsciiChar const *msg) noexcept override {
            if (severity <= Severity::kINFO) {
                if (severity == Severity::kWARNING) {
                    // yellow warning
                    printf("\033[33m[%s]: %s\033[0m\n", severity_string(severity), msg);
                } else if (severity <= Severity::kERROR) {
                    // red error
                    printf("\033[31m[%s]: %s\033[0m\n", severity_string(severity), msg);
                } else {
                    printf("[%s]: %s\n", severity_string(severity), msg);
                }
            }
        }
    };
    
    int main() {
        
        TRTLogger logger;
        // ...
        return 0;
    }
    ```

    * `noexcept`表示这个日志函数永远不会抛出异常，避免编译器生成异常相关代码影响性能。
    * `override`表示你这个函数是重写的父类的虚函数，加上了`override`能在编译器检查你是不是真的重写了父类的虚函数，而不是定义了一个相同名称的函数（这种应该在编译期报错）。

    * 日志等级`Severity`在`ILogger`类中有定义, 类型是`enum`:

      ```cpp
      enum class Severity: int32_t {
          kINTERNAL_ERROR = 0,
          kERROR = 1,
          kWARNING = 2,
          kINFO = 3,
          kVERBOSE = 4,
      };
      ```

      * `int32_t`在`<cstdint>`中，可以保证所有平台上枚举的数据类型都是32位有符号整数。
      * 日志等级越高，错误越不严重：
        * `kVERBOSE`: 一些debug信息。
        * `kINFO`: 其他一些信息。
        * `kWARNING`: 应用程序有错，但是TensorRT能够恢复。
        * `kERROR`: 应用程序出现错误.
        * `kINTERNEL_ERROR`: TensorRT内部的错误。



> 第二步：创建`nvinfer1::IBuilder`对象：

* IBuilder是一个核心组件：

  * 输入：神经网络的定义，和转换引擎相关的配置参数。

  * 输出：神经网络的推理引擎。


  ```cpp
  // 智能指针在#include<memory>, std namespace下
  auto builder = std::unique_ptr<nvinfer1::IBuilder>(nvinfer1::createInferBuilder(logger));
  if (!builder) return false;
  ```



> 第三步：创建`nvinfer1::IBuilderConfig`对象，用来配置转换引擎的参数：

```cpp
auto config = std::unique_ptr<nvinfer1::IBuilderConfig>(builder->createBuilderConfig());
if (!config) return false;
```



> 第四步：将`onnx`模型文件导入

* 首先从ibuilder生成神经网络的定义`nvinfer1::INetworkDefinition`:

  ```cpp
  // 这个flag本质上是1，表示batch size是需要人为显式指定的.
  auto flag = nvinfer1::NetworkDefinition::KSTRONGLY_TYPED:
  auto network = std::unique_ptr<nvinfer1::INetworkDefinition>(builder->createNetworkV2(flag);
  if (!network) return false;
  ```

* 然后创建`onnx` parser, `nvonnxparser::IParser`这个类在`NvOnnxParser.h`中：

  ```cpp
  auto parser = std::unique_ptr<nvonnxparser::IParser>(nvonnxparser::createParser(*network, logger));
  if (!parser) return false;
  ```

* 然后通过parser将神经网络导入：

  ```cpp
  const std::string onnx_path = "...";
  auto parsed = parser->parseFromFile(onnx_path.c_str(), static_cast<int>(Logger::Severity::kWARNING));
  if (!parsed) return false;
  ```



>第五步： 根据`INetworkDefinition`的一些api查看模型的输入输出信息：

```cpp
// Print Network IO Dimensions
void printNetworkIODims(const std::unique_ptr<nvinfer1::INetworkDefinition> &network) {
        int nb_inputs = static_cast<int>(network->getNbInputs());
        int nb_outputs = static_cast<int>(network->getNbOutputs());
        // print input dims
        for (int i = 0; i < nb_inputs; i ++) {
                nvinfer1::ITensor *input = network->getInput(i);
                if (input != nullptr) {
                        std::cout << "Input Dimensions: ";
                        std::cout << '[';
                        nvinfer1::Dims dims = input->getDimensions();
                        for (int d = 0; d < dims.nbDims; d ++) {
                                if (d != dims.nbDims - 1)
                                        std::cout << dims.d[d] << ' ';
                                else
                                        std::cout << dims.d[d];;
                        }
                        std::cout<< ']' << std::endl;
                }
        }
        for (int i = 0; i < nb_outputs; i ++) {
                nvinfer1::ITensor *output = network->getOutput(i);
                if (output != nullptr) {
                        std::cout << "Output Dimensions: ";
                        std::cout << '[';
                        nvinfer1::Dims dims = output->getDimensions();
                        for (int d = 0; d < dims.nbDims; d ++) {
                                if (d != dims.nbDims - 1)
                                        std::cout << dims.d[d] << ' ';
                                else
                                        std::cout << dims.d[d];;
                        }
                        std::cout<< ']' << std::endl;
                }
        }
}
```



> 第六步：根据network和config创建engine和context

```cpp
auto engine = std::unique_ptr<nvinfer1::ICudaEngine>(builder->buildEngineWithConfig(*network, *config));

auto context = std::unique_ptr<nvinfer1::IExecutionContext>(engine->createExecutionContext());
```



> 第七步：在Host端，将推理的输入数据准备好：

* 假设输入的维度是`(B, C, H, W)`, 输出的维度是`(B, N)`, 其中`N`是类别总数。

* 那么在Host端，需要有两个`float`数组：

  ```cpp
  float host_input_data[B * C * H * W];
  // 填充host_input_data
  float host_output_data[B * N];
  ```

  * 注意一定要写成`float a[N]`这种，不能用`malloc`，否则`sizeof`给的是指针的大小，而不是数组的大小，后面`memcpy`会有问题。



> 第八步：初始化Device端的输入输出：

```cpp
float *device_input_data;
float *device_output_data;
cudaMalloc((void**)&device_input_data, sizeof(host_input_data));
cudaMalloc((void**)&device_output_data, sizeof(host_output_data));
cudaMemcpy(device_input_data, host_input_data, sizeof(host_input_data), cudaMemcpyHostToDevice);
```



> 第九步：借用context在GPU端运行推理：

```cpp
// bindings存储device端input和output的指针
float *bindings[] = { device_input_data, device_output_data };
bool success = context->executeV2((void **)bindings);
```



> 第十步：计算结果拷贝回Host端，然后释放Device显存：

```cpp
cudaMemcpy(host_output_data, device_output_data, sizeof(host_output_data), cudaMemcpyDeviceToHost);
cudaFree(device_input_data);
cudaFree(device_output_data);

// 然后host_output_data就是推理结果
```



## TensorRT完整实例代码

* 这份代码的模型选择的是TensorRT官方安装包的`mnist.onnx`模型，输入是MNIST数据集的pgm图像:

```cpp
#include "NvInfer.h"
#include "NvOnnxParser.h"
#include "cuda_runtime.h"

#include <cmath>
#include <memory>
#include <fstream>
#include <iostream>

class Logger : public nvinfer1::ILogger {
  void log(Severity severity, const char *msg) noexcept override {
    if (severity <= Severity::kWARNING)
      std::cout << msg << std::endl;
  }
};

void printNetworkIODims(
	const std::unique_ptr<nvinfer1::INetworkDefinition> &network) {
	int nb_inputs = static_cast<int>(network->getNbInputs());
	int nb_outputs = static_cast<int>(network->getNbOutputs());
	// print input dims
	for (int i = 0; i < nb_inputs; i++) {
		nvinfer1::ITensor *input = network->getInput(i);
		if (input != nullptr) {
			std::cout << "Input Dimensions: ";
			std::cout << '[';
			nvinfer1::Dims dims = input->getDimensions();
			for (int d = 0; d < dims.nbDims; d++) {
				if (d != dims.nbDims - 1)
					std::cout << dims.d[d] << ' ';
				else
					std::cout << dims.d[d];
			}
			std::cout << ']' << std::endl;
		}
	}
	for (int i = 0; i < nb_outputs; i++) {
		nvinfer1::ITensor *output = network->getOutput(i);
		if (output != nullptr) {
			std::cout << "Output Dimensions: ";
			std::cout << '[';
			nvinfer1::Dims dims = output->getDimensions();
			for (int d = 0; d < dims.nbDims; d++) {
				if (d != dims.nbDims - 1)
					std::cout << dims.d[d] << ' ';
				else
					std::cout << dims.d[d];
			}
			std::cout << ']' << std::endl;
		}
	}
}

inline void readPGMFile(const std::string& fileName, uint8_t* buffer, int32_t inH, int32_t inW)
{
    std::ifstream infile(fileName, std::ifstream::binary);
    std::string magic, w, h, max;
    infile >> magic >> w >> h >> max;
    infile.seekg(1, infile.cur);
    infile.read(reinterpret_cast<char*>(buffer), inH * inW);
}

int main() {
	Logger logger;
	auto builder = std::unique_ptr<nvinfer1::IBuilder>(nvinfer1::createInferBuilder(logger));
	auto config = std::unique_ptr<nvinfer1::IBuilderConfig>(builder->createBuilderConfig());
	auto network = std::unique_ptr<nvinfer1::INetworkDefinition>(builder->createNetworkV2(1));
	auto parser = std::unique_ptr<nvonnxparser::IParser>(nvonnxparser::createParser(*network, logger));
	const char *path = "/home/haha/Desktop/tensorrt/TensorRT-10.7.0.23/data/mnist/mnist.onnx";
	auto parsed = parser->parseFromFile(path, static_cast<int>(Logger::Severity::kWARNING));
	printNetworkIODims(network);
	auto engine = std::unique_ptr<nvinfer1::ICudaEngine>(builder->buildEngineWithConfig(*network, *config));
	auto context = std::unique_ptr<nvinfer1::IExecutionContext>(engine->createExecutionContext());

	std::vector<uint8_t> file_data(28 * 28);
	readPGMFile("./0.pgm", file_data.data(), 28, 28);	
	float host_input_data[28 * 28];
	for (int i = 0; i < 28 * 28; i ++) {
		host_input_data[i] = 1.0 - float(file_data[i] / 255.0);
	}
	float host_output_data[10];

	float *device_input_data = nullptr;
	float *device_output_data = nullptr;

	cudaMalloc(&device_input_data, sizeof(host_input_data));
	cudaMalloc(&device_output_data, sizeof(host_output_data));
	cudaMemcpy(device_input_data, host_input_data, sizeof(host_input_data), cudaMemcpyHostToDevice);

	float *bindings[] = { device_input_data, device_output_data };
	bool success = context->executeV2((void **)bindings);
	
	cudaMemcpy(host_output_data, device_output_data, sizeof(host_output_data), cudaMemcpyDeviceToHost);

	float sum{0.0F};
	for (int i = 0; i < 10; i ++) {
		host_output_data[i] = exp(host_output_data[i]);
		sum += host_output_data[i];
	}
	for (int i = 0; i < 10; i ++) {
		host_output_data[i] /= sum;
		printf("Class %d, Prob: %f\n", i, host_output_data[i]);
	}
	cudaFree(device_input_data);
	cudaFree(device_output_data);
  	return 0;
}
```

