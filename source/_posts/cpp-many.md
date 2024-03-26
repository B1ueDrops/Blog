---
title: C++之用到什么学什么
categories: 编程语言
---



## CMake

* `CMakeLists.txt`中添加这个可以生成`compile_commands.json`, 用来给IDE语法解析用.

  ```cmake
  set(CMAKE_EXPORT_COMPILE_COMMANDS 1)
  ```


* 让你的CMake从你给的路径中找链接库, 可以把链接库的绝对路径加到`CMAKE_PREFIX_PATH`中:

  ```cmake
  list(APPEND CMAKE_PREFIX_PATH "你的路径")
  ```

* `CMAKE_CURRENT_SOURCE_DIR`表示`CMakeLists.txt`所在路径.

* **分模块的`CMakeLists.txt`:**

  * 用`add_subdirectory(路径)`可以执行别的路径下的CMake.



## C++一些基础





## C++面向对象

1. 成员变量和成员方法有三种权限修饰符:

   * Class中, 不写默认是`private`.
   * `private`只能在类定义/友元中被访问.
   * `protected`还可以在子类中访问.
   * `public`可以在其他地方访问.

2. 析构函数的作用:

   * 在销毁对象的时候, 需要将对象占用的内存, 文件资源释放.

3. `delete`和`delete[]`的区别:

   * `delete[]`会把数组中的所有元素的析构函数调一遍, 但是`delete`只会调第一个元素的析构函数.
   * `new`和`delete`成对用, `new []`和`delete[]`成对用.

4. 注意, 关于`this`指针, 有时候在没有歧义的情况下`this`可以省略, 读别人代码时注意以下.

5. `const`成员函数:

   * 定义如下: `const`放在了后面.

     ```cpp
     public:
     	int getBorn() const {
         //...
       }
     ```

   * 如果这样写, 说明<font color=blue>这个成员函数不能修改成员变量.</font>

6. `static`的理解:

   * 修饰成员变量: 成员变量和类绑定.
   * 修饰成员函数: 成员函数只能修改静态变量, 不能修改成员变量.

7. 虚函数快速理解:

   * C++多态, 当父类指针指向子类对象时, 如果调用一个函数, 默认调用父类的函数.

     * 但是如果父类函数声明时用了`virtual`, 那么就调子类的.
     * 相当于一种动态运行策略, 如果一个函数是虚函数, 那么在运行时决定使用哪个函数.

   * 用纯虚函数可以把C++的类变成抽象类:

     ```cpp
     class Person
     {
       public:
       	string name;
       // 纯虚函数
       	virtual void print() = 0;
     }
     ```

   * 一个实用的例子: <font color=red>如果定义一个抽象类, 那么析构函数就是虚函数</font>.

8. **`explicit`关键字的作用:**

9. **构造函数`=default`和`=delete`的用法:**



## C++一些高级用法

