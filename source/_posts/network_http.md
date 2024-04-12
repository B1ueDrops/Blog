---
title: 应用层-HTTP协议
categories: 计算机网络
---



## HTTP请求

* HTTP请求报文的结构是:

  ```
  <method> <request-URL> <version>
  <request-headers>
  
  <body>
  ```

  * 注意`<headers>`和`<body>`中有一个空行.
  * `request-headers`是一系列的键值对.

* 常见的HTTP请求报文中, `request-headers`:

| 字段         | 含义                                       |
| ------------ | ------------------------------------------ |
| `User-Agent` | 用户的浏览器/系统标识                      |
| `Cookie`     | 之前服务器通过Set-Cookie告诉客户端的Cookie |
|              |                                            |
|              |                                            |



## HTTP响应

* HTTP响应报文的结构是:

  ```
  <version> <status-code> <reason>
  <response-headers>
  
  <body>
  ```

  * 其中, `reason`是对`status-code`的解释.

* 常见的HTTP响应报文中, `request-headers`:

| 字段         | 含义                     |
| ------------ | ------------------------ |
| `Set-Cookie` | 服务端给客户端设置Cookie |
|              |                          |
|              |                          |

