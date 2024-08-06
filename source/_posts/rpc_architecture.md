---
title: 后端微服务的架构
categories: 软件工程
mathjax: true
---



## 引言

* 后端架构由如下几个部分组成:
  * 微服务: 可以包括HTTP服务, RPC服务, 数据库/分布式缓存服务等.
  * 消息中间件: 使用生产者-消费者模式, 将各种微服务进行互联.



## 存储服务



### MySQL的高可用架构



#### 主从模式

主从模式下:

* 有一台MySQL服务器作为Master, 处理写请求, 其余MySQL服务器作为Slave.
* Master与Slave通过主从复制保持数据一致性.
* Master宕机后, 某个Slave会变成Master.



> 异步主从复制原理:

* Master数据发生变化后, 将数据变更日志写入二进制日志文件`binlog`.
  * MySQL会启动`binlog server`进程, 如果MySQL进程挂了, 还可以通过`binlog server`获取`binlog`.

* Slave启动I/O线程, 与Master进行连接, 读取最新`binlog`, 保存在自身的`relaylog`.
* Slave启动专门的SQL线程, 从`relaylog`中获取日志, 在本地重新执行Master执行过的SQL语句, 保持Master和Slave数据一致.

> 异步主从复制的隐患:

* Master提交事务, 和写入`binlog`之间的时间间隔内, 如果宕机, 那么Slave无法获取更新.



> 半同步主从复制原理:

* Master提交事务前, 就写入`binlog`, 然后等待Slave连接读取`binlog`.
* 至少一个Slave连接成功并读取`binlog`, 并且将更新保存到`relaylog`之后, 向Master发送ACK.
* Master收到至少一个ACK之后, 再提交事务.



> 半同步主从复制的缺陷:

* 适合对数据一致性要求较高的业务场景, 例如金融交易系统.



> 主从复制模式的改进:

* 如果所有Slave都连接Master, 那么Master压力会过大.
* 因此, 只有某些Slave连接Master, 而其余的Slave通过复制后的Slave进行更新.



#### MHA-主从模式管理者

* MHA的全称是Master High Availability, 包括如下几个部分:
  * MHA Manager: 通常部署在单独的服务器上.
  * MHA Node: 部署在每台MySQL服务器上.
* MHA的工作原理如下:
  * MHA Manager周期性地探测所有MySQL集群中Master的心跳, 如果4次检测不到心跳:
    * 有可能是MHA Manager与Master之间的网络问题, 此时MHA Manager会通知Slave (MHA Node)通过SSH对Master进行连接.
    * 如果再连接不上, 则认为Master宕机.
  * 如果Master宕机, 则MHA Manager会通过SSH连接到`binlog server`, 获取Master的`binlog`文件.
  * 之后, MHA Manager识别哪个Slave的`relaylog`更新, 最新的哪个就是新的Master.
  * Slave之间通过增量的方式同步`relaylog`.
  * MHA Manager通过增量的方式同步`binlog`和Slave之间的`relaylog`.
* MHA和半同步复制模式一起使用, 可以大幅度降低数据丢失的风险.



#### MMM-双主模式管理者

#### MGR



### Redis的高可用架构

#### 主从模式



#### 哨兵模式



#### 集群模式



#### 中心化集群模式

