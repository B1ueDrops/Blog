---
title: 多线程编程模型
categories: 并发编程
---



## 消息队列API模型

消息队列模型就是生产者-消费者模型, 分类标准如下:

* 生产者可以多个, 消费者也可以多个.
* 消息队列可以是异步/同步.
  * 同步: 消息队列满了之后, 生产者生产时会被阻塞.
  * 异步: 消息队列的大小没有用, 生产者不会被阻塞.

**例1: 1个生产者和1个消费者通过消息队列发送消息:**

```rust
use std::sync::mpsc;

struct Message {
    name: String,
    age: i32
}
fn main() {
    // async channel, 主线程创建消息队列
    let (tx, rx) = mpsc::channel();

    let sender = std::thread::spawn( move || {
        let message = Message {
            name: String::from("haha"),
            age: 18
        };
        tx.send(message).unwrap();
        println!("Sender has sent data");
    });

    let receiver = std::thread::spawn( move || {
        let message = rx.recv().unwrap();
        println!("Receiver receives {} and {}", message.name, message.age);
    });
		// 等待主线程运行完
    sender.join().unwrap();
    receiver.join().unwrap();
}
```

> move关键字的作用

* 当你在子线程中, 尝试访问外部线程的变量时, 会有隐患.

  * 外部线程可能比你子线程先结束, 然后变量所有权失效, 此时子线程访问就会导致不安全.
  * 子线程和外部线程共享一个变量的所有权, 很危险.

* 为了解决这种情况:
  * 使用`join`, 先让主线程等等, 让原变量生命周期别消失.
  * 用`move`, 表示如果线程内使用了这个变量, 那么变量所有权自动转移到线程内部.
* **用消息队列时就需要用`move`.**



**例2: 多个生产者, 1个消费者, 有一定大小的同步消息队列:**

只需要把`tx`进行克隆即可(`tx.clone()`), 然后不同生产者线程使用不同的`tx`进行发送.

  

## 互斥API模型



### 互斥锁

互斥锁可以让不同线程访问同一个值时, 排队访问.

在Rust中, `Mutex`
