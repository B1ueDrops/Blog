---
title: Rust使用日记
categories: 编程语言
mathjax: true
---



## Rust介绍

> Rust的优势?

* **Zero Cost Abstraction: ** Rust引入的一些机制 (例如所有权机制), 并不会引入额外的运行时开销.



## 命名规范

> https://rust-lang.github.io/api-guidelines/naming.html

| 元素      | 命名方式       |
| --------- | -------------- |
| 局部变量  | snake_case     |
| 函数      | snake_case     |
| 方法      | snake_case     |
| 结构体    | UpperCamelCase |
| Trait     | UpperCamelCase |
| 枚举      | UpperCamelCase |
| 模块(mod) | snake_case     |



## 常量

常量和**不可变的变量**有很大的不同.

* 常量需要使用`const`关键字, 并且**必须标注类型**.

  ```rust
  <pub> const MAX_NUM: i32 = 1000_i32;
  ```

* 常量可以定义在任意作用域, 包括全局作用域.

* 常量必须绑定到**常量表达式**, 必须在编译时期就要算出来.



## 基本数据类型

* 整数的类型转换一定要保证**小转大**.

  * $n$位无符号数范围: $[0, 2^n - 1]$
  * $n$位有符号数范围: $[-(2^{n}-1), 2^{n-1}-1]$​

* 字符转成ASCII码: `let c = 'a' as u8;`

* 除了`byte`, 其他整数类型都可以使用类型后缀, 例如`57_u8, 128_i32`.

* debug模式下会在编译时刻检查整数溢出, 但是release模式不会.

* 打印变量的地址, 首先把它转成裸指针:

  ```rust
  let ptr = &<变量名> as *const <变量类型>
  ```

  * 打印裸指针用`{:p}`.



## 字符串/切片

* Rust中字符串用`UTF-8`编码.

* Rust中, 语言层面只提供了一种字符串类型, 就是`&str`, 就是切片.

  * `&str`本质上就是对存储在其他地方, `UTF-8`编码的字符串的引用.
    * 字符串字面值就存在于二进制文件中.

* `String`本质上是对`Vec<u8>`的封装.

* 创建字符串:

  * `String::from()`函数.
  
    ```rust
    let a = String::from(format!("haha {}", b));
    ```
  
  * `to_string()`方法: 适用于实现了Display trait的数据类型, 包括`&str`.
  
* 从`String`类型字符串中获取字符:

 ```rust
  a.chars().nth(index); // 返回Option, 自己处理
 ```

* 更新String:

  * 将字符附加到`String`: `push`

    ```rust
    let mut s = String::from("foo");
    s.push('l');
    ```

  * 将`&str`附加到`String`: `push_str`

    ```rust
    let mut s = String::from("foo");
    s.push_str("bar");
    ```

  * `String`字符串拼接:

    ```rust
    let s1 = String::from("Hello");
    let s2 = String::from("World");
    let s3 = s1 + &s2
    ```

    * 注意, 拼接的后面必须都是`&String/&str`类型, 因为`+`本质上是调用了类似`fn add(self, a: &str, ...)`这样签名的函数.
    * 注意, 拼接之后, `s1`所有权就没有了, 后面不能再使用.

* 访问字符串:

  * `String`类型不能通过索引访问(没有实现Integer trait).
* 因为索引访问的是`u8`, 而不是字符.
  * Rust中有两种看待字符串的方法:
    * 字节数组: `a.bytes()`
    * 字符数组: `a.chars()`
  * 遍历字符串中的字符:

```rust
  // a可以是String或者&str
  for ch in a.chars() {
    
  }
```


* 字符串逆序:

  ```rust
  // a是String类型
  let a = a.chars().rev().collect::<String>();
  ```

* 字符串转整数:

  ```rust
  let s1 = String::from("42");
  let n1: u64 = s1.parse()?;
  ```

* 在使用`match`做字符串`&str`匹配时, 需要在匹配后面加上`_`的处理, 例如:

  ```rust
      let inst_struct = match inst.as_str() {
          "0110111" => {
              Inst {
                  hex: inst,
                  name: String::from("lui"),
                  rd: 0_u8,
                  rs1: 0_u8,
                  rs2: 0_u8,
              }
          },
        // 如果所有的都没匹配到
          _ => {
              panic!("Not implemented");
          }
      };
  ```

* **字符串切片:** `&str`

	* 切片用来创造数组/字符串某一部分的**不可变引用**.

	* 创建方法是: `&s[startIndex..endIndex]`, 选取的区间是`[startIndex, endIndex - 1]`.
    * 从0开始: `&s[ ..endIndex]`
    * 直到结尾: `&s[0 ..]`
    * 选取全部: `&s[ .. ]`

  * 注意: 切片索引访问的不是字符, 而是`u8`, 需要保证访问的索引在每一个字符的边界位置, 例如下面代码会报错:

    ```rust
    let s = "中国人";
    // 中文在UTF-8中占3个字节, [0..2]正好在'中'的里面
    let a = &s[0..2];
    ```

* 如果一个函数用字符串当作参数, 类型尽量定义成`&str`, 让API更加通用.

## Cargo

* 默认的`cargo build`是用**debug级别**进行编译, 可以用`cargo build --release`进行release级别的编译.
  * 如果要跑benchmark, 一定要用`release`级别, 因为debug级别会增加很多影响性能的东西.
* 更新依赖: `cargo update`.
* cargo下载的包在`CARGO_HOME`下.
  * 默认下载到`~/.cargo`.
* ✨检查代码编译是否正确用`cargo check`, 比`cargo build`快很多.
* cargo添加完dependency后, 下载包用: `cargo build`



## package, crate, module

* Rust中, 默认导入了`std::prelude`, 里面包含了常用的函数, 包括`println!`.
* Rust的引入规范:
  * 引入函数时, 引入到父级模块, 然后用`父级模块::函数名`调用, 用于区分是哪个作用域中的函数.
  * 引入struct, enum这种时, 引入到自身, 因为引入到父级模块的话就太难写了.
* 如果有多个可执行文件, 可以放到`src/bin`中, 运行某个可执行文件就用: `cargo run --bin <可执行文件名字>`

* crate是Rust中最小的编译单元, 分为两种类型:
  * `binary crate`: 编译生成二进制文件.
  * `library crate`: 为其他程序提供服务.
* crate是一颗由`mod`组成的树状结构:
  * 树的根是一个叫做`crate`的模块, 在概念上, 叫做crate root, 它由定义在`src/main.rs`和`src/lib.rs`中的元素组成.
  * 如果要在mod和mod之间建立父子关系, 需要在父模块中使用`mod XXX;`来声明子模块.
  * Rust中, 一个文件夹如果里面有`mod.rs`, 那么会被看成是和文件夹同名的mod, mod中的元素就是在`mod.rs`中的元素.
  * Rust中, 一个文件默认也被看成一个和文件同名的mod.
  
* 如果要在某一个mod中引入其他mod中的元素, 可以使用`use`:
  * 如果使用绝对路径, 需要从`crate::`开始引入.
  * 如果使用相对路径, `super::`表示父亲mod.
  



## 可变, 引用, 所有权

* Stack和Heap:

  * Stack用来存储编译时已知大小的数据.

  * 访问Stack一般比访问Heap快, 因为Heap还需要定位指针, 而Stack一直维护栈顶, 访问内存次数较少.
  
* **借用规则:**

  * 同一作用域内, 对于同一个数据, 只能存在一个可变引用.
  
  * 同一作用域内, 对于同一个数据, 可变引用和不可变引用不能同时存在.
  
    * 有一个`Vec`的场景:
  
      ```rust
      let mut v = vec![1, 2, 3];
      // 可变引用
      let first = &v[0];
      // 不可变引用
      v.push(6);
      println!("{first}");
      // 这段代码会报错
      ```
  
      
  
  * **一个场景: **假设你有一个函数, 使用了变量`a`的不可变引用, 得到一个结果, 如果你后来改变了`a`, 那么得到的这个结果就可能会失效 (例如给定字符串计算长度, 你改了字符串, 这个长度结果肯定就不对), Rust不允许这种情况发生, 后面不能改(也就是不能使用不可变引用来修改).
    * 也就是说, 通过不可变引用, **还能保证一个变量`x`和对应的函数`f(x)`随时可以保持关系**.
  
* **Race condition**发生的条件: Rust的借用规则可以从根本上避免出现race condition
  * 两个/多个指针同时访问一块内存数据.
  * 至少有一个指针尝试写入数据.
  * 没有机制来同步指针对内存数据的访问.
  
* 根据Rust的所有权机制, 自引用结构体很难构造, 例如:
  
  ```rust
  #[derive(Debug)]
  pub struct SelfRef<'a> {
      mystr: String,
      mystr_ref: &'a str,
  }
  
  fn main() {
      let hello = "Hello".to_string();
      let self_ref = SelfRef {
          mystr_ref: &hello,
          mystr: hello
      };
      println!("{:?}", self_ref);
  }
  ```
  
  * 这个代码编译错误, `hello`的所有权会移动到`self_ref.mystr`上, 这个时候, `&hello`这个不可变借用就会失效.
  
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

* 所有权移动时, 一般会涉及变量地址的改变.



## 结构体和方法

* 定义`struct`: 需要为所有Field指明名称和类型:

  ```rust
  struct User {
    name: String,
    sign_in_count: u64,
    active: bool
  }
  ```

  * 结尾没有`;`.

* 实例化:

  ```rust
  let user = User {
    active: true,
    sign_in_count: 554,
    name: String::from("haha")
  };
  ```

  * 创建实例时, 必须同时为所有的字段都赋值.

* 访问/修改`struct`用`.`, 例如`user.name`.

* 一旦一个`struct`变量是`mut`, 那么它的所有字段都是`mut`.

* 当字段的名字和字段 对应值的变量名是一样的时候, 就可以简写:

  ```rust
  let active = true;
  let user = User {
    active,
    sign_in_count: 554,
    name: String::from("haha")
  }
  ```

* `struct`更新语法: 当你想基于某一个已有的结构体创建一个新结构体时, 可以用`..`

  ```rust
  let user = User {
    active: true,
    sign_in_count: 554,
    name: String::from("haha")
  };
  // 没有更新语法
  let user2 = User {
    active: false,
    sign_in_count: user.sign_in_count,
    name: user.name
  }
  // 更新语法..
  let user2 = User {
    active: false,
    ..user
  }
  ```

* `tuple struct`: `struct`有名字, 但是字段没有名字:

  ```rust
  struct Color(i32, i32, i32);
  
  let black = Color(0, 0, 0);
  // 访问元素
  black.0
  black.1
  black.2
  ```

  * Rust函数也可以有多个返回值, 返回值用tuple包裹即可.

* 方法和函数不同, 方法一般定义在`impl`内部, 并且参数一般含有`self/&self/&mut self`.

  * `self`会夺去结构体变量的所有权.

```rust
struct Rectangle {
  width: u32,
  height: u32
}

impl Rectangle {
  fn area(&self) -> u32 {
    self.width * self.height
  }
}
```

* 可以在`impl`块中, 不包含`self`, 这叫做**关联函数**, 不是方法, 例如`String::from()`.

  * 使用时, 用`::`调用.

  * 一般用于构造器:

    ```rust
    impl Rectangle {
      fn square(size: u32) -> {
        Rectangle {
          width: size,
          height: size
        }
      }
    }
    
    let a = Rectangle::square(2);
    ```

* 给结构体/枚举加上`Debug`宏:

  ```rust
  #[derive(Debug)]
  ```

  * 在调试时, 可以使用`{:?}`或者`{:#?}`进行输出, `{:#?}`的输出更优美一些.

## 枚举

* 定义枚举, 例如IP地址分为IPv4和IPv6:

  ```rust
  enum IpAddrKind {
    V4,
    V6
  }
  ```

  * 使用: `let four = IpAddrKind::V4`.

* Rust中, 允许将任意类型的数据附加到枚举的元素中:

  ```rust
  enum IpAddrKind {
    V4(String),
    V6(String)
  }
  ```

  * 每个元素可以存储不同类型的值:

    ```rust
    enum IpAddrKind {
      V4(u8, u8, u8, u8),
      V6(String)
    }
    
    let home = IpAddrKind::V4(127, 0, 0, 1);
    ```

* 为枚举定义方法: 和`struct`一样, 也用`impl`关键字

  ```rust
  impl IpAddrKind {
    
  }
  ```

* Option枚举:

  * 在Rust中没有Null.

  * 定义在`std::prelude`中.

    ```rust
    enum Option<T> {
      Some(T),
      None
    }
    ```

  * 如果一个值可能存在, 也可能不存在(为`None`), 那么就应该是`Option`类型.

  * 如果你要声明一个变量是`None`, 那么需要显式声明类型:

    ```rust
    let absent_number: Option<i32> = None;
    ```




## 模式匹配

* `match`表达式允许一个值和一系列模式进行匹配, 并且执行匹配成功的代码.

  * 模式可以是变量名, 字面值, 或者通配符.

  ```rust
  enum Coin {
    A,
    B,
    C,
    D,
  }
  
  fn value_in_cents(coin: Coin) -> u8 {
    match coin {
      Coin::A => 1,
      Coin::B => 2,
      // 加了花括号, 后面不用,
      Coin::C => {
      }
      Coin::D => 4,
    }
  }
  ```

  * `match`匹配的分支可以绑定到被匹配 对象的部分值.

    * 可以从enum中拿到值:


    ```rust
    enum Coin {
      A,
      B,
      C,
      D(u8),
    }
    
    fn value_in_cents(coin: Coin) -> u8 {
      match coin {
        Coin::A => 1,
        Coin::B => 2,
        // 加了花括号, 后面不用,
        Coin::C => {
          3
        }
        Coin::D(value) => {
         		println!(value); 
          	value
        }
      }
    }
    ```


  * `match`匹配必须穷举所有的可能.
    * `_`代替其余没有被列出来的值.

* `if let`处理只关心一种匹配, 不关心其他匹配的情况:

  ```rust
  if let Some(3) = v {
    println!("three");
  }
  else {
    println!("others");
  }
  ```

  

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
  // 从Some(T)中拿到不可变引用:
  if let Some(x) = a.as_ref() {
    
  }
  // 可变引用
  if let Some(x) = a.as_mut() {
    
  }
  ```

  * `if let`会拿走`Some(x)`中`x`的所有权, 叫做partially move, 同样, `match`也会拿走, 模式匹配时可以考虑`as_ref()/as_mut()`

* 注意, 使用`a.unwrap()`会拿走`a`的所有权(如果没有实现Copy trait), 可以考虑实现`clone()`.

* 从`Option`中获取其中**值的引用**: 假设`x`是`Option<T>`类型的变量

  * **不可变引用**: `x.as_ref().unwrap()`.
  * **可变引用**:  `x.as_mut().unwrap()`.

* 注意, 假设`x`是`&Option<T>`类型, `x.unwrap()`也会拿走`x`的所有权, 因为这里会隐式调用`Deref` trait中的`deref`方法.

  * 也需要用`as_ref()`或者`as_mut()`, 防止所有权转移.



## 异常处理

* Rust中的错误分为两种:
  * 可恢复的错误: Rust提供了`Result<T, E>`类型.
  * 不可恢复的错误: Rust提供了`panic!`宏.

* `panic!`宏使用: `panic!("错误信息")`

  * `panic`有两种处理模式:

    * `unwind`: 程序会展开调用栈, 并且依次清理函数中的数据 (工作量较大).
    * `abort`: 程序直接终止调用栈, 数据由OS回收.

  * 在`release`模式下, 需要用`abort`模式 (可以让二进制文件更小), 在`Cargo.toml`中加上:

    ```toml
    [profile.release]
    panic = 'abort'
    ```

  * `panic`可能发生在源码中, 如果要定位源码的位置, 需要运行时将`RUST_BACKTRACE`设置为1.

* `Result<T, E>`:

  ```rust
  enum Result<T, E> {
    // 操作成功返回T类型
    Ok(T),
    // 操作失败返回E类型
    Err(E)
  }
  ```

* 处理`Result<T, E>`的几种方法:

  * `match`表达式:

    ```rust
    let f = File::open("hello.txt");
    
    let f = match f {
      Ok(file) => file,
      // 错误类型
      Err(error) => {
        panic!("Error opening file {}", error);
      }
    }
    ```

  * `unwrap()`: 直接提取`Ok(T)`中的`T`, 如果失败就会直接`panic`.

    ```rust
    let f = File::open("hello.txt").unwrap();
    ```

  * `expect()`: 在`unwrap()`的基础上, 可以自定制错误信息:

    ```rust
    let f = File::open("hello.txt").expect("错误信息");
    ```

  * 向上传递错误:

    * 在遇到错误的时候, 直接`return Err(e)`.

    * 函数的返回值需要变成`Result<T, E>`.

    * Rust中还提供了`?`宏, 专门用来传播错误:

      ```rust
      let mut f = File::open("hello.txt")?;
      ```

      * 如果函数返回`Ok(T)`, 那么`T`就会自动赋值给`f`.

      * 如果返回`Err(e)`, 那么函数就会直接`return Err(e);`

      * 如果一个函数内部有`?`, 那么最后函数需要写一个返回值`Ok(T)`.

      * 如果在`main`函数中需要使用`?`, 那么`main`函数的签名需要是:

        ```rust
        // Box<dyn Error>表示任何可能的错误类型
        fn main() -> Result<(), Box<dyn Error>> {
          
        }
        ```

    * `?`函数会隐式调用`From`函数, 将返回错误类型自动转换为函数签名所定义的错误类型.




## I/O

* I/O的元素主要在`std::io`中.
* 其中, 最重要的两个traits是`Read`和`Write`, 分别在`std::io::{Read, Write}`中.
  * 实现了`Read` traits的叫`reader`, 可以调用`read`方法将reader中的数据读到buffer中, 返回成功读取的字节数.
  * 实现了`Write` traits的叫`writer`, 可以通过`write`方法将buffer中的内容写入到`writer`中, 返回成功写入的字节数.

* 从标准输入读取`String`:

  ```rust
  use std::io;
  
  let mut a = String::new();
  io::stdin().read_line(&mut a).expect("");
  ```

* Rust中, 默认的`./`是在你cargo项目的根路径.

* `b"hello"`是`&[u8]`类型(切片).

* 使用I/O操作时, 一般会发生Error, 返回值一般是: `io::Result<T>`.

  * 这个是在`std::io`里定义的一个别名, 全称还是`Result<T, Error>`.

* 读取一个文件的每一行:

  ```rust
  use std::fs::File;
  use std::io::BufReader;
  use std::io::{self, BufRead, Read, Write};
  
  fn main() -> io::Result<()> {
  
      let f = File::open("./foo.txt")?;
      let mut reader = BufReader::new(f);
  
      for line in reader.lines() {
          println!("{}", line.unwrap());
      }
  
      Ok(())
  }
  ```

* TCP Server:

  ```rust
  use std::io::{Read, Write};
  use std::net::TcpListener;
  
  fn main() {
  
      // 监听
      let listener = TcpListener::bind("127.0.0.1:3000").unwrap();
  
      for stream in listener.incoming() {
          let mut stream = stream.unwrap();
          let mut buffer = [0; 1024];
          // 读取到buffer
          stream.read(&mut buffer).unwrap();
          // 给客户端返回buffer
          stream.write(&mut buffer).unwrap();
      }
  }
  ```

  * TCP Client:

    ```rust
    use std::io::{Read, Write};
    use std::net::TcpStream;
    use std::str;
    
    fn main() {
    
      let mut stream = TcpStream::connect("127.0.0.1:3000").unwrap();
      // 发送消息
      stream.write("Hello".as_bytes()).unwrap();
      
    	// 接收消息
      let mut buffer = [0; 5];
      stream.read(&mut buffer).unwrap();
      
      println!("Response from server:{:?}", str::from_utf8(&buffer).unwrap());
      
    }
    ```

    



## trait



* PartialEq采用了离散数学中对Equivalence Relation的定义.

  * 如果一个关系是Equivalence Relation, 那么需要满足自反性(reflexivity, `a = a`), 对称性(symmetry, `a = b -> b = a`)以及传递性(transivity `a = b && b = c -> a = c`).

  * 如果一个关系不满足自反性, 那么这个关系就是Partial Equivalence Relation, 例如浮点数中的`NaN`.

    ```rust
    let x = f64::NAN;
    assert!(x != x);
    ```

* 给结构体实现`Default `trait, 可以在使用时获取一个默认值:

  ```rust
  impl Default for Student {
    fn default() -> Self {
      // 构造默认值
    }
  }
  ```

  * 可以使用`Student::default()`返回一个`Student`类型的默认值.

* 如果一个结构体需要比较操作:

  * 如果结构体成员都可以进行比较, 用`#[derive(Eq)]`
  * 如果出现了不满足自反性的成员, 用`#[derive(PartialEq)]`.
  * 比较结构体本质上是对每一个成员进行比较.
  * 一般**枚举**应该都需要比较, 定义枚举时需要考虑.

* 给结构体/枚举实现`From`  trait可以通过`into()`和类型标注转换为对应的结构体类型, 例如:

  ```rust
  #[derive(Debug, PartialEq)]
  pub enum Method {
      Get,
      Post,
      Uninitialized,
  }
  // &str到Method类型的转换
  impl From<&str> for Method {
      fn from(s: &str) -> Self {
          match s {
              "GET" => Method::Get,
              "POST" => Method::Post,
              _ => Method::Uninitialized,
          }
      }
  }
  
  // 可以用如下方法进行类型转换
  // "GET".into()
  // Method::from("GET")
  ```




## Vector

* Vector的类型是`Vec<T>`, 是由标准库提供的.

  * 可以存储多个值, 这些值的类型都必须相同.

  * 如果要存放不同的数据类型, 那么可以用枚举:

    ```rust
    enum Data {
      Int(i32),
      Float(f64),
      String(String),
    }
    
    let mut v = vec![
      Data::Int(3),
      Data::Float(3.3),
      Data::String("3"),
    ];
    ```

    

* 创建`Vec`:

  ```rust
  let v: Vec<i32> = Vec::new();
  ```

* 使用初始值创建`Vec`:

  ```rust
  let v = vec![1, 2, 3];
  ```

* 添加元素: `push`

  ```rust
  let mut v = Vec::new();
  v.push(1);
  ```

* 当`Vec<T>`离开作用域时, 会被清理掉, 但是其中的元素也会被清理掉.

  * 如果其他的地方有`Vec`中元素的索引, 就会发生问题.

* 访问`Vec`中元素的两种方式:

  * 索引: 如果索引超出范围, 程序就会`panic`.
  * `get方法`: 返回值是`Option<T>`, 如果越界会返回`None`.

* `Vec`的各种遍历方式:

  ```rust
  let a = vec![1, 2, 3];
  
  // 只有下标
  // a.len()返回值是usize, i也是usize, 考虑溢出
  for i in 0..a.len() {
    	// a[i]
  }
  
  // T
  for &item in a.iter() {
    // item是T类型
  }
  
  // &T
  for item in a.iter() {
    
  }
  // 带下标&T
  for (i, item) in a.iter().enumerate() {
    
  }
  ```



## HashMap

* 引入:

  ```rust
  use std::collections::HashMap;
  ```

* 创建哈希表:

  ```rust
  let mut hashmap: HashMap<T, U> = HashMap::new();
  ```

  * HashMap中, 所有的Key必须是同一种类型, 所有的Value必须是同一种类型.
  * 数据存储在Heap上.

* 添加元素: `insert`

  ```rust
  let a = String::from("haha");
  let b = 1;
  
  let mut map: HashMap<String, i32> = HashMap::new();
  map.insert(a, b);
  ```

* HashMap的所有权问题:

  * 如果类型实现了Copy trait, 那么值会被复制到HashMap中.
  * 对于有所有权的值, 所有权会转移到HashMap中, 例如调用`insert`.
  * 如果将变量的引用插入HashMap, 那么HashMap有效的期间, 必须保证插入的引用有效.

* 根据键获取值: `get`

  * 参数: `&T`
  * 返回值: `Option<&V>`

  ```rust
  // 传入&T, 返回Option<&U>
  hashmap.get(&key);
  ```

* 遍历HashMap:

  ```rust
  // k, v都是&T, &U类型
  for (k, v) in &hashmap {
    
  }
  ```

* 更新HashMap: 

  * 首先判断Key是否存在:

    ```rust
    if hashmap.contains(&key) {
      // ...
    }
    else {
      
    }
    ```

  * 如果key存在:

    * 如果要用新值覆盖, 直接再次`insert`即可.

    * 如果要基于原来的值计算新值:

      ```rust
      // 得到原来的值的&mut
      let origin = hashmap.entry(&key).or_insert(0);
      // 然后用*origin就可以基于原值计算
      ```

  * 如果key不存在, 直接`insert`插入即可.


## 迭代器

* 一个类型, 如果实现了`Iterator` trait, 那么它就是一个迭代器类型.

* `Iterator` trait的内容如下:

  ```rust
  pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
  }
  ```

* 一个集合如果实现了`IntoIterator` trait, 那么它就能够通过`into_iter(), iter(), iter_mut()`方法变成迭代器:

  * `into_iter()`会拿走集合变量的所有权, 调用`next()`返回的类型是`Some(T)`.
  * `iter()`使用集合的不可变引用, 调用`next()`返回`Some(&T)`.
  * `iter_mut()`使用集合的可变引用, 调用`next()`返回`Some(&mut T)`, 可以通过它修改集合中的元素.

* **注意: **`next()`方法会改变迭代器的状态, 如果一个迭代器变量需要用到`next()`方法, 就需要`mut`.

  ```rust
  let a = vec![1, 2, 3];
  let b = a.iter();
  // 代码会报错, 需要将b定义成mut
  b.next().unwrap();
  ```




* 





## 裸指针

Rust中, 裸指针(raw pointer)分为一下两种类型:

* `*const T`: 不能通过裸指针修改原数据.
* `*mut T`: 可以通过裸指针修改原数据.

> 



## 单元测试

* 快速单元测试的框架, 在你写的函数里面用:

  ```rust
  #[cfg(test)]
  mod tests {
    
    use super::*;
    
    // 测试函数, 可以写多个
    #[test]
    fn test_XXX1() {
      assert_eq!(...);
    }
    
    #[test]
    fn test_XXX2() {
      assert_eq!(...);
    }
  }
  ```

  



## tokio



* 引入包:

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```



### 异步锁

* 默认情况下, `tokio`的任务调度器是**多线程**, 这样, 你就需要考虑一个问题:
  * 使用`.await`时, 任务可能会被阻塞, tokio可能会把任务调度到另一个线程中执行.
  * 这样, 与`.await`在同一个作用域的所有变量, 都会被封装成一个状态, 在线程间传递, 如果在线程间传递, 那么这些变量就需要实现`Send` trait.
  * 最典型的例子是, 如果你在`.await`之前, **在同一作用域内**用`std::sync::Mutex`获取了互斥锁, 由于`MutexGuard<T>`没有实现Send trait, 就会报错.
  * 一个解决方案是, 让`.await`调用的作用域在获取Mutex作用域之后, 另一个解决方案是使用**异步锁**.
* **异步锁: `tokio::sync::Mutex`**:
  * 如果在`.await`之间获取了`std::sync::Mutex`, 那么调用`.await`后, 任务会被阻塞, 但是锁没有被释放, 这时候如果另一个任务尝试获取锁, 就会产生死锁.
  * 使用tokio的异步锁可以在`.await`作用域内获取锁, 但是会有性能开销.



### 异步客户端

* **tokio实现异步客户端的思路: **
  * 首先, `spawn`一个manager异步任务, 这个任务负责管理所有客户端, 相当于客户端的proxy.
  * 然后, 在众多客户端和manager之间建立一个`tokio::sync::mpsc`.
    * 众多`tx`由众多客户端拿到, 客户端用`tx`给manager发送打包好的命令.
    * `rx`由manager拿到, 负责接收客户端发送来的命令.

  * 然后, 在客户端内部, 建立一个`tokio::sync::oneshot` (单生产者, 单消费者, 一次发送一个任务)
    * `oneshot_tx`会被客户端打包到命令中, 发送给manager, manager收到之后会通过`oneshot_tx`把服务器发送来的东西返回给客户端.
    * `oneshot_rx`给客户端用来接收manager返回过来的命令.

  * manager通过connection资源与服务器交互.



### tokio I/O


* tokio I/O的包主要在: `tokio::io`中, 和`std::io`的用法基本一致, 区别在于tokio中的I/O操作是异步的.
* 其中, 最重要的两个traits是`AsyncReadExt`和`AsyncWriteExt`, 实现了这两个trait可以使用异步的`read`和`write`.

* 异步echo服务端:

  ```rust
  use tokio::io;
  use tokio::net::TcpListener;
  
  #[tokio::main]
  async fn main() -> io::Result<()> {
  
      let listener = TcpListener::bind("127.0.0.1:6142").await?;
  
      loop {
          let (mut socket, _) = listener.accept().await?;
          tokio::spawn(async move {
              let (mut rd, mut wr) = socket.split();
              // 异步将reader的内容copy到writer中, copy以后writer就直接发送了
              if io::copy(&mut rd, &mut wr).await.is_err() {
                  eprintln!("failed to copy");
              }
          });
      }
      Ok(())
  }
  ```

* 异步echo客户端:

  ```rust
  use tokio::net::TcpStream;
  use tokio::io::{self, AsyncReadExt, AsyncWriteExt};
  
  #[tokio::main]
  async fn main() -> io::Result<()> {
  
      let socket = TcpStream::connect("127.0.0.1:6142").await?;
    	// 分离reader和writer, 适合同时实现了AsyncRead和AsyncWrite的对象
      let (mut rd, mut wr) = socket.split();
  
      tokio::spawn(async move {
          wr.write_all(b"hello\r\n") .await?;
          wr.write_all(b"hello\r\n").await?;
          Ok::<_, io::Error>(())
      });
  
      let mut buf = vec![0; 128];
      let n = rd.read(&mut buf).await?;
      println!("GOT {:?}", &buf[..n]);
      Ok(())
  }
  ```




## Pin/Unpin

在Rust中, 一个变量的内存地址是可能会被移动的. 这种移动会引发一些问题, 考虑以下几个场景:

* 场景1: **裸指针自引用结构体**

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



## Snippets/Library

* 生成`[a, b)`的随机整数:

  * 使用`rand crate`.

  ```rust
  use rand::Rng;
    
  let secret_number = rand::thread_rng().gen_range(a, b);
  ```
