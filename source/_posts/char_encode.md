---
title: 字符编码深度理解
categories: 编程语言
---



## 字符集与字符编码

* **字符集(Character Set):  **可以理解成一个表格, 用来将一个字符映射到二进制bits.
  * ASCII字符集: 用来将扩展的拉丁字母映射到二进制Bits.
  * Unicode字符集: 可以被映射的范围更广, 映射得到的二进制Bits叫做(Code points).
  * GBK字符集: 中文字符集.
* **字符编码(Character Encoding): **是一种编码算法, 这种算法会将Code points映射到计算机实际存储的字节序列.
  * ASCII字符集有ASCII字符编码算法.
  * Unicode字符集有UTF-8等字符编码算法.
  * GBK字符集有GBK字符编码算法.
  * 字符编码算法分为变长/不变长, 如果每个字符经过编码后生成的字节序列长度一样, 那就是不变长, 否则就是变长.
