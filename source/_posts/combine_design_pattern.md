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



## 桥接模式

