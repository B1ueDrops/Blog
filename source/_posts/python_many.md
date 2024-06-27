---
title: Python的一些基础
categories: 编程语言
---



## Python基础

* 查看dict的keys: `print(list(a.keys())`
* 对某个对象`obj`做深拷贝:
  *  `copyed_obj = obj.copy()`, 对`obj`


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

  

