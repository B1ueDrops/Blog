---
title: 存储微服务的架构
categories: 后端技术
mathjax: true
---



## 引言

* 后端架构由如下几个部分组成:
  * 微服务: 可以包括HTTP服务, RPC服务, 数据库/分布式缓存服务等.
  * 消息中间件: 使用生产者-消费者模式, 将各种微服务进行互联.







## MySQL的高可用架构



### 主从模式

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



### MHA-主从模式管理者

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





### MGR

* MGR的全称是MySQL Group Replication, 它有如下几个特点:
  * 一致性强: 基于分布式共识算法Paxos, 可以保证多个节点数据一致性.
    * 代价是: 性能不如异步主从/半同步主从.
  * 容错性强: `2N + 1`个节点的Group中, 可以容忍`N`个节点宕机.
  * 灵活性强: 同时支持单主/多主.
* MGR的适用场景:
  * 要求数据强一致性.
  * 写操作的请求量不是很大.
* 单主MGR的工作原理:

## Redis的高可用架构

### 主从模式

* Redis主从模式的工作原理:
  * Slave与Master首次建立连接后, 向Master发送`PSYNC`命令准备复制数据.
  * Master收到`PSYNC`命令, 执行`BGSAVE`, 生成全部数据的Snapshot `RDB`文件, 并创建buffer, 用来记录之后Master执行的数据变更命令.
  * Master向所有Slave发送`RDB`文件.
  * Slave收到`RDB`文件后, 先保存到磁盘, 然后从磁盘加载数据到内存, 然后准备接收Master的数据变更命令.
  * Master一进行数据变更, 就会向Slave发送数据变更命令.

### 哨兵模式

* 主从模式中, 如果Master宕机了, 那么需要手动选一个Slave作为Master.

* 哨兵(Sentinel) 模式用来自动选择一个Slave作为Master.
* 哨兵模式的工作原理:
  * 一个Redis Server除了作为Master/Slave, 还可以作为Sentinel.
    * Sentinel负责在Master/Slave宕机之后, 自动选一个Slave重新作为Master.
  * 在主从模式Redis架构中, 部署若干Sentinel节点组成Sentinel集群, Sentinel集群会与Master/Slave维持心跳.
  * 超过`N`个Sentinel节点如果认为Master宕机, 那么Sentinel集群会选一个Slave作为新的Master, 并且与这个新的Master建立连接.
* 在哨兵模式下, 访问Redis首先需要访问Sentinel集群, 从而获取Redis Master的地址.



### 集群模式

* 在哨兵模式下,  只有一个Master对外提供服务, 如果有大量数据存储, 或者海量请求, 那么单个Master难以处理.
  * 也就是没有解决在线扩容的问题.
* 集群模式可以解决:
  * 数据分布式存储到各个Redis节点.
  * 可以接受海量请求访问.
* Redis集群模式工作原理:
  * Redis整体分成若干Redis集群(Redis Cluster), 一个Redis集群有Master, Slave.
    * 每个Redis集群中保存所有数据的一部分, 数据分片算法采用哈希槽算法.
      * 当向Redis中写入数据时, 会进行CRC16校验, 然后和16384进行取模决定放到哪个槽.
    * 每个Redis集群采用主从模式保证高可用, Master宕机后会自动选取一个Slave.
    * 在Redis集群中的每个服务器都有一个哈希槽和Master的映射关系, 这个通过Gossip协议实现.
  * 客户端访问流程是:
    * 客户端访问Redis集群中的任意一个服务器, 获取哈希槽和Master映射关系.
    * 客户端读写数据, 首先基于数据算出哈希槽, 然后获取Master, 然后访问Master.



#### Gossip协议

* Redis集群中, 通过Gossip协议让每个节点都知道哈希槽与Master IP地址的映射关系.
* Gossip协议的内容:
  * 核心: 在节点数量有限的网络中, 每个节点随机和部分节点通信, 经过多次迭代, 多个节点信息会在一定时间内达成一致.

* Gossip协议的缺点:
  * 如果Redis集群节点过多, 那么Gossip协议会严重占用带宽.



### 中心化集群模式

* 中心化集群模式: 为了解决Gossip协议严重占用带宽, 采用中心化的服务器同步哈希槽与Master的存储关系.



## NoSQL数据库



### LSM Tree

* LSM Tree (Log-Structured Merge Tree) 是一种支持高并发写, 查询效率较高的数据结构, 是No SQL数据库的核心, 可以实现高效的键值存储系统.
* LSM Tree的核心:
  * 对于内存/磁盘等存储介质, 顺序读写的性能远高于随机读写.
    * 顺序读写: 按照文件中数据的顺序有序地读写, 例如追加写就是顺序写.
    * 随机读写: 不遵循文件中数据的顺序进行读写.
  * LSM Tree将数据的随机读写转化成数据的顺序读写, 大大提高性能.



### 文档数据库

* 文档数据库可以理解为一个键值存储系统:
  * 键: 字符串.
  * 值: JSON对象, 这个对象可以扩展字段.
* 文档数据库的适用场景:
  * 数据字段定义不明确, 且会不断变化, 例如不同商品的参数.
* 不适用场景:
  * 需要支持事务的强一致性系统.
  * 需要支持复杂查询, 例如join等.



### 列式数据库

* 传统的关系型数据库以行为单位进行存储:
  * 如果需要进行Groupby之类的数据分析, 需要读取所有行数据, 磁盘I/O过多.
* 列式数据库按照列为单位进行存储:
  * 一次可以高效地读取一列的数据.
* 列式数据库可以大幅提高空间利用率:
  * 如果列对应的数据类型是有限枚举(性别等), 可以将原数据映射到枚举值(占用空间小), 然后只存储枚举值.
  * 数据量越大, 空间利用率越高.
* 列式数据库的场景:
  * 数据分析场景: 需要针对某列进行数据分析.
  * 海量数据插入, 但是数据修改极少的场景(诸多的日志监控系统).
* 列式数据库不适用的场景:
  * 数据需要高频写.
  * 需要支持事务的强一致性系统.



### 全文搜索数据库

* 关系型数据库如果需要全文搜索需要使用`like`进行模糊查询, 效率低下, 需要使用`ElasticSearch`等全文搜索数据库.
* 原理: 倒排索引(Inverted Index)
  * 存储每一个关键词在文档中的位置.
* 全文搜索数据库的场景:
  * 关键词检索.
  * 海量数据复杂查询.
  * 数据统计.
* 不适用的场景:
  * 数据需要高频写.
  * 需要支持事务的强一致性系统.



### NewSQL

* 各种数据库的优势/劣势:
  * 关系型数据库: 可以承担高频写, 并且是强一致性系统.
  * NoSQL: 可以针对特定场景提供高性能的查询.
* NewSQL数据库是在NoSQL的基础上, 提供关系型数据库的事务能力.
