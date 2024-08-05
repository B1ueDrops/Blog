---
title: HTTP协议的一些细节
categories: 计算机网络
---

# HTTP协议



## HTTP的报文结构

* HTTP请求报文的结构是:

  ```
  <method> <request-URL> <version>\r\n
  <request-headers>\r\n
  
  <body>
  ```

  * 注意`<headers>`和`<body>`中有一个空行.
  * `request-headers`是一系列的键值对.
* HTTP响应报文的结构是:

  ```
  <version> <status-code> <reason>\r\n
  <response-headers>\r\n
  
  <body>
  ```

  * 其中, `reason`是对`status-code`的解释.



## HTTP的常见header



### HTTP请求报文




* 常见的HTTP请求报文中, `request-headers`:

| 字段                | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| `User-Agent`        | 用户的浏览器/系统标识                                        |
| `Cookie`            | 之前服务器通过Set-Cookie告诉客户端的Cookie                   |
| `Accept`            | 自己能够接受的, 服务端传来的文件类型, 例如`Accept: text/plain` |
| `If-Modified-Since` | 这个是个日期, 如果请求的资源在这个日期后被修改, 就返回200 OK+资源, 否则返回304 Not Modified, 并且数据可以从缓存读取,  只可以用在`GET/HEAD`请求. |
| `Host`              | HTTP请求的域名/IP地址:端口号                                 |
|                     |                                                              |

### HTTP响应报文

* 常见的HTTP响应报文中, `response-headers`:

| 字段            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| `Set-Cookie`    | 服务端给客户端设置Cookie                                     |
| `Content-Type`  | 请求的多媒体文件的类型                                       |
| `Content-Md6`   | 多媒体文件的二进制用base64编码的结果                         |
| `Last-Modified` | 如果请求的文件在`If-Modified-Since`后没被修改, 这个头部就有`Last-Modified`, 标识资源上次被修改的时间. |
|                 |                                                              |





## HTTP的长连接机制

* HTTP的长连接机制是通过重用一个TCP连接, 来处理多个HTTP请求, 用来减少创建TCP连接的开销.

https://www.cnblogs.com/caoweixiong/p/14720254.html



## HTTP协议常见的status code

| status code | 类别             |
| ----------- | ---------------- |
| 2XX         | 成功状态码       |
| 3XX         | 重定向状态码     |
| 4XX         | 客户端错误状态码 |
| 5XX         | 服务器错误状态码 |

> 2系列的状态码?

* 200 OK: 请求被成功处理.
* 201 Create: 比如你用POST请求, 在服务端创建了一个新资源.

> 3系列的状态码?

* 301 Moved Permanently: 比如你的网址已经更换了.

> 4系列的状态码?

* 400 Bad Request: 你的HTTP请求不合法, 例如参数存在错误.

> 5系列的状态码?

* 500 Internal Server Error: 服务端在处理的时候忽然抛出了异常.: 

# 面试题

> 网页浏览的全过程?

* 浏览器中输入制定网页的URL.
* 浏览器通过DNS协议, 获取域名对应的IP地址.
* 浏览器通过你使用的协议, 确定端口号, 例如http协议默认使用90, https协议默认使用443.
* 浏览器根据IP地址和端口号, 向服务器发送一个HTTP请求报文, 获取网页内容.
* 服务器收到请求后, 处理请求, 并且返回一个HTTP响应报文给浏览器.
* 浏览器收到响应报文后, 解析HTML, 渲染网页, 同时对网页中包含的其他静态资源(图片, CSS, JS)等, 再次发起HTTP请求, 获取这些资源, 直到网页完全加载完成.
* 浏览器不需要与服务器通信时, 可以主动关闭TCP连接, 或者等待服务器关闭请求.



> HTTP/1.0 和HTTP /1.1的区别

* **连接方式不同: **HTTP/1.0是短连接, HTTP/1.1是长连接.
* **状态响应码不同: **HTTP/1.1加入了更多的响应码.
* **缓存处理方式不同: **
  * HTTP/1.0主要使用`If-Modified-Since`.
  * HTTP/1.1引入了更多缓存策略.
* **带宽处理: **
  * HTTP/1.0不支持断点续传和部分对象返回, 但是HTTP/1.1支持.



> GET请求和POST请求的区别

* 语义不同:
  * GET请求负责获取资源, POST请求负责修改资源.
* 参数位置不同:
  * GET请求参数位置一般在URL中, 受浏览器限制, POST请求一般在`body`中.
* 幂等性:
  * GET请求是幂等的, 多次GET请求结果不变.
  * POST请求不是.
* 缓存:
  * 由于GET请求幂等, 因此资源可以被缓存.
