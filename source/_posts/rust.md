---
title: rust用到哪里学哪里
categories: 编程语言
---



## rust基础



### rust的闭包

格式:

```
|参数1, 参数2, ...| -> 返回值类型 {
    // 函数体
}
```

例子:

```rust
fn main() {
  let func = |num: i32| -> i32 {
    num + 1
  };
  println!("{}", func(2));
}
```

没有参数/返回值的闭包可以当作线程函数, 写法是:

```rust
|| {
  // 线程逻辑
}
```



### rust所有权

常见的有以下场景:

1. 非基本类型赋值:

   ```rust
   let s1 = String::from("hello");
   let s2 = s1;
   // 此时s1所有权给了s2, s1不能被使用
   // 如果真要共享, 使用引用 let s2 = &s1;
   // 如果只需要s1的值, 可以用let s2 = s1.clone()
   ```

2. 非基本类型作为函数参数:

   ```rust
   let s1 = String::from("hello");
   function(s1);
   // 此处, s1会失效, 不能再次使用.
   ```



### rust分模块编程

* rust中, `package`就相当于`project`的概念.

* 一个`package`中, 可以包含多个`crate`, 一个`crate`是一个最小的编译单元.

  * `crate`分为binary crate和library crate, 两者分别生成可执行文件/库文件.

  * binary crate可以用`cargo new --bin`生成, library crate可以用`cargo new --lib`生成.

  * binary crate默认有`src/main.rs`, library crate默认有`src/lib.rs`.

* 对于rust的编译器来说, 它默认只会看到`src/main.rs`或者`src/lib.rs`, 这个东西叫做`crate root`.

* rust中, 一个`crate`由多个`module`组成, 多个`module`会组成一个树状关系, 一般在`crate root`中需要显式声明`mod`:

```rust
// src/main.rs
mod A;
mod B;
mod C;

use A::funcA;
```

* 分模块编程的思路:
  * 首先, 一个rust文件相当于一个module, 这个module的名字就和文件名一样.
    * 在其中的一些元素如果要供外部使用, 需要加上`pub`.
  * 然后, 一个文件夹也相当于一个module, 这个文件夹内部要有一个`mod.rs`文件, 通过`pub mod ...`, 来让其中的子模块全部导入.
  * 使用时, 可以通过`use`关键字一层层导入, 如果需要在一个子模块中, 引用外部子模块, 可以从`use crate::`开始导入.



## rust一些高级特性



### 异步编程



#### async/await

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

### 简单的例子

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

