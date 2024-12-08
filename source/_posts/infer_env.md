---
title: 深度学习推理环境配置
categories: 开发工具
---



## 推理环境的组成部分

* GPU驱动:
  * 本质上就是一个内核模块`nvidia.ko`.
  * 可以用`modinfo nvidia`查看.

* CUDA:
  * GPU用户端的工具集合, 包括二进制文件, 链接库等.
  * 一般在`/usr/local/cuda-<version>`里面.
* cuDNN:
  * 用CUDA写成的库, 用于针对神经网络提供一些训练/推理的API.
* TensorRT:
  * 一种高速推理引擎, 输入是`onnx`模型, 输出这个模型的推理运行时.



## 配置环境



### GPU驱动

* 首先查看架构, 操作系统和发行版:

  ```bash
  uname -m && cat /etc/*release
  ```

* 然后查看对应的NVIDIA显卡是什么:

  ```bash
  nvidia-smi --query-gpu=name --format=csv,noheader
  ```

* 最后, 去找到合适的GPU驱动下载, 一般提供一个`run`文件(shell脚本), 需要用`sudo`权限执行:

  * 驱动下载: https://www.nvidia.com/en-us/drivers/

* 之后, 验证一下GPU驱动版本:

  ```bash
  modinfo nvidia | grep version
  ```



### CUDA

* 使用`nvidia-smi`查看最高支持的CUDA版本.

* 然后在这里下载CUDA Toolkit:

  * https://developer.nvidia.com/cuda-toolkit-archive
  * 采用下载`run`文件的方式.

* 然后, 在机器上, 新建一个文件夹, 然后在文件夹中安装:

  ```bash
  sh cuda_xxx.run
  ```

* 下载完成后, 针对每个项目, 需要配置三个不同的环境变量:

  * `PATH`需要包括下载的cuda包的`bin`.
  * `LIBRARY_PATH`需要包括cuda包中的`lib64`.
  * `LD_LIBRARY_PATH`需要包括cuda包中的`lib64`.



### cuDNN

* 首先安装:

  ```bash
  sudo apt-get install zlib1g
  ```

* 首先查看与cuDNN兼容的CUDA和Driver的版本: https://docs.nvidia.com/deeplearning/cudnn/latest/reference/support-matrix.html
* 如果有`sudo`权限, 可以下载`deb`: https://developer.nvidia.com/cudnn-downloads
* 如果没有`sudo`权限, 可以下载压缩包: https://developer.download.nvidia.com/compute/cudnn/redist/cudnn/

* 下载完成之后, 将`lib`和`include`目录下所有文件移动到cuda目录的`lib`和`include`:

  ```bash
  cp cudnn/include/cudnn*.h /usr/local/cuda/include/
  cp -P cudnn/lib64/libcudnn* /usr/local/cuda/lib64
  chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
  ```

  

### TensorRT

* 首先需要明确你的架构, 操作系统和CUDA版本.

* 然后从这里下载: https://developer.nvidia.com/tensorrt/download/

  * 推荐下载`tar`文件.

* 之后按照指引配置并验证环境: https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html#installing-tar

  



## 工作区环境

