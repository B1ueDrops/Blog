---
title: rust网络编程
categories: 编程语言
mathjax: true
---

Rust网络编程主要涉及的包是:

* `std::net`: 提供网络的基本通讯功能.
  * 主要使用`TcpListener`和`TcpStream`.
* `std::io`: 提供了网络通讯的相关trait.



## 需要了解的trait

1. `std::io::Read`:
   * 里面有`fn read(&mut self, buf: &mut [u8]) -> Result<usize>;`方法.
   * 实现了这个trait, 每次通过`read()`可以将自己接收到的数据读取到一个`u8`类型的数组`buffer`中.
   * 返回值是`buf`读了多少字节.
2. `std::io::Write`:
   * 里面有: `fn write(&mut self, buf: &[u8]) -> Result<usize>;`方法.
   * 这个方法会把`buf`中的东西写入实现`Write` trait的对象中, 可以通过网络传输.
   * 返回值是写入了多少字节.



## 构建TCP Clinet/Server

* TCP Server的代码如下:

  ```rust
  // TcpStream实现了Read, Write trait需要引入
  use std::io::{ Read, Write };
  use std::net::{ TcpListener, TcpStream };
  
  fn main() {
      
      // 创建连接, 监听127.0.0.1:7878, 其中listener的类型是TcpListener
      let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
      println!("Running on port 7878...");
  
      // 持续监听, 其中stream中是Result<TcpStream, Error>, TcpStream中有监听信息
      // listener.incoming返回一个迭代器
      for stream in listener.incoming() {
          let mut stream = stream.unwrap();
          println!("Connection established");
          // 将客户端传来的消息读取到buffer
          let mut buffer: [u8; 1024] = [0; 1024];
          stream.read(&mut buffer).unwrap();
        	stream.write(&mut buffer);
      }
  }
  ```

* TCP Client的代码如下:

  ```rust
  use std::net::TcpStream;
  use std::io::{ Read, Write };
  use std::net::TcpStream;
  use std::str;
  
  fn main() {
  
      // 客户端连接
      let mut stream = TcpStream::connect("127.0.0.1:7878").unwrap();
  
      // 向服务端发送
      stream.write("Hello".as_bytes()).unwrap();
  
      // 读取服务端响应
      let mut buffer = [0; 5];
      stream.read(&mut buffer).unwrap();
  
      // 将字节流转换为utf8的字符串
      println!("Response from server: {:?}", str::from_utf8(&buffer).unwrap());
  }
  ```

  



## 解析HTTP请求

