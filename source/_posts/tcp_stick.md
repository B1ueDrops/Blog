---
title: 如何解决TCP粘包问题
categories: 计算机网络
---



## TCP粘包是什么?

* 假设TCP的客户端发送了A和B.
* 接收端调用两次`recv`, 从缓冲区中可能拿到的是:
  * 0.5A, 0.5A + B.
  * A + 0.5B, 0.5B.

这个现象就是TCP粘包问题.

## 为什么会有TCP粘包?

* TCP是字节流式传输, 连续两次发送的数据之间边界信息会失去.

> UDP为什么没有粘包?

* UDP不是流式传输, 而是以UDP数据报的形式.
* UDP的服务端是以UDP数据报为单位提取的.

## 如何解决TCP粘包?

* 假设要发送的数据大小是`S`.
* TCP客户端:
  * 首先发送数据大小, 数据大小所占的字节长度需要已知, 可以使用python的`struct`库.
  * 然后, 发送序列化的真实数据.
* TCP服务端:
  * 首先, 第一次接受的buffer需要根据前面定长的数据大小来接受.
  * 然后, 根据数据大小来拼数据.

> TCP客户端代码:

```python
import socket
import pickle
import struct


class TCPClient:

	def __init__(self, ip, port):
		self.ip = ip
		self.port = port

		self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		self.socket.connect((self.ip, self.port))

	def send_data(self, data):
		serialized_data = pickle.dumps(data)
		data_len = struct.pack('i', len(serialized_data))
		self.socket.send(data_len)
		self.socket.send(serialized_data)

	def close(self):
		self.socket.close()


class TCPServer:

	def __init__(self, ip, port):
		self.ip = ip
		self.port = port
		self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
		self.socket.bind((self.ip, self.port))

	def start(self):
		self.socket.listen(5)
		while True:
			client_socket, addr = self.socket.accept()
			data = client_socket.recv(4)
			data_len = struct.unpack('i', data)[0]
			packet = b''
			while True:
				if len(packet) == data_len:
					break
				client_data = client_socket.recv(1024)
				packet += client_data
			true_data = pickle.loads(packet)
			self.process(true_data)

	def process(self, true_data):
		pass

```

