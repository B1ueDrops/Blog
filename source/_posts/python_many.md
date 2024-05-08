---
title: Python用到什么学什么
categories: 编程语言
---



## Python基础

* 查看dict的keys: `print(a.keys())`.
* 对某个对象`obj`做浅拷贝: `copyed_obj = obj.copy()`.

## Python面向对象

* 定义接口的方法:

  ```python
  import abc
  
  # 抽象类
  class Stimulus(metaclass=abc.ABCMeta):
      @abc.abstractmethod
      # 抽象方法
      def start(self):
          pass
  ```

  

## Pandas

* `axis`的参数含义: 如果是0, 表示计算每一列的属性, 如果是1, 表示计算每一行的属性.
* `df.rolling()`方法可以用于滑动窗口计算操作, 通常和`mean(), sum(), std_dev()`等一起使用.
  * `df.ewm()`可以执行指数加权平均(Exponential Weighted Moving Average).

## Snippets/Library

* 读取matlab的`.mat`文件, 选用`mat73`库:

  ```bash
  pip3 install mat73
  ```

  ```python
  # data是字典类型
  data = mat73.loadmat('mat文件路径')
  ```
