---
title: 消息中间件技术
categories: 软件工程
---



## 消息队列模型

* 消息队列模型是一种生产者和消费者的异步通信模型:
  * 生产者只需要向消息队列中塞信息, 就可以去干其他事情了.
  * 消费者如果有了消费能力, 就会从消息队列中拿走信息处理.
* 消息队列的好处:
  * **异步化:** 假设一个用户请求的微服务调用链是`A->B->C->D`.
    * 如果采用同步的方式, 那么用户的响应时间是`T(A) + T(B) + T(C) + T(D)`.
    * 如果采用消息队列, 那么`A`完成后就可以响应用户, 然后写入消息队列就可以了.
  * **流量削峰:** 如果请求速率过高, 可以经过消息队列进行平滑, 然后消费者根据自己能力决定是否处理请求.
  * **解耦合:**
    * 假设没有消息队列, 一个微服务`A`要调用微服务`B`:
      * 需要考虑微服务`B`的各种情况, 因此`A`, `B`耦合在一起.
    * 如果有了消息队列, 那么`A`只需要将消息写到`B`中, `B`怎么样完全不需要理会.



## Kafka的原理

* Kafka是一个高性能, 高可用, 高扩展性的分布式消息队列系统.
* Kafka的组成部分:

  * Producer.
  * Consumer.
  * Topic: 消息类型.
    * Producer可以指定自己能生产的Topic.
    * Consumer可以订阅自己能处理的Topic.
  * Partition: Kafka分布式存储数据的基本单位.
    * 每个Partition存储的消息都是基于Key有序的.
    * 不同Partition之间的消息不保证有序.
    * 一个Topic的消息可以分布式地存储在多个Partition上.
  * Broker:
    * 是部署在真实服务器上的服务.
    * 一个Broker上会有多个Partition.
    * 负责接收Producer的消息, 并存储在Partition上.
  * Consumer Group:
    * 消费同一个Topic类型消息的所有Consumer组成一个Consumer Group.
    * 规定: 在同一时刻, 对于一个Topic的一个Partition, 只允许一个Consumer进行消费.
  * ZooKeeper:
    * 维护Topic和Partition之间的映射关系.
    * 维护Partition和Broker之间的映射关系.
    * 维护Consumer Group和Topic的映射关系.
* Kafka的工作原理:

  * Producer产生某个Topic的消息, 现在要决定存储在哪个Partition上:

    * 可以指定Partition.
    * 可以指定Key, 通过哈希映射到Partition.
    * 都没有指定, 则轮询选择.
  * 决定之后, 通过ZooKeeper, Producer可以获取Partition对应Broker的IP地址.
  * Broker接收Topic, 将消息写入本地磁盘.
  * Consumer根据自身能力, 通过拉取(pull)的方式消费Partition中的消息.
