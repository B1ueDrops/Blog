---
title: 创建型的设计模式
categories: 后端技术
---

[TOC]



## 创建型设计模式

创建型设计模式用于将对象的创建与使用解耦合.



## 单例模式

如果一个类只有一个实例, 那么可以使用单例模式创建对象.

单例模式有两种实现方式: 懒汉与饿汉.

* 懒汉: 也就是懒加载, 当使用对象时才进行创建.
  * 懒汉可以节约资源, 但是要处理多线程的同步问题.


```java
public class Singleton {
  // 实例
  private static Singleton instance;
  // 禁用构造方法
  private Singleton() {}
  // 获取实例
  // synchronized可以保证同一时刻只有一个线程执行getInstance()
  public static synchronized Singleton getInstance() {
    if (instance == null) {
      instance = new Singleton();
    }
  }
}
```

* 饿汉: 在类加载时就会创建实例对象.
  * 饿汉会浪费资源, 但是不要处理多线程同步


```java
public class Singleton {
  
  private static Singleton instance = new Singleton();
  
  private Singleton() {}
  
  public static Singleton getInstance() { return instance; }
}
```



> 懒汉模式的问题

https://blog.csdn.net/qq_40885085/article/details/103363774

## 原型模式

原型模式用于解决大量相同/相似的对象的创建问题.

实现方式: 

* 实现编程语言关于`clone`的接口, 然后调用`clone`方法进行复制.
* 注意: 原型模式有两种, 浅拷贝和深拷贝, 注意区分.



## 工厂方法模式

工厂方法模式有四个角色:

* 抽象产品: 提供化产品的接口.
* 具体产品: 实现了抽象产品的接口, 用于定义某一种具体产品.
* 抽象工厂: 实现了生产产品的接口.
* 具体工厂: 实现创建产品的接口, 用于创造某一种具体产品.

实现方式:

* 抽象产品:

```java
interface Product {
  // 产品的方法
  public void show();
}
```

* 抽象工厂:

```java
interface Factory {
  // 生产产品的方法
  public Product newProduct();
}
```

* 具体产品:

```java
class ConcreteProduct implements Product {
  
  public void show() { System.out.println("Product 1"); }
}
```

* 具体工厂:

```java
class ConcreteFactory implements Factory {
  public Product newProduct() {
    return new ConcreteProduct();
  }
}
```





## 抽象工厂模式

在工厂方法模式种, 一个工厂只能生产一种产品, 但是实际的工厂一般是综合性工厂, 也就是说很有可能一个工厂能够生产一个产品簇, 例如农场既可以生产动物, 也可以生产植物.

与工厂方法模式的区别:

* 在抽象工厂中, 增加了多个生产产品的方法.
* 一个抽象工厂可以对应多个抽象产品.

实现方式:

* 抽象产品:

```java
interface Product1 {
  public void show1();
}

interface Product2 {
  public void show2();
}
```

* 抽象工厂:

```java
interface Factory {
  
  public Product1 newProduct1();
  public Product2 newProduct2();
}
```

* 实际产品:

```java
class ConcreteProduct1 implements Product1 {
  public void show1() { System.out.println("ConcreteProduct1"); }
}

class ConcreteProduct2 implements Product2 {
  public void show2() { System.out.println("ConcreteProduct2"); }
}
```

* 实际工厂:

```java
class ConcreteFactory implements Factory {
  
  public Product1 newProduct1() { return new ConcreteProduct1(); }
  public Product2 newProduct2() { return new ConcreteProduct2(); }
}
```



## 建造者模式

如果你想创建的对象有这两个特点:

* 对象由多个部分组成, 各部分在将来可能会发生变化.
* 对象的构建方式相对固定.

那么就可以用建造者模式.

实现方式:

* 产品类: 产品的各个部分当作成员变量, 并且需要提供对应的setter方法.

```java
class Product {
  
  private String part1;
  private String part2;
  private String part3;
  
  public void setPart1(String part1) {
    this.part1 = part1;
  }
  
  public void setPart2(String part2) {
    this.part2 = part2;
  }
  
  public void setPart3(String part3) {
    this.part3 = part3;
  }
}
```

* 抽象建造者:

```java
abstract class Builder {
  
  // 产品
  protected Product product = new Product();
  // 抽象方法, 建造产品的部分
  public abstract void buildPart1();
  public abstract void buildPart2();
  public abstract void buildPart3();
  
  // 返回生产好的产品
  public Product getResult() { return product; }
}
```

* 具体建造者: 用于实现生产产品每个部分的函数.

```java
public class ConcreteBuilder extends Builder
{
    public void buildPart1()
    {
        product.setPartA("建造 Part1");
    }
    public void buildPart2()
    {
        product.setPartA("建造 Part2");
    }
    public void buildPart3()
    {
        product.setPartA("建造 Part3");
    }
}
```

* 指挥者: 调用建造者的函数完成生产任务.

```java
class Director {
  
  // 建造者
  private Builder builder;
  
  // 构造函数, 通过组合的方式传入建造者
  public Director(Builder builder) {
    this.builder = builder;
  }
  
  // 生产产品
  public Product construct() {
    builder.buildPart1();
    builder.buildPart2();
    builder.buildPart3();
    return builder.getResult();
  }
  
}
```

客户类:

```java
public class Client {
    public static void main(String[] args)
    {
        Builder builder=new ConcreteBuilder();
        Director director=new Director(builder);
        Product product=director.construct();
        product.show();
    }
}
```

通过客户类可以看出:

* 建造者可以很容易扩展(多态).
* 建造者和产品的生产, 通过Director进行解耦合, 只需要给Director传入不同的builder即可.
