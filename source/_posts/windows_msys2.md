---
title: Windows环境配置
categories: 开发工具
mathjax: true
---



## `Unix`兼容层

* Unix兼容层是让使用Unix API的软件运行在Windows系统上的一种解决方案, 主要包括:
  * 特殊的编译器: 将Unix软件编译成特定的二进制文件, 二进制文件中的Symbol会使用兼容层的链接库.
  * 特殊的标准库: 编译后二进制文件运行时, 需要动态链接到兼容层的标准库.
  
* 常见的Unix兼容层:
  
  * **Cygwin**: 
  
    * 使用移植到Windows上的GNU GCC编译.
    * 运行需要依赖`cygwin1.dll`, 链接库中定义了Unix API, 并使用Windows API模拟了行为, 速度较慢.
  
  * **MSYS2**: Mininal GNU System on Windows
  
    * 是Cygwin的一个fork, 只实现了基本的Unix API, 体积较小.
    * 运行同样需要依赖`msys-2.0.dll`.
  
  * **MinGW**: Minimal GNU for Windows
  
    * 使用移植到Windows上的GNU GCC进行编译.
    * 在编译时会将Unix API转换成Windows API, 因此不依赖兼容层的`.dll`, 速度更快.
    * MinGW只能生成Windows32的程序, MinGW-W64可以生成Windows64的程序.
    
    
  
    

## `msys2`配置

* 下载`msys2`后, 会有四种环境可以选择:
  * `MSYS`
  * `MINGW/MINGW64`
  * `UCRT64`
