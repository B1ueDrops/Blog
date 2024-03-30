---
title: rust阅读学习
categories: 编程语言
mathjax: true
---



## rust基础



### rust工具

* rustup: 安装Rust, 以及切换Rust和标准库的版本.
* cargo: Rust包管理器.



### cargo用法





### 基本数据类型

* `let z = 10`, 其中, 这个`z`的类型会推导成`i32`.
* 强制类型转换用`as`: `let v: u16 = 38_u8 as u16;`
* `x`位无符号整数最大值是$2^x - 1$, 最小值是0.
* `x`位有符号整数最大值是$2^{x-1}-1$, 最小值是$-2^{x-1}$​.



### 可变性, 引用, 所有权

* 最基础的一点, rust如果变量想要可变, 需要前面加上`mut`.

* 假设一个变量叫`a`, 是可变的, 你想取它的引用, 那么需要这样写: `&mut a`, 而不是`&a`.

* 引用可以实现<font color=red>所有权的借用(borrow)</font>, 借用规则是:
  * 同一个作用域内, 可以有多个不可变引用, 但是只能有一个可变引用.

  * 不能对一个不可变的值进行可变借用, 例如下面代码是错误的:
  
    ```rust
    fn main() {
        let x = 5;
        let y = &mut x;
    }
    ```
  
* 一个提醒: 如果你见到一个函数传参数不是传的引用, 那么就需要考虑**所有权转移**的问题.

* 数组索引, 一般都要用引用取, 例如`let query = &args[1]`.



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



### 分模块编程

rust中, 一个package可以分为多个binary crate和至多一个libary crate:

* crate root是`src/main.rs`和`src/lib.rs`.
* rust编译器只能看到crate root.

一般来说, 你需要在crate root中声明依赖的模块, 然后再用`use`导入元素:

```rust
// src/main.rs或src/lib.rs
// 声明crate(main.rs)所依赖的module
mod A;
mod B;
mod C;

use A::funcA;
```

对于模块, 你只需要理解下面几点:

* 一个`rs`文件是一个模块, 其中的元素你可以用`pub`导出.
* 一个文件夹中, 你需要创建`mod.rs`, 然后在其中用`pub mod XX;`来将文件夹中的所有rust文件导出.
* 在crate root中需要用`mod XX;`声明上级模块.
* 如果在子模块中需要导入上级模块的东西, 可以用`use package名字::XXX`导入.





### 异常处理

Rust中的异常分为两类:

* **可恢复异常**: 用`Result<T, E>`恢复.
* **不可恢复异常: **用`panic!()`处理.



#### 不可恢复异常

* `panic!()`如果在main线程中, 会直接停止main线程, 但是在子线程中panic不会让main线程挂, 因此不要让main线程承担大多数任务.

* `panic`有两种退出模式:

  * 栈回溯(backtrace): 退出时打印调用栈, 可以用在开发环境.

  * 终止(abort): 直接退出, 适合生产环境, 能减少编译出的二进制文件大小.

    ```toml
    [profile.release]
    panic = 'abort'
    ```



#### 可恢复异常

一般来说, 可能存在异常的函数返回值都是`Result<T, E>`:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

这个类型之后可以用`unwrap()`或者`expect()`处理, 两者类似, 区别在于`expect()`可以接受一个字符串, panic后打印这个字符串信息:

* `unwrap/expect`: 如果成功, 直接返回返回值, 如果失败, 直接`panic`.
* 这是异常处理最快的方法.

> 处理方法1: Match表达式

```rust
use std::fs::File

fn main() {
  let f = match File::open("hello.txt") {
    	Ok(file) => file,
    	Error(error) => {
        panic!("Cannot read file");
    }
  };
}
```

可以在`Error`后再用Match表达式处理:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

也可以直接不处理, 向上传播异常:

* 原函数需要返回`Result<T, E>`
* match表达式如果是`Error`, 需要`return Err(error)`.

Rust提供了专门的宏`?`来简化传播异常的写法:

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    // ...
}
```

这个写法就等价于:

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = match File::open("hello.txt") {
      	Ok(file) => file,
      	Err(error) => Err(error)
  	};
    // ...
}
```

> ?相比于match表达式, 还有另一种好处.

* `?`传播的异常, 返回的本质上是`Box<dyn std::error:Error>`类型.
* 在一个系统中, 你可以基于`Error`, 自定义异常.
* 自定义的异常需要实现`From` trait中的`from`函数, 用来转换到更大的异常类型.
* 这样, 在使用`?`时, rust就会根据你的签名, 自动给你转换到对应级别的异常.

* `Option<T>`也支持`?`.

如果在main函数中使用了`?`, 那么main函数的签名需要改动:

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}
```



### 生命周期






## rust面向对象

1. 大致写法:

```rust
// 成员变量
pub struct Student {
  pub name: String,
  age: i8
}

// 成员方法
// impl可以不用pub, 但是成员方法要pub
impl Student {
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





## rust高级特性





### 常见的trait



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

在Rust中, 引用类型其实不用你手动`*`才能访问值, 例如你有一个结构体引用, 你不用`(*x).name`才能访问成员, 而是直接用`x.name`就可以, 背后的机制就是`deref trait`.



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



### 智能指针



#### Box\<T\>

Box指针本质上还是引用类型, 只是这个引用会把你原本的数据放到堆上, 然后再给你返回一个引用.

```rust
// a数据存在3上
let a = Box::new(3);
```

如果你想打印`a`:

```rust
println!("{}", a);
```

这时可以打印, 并且不会报错的, 因为`a`会隐式调用`deref`进行解引用.

但是如果你想拿这个值做运算:

```rust
let b = a + 1;
```

这就会报错, 正确写法是`let b = *a + 1`, 这个时候需要手动调用`deref`.

<font color=red>注意: Box指针会导致所有权的转移</font>, `Box::new(xxx)`之后, `xxx`的所有权会转移到接收变量中.

```rust
fn main() {
    let s = String::from("hello, world");
    // s在这里被转移给a
    let a = Box::new(s);
    // 报错！此处继续尝试将 s 转移给 b
    let b = Box::new(s);
}
```



**Box指针有一个非常有效的应用场景:**

* 在Rust中, 编译器要求类型的大小必须是固定的.
* 但是有些类型是无法在编译时期知道大小的, 例如递归的类型定义.
* 这时, 可以将递归的成员变量转换成`Box<T>`指针类型.

例如, 如果你想用一个数组, 存储所有实现了某一个`trait`的对象(动态类型), 就需要用到`Box`指针, 例子如下:

```rust
trait Draw {
    fn draw(&self);
}
// 下面Button和Select都实现了Draw这个trait
struct Button {
    id: u32
}
impl Draw for Button {
    fn draw(&self) {
        println!("Button {}", self.id);
    }
}
struct Select {
    id: u32
}
impl Draw for Select {
    fn draw(&self) {
        println!("Select {}", self.id)
    }
}

fn main() {
  	// 定义类型时需要加上dyn, 因为实现trait的对象都是一个动态类型
    let elems: Vec<Box<dyn Draw>> = vec![ Box::new(Button {id:1}), Box::new(Select {id: 1}) ];
    for e in elems {
        e.draw();
    }
}
```

如果你要从Box指针数组中取出元素时, 你需要解两次引用, 例如下面这个例子:

```rust
fn main() {
    let arr = vec![ Box::new(1), Box::new(2) ];
    // 注意, 这里必须取引用, 否则会发生所有权转移
    let (first, second) = (&arr[0], &arr[1]);
    // 第一次将&Box<i32>转成Box<i32>, 第二次将Box<i32>转成i32
    let sum = **first + **second;
    
}
```



#### Rc\<T\>

Rc\<T\>可以在单线程环境下, 创建一个数据的多个不可变引用, 并且提供了如下功能:

* 实现了`Deref`和`Drop`.
* 提供了引用计数(reference counting), 一旦数据的引用计数是0, 就会自动释放.

用法如下:

```rust
use std::rc::Rc;

fn main() {
  	// 用Rc::new创建一个数据的引用, 类型是Rc<String>
    let a = Rc::new(String::from("hello world"));
  	// 用Rc::clone创建数据的新的不可变引用, 这个clone是浅拷贝, 性能比较高
    let b = Rc::clone(&a);
		// 用Rc::strong_count输出变量的计数
    assert_eq!(2, Rc::strong_count(&a));
    assert_eq!(Rc::strong_count(&a), Rc::strong_count(&b));
}
```



#### Arc\<T\>

Arc\<T\>可以在多线程环境下, 实现多个线程拥有同一个数据的多个不可变引用, 和`Rc<T>`的API完全相同.

* 和`Rc<T>`不同之处在于, `Arc<T>`采用原子指令维护了引用计数, 会有一定的性能损耗.



### 并发编程

#### Mutex





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



## rust开发实践

* `dbg!()`: 快速打印变量/表达式, 方便调试.

* 基础命令行参数读取:

  ```rust
  use std::env;
  
  fn main() {
      let args:Vec<String> = env::args().collect();
      for arg in args {
          println!("{}", arg);
      }
  }
  ```

* 读文件:

  ```rust
  use std::fs;
  
  fn main() {
    	// 读取到String中
      let text = fs::read_to_string("D:\\text.txt").expect("文件读取失败的错误信息");
      println!("{}", text);
  }
  ```

* `if let`表达式简化代码: 

  ```rust
  // 简化异常处理
  if let Err(e) = run(config) {
      println!("Application error {}", e);
      process::exit(1);
  };
  ```

* 实战来说, 一些顶层函数一般放到`lib.rs`中, `main.rs`一般保留最小的需要支持的元素和流程.



### TDD测试驱动开发

rust TDD的一般流程是:

* 在`src`同级目录下创建`test`.

* 里面创建很多个rust文件, 随便起名.

* 每个文件的格式如下:

  ```rust
  #[cfg(test)]
  mod tests {
  
      use package名字::*;
  
      #[test]
      fn two_result() {
          let query = "duct";
          let contents = "Rust:\nSafe duct\nniubi duct";
          assert_eq!(vec!["Safe duct", "niubi duct"], search(query, contents))
      }
  }
  ```

  * `#[cfg(test)]`表示只有在`cargo test`时才运行测试.
  * `#[test]`表示单元测试.

* 错误信息采用`eprintln`输出到`stderr`.

之后只要运行`cargo test`就会运行所有单元测试.

<font color=red>上述函数中, 无法对私有函数进行测试</font>, 如果真要对私有函数进行测试, 需要把测试函数`#[test]`写私有函数定义的地方.

