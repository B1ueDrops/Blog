---
title: C++使用日记
categories: 编程语言
---





## 引用

* 引用就相当于对原始变量所有权的租借, 对引用的操作等价于对原变量的操作:

  ```cpp
  int num = 0;
  // 取num引用
  int &b = num;
  
  Student s(string("haha"), 1);
  Student &stu = s;
  // 用引用操作变量和用指针一样
  cout << s->print() << endl;
  ```

  * <font color=red>**引用变量必须初始化 ! 编译器会在编译层面保证这一点.**</font>



### 不可变引用

* 借助`const`可以实现不可变引用.

* 不可变引用: `const Type &`.

  ```cpp
  // 获取变量不可变引用
  int num = 0;
  const int &b = num;
  ```

* 可变引用: `Type &`.



### 常见问题

> 引用和指针的区别?

* 指针, 本质上是一种所有权的量化, 它代表所有权本身, 这个所有权会由一个变量承载.
* 引用, 本质上是对所有权的租借, 它可以让多个变量承载同一个所有权.

## 函数

### 函数重载

* 注意: 重载是C++的特性, 而不是C语言的特性.

* 函数重载:
  * 可以同时定义多个重名函数, 这些函数参数, 返回值类型可以不同.
  * 具体执行哪个函数是在编译时完成.

## 动态内存管理

> `malloc`和`new`的区别?

* `malloc`是C++标准库函数, `new`是C++的一个运算符.
* `malloc`需要手动计算内存大小, `new`可以自动计算.

* `malloc`返回`void *`类型的指针, 而`new`返回具体类型的指针.

* `malloc`不是类型安全的, 而`new`是类型安全的.
  * 假设你申请了一大块内存, 但是赋给了一个小变量(例如`int`类型), `new`会在编译时期报错, 而`malloc`不会.
* `new`由 `malloc`, 指针转换, 构造函数三部分组成.



> `malloc`申请的内存可以用`delete`释放吗? `new`申请的内存可以用`free`释放吗?

* 都不可以, 核心在于:
  * `malloc`不会调用构造函数, 而`new`会调用构造函数.
    * 如果类的构造函数中有动态申请内存, 那么就会出现问题.
  * `free`不会调用析构函数, 而`delete`会调用析构函数.
    * 如果类的析构函数中有释放成员变量占用的内存, 那么就会出现内存泄漏.



## 内存分区

* 由低地址向高地址为:
  * `.text`: 代码段.
  * `.data`: 初始化的全局变量与静态变量.
  * `.bss`: 全称为Block Started by Symbol, 包括未初始化的全局变量和静态变量.
  * `heap`: 由低向高延伸, 存储动态申请的内存.
  * `stack`: 由高向低延伸, 存储函数参数, 局部变量等栈帧信息. 

> 栈和堆的区别 (几个重要的点)?

* 栈是固定大小, 连续的内存区域 (不会产生碎片), 在CPU级别会提供栈的操作指令及相关寄存器. 栈由编译器进行管理.
* 堆是不固定大小, 不连续的内存区域 (OS链表管理, 会产生碎片), 堆的操作需要C++标准库提供, 堆由程序员管理.



## `inline`

* `inline`的作用: 在编译时, 将`inline`函数体直接替换到调用处, 而不是创建栈帧.

  ```cpp
  inline void call_function() {
    cout << "niubi" << endl;
  }
  
  int main() {
    call_function();
    return 0;
  }
  ```

* 优点:

  *  `inline`适用于体积较小的函数, 可以提高可读性.
  * 可以减少小函数创建/销毁栈帧的开销.

* 缺点:

  * 二进制文件体积较大 (每个调用都要重复复制代码).
  * 对I-Cache的利用率较低 (同一个I-Cache很有可能)
  * 调试时, 调用栈可能过于复杂.

## `struct`

* 定义结构体:

  ```cpp
  typedef struct Student {
    
    string name;
    int age;
    
  } Student;
  ```

* 例化结构体:

  ```cpp
  auto *stu = new Student {
    string("haha"),
    10
  };
  //...
  free(stu);
  ```

  * 注意, 由于C++本身的特性, 这里参数名称和值没法对应, 需要用IDE的inlay hints.

* 使用结构体变量:

  ```cpp
  cout << stu->name << endl;
  ```

* 结构体字段的权限:
  * 继承访问权: `public`.
  * 外界访问权: `public`.



> C语言的`struct`和C++的`struct`区别:

1. C++中的`struct`可以包含成员函数.
2. C++中的`struct`支持继承.
3. C++中的`struct`支持多态.



## `class`

### 基本使用

* 定义类成员变量/成员函数:

  * 在`.hpp`文件中定义类:

    ```cpp
    class Student {
    
    public:
      string name;
      int age;
    
      // 构造函数
      Student(string name, int age): name(std::move(name)), age(age) {};
    
      void print() const;
    };
    ```

  * 在`.cpp`文件中实现类中的成员函数:

    ```cpp
    void Student::print() const {
      cout << this->name << endl;
    }
    ```

  * <font color=red>注意: 成员函数分为两类:</font>

    * 不修改成员变量: 在函数后面加上`const`, 例如: `void print() const;`
    * 修改成员变量: 不用加`const`.

* 创建类的对象:

  ```cpp
  auto *stu = new Student(string("haha"), 10);
  
  free(stu);
  ```

* 使用类的成员变量/函数:

  ```cpp
  cout << stu->name << endl;
  stu->print();
  ```



### 成员权限

* C++中成员变量/成员函数的三种权限:
  * `private`: 类内成员函数可访问, 子类, 类外不可访问.
  * `protected`: 类内成员函数, 子类可访问, 类外不可访问.
  * `public`: 类内成员函数, 子类, 类外都可访问.
  * 注意: 友元类可以无视权限, 直接访问到成员变量/函数.




### 构造函数

* C++中, 如果提供了有参数的构造函数, 则不提供默认构造函数, 如果非要提供, 需要用`default`关键字:

  ```cpp
  class Student {
    public:
    	Student();
    	Student(string name, int age): name(std::move(name)), age(age) {};
  }
  
  Student::Student() = default;
  ```

* C++的类中存在拷贝构造函数(copy constructor), 签名如下:

  ```cpp
  Student(const Student &stu);
  ```

  * 当需要对C++的非基本类型进行拷贝时, 就会自动调用拷贝构造函数.

  * 如果没有显式重载拷贝构造函数, 编译器会提供默认实现, 递归调用非静态的成员变量.

  * **调用拷贝构造函数的场景:**

    * 函数参数值传递.
    * 函数返回值.
    * `Student stu(stu2);`
    * `Student stu = stu2;`这种赋值操作, **注意, `stu = stu2`是调用的运算符**`=`.
    
  * C++类中的运算符`=`的签名为:
  
    ```cpp
    Student &operator=(Student &stu);
    ```
    
    * `=`的默认实现和拷贝构造函数一致.



### 析构函数

* 析构函数: 当对类的对象调用`delete`之前, 会首先调用析构函数:

  ```cpp
  class Student {
    
    // ...
    public:
    	char name[4];
    
    	~Student() {
        delete [] name;
      }
  }
  ```

  * 使用场景: 如果类的成员变量是动态内存申请得到的, 那么就需要在析构函数中手动`delete`.





### 运算符重载

* 重载的模板如下:

  ```cpp
  class Student {
    
    public:
    	string name;
    	int age;
    	
    	// 重载+
    	Student operator+(const Student &stu);
    	Student operator+(int age);
    	// 为了实现交换律, 需要用友元函数重载
    	friend Student operator+(int age, const Student &stu);
    
    	// 实现Java中的toString()
    	friend ostream& operator<<(ostream &out, const Student &stu);
  };
  
  // 在.cpp中实现
  Student Student::operator+(const Student &stu) {
    Student stu;
    stu.name = this->name;
    stu.age = this->age + stu->age;
    return stu;
  }
  
  Student Student::operator+(int age) {
  
  }
  
  // 友元函数
  friend Student operator+(int age, const Student &stu) {
    // ...
  }
  
  // toString
  friend ostream& operator<<(ostream & out, const Student &stu) {
    out << "这里自己格式化, 可以访问stu的所有变量";
    return out;
  }
  ```

  



### 常见问题

> C++中`struct`和`class`的区别?

* `struct`中成员变量/函数默认权限为`public`.
* `class`中成员变量/函数默认权限为`private`.



> 什么时候需要重载copy constructor?

* 当类的成员变量有指针的时候.



## `CMake`

### `compile_commands`

* `CMakeLists.txt`中添加这个可以生成`compile_commands.json`, 用来给IDE语法解析用.

  ```cmake
  set(CMAKE_EXPORT_COMPILE_COMMANDS 1)
  ```
