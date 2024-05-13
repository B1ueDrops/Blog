---
title: 代码整洁之道
categories: 软件工程
---



## 注释

* 理解注释的地位:

  * 注释不是越多越好: <font color=red>注释只是对代码的辩解, 好的代码就是文档, 根本不需要注释.</font>

* 不要在代码中保留注释掉的代码, 借助git.

* <font color=red>仅仅对包含复杂业务逻辑的东西进行注释, 不要对大家都懂的东西进行注释.</font>

* 不要占位符, 借助合理的缩进和格式化提供好的代码结构:

  ```java
  ////////////////////////////////////////////////////////////////////////////////
  // Scope Model Instantiation
  ////////////////////////////////////////////////////////////////////////////////
  String[] model = {"foo","bar"};
  
  ////////////////////////////////////////////////////////////////////////////////
  // Action setup
  ///////////////////////////////////////////////////////////////////////////////
  void action(){
      //...
  }
  ```

  

## 变量

* 变量名需要具有足够的可读性, 不要强迫读者猜测变量的意义, 简单明了胜于晦涩难懂.

  * 反例1: `ymdstr = datetime.date.today().strftime("%y-%m-%d")`, `ymdstr`是什么?

    * `current_date: str = datetime.date.today().strftime("%y-%m-%d")`.

  * 反例2:

    ```python
    # seq是什么东西?
    seq = ( 'Austin', 'New York', 'San Francisco' )
    for item in seq:
      # item是什么东西
      do_something(item)
      
    # 修改后
    locations = ( 'Austin', 'New York', 'San Francisco' )
    for location in locations:
      # item是什么东西
      do_something(location)
    ```

  * 反例3: `time.sleep(86400)`, 86400是什么意思?

    * 修改:

      ```python
      SECONDS_IN_A_DAY = 60 * 60 * 24
      time.sleep(SECONDS_IN_A_DAY)
      ```

  * 反例4: 在程序中出现`matches[0], matches[1]`这种, 这是什么东西都不知道, 应该用一个变量去接收, 这个变量的名称要具有可读可搜索性.
  
  * 变量的名称中, 不要出现类型信息.
  
    * 例如一个结构体叫`user`, 成员变量不要是`user_name`, 冗余.
  



## 函数

* 一个函数应该简洁到只做一件事情/只有一个动作, 这样函数能够更容易被测试/维护.
* 一个函数最好只有一个入口点, 一个出口点(只有一处有`return`).
* 函数的内部尽量不要包含`try catch`等异常处理结构 (只做一件事的原则), 异常处理尽量交给别的函数统一处理.
* 函数的参数个数不要超过两个, 因为一旦超过两个, 函数大概率做了很多事情, 耦合度较高, 不容易被重构.

* 函数的名称应该具有足够简洁, 直观, 具有足够的可读性, 能够直接说明这个函数做了什么.

* 如果一个函数做了除了接收一个值, 计算, 返回一个值以外的动作, 例如修改了全局变量, 或者写入了文件, 那么:

  * 尽量不要这样干.

  * 如果真的需要这样干, 可以在全局提供一个service函数, 通过调用这个service函数集中解决 (集中方便维护).

    ```java
    // 不符合规范的代码
    String name = "haha";
    void changeName() {
      name = name.split("a").toString();
    }
    changeName();
    System.out.println(name);
    
    // 符合规范的代码
    String name = "haha";
    void changeName(String name) {
      return name.split("a").toString();
    }
    changeName(name);
    System.out.println(name);
    ```

* 关于条件语句的几点:

  * 尽量避免条件语句, 因为你一旦引入条件语句, 就说明函数一定做了不止一件事.

    * 可以使用多态在一定程度上减少条件语句.

  * 如果无法避免, 那么需要对条件语句的布尔表达式进行封装, 例如:

    ```java
    if(fsm.state.equals("fetching")&&listNode.isEmpty(){
    }
       
    // 修改后
    void shouldShowSpinner(Fsm fsm, String listNode) {
        return fsm.state.equals("fetching")&&listNode.isEmpty();
    }
    
    if (shouldShowSpinner(fsmInstance, listNodeInstance)) {
                // ...
    }
    ```

  * 尽量不要使用反面的条件, 例如 `if (!isDOMNodeNotPresent(node)) {`.

* 避免冗余代码:

  * 如果你的代码中, 出现了修改一处, 就需要修改更多处代码的情况, 那么很可能就出现了冗余代码, 需要避免.

* 避免僵尸代码:

  * 代码注释掉的代码不要保留, 因为你有git.

* 用Getter和Setter方法实现参数设置, 而尽量不要写死参数.

* 函数的调用者和函数的定义在竖直方向上应该靠近, 最理想的情况是: 函数调用者的正下方就是函数的定义, 因为人的阅读习惯就是从上向下阅读.



## 方法链简化代码

在Setter方法中返回`this`, 可以简化代码:

```java
    class Car{
        private String make;
        private String model;
        private String color;

        public Car setMake(String make) {
            this.make = make;
            return this;
        }

        public Car setModel(String model) {
            this.model = model;
            return this;
        }

        public Car setColor(String color) {
            this.color = color;
            return this;
        }

        public Car save(){
            console.log(this.make, this.model, this.color);
            return this;
        }
    }

    Car car=new Car().setColor("pink").setMake("Ford").setModel("F-150").save();
```



## 函数式编程简化代码

使用函数式编程可以简化代码, 并且不失直观性:

```java
// 没有函数式编程
    List<Integer> programmerOutput=new ArrayList<>();
    programmerOutput.add(500);
    programmerOutput.add(1500);
    programmerOutput.add(150);
    programmerOutput.add(1000);
    int totalOutput=0;
    for(int i=0;i<programmerOutput.size();i++){
        totalOutput+=programmerOutput.get(i);
    }
// 有函数式编程
    List<Integer> programmerOutput=new ArrayList<>();
    programmerOutput.add(500);
    programmerOutput.add(1500);
    programmerOutput.add(150);
    programmerOutput.add(1000);
    int totalOutput= programmerOutput.stream().filter(programmer -> programmer > 500).mapToInt(programmer -> programmer).sum();
```



## 测试

* 测试驱动开发!
* 测试需要有100%的覆盖率来保证代码质量.
* 一项测试只测试一个方面, 不要一个测试包含多个方面.