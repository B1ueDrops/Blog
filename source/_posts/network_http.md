---
title: HTTP协议
categories: 计算机网络
---



## HTTP的报文结构

* HTTP请求报文的结构是:

  ```
  <method> <request-URL> <version>
  <request-headers>
  
  <body>
  ```

  * 注意`<headers>`和`<body>`中有一个空行.
  * `request-headers`是一系列的键值对.
* HTTP响应报文的结构是:

  ```
  <version> <status-code> <reason>
  <response-headers>
  
  <body>
  ```

  * 其中, `reason`是对`status-code`的解释.



## HTTP的header常见的字段




* 常见的HTTP请求报文中, `request-headers`:

| 字段         | 含义                                       |
| ------------ | ------------------------------------------ |
| `User-Agent` | 用户的浏览器/系统标识                      |
| `Cookie`     | 之前服务器通过Set-Cookie告诉客户端的Cookie |
|              |                                            |
|              |                                            |

* 常见的HTTP响应报文中, `request-headers`:

| 字段         | 含义                     |
| ------------ | ------------------------ |
| `Set-Cookie` | 服务端给客户端设置Cookie |
|              |                          |
|              |                          |



## HTTP的长连接机制

* HTTP的长连接机制是通过重用一个TCP连接, 来处理多个HTTP请求, 用来减少创建TCP连接的开销.

https://www.cnblogs.com/caoweixiong/p/14720254.html
