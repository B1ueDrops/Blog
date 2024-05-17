---
title: Go语言使用日记
categories: 编程语言
---



## 配置环境

* 官网下载`pkg`.

* 环境变量:

```bash
export GO111MODULE=on
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GOPROXY=https://goproxy.cn
export PATH=$PATH:$GOROOT/bin
```



## 命名规范

* package: 小写单词, 不用下划线.
* 文件名: 小写单词, 下划线.
* 变量: 驼峰.
* 函数: 驼峰.
* 结构体: 驼峰.



## 实数

* Go语言中有两种浮点数:
  * `float64`: 
    * 浮点数默认都是float64.
    * 占用8字节内存.
  * `float32`: 占用4字节内存.
* 如果要使用单精度, 必须显式声明`float32`: 
  * `var num float32 = 42.0`.

* Go语言中, 每一种类型都有一个默认值, 它叫做零值.
* 如果你声明但是没有初始化变量, 那么变量就会变成零值.
* `%f`格式化占位符: `%m.nf`
  * `m`表示整个浮点数的输出宽度 (如果数长, 超过这个宽度, 没有问题, 如果达不到这个宽度会在左边填充空格).
    * 如果不想用空格, 想用0填充, 可以用`%0m.nf`.
  * `n`表示小数点的位数.
* 浮点数不精确, 尽量先做乘法, 再做➗.
* 如果要比较浮点数`a`和`b`, 用`math.Abs(a - b) < eps`, 其中`eps`需要自己确定.
* 如果没有为指数形式(例如`24e18`)指明类型, 那么默认是`float64`类型.



## 整数

* 最常用的有符号整数类型是`int`, 无符号的是`uint`.
* 自动推断整数类型一般是`int`.
* Go语言中的整数类型只有8种:
  * `int8, int16, int32, int64`.
  * `uint8, uint16, uint32, uint64`.
  * 这8中类型和CPU架构没有关系.
* `int`和`uint`是和目标设备有关的, 不同平台位宽可能不一样.
  * 最好用`int64`或者`uint64`代替`int`和`uint`.
* Go语言中加上`0x`前缀, 就可以用十六进制的形式表示.
* 整数以16进制打印, 可以用`%0mx`.
  * `0`表示前面如果宽度不够, 用0填充.
  * `m`表示最小宽度.
* 整数类型都有范围, 超过这个范围就会发生环绕.
  * 因此, 如果代表时间, 应该用`int64`或者`uint64`.
* 整数以2进制打印, 可以用`%0mb`, 和`%0mx`用法一样.
* `math`包中, 为与架构无关的整数, 预定义了最大值和最小值:
  * 例如`math.MaxInt16`和`math.MaxInt32`等.

* 对于超大数, 可以使用`big`包:

  * 超大整数: `big.Int`.

    ```go
    a := new(big.Int)
    // 10代表进制
    a.SetString("2400000000000000000000000", 10)
    ```

    * 一旦一个整数使用了`big.Int`, 那么等式的其他变量也必须是`big.Int`类型
    * `big.NewInt()`可以把`int64`转换成`big.Int`类型.

  * 任意精度浮点数: `big.Float`

  * 分数: `big.Rat`.

* 在Go语言中, 常量可以是没有类型`untype`的:

  ```go
  const a = 24000000000000000000000000000
  ```

  * 在底层, 这种无类型的常量, 也是用`big`包进行处理的.

* 用`fmt.Printf`打印时, 通用的占位符类型是`%v`.



## 字符串

* 字符串的类型是`string`.

* 字符串的零值是`""`.

* 字符串字面值和原始字符串字面值(raw string literal):

  * 字符串字面值会处理转义字符: `fmt.Println("haha\nhaha")`

  * 原始字符串字面值不用`""`, 不会处理其中的转义字符.

    ```go
    fmt.Println(`haha\nhaha`)
    ```

* 字符:

  * ASCII码中包含128个字符.

  * Unicode联盟为超过100万个字符都分配了数值, 这个数值叫做code points.
    * 65表示`'A'`.

  * 为了表示code points, Go语言提供了`rune`类型, 本质上是`int32`.
  * `byte`是`uint8`的别名.

* 给类型起别名:

  ```go
  type rune = int32
  ```

* 如果想打印字符而不是数值, 就用`%c`.

* 如果想输出code points, 就用`%v`.

* 任何整数类型都可以用`%c`打印, 但是`rune`意味着该数值表示了一个字符.

* 如果没有指定字符类型, Go语言会推断类型为`rune`.

  * 字符字面值也可以用`byte`类型.

* 可以给某一个变量赋予不同的`string`值, 但是`string`本身是不可变的, 例如下面代码会报错:

  ```go
  func main() {
  	message := "shalom"
  	message[0] = 'd'
  }
  ```

* 可以用`len(message)`获取字符串的**字节数**, `len()`是内置函数.

  * 如果需要获取字符串的字符个数, 需要用`utf8`包

    ```go
    import "unicode/utf8"
    
    a = "haha"
    fmt.Println(utf8.RuneCountInString(a))
    ```

* Go语言中的字符串采用`UTF-8`编码, `UTF-8`编码是Unicode code points的编码方式之一.

  * `UTF-8`是一种可变长度编码, 每个code point可以是8位, 16位, 32位.

* Go语言中的字符可以通过`+`整数, 变成另一个字符.

* 遍历字符串字符:

  ```go
  a := "haha"
  
  for _, c := range a {
    fmt.Printf("%v %c", i, c);
  }
  ```

  

## 类型转换

* 直接连接字符串和数值会报错:

  ```go
  count := "haha" + 10 + "haha"
  ```

* 整数和浮点数也不能直接运算:

  ```go
  a := 1
  b := 2.0
  c := a * b //报错
  ```

* 类型转换: 

  ```go
  a := 1
  b := float64(a)
  ```

* 从浮点类型转换为整数类型, 小数点后面的部分会被截断.

* 无符号和有符号之间, 不同大小的整数类型之间都需要转换!

* 类型转换需要谨慎, 避免环绕行为.

  * 可以通过`math`包中的`min/max`判断是否超过最大/最小值.

* 如果想把`rune`或者`byte`转换为`string`, 语法是一样的:

  ```go
  var a byte = 12
  var b rune = 199
  
  c := string(a)
  d := string(b)
  ```

  * 但是, 必须要证`rune/byte`的数值在合法范围内.
  * 注意, 这个转换是把unicode code point转换成字符串, 例如`97`转换成`'a'`.
  
* 整数转字符串: `strconv.Itoa(10)`, 可以把10转为`"10"`.

* 字符串转整数:

  ```go
  data, err := strconv.Atoi("10")
  if err != nil {
    
  }
  ```



## 函数

* 函数声明: `func`

  ```go
  func Func_name(a int32, b int64) string {
    
  }
  ```

* 大写字母开头的函数/变量/其他标识符都会被默认导出, 小写字母开头就不行.

* 函数可以有多个返回值:

  ```go
  func Func_name(a int32, b int64) (string, error) {
    
  }
  ```

* 匿名函数:

  ```go
  func() {
    fmt.Println("只用一次的匿名函数")
  }()
  ```

  

## 方法

* 方法不一定要和`struct`绑定, 可以和你自己声明的任何类型绑定:

  ```go
  // 通过type声明的新类型
  type TypeA int32
  
  // 为TypeA绑定方法
  func (a TypeA) getNum() i32 {
    
  }
  
  var a TypeA = 2
  a.getNum()
  ```




## 数组

* 数组是一种固定长度的数据类型:

  ```go
  var planets [8]string
  ```

  * 二维数组: `var planets [8][8]string`

* 数组长度可以用`len()`确定:

  ```go
  len(planets)
  ```

  * 数组的长度也是数组的一部分.

* 如果没有初始化数组中的值, 那么数组中的值默认是零值.

* Go语言编译时会检查数组越界, 如果越界就会报错.

  * 如果编译时没发现, 那么运行时出现就会`panic`.

* Go语言定义并初始化数组:

  ```go
  dwarfs := [8]string {
    "A",
    "B",
    "C",
    "D",
    "E",
    "F",
    "G",
    "H"
  }
  ```

  * 可以使用`...`代替数组长度, Go编译器会帮你算出数组长度:

    ```go
    dwarfs := [...]string {
      "A",
      "B",
      "C",
      "D",
      "E",
      "F",
      "G",
      "H"
    }
    ```

* 遍历数组:

  * 使用`for`循环:

    ```go
    for i := 0; i < len(dwarfs); i ++ {
      
    }
    ```

  * 使用`range`:

    ```go
    for i, item := range dwarfs {
      
    }
    ```

* 数组复制:

  * Go语言中, 将一个数组赋值/作为函数参数, 会发生深拷贝.



## 切片

* `slice`是数组某一部分的副本.

  * `a[0:4]`就是一个切片.

* `slice`的默认索引:

  * 忽略起始索引, 从开始切分, `a[:4]`
  * 忽略结束索引, 切分到结束, `a[1:]`

* 切分数组的语法也可以引用于字符串.

  * 切分字符串时, 索引代表的是字节数, 而不是`rune`.

* 定义并初始化切片:

  ```go
  dwarfs := []string {
    "haha",
    "haha",
    "haha"
  }
  ```

  * 想要获得和数组相同元素的`slice`, 可以用`[:]`切分.

* Go语言中, 很多函数倾向于使用slice, 而不是数组作为参数.

* slice在进行赋值/函数参数传递时, 会发生浅拷贝.

* `append`函数:

  * `append`函数是内置函数, 可以将元素添加到slice中.

    ```go
    dwarfs := []string { "A", "B" }
    dwarfs = append(dwarfs, "C")
    // 也可以添加多个元素
    dwarfs = append(dwarfs, "D", "E")
    ```

* `length`和`capacity`的区别:

  * `length`: `slice`中有多少个元素, 用`len()`.
  * `capacity`: `slice`对应的底层数组有多少个元素, 用`cap()`.

* `slice`预分配:

  * `slice`本质上是一个动态数组, 如果`append`过多, 会发生过多的内存拷贝, 影响性能.

  * 可以使用`make`函数进行预分配:

    ```go
    // 初始length是0, capacity是10
    dwarfs = make([]string, 0, 10)
    ```

* 声明可变参数的函数:

  * 需要在函数最后一个参数的前面加上`...`:

    ```go
    // worlds本质上是[]string类型
    func terraform(prefix string, worlds ...string) []string {
      for i := range worlds {
        worlds[i]
      }
    }
    ```




## map

* 声明map, 必须指明key和value的类型:

  ```go
  map[string]int
  ```

* 根据已有的值直接创建`map`:

  ```go
  temperature := map[string]int {
    "Earth": 15,
    "Mars": -15,
  }
  ```

* 插入一个键值对:

  ```go
  temperature["Venus"] = 20
  ```

* 当访问一个不存在的值时, map会返回类型对应的零值:

  ```go
  // value的值是0
  value := temperature["Moon"]
  ```

* 逗号和OK写法: 用来判断key在键值对存不存在:

  ```go
  if moon, ok := temperature["Moon"]; ok {
    // 如果键值对存在
  } else {
    // 如果键值对不存在
  }
  ```

* map在赋值/当作函数参数时, 只会发生浅拷贝:

  ```go
  a = map[string]int {
    "haha": 1,
    "heihei": 2,
  }
  // b和a指向同一个map
  b := a
  ```

* 删除键值对:

  ```go
  delete(tempearture, "Moon")
  ```

* `make`也可以对map进行预分配: 第二个参数是预分配的空间

  ```go
  temperature = make(map[string]float64, 10)
  ```

* 遍历map:

  ```go
  for key, value := range temperature {
    
  }
  ```

  

## struct

* 声明结构体:

  ```go
  type Student struct {
    name string
    age int64
  }
  ```

  * 结构体字段之间不用`,`隔开.

* 打印结构体用`%+v`.

* 用复合字面值创建struct:

  ```go
  student := Student {
    name: "haha",
    age: 18,
  }
  ```

* struct的复制:

  * struct赋值/当作函数参数时, 会发生深拷贝.

* 将struct转换为json:

  * 首先需要在结构体定义时, 规定每个字段转化为json的时候是什么名字:

    ```go
    type Student struct {
      name string `json:"Name"`
      age int `json:"Age"`
    }
    ```
  
  * 需要`import "encoding/json"`
  
    ```go
    // 转换成功可以得到[]byte
    bytes, err := json.Marshal(student)
    fmt.Printf("%+v", string(bytes))
    ```
  
* 将方法绑定到struct:

  * 构造函数: Go语言中没有特殊规定构造函数, 构造函数就是普通函数, 构造函数一般就叫`New()`

    ```go
    type Location struct {
      lat float64
    long float64
    }
    
    func NewLocation(lat float64, long float64) Location {
      return Location { lat.decimal(), long.decimal() }
    }
    ```
  
  

## 接口

* 接口定义了一个类型必须实现的一种方法.

* 按约定, 接口必须以`er`为结尾.

  ```go
  type talker interface {
    talk() string
  }
  
  func (stu Student) talk() string {
    // Student实现了talker接口
  }
  ```

* 接口类型的含义: 例如`t talker`, 表示所有实现了`talker`接口的变量

* `interface{}`这个类型表达式, 表示什么类型都行.



## 指针

* 通过`&`可以获得变量的指针, 解引用用`*`.

  * 类型叫做`*int`, 表示`int`类型的指针.

* Go语言不允许对地址进行运算.

* 如果两个指针指向相同的内存地址, 那么他们就是相等的.

* struct和指针:

  ```go
  timmy = &person {
    name: "Timmy",
    age: 10,
  }
  // 这个时候, 不用加上解引用&
  // 等价于(*timmy).superpower
  fmt.Println(timmy.superpower)
  ```

  * slice和map也可以用`&`, 但是不能自动解引用.

* 函数/方法和指针:

  * Go语言中函数和方法都是值传递, 函数总是操作副本(除了map), 如果要操作原变量要传指针.

  ```go
  func birthday(p *Person) {
    p.age++
  }
  ```

  * Go语言中的变量在使用`.`时, 会自动调用`&`获取地址:

  ```go
  func (stu *Student) setName(name string) {
    stu.name = name
  }
  
  stu.setName()
  // 不用(&stu).setName()
  ```

  * 如果一个struct的方法用到的是指针作为接收者, 那么这种类型的所有方法都应该用指针作为接收者.



## nil

* `nil`是指针, slice, map, 接口, 函数类型的零值.

* `nil`会导致`panic`:

  * 对`nil`进行解引用会导致`panic`.

* 在设计方法时, 如果方法的接收者是指针, 应该判断指针是否为`nil`, 避免`panic`:

  ```go
  func (p *person) birthday() {
    if p == nil {
      return
    }
  }
  ```

* 对于nil类型的slice, `range, len, append`等函数都能正确处理slice, 不会导致panic.

  * 虽然空的slice和`nil`的slice并不相等, 但是它们通常可以替换使用.

* 对值为`nil`的map进行读取, 不会`panic`, 但是插入值会发生`panic`.



## 异常处理

* 按照惯例, 函数在返回错误时, 最后面的返回值应该表示错误, 如果没有错误返回`nil`.

  * 如果有错误, 返回`error`类型的变量, `error`类型是一个接口类型, 表示所有实现了`Error`接口的变量
  * `import errors`
  * 如果要返回, 就返回`return errors.New("错误信息")`

* 如果要自定义错误类型, 可以实现`Error`接口:

  ```go
  func (err AError) Error() string {
    // ...
  }
  ```

  * 那么这个类型就可以是`Error`的一种子类型`AError`.

  * 如果需要判断一个`Error`变量是哪种错误类型:

    ```go
    if err != nil {
      // 如果err是AError类型, 那么ok就是true
      if errs, ok := err.(AError); ok {
        
      }
    }
    ```

    

* 调用函数后, 应该立即检查是否有错误:

  * 当发生错误时, 函数的其他返回值就不能用.

* `defer`语句:

  * `defer f.Close()`: 保证函数在返回之前, `f.Close()`一定可以被执行.
  * 即使发生了`panic`, `defer`的语句也会执行.
  * 一般用于释放资源, 避免忘记.

## package

* 假设你有一个文件夹`utils`, 里面所有的`.go`文件, 第一行都是`package utils`
* 里面的`.go`文件, 各种成分互相调用, 不用写`import`.
* 需要别的包调用的, 记得名字<font color=blue>首字母大写</font>, 不需要的小写.
* 在另外一个package中, 调用其他package的成分:
  * 写上`import "Go项目的名字/package文件夹的路径"`
  * `package`文件夹的路径就是相对于Go项目根目录的路径.



## MPG并发模型

假设一个机器中有$n$个CPU, 那么这个机器就有$n$个进程.

* M: 全称是Machine, 是在用户态对于CPU的一种抽象, 一个进程对应一个Machine, 用于真正执行Goroutine.
* P: 全称是Processor, 可以理解为存储Goroutine的一个队列.
* G: 全称是Goroutine, 也叫协程, 是用户代码的运行时的状态.

协程在放到Processor之前, 会被放到一个协程的全局队列runqueue中.

1. 当创建第一个协程时, 会发生:

* 系统创建一个Machine, 并绑定到一个进程上.
* 系统创建一个Processor, 并绑定到当前的Machine上.
* 协程Goroutine会被放到Processor上, 然后开始在Machine执行.

2. 如果协程数量很多时, 会发生:

* 系统会创建更多的Machine, 绑定到对应的进程上.
* 系统会创建更多的Processor, 用来存储将要运行的Goroutine.

3. 如果Machine在执行一个Goroutine时, 发生了I/O这种高耗时任务或系统调用, 会发生:

* 绑定到这个Machine的Processor会被切换到其他的Machine.
* 这个Machine仍在处理发生I/O的Goroutine, 直到结束.
* 结束后, 这个Machine会处于短暂的空闲状态



## 协程

Go语言中, 创建协程的办法是在函数调用之前加上一个`go`, 例如:

```go
package main

import "fmt"

func foo() {
  fmt.Println("Hello")
}

func main() {
  // 创建了一个运行foo函数的协程
  go foo()
}
```

<font color=red>注意, 使用`go`创建的所有协程, 它们并不存在父子关系, 在调度层面是并列关系.</font>

在Go语言中, 有一个主协程(也叫`main`协程), 主协程和其他协程的关系是:

* 主协程结束会导致所有其他协程强制结束.
* 其他协程如果出现panic, 会导致主协程也强制结束, 这个需要避免.



### 主协程等待子协程

如果要让主协程等待所有子协程完成之后再结束, 可以使用`sync.WaitGroup{}`.

具体例子如下:

```go
package main

import "sync"

func main() {
  
  wg := sync.WaitGroup{}
  
  /* 这里的2是main协程需要等待的子协程的个数 */
  wg.Add(2)
  
  /* 创建两个子协程, 然后子协程运行结束后执行Done */
  go func() {
    defer wg.Done()
    fmt.Println("...")
  }()
  
  go func() {
    defer wg.Done()
    fmt.Println("....")
  }()
  
  /* main协程等待子协程 */
  wg.Wait()
}
```

这个`WaitGroup`可以写很多个, 最终的效果等于所有写的WaitGroup拼接起来.



### 子协程故障不导致主协程挂

如果要子协程发生`panic`后不让主协程挂掉, 只需要在可能会发生`panic`的子协程的函数最前面加上这句话:

```go
func f1() {
  
  /* 如果发生了panic, 那么最终还会执行这个defer */
  defer func() {
    
    err := recover()
    /* 如果err不是nil, 证明发生了panic, 后面是panic的处理逻辑 */
    if err != nil {
      fmt.Println(err)
    }
    
  }
  
}
```



## 临界区和锁

临界区(Critical Section): 临界区是指一个协程中的一段代码, 这段代码中涉及修改全局变量的部分.

需要保证: <font color=red>同一时间段内, 有且仅有一个协程能够进入它的临界区.</font>

实现这个原则可以通过两种方式:

* 对临界资源加锁, 可以使用互斥锁mutex, 在Go语言中是`sync.Mutex`.

  ```go
  import "sync"
  
  /* mutex锁是全局变量 */
  var lock = sync.Mutex{}
  
  func f1() {
    
    
    lock.lock()
    /* 临界区代码 */
    lock.unlock()
  }
  ```



## Channel

Channel的本质是一个固定长度的队列, 它的类型关键词是`chan type`, 其中`type`可以是`int`或者`struct{}`这种数据类型.

* Channel可以使用`make`创建:

```go
// 创建一个管道, 里面元素是bool类型, 大小是3
ch := make(chan bool, 3)
```

* 写Channel, 可以使用: `ch<-你要写的元素`

  * 注意: <font color=green>当你向Channel中写入元素后, 如果这个Channel原始为空, 那么它会唤醒所有等待这个Channel读取的协程, 相当于内置实现了生产者-消费者算法</font>.

* 读Channel, 可以使用`<-ch`.

  * 注意1: `<-ch`这个整体是一个表达式, 这个整体就代表从Channel中读取的队头元素.

  * 注意2:  此时需要处理异常:

    ```go
    a, ok := <-ch
    // 当且仅当ch被close, 并且ch为空时, ok才是false, 此时需要处理异常
    ```

  * 注意3: 当使用`<-ch`后, <font color=green>如果`ch`为空, 那么该协程会被阻塞, 直到有协程写入.</font>


* 关闭Channel可以使用: `close(ch)`, 注意: <font color=red>关闭Channel仅代表不可以向Channel中写入元素, 但是还可以读取</font>.



### 使用Channel实现主协程等待子协程

借助Channel可以阻塞协程的特性, 可以实现主协程等待一个子协程:

```go
func main() {
  
  /* 1. 创建一个没有buffer的Channel */
  ch := make(chan struct{}, 0)
  
  /* 一个子协程 */
  go func() {
    /* 协程代码 */
    
    /* 3. 需要被等待的子协程再结束后写入Channel, 唤醒main协程 */
    defer ch<-struct{}{}
  }
  
  /* 2. 读取Channel, 由于Channel容量为0, main协程会被阻塞 */
  <-ch
}
```

> 为什么Channel的数据类型是`struct{}` ?

因为`struct{}`这种类型在创建实例`struct{}{}`时, 并不实际占用内存.



> 什么是没有Buffer的Channel

如果使用`ch := make(chan struct{}, 0)`或者`ch := make(chan struct{})`创建一个Channel, 那么这个Channel就是没有Buffer的, 此时, <font color=red>向这个Channel中读/写元素都会导致Channel被阻塞.</font>



<font color=purple>因此, 借助这个思路, 对于每一个协程之间的先后关系, 都可以通过一个容量为0的Channel进行实现.</font>



## 死锁

### 例子1

```go
package main

import (
	"fmt"
)

func main() {
	
	ch := make(chan struct{}, 0)

	go func() {
		/* 向没有Buffer的Channel写入会阻塞协程 */
		ch <- struct{}{}
		fmt.Println("Hello")
	}()
	/* main协程也会被阻塞 */
	ch <- struct{}{}
}
```

在这个例子中, 子协程和main协程都会被永远阻塞, go语言会爆出deadlock的运行时异常.



### 例子2

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	
	ch := make(chan struct{}, 0)

	go func() {
		/* 向没有Buffer的Channel写入会阻塞协程 */
		ch <- struct{}{}
		fmt.Println("Hello")
	}()

	/* 增加一个子协程 */
	go func() {
		time.Sleep(1 * time.Hour)
	}()

	/* main协程也会被阻塞 */
	ch <- struct{}{}
}
```

这个例子在上面的基础上增加了一个休眠1小时的子协程, 运行这个代码, go语言居然就不会爆deadlock的Error了.

这是因为: 增加一个子协程后, 在运行时子协程休眠了, 但是Go语言不知道它后面会执行什么代码, 有可能要执行的代码会帮助解除main协程或前面协程的阻塞, 因此是有希望不出现deadlock的, 所以go语言没有爆deadlock.



因此:

<font color=purple>如果Go代码没有爆deadlock, 不代表它真的没有deadlock, Go语言会假设现在还没有结束的协程可能会帮助解决deadlock, 因此要注意代码逻辑.</font>
