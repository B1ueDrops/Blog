---
title: Rust系统总结
categories: 编程语言
mathjax: true
---



## 常用Snippets

* 只遍历元素:

  ```rust
  // item不是引用,此处会发生所有权转移
  for item in arr {
    
  }
  // item是不可变引用
  for item in &arr {
    
  }
  // item是可变引用
  for item in &mut arr {
    
  }
  ```
  
* 同时遍历下标和元素:

  ```rust
  // 注意这里的i类型是usize
  // 注意这里的v是不可变引用
  // a可以是vec
  for (i, v) in a.iter().enumerate() {
          println!("第{}个元素是{}", i + 1, v);
  }
  ```
  
* 遍历String的每一个字符:

  ```rust
  for (i, c) in s.chars().enumerate() {
    // c是char类型
  }
  ```

* 找String的第`i`个字符: 

  ```rust
  // x是char
  let x = s.chars().nth(i).unwrap();
  ```
* 通过循环修改原数组
  ```rust
  // 因为要对原数组进行修改, 所以数组定义成可变mut
  let mut arr = vec![1, 2, 3, 4];
  // 对数组进行可变的借用, item类型是&mut i32
  for item in &mut arr {
      *item = *item + 1;
  }
  ```
* 把一个数组中的内容放到另一个数组:
  ```rust
  let a = vec![
      String::from("1"),
      String::from("2"),
      String::from("3")
  ];
  
  let mut b: Vec<String> = vec![];
  
  // to_owned方法可以给引用调, 用来拷贝引用指向的对象, 并且返回值是拷贝后的对象, 而不是引用
  for item in &a {
      println!("{}", item.to_owned());
    	b.push(iten.to_owned());
  }
  ```
  
  

## 变量



### 栈/堆

* 基本类型的值存储在线程的栈上, 因为基本类型大小固定, 并且栈的存储/访问速度一般快.
  * 基本类型有: 整数, 浮点, 字符, 以及仅包含它们的元组.

* 非基本类型存储在堆上, 因为非基本类型大小有可能在编译时期不固定, 在栈中会存储数据在堆中地址.



### 可变/不可变

* 对于一个变量, 如果你要修改它的值, 那么它就是可变变量, 需要用`mut`.
  * 例如: 数组中修改元素的值, 结构体中修改成员变量的值.




### 所有权

* 所有权是指**数据的所有权**.
* 所有权的核心: **对于一个数据, 仅可能通过一个变量去访问, 也就是一个存储在内存中的值, 有且仅有一个ownership**.
  * 如果出现赋值语句, 函数参数传递, 或者函数返回值返回, 都会发生所有权的转移, 原变量会失效.
  * 但是对于实现了Copy trait的类型来说, 如果出现上述情况, **不会发生所有权转移**, 实现Copy trait的类型有:
    * 基本类型.
    * 基本类型组成的元组.
    * **不可变引用`&T`**.
* 可以使用`clone()`函数对堆上的数据进行**深拷贝**, 但是`clone()`会极大拖慢程序性能.



### 引用

* 引用是对所有权的**租借(borrow)**, 也是实现**浅拷贝**的方法.

  * 无论你如何使用引用, 都不会改变原变量的所有权, 只有使用权.

* 引用是个变量, 本身具有可变/不可变两种类型:

  * 可变引用: 这个引用变量可以租借其他变量所有权.
  * 不可变引用: 这个引用变量只能租借一个变量的所有权.

* 引用有两种内置类型, 注意这里的类型与上面描述不同, type的描述词是`&T`和`&mut T`:

  * `&T`: 不可以通过修改引用来改变原变量的值.
  * `&mut T`: 可以通过修改引用修改原变量的值.
    * 原变量必须是可变变量(有`mut`).


  * **租借规则:**

    * 对于一个变量, 在同一作用域内, 允许有多个`&T`引用, 但是**只能有一个`&mut T`引用.**
    * 对于一个变量, 在同一作用域内, 不允许同时出现`&T`和`&mut T`引用.

  * 引用本质上也是一种**非基本类型**, 并且**实现了Deref trait**, 因此能和普通变量一样使用, 不需要显式解引用. 

    * 对一个引用调用`clone()`, 本质上是克隆的原始变量.



### 生命周期

* 变量在定义时, 生命周期就会固定.
* **堆上的数据是否被释放, 取决于它的所有权在哪个变量**, 变量生命周期过了就会被释放.

```rust
fn main() {
    let b;
    {
        let a = String::from("haha");
        // String::from本来在这个生命周期会被释放, 但是值的所有权转移后, 就不会被释放.
        b = a;
    }
    println!("{}", b);
}
```

* **最基本的生命周期约束:** 使用引用租借一个变量的所有权时, 需要保证**引用指向的原变量的生命周期, 小于等于依赖引用指向的所有变量的生命周期**.



## 函数



### 生命周期约束

* 可以对引用进行生命周期的标注, 例如: `first: &'a i32`:

  * 标注的意思是: **这个引用所指向的原变量的生命周期是`a`**.

* **函数的生命周期约束: **如果函数返回值是**引用**, 那么**返回值引用指向的原变量**的生命周期, 要小于等于所有参数生命周期的最小值.

  * 函数如果返回引用, 那么这个引用只有两个来源:
    * 函数参数.
    * 函数内部新的变量的引用, 这种情况会出现Dangling Reference, 需要返回原变量, 让其发生所有权转移.

* **Rust消除生命周期标注的规则:**

  * 对于输入的每个引用, 都会分配一个生命周期标注:

    ```rust
    // 分配前
    fn foo(x: &i32, y: &i32);
    // 分配后
    fn foo<'a, 'b>(x: &'a i32, y: &'b i32);
    ```

  * 如果只有一个输入生命周期`a`, 并且函数返回值是引用, 那么返回值引用的生命周期也会被标注为`a`.

  * 如果存在多个输入生命周期, 但是其中有一个引用是`&self`或者是`&mut self`, 并且函数返回值是引用, 那么返回值引用的生命周期会和`&self/&mut self`标注的一样.

* 如果实际情况不符合生命周期标注规则, 那么就需要**手动标注**, 否则编译不通过:

  * 最典型情况: 函数参数有多个引用类型, 并且返回值也是引用类型.

    * 为什么这种情况需要标注?

      * 因为返回值引用必然依赖于函数参数引用, 但是可能函数在运行时才能决定依赖于那些参数引用, 例如这个例子:

        ```rust
        // 函数用来返回长度较长的字符串的引用, 具体返回哪个参数取决于运行时
        fn longest(x: &str, y: &str) -> &str {
            if x.len() > y.len() {
                x
            } else {
                y
            }
        }
        ```

  * 怎么标注?

    ```rust
    fn longest<'a>(x: &'a str, y: &'a &str) -> &'a str {
      	if x.len() > y.len() {
            x
        } else {
            y
        }
    }
    ```

    * 表示返回值的引用指向的原变量的生命周期, 和函数参数`x, y`具有相同的生命周期, 这才能通过编译.

* **最长的生命周期:** `'static`, 这个生命周期能和程序活得一样久.

  * 字符串字面值: `let s: &'static str = "我没啥优点，就是活得久，嘿嘿";`



## 面向对象



### 结构体

* 定义语法:

  ```rust
  // 最后没有;
  struct User {
      active: bool,
      username: String,
      email: String,
      sign_in_count: u64
  }
  ```




### 方法

* 方法的定义语法:

  ```rust
  struct Circle {
      x: f64,
      y: f64,
      radius: f64,
  }
  // 同样没有;
  impl Circle {
    // 调用: Circle::new
      fn new(x: f64, y: f64, radius: f64) -> Circle {
          Circle {
              x: x,
              y: y,
              radius: radius,
          }
      }
      fn area(&self) -> f64 {
          std::f64::consts::PI * (self.radius * self.radius)
      }
  }
  ```

* 方法有三种类型:

  * **关联方法(associated method): **参数中不含任何`self`, 可以用来写构造方法.
  * 参数带`&self`的方法: 不可以修改成员变量.
  * 参数带`&mut self`的方法: 可以修改成员变量.

* **构造方法: **一般用`new`当作方法名, Rust中`new`不是关键字.

  ```rust
  // Self代指原结构体类型
  pub fn new(width: u32, height: u32) -> Self {
          Self { width, height }
  }
  ```

* **Getter方法:** 方法名称和私有成员变量同名.

  ```rust
  pub fn width(&self) -> u32 {
      return self.width;
  }
  ```



### 泛型

* 语法:

  ```rust
  // 函数
  fn largest<T>(list: &[T]) -> T {
  }
  
  // 结构体
  struct Point<T,U> {
      x: T,
      y: U,
  }
  // 方法
  impl<T> Point<T> {
      fn x(&self) -> &T {
          &self.x
      }
  }
  // 针对具体类型实现方法
  impl Point<f32> {
  
  }
  // trait
  pub trait Iterator<T> {
      fn next(&mut self) -> Option<T>;
  }
  ```

  



### trait

* trait类似于接口, 用来把行为标准和具体实现解耦, 减少冗余代码.

* 语法:

  ```rust
  pub trait Summary {
    fn summarize(&self) -> String;
  }
  
  pub struct Student {
    //...
  }
  // impl <trait> for <struct>
  impl Summary for Student {
    fn summarize(&self) -> String {
      // ...
    }
  }
  ```

* trait中也可以定义默认方法(与接口不同之处), 实现trait的人可以选择重载.

* **孤儿规则(orphan rule, OR): **

  * 如果要实现外部定义的trait, 需要将其导入作用域.
  * 可以对自定义类型实现外部trait.
  * 可以对外部类型实现当前作用域自定义的trait.
  * 不允许对外部类型实现外部trait.
  * **一句话概括: **如果要为类型A实现trait T, 那么A和T的定义至少要有一个在当前作用域.

* **特征约束: **

  * 场景: 你传入了一个泛型, 你要求这个泛型必须实现某几个trait.

  * 语法: 

    ```rust
    // T: Summary表示T这个泛型必须实现Summary trait
    pub fn notify<T: Summary>(item: &T) {
        
    }
    // 表示T必须同时实现Summary和Display
    pub fn notify<T: Summary + Display>(item: &T) {}
    
    // where防止特征约束过于复杂
    fn some_function<T, U>(t: &T, u: &U) -> i32
        where T: Display + Clone,
              U: Clone + Debug
    {}
    ```

  * 语法糖: `impl <trait>`表示实现了`trait`特征的类型

    ```rust
    // impl Summary表示所有实现了Summary trait的结构体
    pub fn notify(item: &impl Summary) {
    
    }
    pub fn notify(item: &(impl Summary + Display)) {}
    ```

  * **语法糖的限制:** 返回值约束

    * 这样写是正确的, 但是, Rust要求函数的返回值只能是一种类型, 假设类型`A`和类型`B`都实现了Summary trait, 你写的函数既可能返回`A`, 又可能返回`B`, 这种是不允许的, 因为返回值要求只能返回一种类型.

    ```rust
    fn returns_summarizable() -> impl Summary {
        // ...
    }
    ```

* **特征对象:**

  * 场景: 假设你有一个数组, 这个数组中需要存储所有实现了某个trait的对象.

  * 语法: 使用`&dyn trait`或者`Box<dyn trait>`表示实现了trait的对象, 这个类型本质上是引用类型.

    ```rust
    pub struct Screen {
      // components存储所有实现了Draw trait的对象的引用
        pub components: Vec<Box<dyn Draw>>,
    }
    ```

  * **为什么不定义成`dyn trait` ?**

    * 因为特征对象本质上是**动态类型**, 编译时不知道大小, 而Rust要求对象在编译时知道大小.
    * 但是`&dyn trait`和`Box<dyn trait>`是在编译时就知道大小的.

  * **特征对象的限制:**

    * 并不是所有的trait都能有特征对象, 只有对象安全的trait才可以有.
  * 对象安全的定义:
    
    * trait中方法中没有泛型参数.
    
    * trait方法中返回值没有`Self`.
  
* **特征继承(super trait)**

  * 场景: 有时候在写一个trait时, 需要继承其他trait的功能才能完成这个trait.

  * 语法:

    ```rust
    // OutlinePrint: Display表示, 要实现OutlinePrint这个trait, 你还得先实现Display
    trait OutlinePrint: Display {
    }
    // 同时继承多个trait
    pub trait CacheableItem: Clone + Default + fmt::Debug + Decodable + Encodable {
    }
    ```

* **关联类型(associated type)**

  * 场景: 在trait中想使用泛型的场景都可以用关联类型代替, 这样就不用再使用泛型时都带上`<T>`, 提高可读性

  * 语法:

    ```rust
    // 泛型
    pub trait Iterator<T> {
        fn next(&mut self) -> Option<T>;
    }
    // 实现trait
    impl<T> Iterator<T> for Counter <T> {
      fn next<T>(&mut self) -> Option<T> {
      }
    }
    
    // 关联类型
    pub trait Iterator {
      type Item;
      fn next(&mut self) -> Option<Self::Item>;
    }
    // 实现trait, 这时候就不用写
    impl Iterator for Counter {
        type Item = u32;
        fn next(&mut self) -> Option<Self::Item> {
        }
    }
    ```
  
* **完全限定语法:**

  * 场景: 为一个结构体实现了多个trait, 但是多个trait中有方法重名, 怎么告诉编译器调哪一个?

  * 语法:

    ```rust
    <Type as Trait>::function(receiver_if_method, next_arg, ...);
    
    trait Animal {
        fn baby_name() -> String;
      	fn fly(&self);
    }
    
    struct Dog;
    
    impl Dog {
      fn baby_name() -> String {
          String::from("Spot")
      }
      fn fly(&self) {   
      }
    }
    
    impl Animal for Dog {
      fn baby_name() -> String {
          String::from("puppy")
      }
      fn fly(&self) {
        
      }
    }
    
    fn main() {
      // 调Animal Trait中的baby_name方法
        println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
      let a = Dog {};
      // 默认调用impl Dog中的fly方法
      a.fly();
      // 如果要调用Animal中的fly
      <Dog as Animal>::fly(&a);
    }
    ```
  
    
  



### 生命周期约束

* **结构体的生命周期约束:** 结构体的成员变量如果存在引用, 那么结构体的生命周期一定要小于等于引用成员变量所指向的原变量的生命周期.

* **生命周期标注语法:**

  ```rust
  struct ImportantExcept<'a> {
      part: &'a str,
  }
  
  impl<'a> ImportantExcerpt<'a> {
      fn level(&self) -> i32 {
          3
      }
  }
  ```

* **生命周期约束语法:** `'a:'b`表示`'b`这个生命周期小于`'a`这个生命周期.

  ```rust
  // 假设有这样一个方法
  impl<'a: 'b, 'b> ImportantExcept<'a> {
      fn announce_and_return_part(&'a self, announcement: &'b str) -> &'b str {
          println!("Attention please: {}", announcement);
          self.part
      }
  }
  ```

* 生命周期标注如果和泛型出现在一起, 可以同时写到`<>`中: `fn longest_with_an_announcement<'a, T>`



## 软件工程



### 权限

* Rust中, 很多元素都有权限, 这个权限是相对于其他mod说明的:
  * `pub`权限: 可以在其他mod中导入元素.
  * 不写`pub`: 就是private权限, 只可以在当前mod中使用权限.



### 多文件编程

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



### 集合类型



#### Vector

* 创建Vector:

  ```rust
  let vec: Vec<i32> = Vec::new();
  let vec: Vec<i32> = Vec::from([1, 2, 3]);
  ```

* 插入元素: `push`

  ```rust
  vec.push(1);
  ```

  * **<font color=red>注意, `push`会对`vec`进行可变借用, 如果前面存在不可变借用, 那么就会报错:</font>**

    ```rust
    // 报错代码
    let mut v = vec![1, 2, 3, 4];
    let first = &v[0]; // 对v的不可变借用
    v.push(6); // 对v的可变借用
    ```

    * 由于Vector是变长数组, 如果你`push`, 会有可能导致扩容, 扩容后, Rust会新分配一块内存, 然后把原来的数据拷贝, 那么之前的不可变引用就会报错.

* 索引元素:

  * 第一种, 用下标, `vec[2]`, 但是如果越界, 程序会发生`panic`.
  * 第二种, 用`vec.get(2)`, 返回`Option<&>`


#### HashMap

* 导入包:

  ```rust
  use std::collections::HashMap;
  ```

  

* 创建HashMap:

  ```rust
  let hash = HashMap::new()
  ```

* 插入值:

  ```rust
  // 注意, 这里x, y会发生所有权转移
  // 如果插入引用, 需要保证HashMap的生命周期小于等于插入的引用的生命周期
  hash.insert(x, y)
  ```
  
* 根据key判断是否存在键值对:

  ```rust
  // 传入的Key是&
  // 返回值是Option<&>
  hash.get(<你的key>)
  
  // x是原值
  if let Some(&x) = hash.get(&y) {
     
  }
  // x是&
  if let Some(x) = hash.get(&y) {
    
  }
  ```

* 遍历key, value:

  ```rust
  // key, value不是&
  for (key, value) in scores {
      println!("{}: {}", key, value);
  }
  
  // key, value是&
  for (key, value) in &scores {
      println!("{}: {}", key, value);
  }
  
  // key是&, value是&mut, 可以修改原值
  for (key, value) in &mut scores {
      println!("{}: {}", key, value);
  }
  ```

* 查询已有值, 若不存在则插入新值:

  ```rust
  // x, y都是值, 不是&, 返回&mut, 可以根据z修改哈希表中的值
  let z = scores.entry(x).or_insert(y);
  *z = *z + 1;
  ```





### 函数式编程



#### 闭包

* 闭包是一种匿名函数, 可以赋值给变量, 也可以捕获闭包之外的作用域中的值.

* 语法:

  ```rust
  let a = |x: i32, y: i32| -> i32 {
    	x + y
  };
  ```

* **闭包的一个用法: 简化冗余代码.**

  * 假设你有一个目的, 有很多种方法实现, 目的留出接口, 那么实现就可以用不同的闭包.

* 闭包可以进行自动类型推导, 但是函数必须要标注类型.

* **闭包作为结构体成员变量:**

  ```rust
  struct Cacher<T> where T: Fn(u32) -> u32 {
    query: T,
    value: Option<u32>
  }
  // 调用闭包
  (self.query)(args);
  ```

  * 其中, `Fn(u32) -> u32`是一个特征用来规范闭包类型`T`.

* **闭包作为函数参数:**

  ```rust
  fn some_function<F>(func: F) where F: Fn(usize) -> bool {
    
  }
  ```

* **闭包的特征, 以及所有权问题:**

  * 闭包能够捕获闭包作用域之外的值, 但是这个捕获就涉及所有权转移的问题.

  * **特征1: `FnOnce`**: 实现这个trait的闭包, 会把参数/里面捕获的变量的所有权拿走, 因此只能运行一次.

    ```rust
    fn some_function<F>(func: F) where F: FnOnce(usize) -> bool, {
      func(3);
      // 调用第二次时, x的所有权已经被第一个闭包拿走, 第二个闭包没权利用x
      func(4);
    }
    
    fn main() {
      let x = vec![1, 2, 3];
     	let closure = |z| { z == x.len() };
      fn_once(closure);
    }
    ```

  * **`FnOnce`加`Copy`**: 如果闭包trait实现了Copy trait, 那么在使用时, 就会调用闭包的拷贝, 可以运行多次.

    * 注意, 如果一个闭包实现了Copy trait, 那么不管其中捕获的变量是否实现Copy trait, 都没有关系, 都会调用闭包的拷贝.

    ```rust
    fn some_function<F>(func: F) where F: FnOnce(usize) -> bool + Copy, {
      func(3);
      // 调用第二次时, x的所有权已经被第一个闭包拿走, 第二个闭包没权利用x
      func(4);
    }
    
    fn main() {
      let x = vec![1, 2, 3];
     	let closure = |z| { z == x.len() };
      fn_once(closure);
    }
    ```

    * **如何判断一个闭包是否实现了`Copy`特征?**
      * 如果一个闭包捕获的所有变量的类型都实现了`Copy`特征, 那么这个闭包就会自动实现Copy特征.

  * **特征2: `FnMut`**: 以可变引用`&mut`的方式捕获

    * 只要闭包声明时带了`mut`, 那么这个闭包就默认实现了`FnMut`.

      ```rust
      fn main() {
          let mut s = String::new();
          let mut update_string = |str| s.push_str(str);
      }
      ```

  * **特征3: `Fn`**: 以不可变借用`&`的方式捕获:

    ```rust
    fn main() {
        let mut s = String::new();
        let update_string =  |str| s.push_str(str);
    }
    ```

  * **闭包实现特征的规律:**

    * 一个闭包可以同时实现多个特征.
    * 所有闭包默认实现`FnOnce`, 保证一个闭包至少可以被调用一次.
    * 如果闭包内没有对捕获变量进行改变, 那么实现了`Fn`特征.
    * 如果闭包没有把捕获变量的所有权返回, 那么实现了`FnMut`特征.

* **`move`关键字:** 定义在参数列表`||`之前, 表示闭包会强制把捕获变量的所有权拿走, 适用于**闭包生命周期大于捕获变量生命周期**.



#### 迭代器

* Rust中迭代器是懒加载的, 只创建不使用迭代器不会产生任何性能损失.

  ```rust
  let v = vec![1, 2, 3];
  // 将数组转化为迭代器
  let v_iter = v.iter();
  // 使用迭代器遍历
  for item in v_iter {
    //...
  }
  ```

* 所有迭代器默认实现了`Iterator` trait

  ```rust
  pub trait Iterator {
    type Item;
    // next方法返回下一个元素, 并且会消耗掉, 没有元素后就会返回None
    fn next(&mut self) -> Option<Self::Item>;
  }
  ```

  * 还有一个trait叫`IntoIterator`, 如果一个类型实现了这个trait, 那么它就可以通过`iter()/into_iter()`变成迭代器.

* 迭代器和所有权问题: 将数组转换为迭代器时, 有可能会把数组中元素的所有权给拿走, 具体分为以下三种情况:

  * 不可变借用: `iter()`

    ```rust
    let values = vec![1, 2, 3];
    for v in values.iter() {
      // v的类型是&T
    }
    ```

  * 可变借用: `iter_mut()`

    ```rust
    // 原数组必须可变
    let mut values = vec![1, 2, 3];
    for item in a.iter_mut() {
      // item类型是&mut T, 可以修改原数组元素
    }
    ```

  * 会夺走元素所有权: `into_iter()`

    ```rust
    let values = vec![1, 2, 3];
    for item in a.into_iter() {
      // item的类型是T
    }
    ```

* **迭代器适配器/迭代器消费者:**

  * 迭代器适配器非常强大, 可以把一个迭代器转换成你想要的迭代器, 然后配合迭代器消费者, 可以将一个数组转换成你想要的数组.

  * `map + collect` : `map`方法可以传入一个闭包, 对原数组进行操作, 然后用迭代器消费者`collect()`转换成数组.

    * 注意, 使用`collect()`必须进行类型标注, `_`表示让编译器进行标注.

    ```rust
    let v = vec![1, 2, 3];
    // 把v中的元素加1, 放到新数组中, 对原数组所有权没有影响
    let v1: Vec<_> = v.iter().map(|x| x + 1).collect();
    ```

  * `filter + collect`: `filter`方法可以传入一个返回bool的闭包, 如果返回值是true, 那么会在原数组中保留, 否则删除.

    ```rust
    let v = vec![1, 2, 3];
    // 筛选原数组中的偶数, 注意v2中的元素是&i32类型
    let v2: Vec<&i32> = v.iter().filter(|x| **x % 2 == 0).collect();
    ```

    



### 异常处理



#### Option

* 如果一个值的类型是`Option<T>`, 说明这个值可能是`None`.

* `Option<T>`的定义:

  ```rust
  enum Option<T> {
    Some(T),
    None
  }
  ```

* 构造`Option<T>`

  ```rust
  let value: Option<i32> = Some(5);
  let non_value: Option<i32> = None;
  ```

* 拆解`Option<T>`

  * 如果确定不会是None, 可以用`unwrap()`, 如果出现None, 程序就会`panic`.

  * 用match表达式:

    ```rust
    match some_value {
        Some(value) => println!("The value is {}", value),
        None => println!("There is no value"),
    }
    ```

  * 用`if let`表达式:

    ```rust
    let value: Option<i32> = Some(5);
    
    if let Some(value) = some_value {
        println!("The value is {}", value);
    } else {
      // None
        println!("There is no value");
    }
    ```

* 判断是否有值/None:

  ```rust
  let value1: Option<i32> = Some(5);
  let value2: Option<i32> = None;
  
  println!("some_value is {}", value1.is_some());
  println!("none_value is {}", value2.is_none());
  ```




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



### 单元测试

rust TDD (test driven development)的一般流程是:

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

## 高级特性

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

Rc\<T\>可以在单线程环境下, 创建一个数据的多个**不可变引用**, 并且提供了如下功能:

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
  	let c = Rc::clone(&a);
  
		// 用Rc::strong_count输出变量的计数
    assert_eq!(3, Rc::strong_count(&a));
    assert_eq!(Rc::strong_count(&a), Rc::strong_count(&b));
}
```



#### Arc\<T\>

Arc\<T\>可以在多线程环境下, 实现多个线程拥有同一个数据的多个不可变引用, 和`Rc<T>`的API完全相同.

* 和`Rc<T>`不同之处在于, `Arc<T>`采用原子指令维护了引用计数, 会有一定的性能损耗.



#### RefCell\<T\>

* RefCell\<T\>提供了变量的内部可变性, 也就是**原变量不可变的前提下, 你还可以创建可变引用**.

  ```rust
  // val的类型是RefCell<i32>
  let val = RefCell::new(10);
  
  // 可变引用修改
  *val.borrow_mut() += 1;
  
  println!("{}", *val.borrow());
  ```

* 但是, `RefCell<T>`仍然要遵守rust的借用规则, 例如不能让可变引用和不可变引用共存.

  ```rust
  // 报错, 同时有了可变借用和不可变借用
  let a = RefCell::new(10);
  let b = a.borrow();
  let c = a.borrow_mut();
  println!("{}, {}", b, c);
  ```

* 一个例子: 假设你要为自己的类型实现trait, 但是trait中方法签名是`&self`, 但是你就想在这个方法中修改成员变量, 这个时候需要将成员变量定义成`RefCell<T>`类型.

* `Rc + Refcell`: Rc可以实现一个数据有多个不可变引用, 结合`Refcell`可以通过不可变引用修改数据.

  ```rust
  let a = Rc::new(RefCell::new(10));
  
  let a1 = Rc::clone(&a);
  let a2 = Rc::clone(&a);
  
  *a1.borrow_mut() += 1;
  *a2.borrow_mut() += 2;
  
  println!("{}", a.borrow()); //13
  ```
