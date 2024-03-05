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
  // 这里sleep后, 会调度到niubi
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





