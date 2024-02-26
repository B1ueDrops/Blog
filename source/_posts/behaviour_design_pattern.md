---
title: 行为型的设计模式
categories: 软件工程
---



## 行为型设计模式

行为型设计模式用来描述类/对象之间如何通过协作, 以耦合度最小的前提下完成任务.



## 策略模式

如果一个需求可以用非常多的算法实现, 那么就可以考虑策略模式.

实现方式:

* 抽象策略接口:

```java
interface Strategy {
  // 这里面定义了实现策略的方法
  public void strategyMethod();
}
```

* 具体策略类: 用于实现抽象策略接口.

```java
class ConcreteStrategy implements Strategy {
  
  public void strategyMethod() {
    System.out.println("haha");
  }
}
```

* 策略调用者:

```java
class Context {
  
  // 成员变量是策略的抽象类
  private Strategy strategy;
  
  // 设置具体策略
  public void setStrategy(Strategy strategy) {
    this.strategy = strategy;
  }
  
  // 调用策略方法
  public void strategyMethod() {
    this.strategy.strategyMethod();
  }
}
```

* 策略模式的使用方式:

```java
public static void main(String[] args) {
  
  Context context = new Context();
  // 利用多态选择一个策略
  Strategy strategy = new ConcreteStrategy();
  // 将选定的策略设置给Context
  context.setStrategy(strategy);
  // Context执行策略
  context.strategyMethod();
  // 之后换了策略, 只要给Context重新设置一个策略即可
}
```