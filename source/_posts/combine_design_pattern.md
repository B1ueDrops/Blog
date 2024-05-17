---
title: 组合型的设计模式
categories: 软件工程
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

  
