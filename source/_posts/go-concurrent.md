---
title: 使用goroutine实现并发算法题
categories: 编程语言
---



## 协程顺序

> https://leetcode.cn/problems/print-in-order/description/

直接使用3个Channel, 分别控制:

* f1和f2的先后关系.
* f2和f3的先后关系.
* f3和main的先后关系.

```go
package main

import "fmt"

var c1 = make(chan struct{})
var c2 = make(chan struct{})
var c3 = make(chan struct{})

func f1() {
	fmt.Println("first")
	/* f1执行完毕，解除f2的阻塞 */
	c1 <- struct{}{}
}

func f2() {
	/* 等待f1 */
	<-c1
	fmt.Println("second")
	/* f2执行完毕， 解除f3的阻塞 */
	c2 <- struct{}{}
}

func f3() {
	/* 等待f2 */
	<-c2
	fmt.Println("third")
	/* f3执行完毕， 解除main的阻塞 */
	c3 <- struct{}{}
}


func main() {
	
	routineMap := map[int]func() {
		1: f1,
		2: f2,
		3: f3,
	}

	for _, f := range routineMap {
		go f()
	}

  defer close(c1)
  defer close(c2)
  defer close(c3)
  
	<-c3
	
}

```

<font color=red>注意: 一定要用`close`关掉Channel !!!, 还要注意要多定义一个Channel用来阻塞main协程.</font>



## 协程交替打印

> https://leetcode.cn/problems/print-foobar-alternately/description/

实现方式非常简单, 使用两个Channel:

* 一个Channel容量是1, 里面里面有一个元素, 叫c1
* 一个Channel容量是0, 叫c2

对于需要交替执行的两个Part (Part1和Part2):

* Part1先从c1中拿到元素, 然后执行, 执行完一次后解除c2阻塞.
* Part2解除阻塞后执行, 执行完之后解除c1的阻塞.
* 然后循环.

```go
package main

import (
	"fmt"
	"sync"
)

/* 创建两个Channel */
var c1 = make(chan struct{}, 1)
var c2 = make(chan struct{})


func main() {
	/* 初始化c1有一个元素 */
	c1 <- struct{}{}
	
	/* 创建WaitGroup */
	wg := sync.WaitGroup{}
	wg.Add(2)
	

	var n int

	/* 从标准输入读入n */
	fmt.Scanf("%d", &n)

	/* 开启协程 */
	go func() {
		for i := 0; i < n; i ++ {
			<-c1
			fmt.Print("Foo")
			c2 <- struct{}{}
		}
		wg.Done()
	}()

	go func() {
		for i := 0; i < n; i ++ {
			<-c2
			fmt.Print("Bar")
			c1 <- struct{}{}
		}
		wg.Done()
	}()

	/* 延迟关闭管道 */
	defer close(c1)
	defer close(c2)


	wg.Wait()
}
```



## 协程屏障

> https://leetcode.cn/problems/building-h2o/description/