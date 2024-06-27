---
title: 使用matplotlib做可视化
categories: 编程语言
---



## 导入模块

```python
from matplotlib import pyplot as plt
```



## 创建`figure`对象

figure对象可以理解成一个画布(canvas).

* `fig = plt.figure()`:
  * `figsize`: 画布大小, 一般是一个tuple, `(宽度, 高度)`.
  * `dpi`: 分辨率, 默认80, 表示一个英寸中有几个像素.



## 创建axes对象

axes对象是一个绘图区域:

* `ax = fig.add_axes([x, y, width, height])`:
  * `x, y, width, height`是这个绘图区域在画布上的位置和大小, 是一个`[0, 1]`上的浮点数, 表示画布的比例大小.



## axes对象配置

* 设置title: `ax.set_title('title')`
* 设置x label和y label:
  * `ax.set_xlabel('xlabel')`
  * `ax.set_ylabel('ylabel')`

* `ax.plot(x, y, str)`:

  * `x`: 横轴数据.
  * `y`: 纵轴数据.
  * `str`: 折线样式, 文法是`颜色标记字符`
    * 例如`ys-` : 表示黄色, 正方形标记, 线的类型是线段连接.

* 如果要在一个`ax`上画很多线, 那么就要调用多次`ax.plot`.




## 等分画布

* 通过`fig, axes = plt.subplots(nrows, ncols)`, 可以等分画布, 得到一个`nrows x ncols`的矩阵.
  * 每个矩阵都有一个`ax`对象, 例如 `ax[0][0]`, 然后对每一个对象都进行配置就可以在画布上画很多图.