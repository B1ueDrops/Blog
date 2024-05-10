---
title: Go原理与实战记录
categories: 编程语言
---



# 原理



# 实战



## 配置环境

* 官网下载`pkg`.

* 环境变量:

```bash
export GO111MODULE=on
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GOPROXY=https://goproxy.cn
export PATH=$PATH:$GOROOT/bin
```



## 创建项目

* 创建空文件夹

```bash
mkdir goproject
cd goproject
```

* `go mod init goproject`

* 创建`main.go`, 写上`package main`和`func main() {}`

  ```bash
  mkdir src
  cd src
  touch main.go
  ```



## package

* 假设你有一个文件夹`utils`, 里面所有的`.go`文件, 第一行都是`package utils`
* 里面的`.go`文件, 各种成分互相调用, 不用写`import`.
* 需要别的包调用的, 记得名字<font color=blue>首字母大写</font>, 不需要的小写.
* 在另外一个package中, 调用其他package的成分:
  * 写上`import "Go项目的名字/package文件夹的路径"`
  * `package`文件夹的路径就是相对于Go项目根目录的路径.

## 数据类型

* `fmt.Printf()`的占位符只有一个`%v`.
* 打印数据类型的占位符是`%t`.
* 浮点数默认是`float64`类型.

## 字符串



## 哈希表

* 创建哈希表:

  ```go
  hashmap := make(map[string]int)
  ```

* 判断`key`是否存在: `ok`是一个bool值, true表示key存在

  ```go
  _, ok := hashmap[key]
  ```

  



## Snippets/Library

* 生成`[0, n)`的随机整数:

  ```go
  import "math/rand"
  
  num := rand.Intn(n)
  ```

  