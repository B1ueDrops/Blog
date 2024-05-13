---
title: Linux上的进程控制
categories: UNIX编程
---



## PCB

Linux上的PCB (Process Control Block) 本质上是一个叫做`task_struct`的结构体, 需要包含以下信息:

* `pid`: 进程的唯一ID.
* 进程状态(创建态, 就绪态, 运行态, 阻塞态, 终止态).
* 虚拟地址空间信息.
* 绑定的终端信息 (环境变量等).
* `cwd`.
* `unmask`: 进程创建文件时, 对于这个文件的操作权限.
* 文件描述符表.
* 和信号相关的信息.
* 用户id, 用户组id.
* 进程使用资源上限, 通过`ulimit -a`可以查看.



## 创建进程

* 头文件: `#include <unistd.h>`

* 函数: `pid_t fork(void)`, 其中`pid_t`是进程ID的数据类型.
  * 行为: 将父进程拷贝, 生成子进程, 父进程返回子进程的pid, 子进程返回0, 创建失败返回-1, 子进程从`fork()`后面开始执行.
    * 默认创建的父进程和子进程的优先级是一样的.

```c
#include <cstdio>
#include <unistd.h>

int main() {

    pid_t pid = fork();

    if (pid > 0) {
        printf("Father process with %d\n", pid);
    }
    else {
        puts("Child process");
    }
}
```

* 底层实现: 
  * 父进程和子进程中, 用户区的数据(`.text`, `.data`这些段, 栈, 堆等)是完全一样的, 环境变量, 文件描述符表也一样.
  * 父进程和子进程的内存区域遵循**读时共享, 写时复制(Copy on Write)**.
  * 内核区存储的父进程/子进程的id是不一样的.

一个例子: 下面这个程序创建了多少个进程?

```c
#include <cstdio>
#include <unistd.h>

int main() {
    for (int i = 0; i < 3; i ++) {
        pid_t t = fork();
        printf("Current pid is %d\n", getpid());
    }
}
```

> `getpid()`能获取当前进程的pid.

```c
Current pid is 35943 <-
Current pid is 35944 <-
Current pid is 35943
Current pid is 35945 <-
Current pid is 35943
Current pid is 35944
Current pid is 35945
Current pid is 35947 <-
Current pid is 35944
Current pid is 35948 <-
Current pid is 35946 <-
Current pid is 35949 <-
Current pid is 35946
Current pid is 35950 <-
```

一共8个进程.

假设`for (int i = 0; i < 3; i ++)`改成`for (int i = 0; i < n; i ++)`, 那么会创建多少个进程?

* $2^n$.

* 注意, Linux中, 两个进程不能通过全局变量来进行交互.



## 进程执行可执行文件

进程控制一般用到`execl`和`execlp`这两个函数, 两个函数都在`<unistd.h>`中:

* `int execl(const char *path, const char *arg, ...);`
  * `path`是可执行文件的路径, 推荐绝对路径.
  * `arg`是可执行文件的名字.
  * `...`是执行命令用的参数, 可以写多个, 最后用`NULL`结尾.
  * 返回值: 成功执行没有返回值, 执行失败返回`-1`.
* `int execlp(const char *file, const char *arg, ...);`
  * `file`: 可执行程序的名字, 如果可执行文件在`PATH`中, 就不用指定.

> 例子: 在子进程中执行`ls -l`

```c
#include <stdio.h>
#include <unistd.h>

int main() {

    pid_t pid = fork();

    if (pid > 0) {
        printf("I'm father");
    }
    else {
        execlp("ls", "-l", NULL);
    }
}
```

