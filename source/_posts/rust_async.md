---
title: rust的async/await模型
categories: 并发编程
---



## 简单的例子

```rust
use std::time::Duration;
use futures::executor::block_on;
use async_std::task;

async fn hello_world() {
    println!("hello world!");
  // 这里异步sleep后, 会调度到niubi
    task::sleep(Duration::from_millis(1000)).await;
    println!("aba");
}
async fn niubi() {
    println!("niubi");
}

async fn async_main() {
  // async_main等待两个任务完成, 两个任务独立
    futures::join!(hello_world(), niubi());
}

fn main() {
    block_on(async_main())
}
```



## API



### async/await

* Rust中, 使用`async`定义的函数, 调用后会变成`Future`.
* `Future`需要一个执行器(executor)来执行.
* 当`Future`执行被阻塞后, 它会让出当前线程的控制权, 来执行其他`Future`, 不会导致整个线程被阻塞.
* 在一个`async`函数中, 调用另一个`async`函数可以对`Future`调用`await`, `await`会建立`Future`之间的依赖关系.
  * 因此, 执行`async`有两种方式:
    * 第一, 使用Executor.
    * 第二, 在`async`函数中使用`.await`.



### 多个Future之间并发

对于下面的代码:

```rust
async fn book() {
  println!("book");
}

async fn dance() {
  println!("dance");
}

async fn async_main() {
  book().await
  dance().await
}
```

此处, 由于<font color=red>await是有顺序限制的</font>, `book`和`dance`之间并不是并行.

如果想让`book`和`dance`两个函数并发, 需要用到`future::join`

```rust
use futures::join;

async fn book() {
  println!("book");
}

async fn dance() {
  println!("dance");
}

async fn async_main() {
  join!(book(), dance());
}
```

如果要执行的`Future`很多, 那么可以把`Future`放到一个数组中, 然后调用`futures::future::join_all`.

