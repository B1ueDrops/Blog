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

  

## package

* 假设你有一个文件夹`utils`, 里面所有的`.go`文件, 第一行都是`package utils`
* 里面的`.go`文件, 各种成分互相调用, 不用写`import`.
* 需要别的包调用的, 记得名字<font color=blue>首字母大写</font>, 不需要的小写.
* 在另外一个package中, 调用其他package的成分:
  * 写上`import "Go项目的名字/package文件夹的路径"`
  * `package`文件夹的路径就是相对于Go项目根目录的路径.



## 结构体

* 打印结构体用`%#v`.

## 哈希表

* 创建哈希表:

  ```go
  hashmap := make(map[string]int)
  ```

* 判断`key`是否存在: `ok`是一个bool值, true表示key存在

  ```go
  _, ok := hashmap[key]
  ```

  



## 随机数

* 生成`[0, n)`的随机整数:

  ```go
  import "math/rand"
  
  num := rand.Intn(n)
  ```

  