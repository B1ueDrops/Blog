---
title: 组合型的设计模式
categories: 后端技术
---



## 组合型设计模式

组合型设计模式用于将类/对象以耦合度低的方式组织成更大的对象.



## 代理模式

* 适用场景: 在调用一个对象的方法时, 需要`pre`处理, 以及`post`处理, 这个时候可以提供一个代理.

实现方式:

* 对象的抽象接口:

```java
interface Subject {
  void Request();
}
```

* 真实对象:

  ```java
  class ConcreteSubject implements Subject {
    public void Request() {
      //...
    }
  }
  ```

* 代理:

  ```java
  class Proxy implements Subject {
    
    private RealSubject realSubject;
    
    public void Request() {
      
      if (this.realSubject == null) {
        this.realSubject = new RealSubject();  
      }
      
      preRequest();
      realSubject.Request();
      postRequest();
    }
    
    public void preRequest() {
    	//...  
    }
    public void postRequest() {
      //...
    }
  }
  ```

  

## 适配器模式

* 适用场景: 我现在有一个类, 我要把这个类的接口通过一层适配器转换成目标接口的格式:

实现方式1: 类适配器 

* 目标接口:

```java
interface Target {
  public void request();
}
```

* 我现在有的类:

  ```java
  class Adaptee {
    
    public void myRequest() {
      // ...
    }
  }
  ```

* 加一个Adaptor类:

  ```java
  class Adapter extends Adaptee implements Target {
    
    // 实现接口方法
    public void request() {
      // 用myRequest()
    }
  }
  ```

* main函数:

  ```java
  public static void main(String[] args) {
    Target target = new Adapter();
    target.request();
  }
  ```



实现方式二: 对象适配器

* Adapter类:

  ```java
  class Adapter implements Target {
    
    private Adaptee adaptee;
    
    public Adapter(Adaptee adaptee) {
      this.adaptee = adaptee;
    }
    
    public void request() {
      // 做适配
      this.adaptee.myRequest();
    }
  }
  ```

* main函数:

  ```java
  public static void main(String[] args) {
    Adaptee adaptee = new Adaptee();
    Target target = new Adapter(adaptee);
    target.request();
  }
  ```



## 桥接模式

* 适用场景: 某些对象具有多种特征维度的变化, 例如m种颜色和n种大小, 那么就会有mxn种的子类.

实现方式:

* 抽象特征接口:

```java
interface Feature {
  //...
}
```

* 具体特征:

  ```java
  class ConcreteFeature implements Feature {
    
  }
  ```

* 抽象对象:

  ```java
  abstract class Subject {
    protected Feature feature;
    
    public Subject() {
      this.feature = feature;
    }
    public void setFeature(Feature feature) {
      this.feature = feature;
    }
    // ...
  }
  ```

* 具体对象:

  ```java
  class ConcreteSubject extends Subject {
    
    // ...
  }
  ```

* main函数:

  ```java
  public static void main(String[] args) {
    
    Feature feature = new ConcreteFeature();
    Subject subject = new ConcreteSubject(feature);
  }
  ```

本质上来说, 就是把最主要的Feature用继承解决, 其他的Feature都用组合解决.



## 装饰模式

* 适用场景: 在保持原来对象的结构不变的情况下, 动态增加新的功能.

实现方式:

* 抽象对象:

  ```java
  interface Subject {
    public void operation();
  }
  ```

* 具体对象:

  ```java
  class ConcreteSubject implements Subject {
    public void operation() {
      //...
    }
  }
  ```

* 装饰器类:

  ```java
  class SubjectDecorator implements Subject {
    
    private Subject subject;
    
    public SubjectDecorator(Subject subject) {
      this.subject = subject;
    }
    
    public void operation() {
      this.subject.operation();
    }
  }
  ```

* 具体要装饰什么, 直接继承装饰器类:

  ```java
  class ConcreteDecorator extends SubjectDecorator {
    
    public ConcreteDecorator(Subject subject) {
      super(subject);
    }
    
    public void operation() {
      // super.operation和我自己加什么功能组合
    }
  }
  ```




## 外观模式

* 适用场景: 外观模式负责将若干个不同的子系统整合成一个大的系统, 并向用户提供简单的接口, 掩饰系统的复杂性.

实现方式:

* 若干个子系统:

  ```java
  class SubSystem1 {
      public  void method1() {
          System.out.println("子系统1的method1()被调用！");
      }   
  }
  class SubSystem2 {
      public  void method2() {
          System.out.println("子系统2的method2()被调用！");
      }   
  }
  class SubSystem3 {
      public  void method3() {
          System.out.println("子系统3的method3()被调用！");
      }   
  }
  ```

* 外观类:

  ```java
  class Facade
  {
      private SubSystem1 obj1=new SubSystem1();
      private SubSystem2 obj2=new SubSystem2();
      private SubSystem3 obj3=new SubSystem3();
    
      public void method()
      {
          obj1.method1();
          obj2.method2();
          obj3.method3();
      }
  }
  ```

* main函数:

  ```java
  public static void main(String[] args) {
    
    Facade f = new Facade();
    f.method();
  }
  ```



## 组合模式

* 适用场景: 当你的系统中出现了树状逻辑结果, 例如文件/文件夹等, 组合模式可以让你的系统中, 单一对象和复合对象具有相同的接口, 简化系统设计.

实现方式:

* 节点接口:

```java 
interface Component
{
    public void add(Component c);
    public void remove(Component c);
    public Component getChild(int i);
    public void operation();
}
```

* 中间节点:

  ```java
  class Composite implements Component
  {
      private ArrayList<Component> children=new ArrayList<Component>();   
    
      public void add(Component c) {
          this.children.add(c);
      }
    
      public void remove(Component c) {
          children.remove(c);
      }
    
      public Component getChild(int i) {
          return children.get(i);
      }
    
      public void operation() {
          for(Object obj:children) {
              ((Component)obj).operation();
          }
      }    
  }
  
  ```

* 叶子节点:

  ```java
  class Leaf implements Component
  {
      private String name;
    
      public Leaf(String name) {
          this.name=name;
      }
      public void add(Component c){ } 
    
      public void remove(Component c){ }
    
      public Component getChild(int i) {
          return null;
      }
    
      public void operation()
      {
          System.out.println("树叶"+name+"：被访问！"); 
      }
  }
  ```

* main函数:

  ```java
  public static void main(String[] args) {
      Component c0 = new Composite(); 
      Component c1 = new Composite(); 
      Component leaf1 = new Leaf("1"); 
      Component leaf2 = new Leaf("2"); 
      Component leaf3 = new Leaf("3");
    
      c0.add(leaf1); 
      c0.add(c1);
      c1.add(leaf2); 
      c1.add(leaf3);          
      c0.operation(); 
  }
  ```

  

## 享元模式

* 适用场景: 程序中要出现大量的相同对象 (例如不同颜色的棋子、 图形), 享元模式可以共享这些对象相同的部分, 节省系统资源.

实现方式:

* 首先需要确定, 这些对象中, 可以被共享的成分和不能被共享的成分.

* 非享元部分 (不能被共享的成分)

  ```java
  class UnsharedConcreteFlyweight {
      private String info;
    
      UnsharedConcreteFlyweight(String info) {
          this.info = info;
      }
      public String getInfo() {
          return info;
      }
      public void setInfo(String info) {
          this.info = info;
      }
  }
  ```

* 享元对象接口:

  ```java
  interface Flyweight {
      public void operation(UnsharedConcreteFlyweight state);
  }
  ```

* 具体的享元对象:

  ```java
  class ConcreteFlyweight implements Flyweight
  {
    // key是可以被共享的属性, 例如颜色
      private String key;
      ConcreteFlyweight(String key)
      {
          this.key = key;
          System.out.println("具体享元"+key+"被创建！");
      }
      public void operation(UnsharedConcreteFlyweight outState)
      {
        // 这里可以结合非享元部分进行操作
          System.out.print("具体享元"+key+"被调用，");
          System.out.println("非享元信息是:"+outState.getInfo());
      }
  }
  ```

* 享元工厂: 用来生产享元对象:

  ```java
  class FlyweightFactory {
    
    private HashMap<String, Flyweight> flyweights = new HashMap<String, Flyweight>();
    
    // 如果HashMap中存在, 就直接返回, 否则新建一个
    public Flyweight getFlyweight(String key) {
      
      Flyweight flyweight=(Flyweight)flyweights.get(key);
          
      if(flyweight != null) {
          System.out.println("具体享元"+key+"已经存在，被成功获取！");
      }
      else {
          flyweight=new ConcreteFlyweight(key);
          flyweights.put(key, flyweight);
      }
      
      return flyweight;
    }
  }
  ```

* main函数:

  ```java
  public static void main(String[] args) {
    
    FlyweightFactory factory=new FlyweightFactory();
    
    Flyweight f1=factory.getFlyweight("a");
    Flyweight f2=factory.getFlyweight("a");
    
    // 传递非共享部分
    f1.operation(new UnsharedConcreteFlyweight("first"));       
    f2.operation(new UnsharedConcreteFlyweight("second")); 
  }
  ```

  
