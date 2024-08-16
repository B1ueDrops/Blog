---
title: 行为型的设计模式
categories: 后端技术
---

[TOC]



## 行为型设计模式

行为型设计模式用来描述类/对象之间如何通过协作, 以耦合度最小的前提下完成任务.



## 策略模式

如果一个需求可以用非常多的代替算法实现, 那么就可以考虑策略模式.

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



## 观察者模式

如果当一个对象的状态发生改变时, 依赖这个对象的所有对象都需要发生变化, 就可以考虑使用观察者模式.

实现方式:

* 抽象观察者接口:

```java
interface Observer {
  // response是抽象观察者更新自己的方法
  // response也可以接收Subject中的参数
  void response();
}
```

* 具体观察者:

```java
class ConcreteObserver implements Observer {
  public void response() {
    System.out.println("更新自己");
  }
}
```

* 观察者依赖的对象:

```java
abstract class Subject {
  
  // 依赖这个对象的观察者们
  protected List<Observer> observers = new ArrayList<Observer>();
  
  // 添加一个观察者
  public void add(Observer observer) {
    this.observers.add(observer);
  }
  
  // 删除观察者
  public void remove(Observer observer) {
    observers.remove(observer);
  }
  
  // 通知观察者的方法
  public abstract void notifyObserver();
}
```

* 实现`Subject`:

```java
class ConcreteSubject extends Subject {
  
  public void notifyObserver() {
    for (Object obs: observers) {
      obs.response();
    }
  }
}
```



## 模板方法模式

* 适用场景:
  * 一个算法有若干个小算法, 按照顺序组合而成.
  * 这些小算法的实现不固定.

实现方式:

* 模板方法抽象类:

```java
abstract class TemplateMethodClass {
  
  public void specificMethod() {
    // 已经可以确定下来的具体方法
  }
  // 不确定的两个抽象方法
  public abstract void abstractMethod1();
  public abstract void abstractMethod2();
  
  // 整个大算法的实现组成
  public void templateMethod() {
    specificMethod();
    abstractMethod1();
    abstractMethod2();
  }
  
}
```

* 每一种具体的小方法组合都要实现模板方法抽象类:

  ```java
  class ConcreteTemplateMethod extends TemplateMethodClass
  {
      public void abstractMethod1()
      {
          System.out.println("抽象方法1的实现被调用...");
      }   
      public void abstractMethod2()
      {
          System.out.println("抽象方法2的实现被调用...");
      }
  }
  
  ```

* 使用:

  ```java
  public class TemplateMethodPattern
  {
      public static void main(String[] args)
      {
          AbstractClass tm=new ConcreteClass();
          tm.TemplateMethod();
      }
  }
  ```




## 命令模式

* 适用场景:
  * 在一个系统中, 有一个命令需要执行, 这个命令是动态的, 可以在运行时被改变.

实现方式:

* 抽象的命令接口:

  ```java
  interface Command {
    public abstract void execute();
  }
  ```

* 具体的命令, 每一个具体命令都会有一个Receiver去执行:

  ```java
  class ConcreteCommand implements Command {
    
    private Receiver receiver;
    
    ConcreteCommand() {
      receiver = new Receiver();
    }
    // 执行
    public void execute() {
      receiver.action();
    }
  }
  ```

* Receiver类: 负责真实处理Command:

  ```java
  class Receiver
  {
      public void action()
      {
        // ...
      }
  }
  ```

* Invoker类: 负责接受不同类型的Command, 然后执行:

  ```java
  class Invoker {
    
    private Command command;
    
    public Invoker(Command command) {
      this.command = command;
    }
    
    public void setCommand(Command command) {
      this.command = command;
    }
    
    public void call() {
      command.execute();
    }
  }
  ```

* main方法:

  ```java
  public static void main(String[] args) {
    // 用多态切换Command
    Command cmd = new ConcreteCommand();
    Invoker invoker = new Invoker(cmd);
    invoker.call();
  }
  ```

  

## 责任链模式

* 适用场景:
  * 我现在有一个请求/算法, 实现这个请求/算法需要多个Handler, 每一个Handler可能只做某一部分, 并且Handler之间是有顺序的.

实现方式:

* 抽象Handler类:

  ```java
  abstract class Handler {
    
    private Handler next;
    
    public void setNext(Handler next) {
      this.next = next;
    }
    
    public Handler getNext() {
      return this.next;
    }
    
    // 当前Handler处理请求
    public abstract void handleRequest(String request);
  }
  ```

* 具体Handler类:

  ```java
  class ConcreteHandler extends Handler {
    
    public void handleRequest(String request) {
      if (/*如果能处理*/) {
        // 处理请求
      }
      else {
        // 看看下一个Handler是否有
        if (this.getNext() != null) {
          // 调用下一个
          this.getNext().handleRequest(request);
        }
        else {
          // 没人处理请求
        }
      }
    }
  }
  ```

* main函数:

  ```java
  public class ChainOfResponsibilityPattern
  {
      public static void main(String[] args) {
          //组装责任链 
          Handler handler1=new ConcreteHandler1(); 
          Handler handler2=new ConcreteHandler2(); 
          handler1.setNext(handler2); 
          //提交请求 
          handler1.handleRequest("two");
      }
  }
  ```

  

## 状态模式

* 适用场景:
  * 如果一个对象有状态, 并且状态不同, 对象的行为不同时, 就可以用到状态模式

实现方式:

* Context类 (有状态的对象类):

  ```java
  class Context {
    
    private State state;
    
    // 定义初始状态
    public Context() {
      this.state = state;
    }
    
    // 设置新状态
    public void setState(State state) {
      this.state = state;
    }
    
    // 读取状态
    public State getState() {
      return this.state;
    }
    
    // 根据状态做出行为
    public void handle() {
      this.state.handle(this);
    }
  }
  ```

* State抽象类:

  ```java
  abstract class State {
    public abstract void handle(Context context);
  }
  ```

* ConcreteState:

  ```java
  class ConcreteState extends State {
    public void handle(Context context) {
      // 处理逻辑
      // 状态切换逻辑: 根据Context的变量决定切换
      context.setState(new ConcreteStateB());
    }
  }
  ```

* main函数:

  ```java
  public class StatePatternClient
  {
      public static void main(String[] args)
      {       
          Context context=new Context();    //创建环境       
          context.Handle();    //处理请求
          context.Handle();
          context.Handle();
          context.Handle();
      }
  }
  ```

  

## 中介模式

* 适用场景:
  * 当你有一个服务时, 客户端有一大堆, 服务端有一大堆, 这时你就需要一个中介者去临时接收, 转发.

实现方式:

* 抽象同事类:

  ```java
  abstract class Colleague {
    
    // 中介
    protected Mediator mediator;
    
    public void setMedium(Mediator mediator) {
      this.mediator = mediator;
    }
    
    public abstract void send();
    public abstract void receive();
  }
  ```

* 具体同事类:

  ```java
  class ConcreteColleague extends Colleague {
    
    public void send() {
      // 请中介转发
      mediator.relay(this);
    }
    
    public void receive() {
      // 收到来自中介的请求
    }
  }
  ```

* 抽象中介类:

  ```java
  abstract class Mediator {
    
    public abstract void register(Colleague colleague);
    // 转发来自cl的请求
    public abstract void relay(Colleague cl);
  }
  ```

* 具体中介类:

  ```java
  class ConcreteMediator extends Mediator {
    
    // 注册的同事
    private List<Colleague> colleagues = new ArrayList<Colleague>();
    
    // 注册同事
   	public void register(Colleague colleague) {
      if (colleagues.contains(colleague)) {
        colleagues.add(colleague);
        colleague.setMedium(this);
      }
    }
    
    // 转发给其他同事
    public void relay(Colleague cl) {
      for (Colleague ob: colleagues) {
        if (!ob.equals(cl))
          ((Colleague)ob).receive();
      }
    }
  }
  ```

* main函数:

  ```java
  public class MediatorPattern {
      public static void main(String[] args) {
        
          Mediator mediator = new ConcreteMediator();
        
          Colleague c1 = new ConcreteColleague1();
          Colleague c2=new ConcreteColleague2();
        
          mediator.register(c1);
          mediator.register(c2);
        
          c1.send();
          c2.send();
      }
  }
  ```

  

## 迭代器模式

* 适用场景:
  * 如果你有一个集合, 这个集合需要在运行时改变它的遍历顺序, 这就需要迭代器模式.

实现方式:

* 抽象迭代器:

  ```java
  interface Iterator {
    boolean hasNext();
    Object next();
  }
  ```

* 具体迭代器:

  ```java
  class ConcreteIterator implements Iterator {
    
    // 成员变量设置成你的集合/集合内部的核心存储元素(List等)
    // 这里拿list举个例子
    private Set set = null;
    
    // 这里构造函数的参数同上
    public ConcreteIterator(Set set) {
      this.set = set;
    }
  	// 根据Set的逻辑实现这两个接口
    public boolean hasNext() {
      
    }
    public Object next() {
      
    }
  }
  ```

* 抽象集合类:

  ```java
  abstract Set {
    // 获取迭代器
    public abstract Iterator getIterator();
  }
  ```

* 集合类:

  ```java
  class ConcreteSet implements Set {
    
    public Iterator getIterator() {
      return new ConcreteIterator(this);
    }
  }
  ```

* main方法:

  ```java
  public static void main(String[] args) {
    Set set = new ConcreteSet();
    Iterator iter = set.getIterator();
    
    while (iter.hasNext) {
      Object obj = iter.next();
      //...
    }
  }
  ```

  

## 访问者模式

* 适用场景:
  * 假设你的系统中有多个组成元素, 你的系统通过**对这些元素的不同使用方式**来达到一个目的.
  * 变量是不同的使用方式.

实现方式: 不同的使用方式就可以抽象成不同的Visitor.

* 抽象Visitor接口:

  ```java
  interface Visitor {
    
    // visitor访问不同元素的方式
    void visit(ConcreteElementA element);
    void visit(ConcreteElementB element);
    
  }
  ```

* 具体Visitor:

  ```java
  //具体访问者A类
  class ConcreteVisitorA implements Visitor
  {
      public void visit(ConcreteElementA element)
      {
          System.out.println("具体访问者A访问-->"+element.operationA());
      }
      public void visit(ConcreteElementB element)
      {
          System.out.println("具体访问者A访问-->"+element.operationB());
      }
  }
  ```

* 抽象元素类:

  ```java
  interface Element {
    
    void accept(Visitor visitor);
  }
  ```

* 具体元素类:

  ```java
  class ConcreteElement implements Element {
    
    public void accept(Visitor visitor) {
      visitor.visit(this);
    }
    
    public String operationA() {
      return "haha";
    }
  }
  ```

* 系统类:

  ```java
  class System
  {   
      private List<Element> list=new ArrayList<Element>(); 
    
      public void accept(Visitor visitor)
      {
          Iterator<Element> i = list.iterator();
          while(i.hasNext())
          {
              ((Element) i.next()).accept(visitor);
          }      
      }
    
      public void add(Element element)
      {
          list.add(element);
      }
      public void remove(Element element)
      {
          list.remove(element);
      }
  }
  ```

* main方法:

  ```java
  public static void main(String[] args) {
    
      System sys = new System();
    
      sys.add(new ConcreteElementA());
      sys.add(new ConcreteElementB());
    
      Visitor visitor=new ConcreteVisitorA();
    
      sys.accept(visitor);
  
      visitor=new ConcreteVisitorB();
    
      sys.accept(visitor);
  }
  ```

  

## 备忘录模式

* 适用场景:
  * 程序中的某个对象需要恢复到之前的某个状态.

实现方式:

* 假设你需要恢复状态的对象叫`Originator`:

  ```java
  public class Originator {
    
    // 这里的state需要自己设计, 需要恢复状态所用的变量
    private String state;
    
    public String getState() {
      return this.state;
    }
    public void setState(String state) {
      this.state = state;
    }
    
    // 生成存储在备忘录中的元素
    public Snapshot getSnapShot() {
      return new Snapshot(state);
    }
    
    // 从备忘录中获取元素, 并且恢复
    public void restore(Snapshot snapshot) {
      this.state = snapshot.getState();
    }
  }
  ```

* 备忘录类:

  ```java
  class Snapshot {
    
    private String state;
    
    public Snapshot(String state) {
      this.state = state;
    }
    
    public String getState() {
      return state;
    }
  }
  ```

* 备忘录管理者:

  ```java
  class SnapshotManger {
    
    // 存储所有Memento, 一般是栈
    private Stack<Snapshot> snapshotList = new Stack<Snapshot>();
    
    public void backup(Snapshot snapshot){
        this.snapshotList.push(snapshot);
    }
   
     public Memento getRestoreSnapshot(){
        return this.snapshotList.pop();
     }
    
  }
  ```

* main方法:

  ```java
  public static void main(String[] args) {
    
    // 创建系统
    Originator originator = new Originator();
    
    // 创建备忘录管理者
    SnapshotManager manager = new SnapshotManager();
    
    originator.setState("State 0");
    // 备份
    manager.backup(originator.getSnapshot());
    
    // 恢复
    originator.restore(manager.getRestoreSnapshot());
  }
  ```
