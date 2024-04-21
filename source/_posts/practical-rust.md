---
title: Rust原理与实战记录
categories: 编程语言
mathjax: true
---



# 原理

## Rust

> Rust的优势?

* Rust的所有权机制能让这个语言不使用GC, 并且这种机制不会产生任何的运行时开销.

## 常量

* 常量和**不可变的变量**有很大的不同.

  * 常量需要使用`const`关键字, 并且**必须标注类型**.

    ```rust
    const MAX_NUM: i32 = 1000_i32;
    ```

  * 常量可以定义在任意作用域, 包括全局作用域.

  * 常量必须绑定到**常量表达式**, 必须在编译时期就要算出来.

## crate

* crate是Rust中最小的编译单元, 分为两种类型:
  * `binary crate`: 编译生成二进制文件.
  * `library crate`: 为其他程序提供服务.



## 可变, 引用, 所有权

* Stack和Heap:

  * Stack用来存储编译时已知大小的数据.

  * 访问Stack一般比访问Heap快, 因为Heap还需要定位指针, 而Stack一直维护栈顶, 访问内存次数较少.

* **借用规则:**

  * 同一作用域内, 对于同一个数据, 只能存在一个可变引用.
  * 同一作用域内, 对于同一个数据, 可变引用和不可变引用不能同时存在.

* **Race condition**发生的条件:

  * 两个/多个指针同时访问一块内存数据.
  * 至少有一个指针尝试写入数据.
  * 没有机制来同步指针对内存数据的访问.

## I/O操作



## 裸指针

Rust中, 裸指针(raw pointer)分为一下两种类型:

* `*const T`: 不能通过裸指针修改原数据.
* `*mut T`: 可以通过裸指针修改原数据.

## Pin/Unpin

在Rust中, 一个变量的内存地址是可能会被移动的. 这种移动会引发一些问题, 考虑以下几个场景:

* 场景1: **自引用结构体**
  * 考虑如下这个自引用结构体:
  
    ```rust
    struct SelfRef {
      value: String,
      // 这个裸指针指向成员变量value
      pointer_to_value: *mut String
    }
    ```
  
  * 如果`value`在内存中的位置发生了移动, 那么`pointer_to_value`存储的内存地址就不对了.

> 构造一个样例, 来解释变量在栈内存中的移动?

```rust
struct Data {
    data: usize
}

struct SelfRef {
    data: Data,
    data_ref: *const Data
}

impl SelfRef {
    fn new(data: Data) -> Self {
        Self {
            data,
            data_ref: std::ptr::null()
        }
    }
    fn init(&mut self) {
        self.data_ref = &self.data;
    }
    fn print_info(&self) {
        println!(
            "Actual Data Address: {:p}, Actual Data Content: {}, Value of data_ref: {:p}",
            &self.data,
            self.data.data,
            self.data_ref
        );
    }
}

// 在函数中, 将SelfRef变量的所有权返回, 那么原始变量在
// 栈中的内存会发生移动
fn test() -> SelfRef {
    let mut self_ref = SelfRef::new(Data { data: 1 });
    self_ref.init();
    self_ref.print_info();
    self_ref
}

fn main() {
    let self_ref = test();
    self_ref.print_info();
}
```

这段程序的运行结果如下:

```
Actual Data Address: 0x16eec2288, Actual Data Content: 1, Value of data_ref: 0x16eec2288
Actual Data Address: 0x16eec22b0, Actual Data Content: 1, Value of data_ref: 0x16eec2288
```

很明显, `Actual Data Address`已经发生了改变, 但是`Value of data_ref`没有改变, 再使用`data_ref`就会发生不可预期的错误.

> 构造一个样例, 解释变量在堆内存中的移动?



## Tokio

### Tokio I/O



# 实战



## 命名规范

* https://rust-lang.github.io/api-guidelines/naming.html
* 数字字面值之间加入`_`可以增加可读性.
* 如果出现了多个`else if`, 考虑用`match`重构.



## Cargo

* 默认的`cargo build`是用**debug级别**进行编译, 可以用`cargo build --release`进行release级别的编译.
  * 如果要跑benchmark, 一定要用`release`级别.
* 更新依赖: `cargo update`.
* cargo下载的包在`CARGO_HOME`下.
  * 默认下载到`~/.cargo`.
* ✨检查代码编译是否正确用`cargo check`, 比`cargo build`快很多.

## crate

* Rust中, 默认导入了`std::prelude`, 里面包含了常用的函数, 包括`println!`.

## 数据类型

* 整数的类型转换一定要保证**小转大**.
  * $n$位无符号数范围: $[0, 2^n - 1]$
  * $n$位有符号数范围: $[-(2^{n}-1), 2^{n-1}-1]$​
* 字符转成ASCII码: `let c = 'a' as u8;`
* 除了`byte`, 其他整数类型都可以使用类型后缀, 例如`57_u8, 128_i32`.
* debug模式下会在编译时刻检查整数溢出, 但是release模式不会.



### 字符串

* 格式化字符串:

  ```rust
  let a = String::from(format!("haha {}", b));
  ```




## Vector

* 不可变方式遍历vector:

  ```rust
  let a = vec![1, 2, 3];
  // item是&T类型
  for item in a.iter() {
    
  }
  ```

* 带下标的遍历:

  ```rust
  for (i, item) in a.iter().enumerate() {
    
  }
  ```

* `a.len()`返回的类型是`usize`.

## HashMap

* 引入:

  ```rust
  use std::collections::HashMap;
  ```

* 创建哈希表:

  ```rust
  let mut hashmap: HashMap<T, U> = HashMap::new();
  ```

* 根据键获取值:

  ```rust
  // 传入引用, 返回Option<T>
  hashmap.get(&key);
  ```

* 根据原有的值更新值:

  ```rust
  let value = hashmap.entry(key).or_insert(0);
  // 更新
  *value += 1;
  ```

  

## I/O操作

* 从`stdin`读取字符串:

  ```rust
  use std::io;
  let mut a = String::new();
  // read_line会返回io::Result<Ok, Err>
  io::stdin().read_line(&mut a).expect("");
  ```

  

## 可变, 引用, 所有权

* 当你定义一个变量的时候, 你就需要问自己, 这个变量会不会在未来可变, 如果可变就定义成`mut`.

* 函数的返回值一般都会把所有权返回, 可用不可变变量接收/`mut`接收 (如果返回引用需要处理生命周期).

* 注意, 当一个变量的所有权转移后, 这个变量值的可变性可能改变, **可以用所有权转移的方式改变一个原本不能被改变的值**

  ```rust
  let a = String::from("a");
  let mut b = a;
  b.push_str("haha"); // 不会报错
  ```

* 看一个结构体成员方法时, 注意方法参数:
  * `self`: 会直接拿走结构体变量的所有权, 结构体变量后面就不能用了.
  * `&self`: 没事, 随便用.
  * `&mut self`: 结构体成员可能会发生变化, 具体什么变化看函数实现.

* 结构体中的`new`关联方法中,  参数列表一般都不用引用, **直接拿走原变量的所有权**, 让所有权转移到结构体成员变量中.

* 注意, 你在用`for .. in`遍历数组时, 会调用`into_iter()`, 这个方法会拿走原始变量的所有权, 因此需要考虑原始变量是否实现了`Copy trait`.

* 如果你看到一个变量是`&mut`类型, 那么**你一定要记得, 这个类型没有实现`Copy trait`**, 需要注意所有权.

## Option

* 如果一个变量在之后可能被设置为None, 那么:

  * 在定义时:

    ```rust
    // 注意, 一定是mut
    let mut a: Option<T> = Some(...);
    ```

  * 后期, 如果要把取`a`的值, 并且把`a`设置为None, 那么就用: `let b = a.take().unwrap();`, `take()`方法会直接让原来`Option`变量变成`None`.

  * 此时, `a.is_none()`的返回值就是True.
  
* 快速接收`Option`变量: `if let`

  ```rust
  if let Some(x) = a {
    // x就是其中的值
  }
  else {
    // 如果是None怎么处理
  }
  
  // 如果要使用可变的值
  if let Some(mut x) = a {
    
  }
  ```
  
  * `if let`会拿走`Some(x)`中`x`的所有权, 叫做partially move.
  
* 注意, 使用`a.unwrap()`会拿走`a`的所有权(如果没有实现Copy trait), 可以考虑实现`clone()`.

* 从`Option`中获取其中**值的引用**: 假设`x`是`Option<T>`类型的变量

  * **不可变引用**: `x.as_ref().unwrap()`.
  * **可变引用**:  `x.as_mut().unwrap()`.
  
* 注意, 假设`x`是`&Option<T>`类型, `x.unwrap()`也会拿走`x`的所有权, 因为这里会隐式调用`Deref` trait中的`deref`方法.

  * 也需要用`as_ref()`或者`as_mut()`, 防止所有权转移.





## 多线程

* 如果一个变量可能存在于多个线程之间, 并且可能需要被修改:

  * 它的类型应该定义为: `Arc<Mutex<T>>`.

  * 对它进行克隆用`Arc::clone(&T)`.

  * 然后, 如果一个函数/方法中, 要接收`Arc<Mutex<T>>`类型的变量, 可以直接转移所有权, 不用写成引用.

  * 接收时:

    ```rust
    // 定义成mut, 然后后面进行修改
    let mut a = b.lock().unwrap();
    ```

* 如果一个变量可能存在于多个线程之间, 但是不需要被修改:

  * 类型定义为: `Arc<T>`.
  * 克隆同样用`Arc::clone(&T)`.



## 异常处理

### 基本方法

* 最快方法: `unwrap()`, 失败后`panic`.

* 其次方法: `expect("字符串")`, 失败后打印给定字符串, 然后`panic`.

* Match方法:

  ```rust
  let mut f = match File::open("hello.txt") {
       Ok(file) => file,
       Err(error) => Err(error)
  };
  ```

* `?`宏:

  ```rust
  let mut f = File::open("hello.txt")?;
  ```

  * 这个语句等价为:

    ```rust
    let mut f = match File::open("hello.txt") {
          	Ok(file) => file,
          	Err(e) => return Err(e),
    };
    ```

  * 因此, 如果`?`出现在一个函数中, 函数需要返回`Result<T, Box<dyn std::error::Error>>`.

  * 如果在`main`函数中使用`?`, 那么`main`函数签名要改成: `fn main() -> Result<(), Box<dyn Error>>`



## tokio

* 引入包:

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

* 默认情况下, `tokio`的任务调度器是**多线程**, 这样, 你就需要考虑一个问题:
  * 使用`.await`时, 任务可能会被阻塞, tokio可能会把任务调度到另一个线程中执行.
  * 这样, 与`.await`在同一个作用域的所有变量, 都会被封装成一个状态, 在线程间传递, 如果在线程间传递, 那么这些变量就需要实现`Send` trait.
  * 最典型的例子是, 如果你在`.await`之前, **在同一作用域内**用`std::sync::Mutex`获取了互斥锁, 由于`MutexGuard<T>`没有实现Send trait, 就会报错.
  * 一个解决方案是, 让`.await`调用的作用域在获取Mutex作用域之后, 另一个解决方案是使用**异步锁**.
* **异步锁: `tokio::sync::Mutex`**:
  * 如果在`.await`之间获取了`std::sync::Mutex`, 那么调用`.await`后, 任务会被阻塞, 但是锁没有被释放, 这时候如果另一个任务尝试获取锁, 就会产生死锁.
  * 使用tokio的异步锁可以在`.await`作用域内获取锁, 但是会有性能开销.
* **`tokio::sync::mpsc`和`std::sync::mspc`的区别:**
  * 标准库中, 如果队列满了, 生产者线程会被阻塞, 但是`tokio`不会.



## Snippets

* 生成`[a, b)`的随机数:

  * 使用`rand crate`.

  * ```rust
    use rand::Rng;
    
    let secret_number = rand::thread_rng().gen_range(a, b);
    ```

* 字符串转整数: `let target = source.trim().parse().expect("");`


