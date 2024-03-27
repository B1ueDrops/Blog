---
title: rust用到哪里学哪里
categories: 编程语言
---



## rust基础



### rust工具

* rustup: 安装Rust, 以及切换Rust和标准库的版本.
* cargo: Rust包管理器.



### 可变性和引用

* 最基础的一点, rust如果变量想要可变, 需要前面加上`mut`.
* 假设一个变量叫`a`, 是可变的, 你想取它的引用, 那么需要这样写: `&mut a`, 而不是`&a`.



### 元组结构体

```rust
struct Color(i32, i32, i32);

let black = Color(0, 0, 0);
// 获取第0, 1, 2个元素
println!("{}", black.0);
println!("{}", black.1);
println!("{}", black.2);
```





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



### rust所有权转移

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

> 内部原理

* 首先, 基本类型完全存储在栈上, rust会直接拷贝, 不会发生所有权转移.
* 非基本类型数据存储在堆上, 引用存储在栈上, rust处于安全考虑, 会让原来的所有权失效.
* 堆上数据深拷贝可以用`clone`方法.



### rust分模块编程

* rust中, `package`就相当于`project`的概念.

* 一个`package`中, 可以包含多个`crate`, 一个`crate`是一个最小的编译单元.

  * `crate`分为binary crate和library crate, 两者分别生成可执行文件/库文件.

  * binary crate可以用`cargo new --bin`生成, library crate可以用`cargo new --lib`生成.

  * binary crate默认有`src/main.rs`, library crate默认有`src/lib.rs`.

  * 一个`package`至少有一个crate, 可以有任意多个binary crate, 最多有一个library crate.

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



## rust面向对象

1. 大致写法:

```rust
// 成员变量
pub struct Student {
  pub name: String,
  age: i8
}

// 成员方法
pub impl Student {
  // 构造方法, 也是类方法
  pub fn new(name: String, age: i8) -> Student {
    Student {
      name: name,
      age: age
    }
  }
  // public
  // 如果需要改成员变量, 用&mut self
  pub fn get_age(&self) -> i8 {
    return self.age;
  }
  // private
  fn haha(&self) {
    
  }
}

fn main() {
  // 创建对象
  let student = Student::new(String::from("haha"), 18);
  student.get_age();
}
```





## rust一些高级特性



### 智能指针



#### Deref

Deref是一个有关于解引用的trait.

假设我有一个结构体:

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
  fn new(x: T) -> MyBox<T> {
    MyBox(x);
  }
}
```

现在, 你可以为这个结构体实现`Deref` 这个trait, 来让你能`*`一个结构体引用.

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
  // &self是结构体引用类型, 在这个函数中, 你要通过这个结构体引用, find一个基本类型的引用, 然后返回
  fn deref(&self) -> &T {
    &self.0
  }
}
```

本质上, `deref`会把一个结构体引用, 转换成一个基本类型引用.

实现``deref`后, 你再`*`一个结构体引用`x`, 就等价于`*(x.deref())`



> 自动deref机制

看这段代码:

```rust
fn display(s: &str) {
    println!("{}",s);
}

fn main() {
    let s = MyBox::new(String::from("hello world"));
    display(&s)
}
```

对于`&s`, 它会被连续`deref` , 直到转换成`&str`为止.



#### Drop

Drop是一个`trait`, 如果一个结构体实现了`Drop`, 那么它的生命周期结束之后, 就会自动调用`drop`方法实现收尾工作, rust就是因为这个才可以不实现`GC`.

```rust
impl Drop for XXX {
  fn drop(&mut self) {
  }
}
```

但是, 在这段代码中, `drop`的参数是`&mut self`, 就表示一个结构体调用`drop`函数后, 它的所有权没有被收走.

* 因此, 你不可以对一个结构体`x`显示地调用`x.drop()`函数.

但是, 有些场景下, 你需要提前释放某些资源, 例如互斥锁等, 这个时候可以使用`std::prelude::drop`函数.

这个函数签名是:

```rust
pub fn drop<T>(_x: T)
```

会直接把所有权拿走, 调用完这个函数, 如果在下面再使用原变量, 就会编译错误.





#### Box\<T\>

Box指针能让你把数据分配在堆上.

```rust
// a数据存在3上
let a = 3;
let a = Box::new(3);
```



> 关于数据分配在栈/堆上对性能影响的规律

* 小数据: 在栈上分配/读取比堆快.
* 中数据: 在栈上分配快, 读取不一定.
* 大数据: 在堆上分配/读取.





### 异步编程



#### async/await

* Rust中, 使用`async`定义的函数, 调用后会变成`Future`.
* `Future`需要一个执行器(executor)来执行.
* 当`Future`执行被阻塞后, 它会让出当前线程的控制权, 来执行其他`Future`, 不会导致整个线程被阻塞.
* 在一个`async`函数中, 调用另一个`async`函数可以对`Future`调用`await`, `await`会建立`Future`之间的依赖关系.
  * 因此, 执行`async`有两种方式:
    * 第一, 使用Executor.
    * 第二, 在`async`函数中使用`.await`.



#### 多个Future之间并发

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

#### 一个简单的例子

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

