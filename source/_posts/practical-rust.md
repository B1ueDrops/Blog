---
title: Rust原理与实战记录
categories: 编程语言
---



# 原理

## 裸指针



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

# 实战



## 命名规范

https://rust-lang.github.io/api-guidelines/naming.html



## crate

* Rust中, 默认导入了`std::prelude`, 里面包含了常用的函数, 包括`println!`.

## 类型转换

* 整数的类型转换一定要保证**小转大**.
  * $n$位无符号数范围: $[0, 2^n - 1]$
  * $n$位有符号数范围: $[-2^{n-1}, 2^{n-1}-1]$​
* 字符转成ASCII码: `let c = 'a' as u8;`



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
  ```

  



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

