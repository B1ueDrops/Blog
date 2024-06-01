---
title: 用rust实现一个简易线程池
categories: 编程语言
---





## 一些预备知识

* `JoinHandle<T>`类型:
  * 使用`let handle = thread::spawn(|| {})`的返回值是`JoinHandle<T>`类型, 其中`T`是线程的返回值类型.
  * 可以通过`handle.join()`获取线程的返回值, 返回值类型是`Result<T, E>`.

## 线程池的设计

* 工作线程结构体`Worker`:
  * 字段: 需要有一个`id`和一个封装的线程对象.
    * 线程对象
  * 方法: 需要有一个构造方法, 参数有`id`以及消息队列接收者的引用`rx`.
    * `rx`需要在多个工作线程之间共享, 因此类型需要是`Arc<Mutex<T>>`类型.
    * 构造方法中创建线程, 持续尝试获取`rx`的互斥锁, 获取成功后执行任务.
* 线程池对象`ThreadPool`:
  * 字段: 
    * 需要有一个`Vec`存储所有的`Workers`.
    * 需要有一个`tx`用来发送任务.
  * 方法:
    * 构造方法: 初始化线程池, 参数就一个线程池中线程个数就行.
    * 执行任务: 注意, 任务这个闭包需要实现`FnOnce() + Send + 'static`
      * `'static`是因为任务不知道什么时候结束, 生命周期不确定.
  * 实现`drop`:
    * 首先需要`drop`掉`sender`, 这样工作线程调用`rx.recv()`时就会出现`Err`, 自动退出.
    * 然后遍历工作线程, 调用`join()`, 保证工作线程获取结果, 全部退出销毁后, 线程池再销毁.



## 线程池的源代码

```rust
use std::thread;
use std::sync::{Arc, Mutex, mpsc};

// Job是一个闭包类型
type Job = Box<dyn FnOnce() + Send + 'static>;

struct Worker {
    // 工作线程的id
    id: usize,
    // 实际的OS线程
    thread: Option<thread::JoinHandle<()>>
}

impl Worker {
    // 每一个Worker有一个接收端, 接收线程池发送的Job
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // 每一个Worker持续尝试获取receiver的lock, 获取后尝试执行job
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();
            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing");
                    job();
                }
                Err(_) => {
                    // 线程池drop后
                    println!("Worker {id} disconnected, shutting down");
                    break;
                }
            }
        });
        Worker { id, thread: Some(thread) }
    }
}

pub struct ThreadPool {
    // 线程池中所有的工作线程
    workers: Vec<Worker>,
    // 工作线程从线程池中获取Job
    sender: Option<mpsc::Sender<Job>>, 
}

impl ThreadPool {

    pub fn new(size: usize) -> Self {
        assert!(size > 0);
        
        // 线程池通过channel给Worker分配任务
        let (tx, rx) = mpsc::channel();

        // 让每一个Worker都能持有rx
        let rx = Arc::new(Mutex::new(rx));

        // 创建所有的Worker线程
        let mut workers = Vec::with_capacity(size);

        // 把receiver的copy发送给Worker
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&rx)));
        }

        ThreadPool { workers, sender: Some(tx) }
    }

    // 执行时, 根据闭包创建Job, 然后通过channel发送
    pub fn execute<F>(&self, f: F) where F: FnOnce() + Send + 'static {
        let job = Box::new(f);
        // 注意unwrap会拿走所有权, 需要用as_ref转成引用
        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        // drop sender, 这样所有worker线程的channel都会报错
        drop(self.sender.take());
        // 对所有的线程调用join, 等待所有工作线程结束后再结束
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);
            // take方法可以拿走Option中值的所有权
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

