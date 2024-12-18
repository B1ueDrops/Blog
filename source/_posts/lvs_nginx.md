---
title: LVS/Nginx-服务器调度的技术
categories: 后端技术
mathjax: true
---



## 引言

客户端的请求连接到机房后, 机房的入口设备需要负责把请求转发到某一台具体的服务器中.

如果没有转发, 那么就是最简单的架构:

* DNS直接解析出来服务器的公网IP地址.
* 客户端直接访问IP地址.

这种架构有如下问题:

* 可用性问题: 如果服务器垮了, 那么DNS服务器不能快速感知到IP地址不可用.
* 安全性问题: 所有服务器如果都有公网IP, 那么存在安全隐患.
* 扩展性问题: 如果一个服务需要扩容(增加服务器数量), 那么就需要额外配置DNS服务器.
  * 但是DNS缓存有TTL值, 需要等到缓存失效, 额外配置的服务器IP地址才能生效, 新的服务器不能马上派上用场.

因此, 为了解决这些问题, 需要加一层调度层:

* DNS需要将域名映射到调度层机器的公网IP.
* 调度层负责将请求进行转发, 需要有很高的性能.



## Nginx基础



### 代理模式

代理模式分为两种:

* 正向代理(forward proxy): 代理客户端, 客户端负责把请求发给正向代理, 让代理办事, 例如VPN代理.
* 反向代理(reverse proxy): 代理服务器: 反向代理负责接收客户端请求, 然后转发到内部网络的服务器.



### Nginx的介绍

* Nginx可以作为:
  * HTTP服务器.
  * 各种服务器的反向代理服务器, 并且内置负载均衡器.



### Nginx配置文件

Nginx的配置文件是`nginx.conf`.

例子:



#### `http`配置块

```ini
http {
	
	upstream api_feed {
		server 10.1.0.1:8080;
		server 10.1.0.2:8080;
	}
	
	upstream api_like {
		server 10.1.0.1:8081;
		server 10.1.0.1:8081;
	}
	
	server {
		listen 80;
		server_name: www.haha.com
		
		location /feed {
			proxy_pass http://api_feed;
			proxy_pass http://api_like;
		}
		
	}
}
```



`http`配置块包括两个重要的子配置: `server`和`upstream`:

* `server`: Nginx作为HTTP服务器启动时需要的各种参数, 包括:
  * `listen`: 监听本地哪个端口.
  * `server_name`:
    * 首先, 一个机器上可能会部署多个HTTP服务器, 也就会有多个`server`配置块.
    * 此时, HTTP请求来了之后, 会将header中的`HOST`字段和`server_name`对比, 然后才知道请求被哪个`server`配置处理.
  * `location`: 根据API, 看这个HTTP请求要被哪个`upstream`块处理.
* `upstream`: Nginx作为反向代理服务器启动时需要的各种参数.
  * 其中包含多个服务池, 每个服务池中是服务器实例的内网IP.
  * 如果没有额外配置, Nginx会采用Round-Robin策略, 选择一个服务器实例进行转发.



### Nginx负载均衡

* Round-Robin: 轮询.

  * 可以加权, 每次优先选择权值最高的转发.
* IP Hash:
  * 根据每个客户端IP地址的哈希结果进行转发.
  * 同一个IP地址总是会分配到同一个机器, 不保证负载均衡.
  
* URL Hash:
  * 根据客户端URL哈希结果进行转发.
  
* least_conn:
  * 将请求转发给连接数最少的服务器.
  
  
  

### Nginx服务发现

* 如果一个服务需要扩容 (增加几台后端服务器), 那么就需要频繁修改`nginx.conf`, 不太现实.
* Nginx为了实现热更新, 有如下两个模块:
  * `ngx_lua`: 内置Lua解释器, 可以编写Lua脚本部署到Nginx上执行.
  * `ngx_http_dyups_module`: 热更新upstream配置.
  * 这样, 从服务发现中心获取最新`upstream`配置后, 就可以热更新到Nginx工作进程.





## LVS实现Nginx集群管理

* 由于单台Nginx服务器能承载的用户请求有限, 因此, 当用户数量增多时, 就需要Nginx集群才能顶住压力, 此时还需要一个中间层, 将请求转发到Nginx服务器, 这个中间层就是Linux Virtual Server (LVS).
  * 也就是说, LVS可以将对公网的请求转发到自身内网的服务器.
  * LVS内置在Linux内核中, 性能更高.
* LVS工作在网络层.



### LVS的转发模式



#### NAT模式

NAT模式的工作流程如下:

* 客户端发送请求到LVS服务器.
* LVS服务器选择一个内网机器, 然后修改客户端请求的目的IP, 变成服务器内网IP, 然后转发.
* RS处理之后, 返回响应, 其中packet的源IP是自己的内网IP, 目的IP是客户端的IP.
* 到达LVS服务器后, LVS服务器作为网关, 会将packet的源IP设置为自己的公网IP, 然后给客户端.

NAT模式要求:

* LVS服务器有两个网卡, 一个提供公网IP, 一个提供内网IP.
* LVS服务器需要和内网服务器在一个局域网, 并且是网关, 否则内网服务器的响应不会经过LVS.

NAT模式的缺点: LVS服务器需要同时修改请求和响应, 网络带宽压力较大.



#### FULLNAT模式

* NAT模式的优化: LVS服务器收到客户端请求时:
  * 请求源IP地址改为LVS服务器的内网IP.
  * 请求目的IP地址改为内网服务器的内网IP.
  * LVS服务器需要存储客户端IP地址, 内网服务器返回响应时需要修改目的IP为客户端IP地址.
* FULLNAT的缺点: 内网服务器无法获取客户端的IP.



#### TUN模式

TUN模式采用了IP隧道实现.

* 当请求到达LVS服务器时, 会将报文作为新的IP数据包的数据部分, 其中:
  * 源IP是LVS服务器的内网IP.
  * 目的IP是后端服务器的内网IP.
* 到达内网服务器后, 内网服务器解析, 发现数据部分中, 目的IP是LVS服务器的公网IP, 然后开始解析.
  * 需要将LVS服务器的公网IP配置到内网服务器的lo网卡.
* RS返回IP数据包, 其中:
  * 源IP是LVS的公网IP.
  * 目的IP是客户端的IP.

* TUN模式的优点:
  * 性能好: LVS服务器只负责转发, 后端服务器的返回结果不经过LVS服务器.
* TUN模式的缺点:
  * 后端服务器需要支持IP隧道协议.
  * 后端服务器lo网卡需要配置LVS服务器的公网IP.



#### DR模式

DR (Direct Routing) 模式的工作过程如下:

* 请求到达LVS服务器.
* LVS服务器选择一个后端服务器:
  * LVS服务器和后端服务器在同一个局域网.
  * 源MAC地址改为LVS服务器内网网卡的MAC地址.
  * 目的MAC地址改成后端服务器的内网网卡的MAC地址.
  * 然后LVS服务器将请求转发给后端服务器.
* 后端服务器lo网卡上配置LVS服务器公网IP, 发现公网IP匹配后处理, 然后封装请求:
  * 源IP地址: LVS服务器公网IP.
  * 目的IP地址: 客户端公网IP.
  * 源MAC地址: 后端服务器内网网卡MAC地址.
  * 目的MAC地址: 客户端的MAC地址.

* DR模式的优势:
  * 比TUN模式性能高, 因为不涉及加解密IP隧道.
* DR模式的缺点:
  * LVS服务器要和后端服务器在同一个局域网.
  * 后端服务器lo网卡需要配置LVS服务器的公网IP.

### LVS的高可用问题

* LVS服务器一般是单点, 为了保证LVS服务器高可用, 需要实现双机热备.

## LVS+Nginx服务器调度层

* 服务器调度层的架构如下:
  * 配置多台高可用(实现双机热备)的LVS服务器的公网IP, 并且配置DNS, 让这些公网IP指向同一个域名.
    * 客户端通过DNS轮询决定访问哪个LVS.
  * LVS使用FULLNAT模式, 将请求转发到某台Nginx服务器上.
  * Nginx服务器再根据URL, 以及负载均衡策略, 将请求转发到后端一台真实的服务器.
