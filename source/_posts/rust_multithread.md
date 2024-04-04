---
title: Rust多线程编程模型
categories: 并发编程
mathjax: true
---



## Rust的线程模型

rust中封装的线程和OS线程是1:1的, 因为rust要保证尽量小的运行时环境.

rust的线程库是`use std::thread;`



## 创建线程/等待

* Rust中用`thread::spawn`创建线程, 传入一个闭包作为参数:

  * `thread::spawn`返回线程的句柄.
* main线程结束后, 所有线程都会自动结束, 如果要让父线程等待子线程结束, 需要用`join`方法.
  * 但是父线程结束后, 子线程不会结束.
* 在线程内部如果使用了外部变量, 需要在闭包前用`move`关键字, 让线程夺取所有权.

```rust
use std::thread;

fn main() {
    
    let mut handles = Vec::new();

    // 创建5个线程
    for i in 0..5 {
        let handle = thread::spawn(move || {
            for j in 0..5 {
                println!("thread {} print {}", i, j);
            }
        });
        handles.push(handle);
    } 

    // main线程等待子线程
    for handle in handles {
        handle.join().unwrap();
    }
}
```

* 注意: 线程的创建开销较大, 非必要的任务不要使用线程.



## 通道

* Rust在标准库中提供了多发送者, 单接收者的通道`std::sync::mspc`.
* Rust中的通道具有大小:
  * 如果大小是无穷大: 那么就相当于异步通道, 发送者不必等到接收者接受信息之后再运行.
  * 如果大小是0: 相当于同步通道, 发送者必须等到接收者接受信息后, 在开始运行.
  * 如果大小是固定大小: 那么只有在通道满了之后才会阻塞发送者.
* 如果使用通道发送数据:
  * 如果数据实现了`Copy` trait, 那么不会导致所有权转移, 如果没有, 那么通道发送数据后, 原线程不能再使用原数据.

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    
    // 创建通道
    let (tx, rx) = mpsc::channel();
  
    // 两个发送者
    let tx1 = tx.clone();

    // 线程1发送者
    // 这里需要用move因为要拿走tx所有权
    let thread1 = thread::spawn( move || {
        tx.send(1).unwrap();
    });

    // 线程2发送者
    let thread2 = thread::spawn( move || {
        tx1.send(2).unwrap()
    });

    // 线程3接收者
    let thread3 = thread::spawn( move || {
        for received in rx {
            println!("Received {}", received);
        }
    });

    thread1.join().unwrap();
    thread2.join().unwrap();
    thread3.join().unwrap();
}
```

* 只有当所有发送者被drop/所有接收者都被drop之后, 通道才会被drop, 这一点有一个坑;

  ```rust
  use std::thread;
  use std::sync::mpsc;
  
  fn main() {
  
      let (tx, rx) = mpsc::channel();
  
      for i in 0..3 {
          let thread_tx = tx.clone();
          thread::spawn(move || {
              thread_tx.send(i).unwrap();
              println!("thread {} finished", i);
          });
      }
  
      for x in rx {
          println!("main Got {}", x);
      }
  
      println!("Finished");
  }
  ```

  * 以下代码中, 有4个发送者(`tx`, 还有3个`thread_tx`).
  * 其中3个`thread_tx`会在子线程结束后被释放, 但是`tx`会在main线程结束后被释放.
  * 但是main线程一直在监听通道, `tx`一直没发, 所以就会卡住, 等待`tx`被drop, 因此最后一行打印不会打出来.



## 互斥锁

* 互斥锁可以保证线程对共享数据的操作是原子操作(一个线程操作数据时, 不会被打断) (所有线程对共享数据的访问互斥).

* 在Rust中, 互斥锁在`std::sync::Mutex`, 本质上互斥锁算是一种智能指针.
* Mutex可以实现多线程的内部可变性, 相当于`RefCell`的多线程版本, mutex无需声明为`mut`.
* 由于多个线程要共享`Mutex`的所有权, 因此一般和`Arc<T>`一起使用.
* `Mutex<T>`变量如果调用`lock().unwrap()`, 会返回一个`MutexGuard<T>`智能指针
  * `MutexGuard`实现了`Deref` trait, 直接`*`一个`MutexGuard<T>`变量可以拿到其中可变的值, 这个值需要定义为`mut`.
  * `MutexGuard`还实现了`Drop` trait, 作用域结束会自动释放锁, 如果需要手动释放可以`drop(&MutexGuard)`.
* 如果获取`Mutex`失败, 那么线程会被**阻塞**, 可以使用`try_lock()`, 这样线程获取锁失败后, 会返回错误, 然后继续执行.

```rust
use std::thread;
use std::sync::{ Arc, Mutex };

fn main() {

    // 多线程共享数据
    let counter = Arc::new(Mutex::new(0));
    let mut handles = Vec::new();

    // 10个线程, 每个人给这个数据+1
    for _ in 0..10 {
        // clone互斥锁
        let mutex_counter = Arc::clone(&counter);
      
        let handle = thread::spawn(move || {
            // 上锁
            let mut num = mutex_counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }
    println!("{}", *counter.lock().unwrap());
}
```



## 条件变量

* Mutex可以保证线程互斥访问, 但是如果要保证线程访问顺序, 需要条件变量.
* 条件变量在`std::sync::Condvar`中.
* 条件变量的API有两个:
  * `wait`: 当线程拿到锁后, 发现共享数据没准备好, 可以wait`MutexGuard<T>`, 此时会把锁释放, 并阻塞线程.
  * `notify`: 当其他线程获取锁后, 进行处理完, 可以通过`notify`唤醒被阻塞的线程, 被阻塞的线程就可以获取处理好的共享数据.

```rust
use std::thread;
use std::sync::{ Arc, Mutex, Condvar };

fn main() {

    let mutex = Arc::new(Mutex::new(0));
    let cond = Arc::new(Condvar::new());

    let mutex_clone = Arc::clone(&mutex);
    let cond_clone = Arc::clone(&cond);

    let mutex_clone_clone = Arc::clone(&mutex);
    let cond_clone_clone = Arc::clone(&cond);

    let handle1 = thread::spawn( move || {
        // 尝试获取锁
        let mut data = mutex_clone.lock().unwrap();
        // 如果发现数据是0, 用条件变量等待, 直到数据是1
        // 此时会先释放锁
        if *data == 0 {
            println!("Data is 0, waiting for changer...");
            data = cond_clone.wait(data).unwrap();
        }
        *data = 0;
        println!("Data changed back to 0!");
    });

    let handle2 = thread::spawn(move || {
        let mut data = mutex_clone_clone.lock().unwrap();
        *data = 1;
        println!("Changer change 0 to 1!");
        // 唤醒条件变量
        cond_clone_clone.notify_one();
        // 释放锁
        drop(data);
        println!("Changer exit...");
    });

    handle1.join().unwrap();
    handle2.join().unwrap();
}
```