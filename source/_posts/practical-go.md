---
title: Go语言原理与实战记录
categories: 编程语言
---



# 原理



# 实战



## 数据类型

* `fmt.Printf()`的占位符只有一个`%v`.
* 打印数据类型的占位符是`%t`.
* 浮点数默认是`float64`类型.

## 字符串





## Snippets/Library

* 生成`[0, n)`的随机整数:

  ```go
  import "math/rand"
  
  num := rand.Intn(n)
  ```

  