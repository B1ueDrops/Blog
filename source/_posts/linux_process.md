---
title: Linux上的进程控制
categories: UNIX编程
---



## 进程结构体`PCB`

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



## 创建进程`fork()`

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



## 进程执行`exec`

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



## 孤儿进程

> 孤儿进程是怎么产生的?

* 父进程创建了子进程, 子进程会和父进程同时运行.
* 如果父进程先于子进程结束, 那么子进程就没有父亲, 就会成为孤儿进程(Orphan Process).
* 在Linux中, 如果出现了孤儿进程, 会把孤儿进程的父进程重新设置:
  * 如果有终端, 那么父进程就是终端.
  * 如果没有终端, 那么父进程就是`init`进程(`pid = 1`).



> 为什么要给孤儿进程重新设置父进程?

* 子进程退出时, 用户区的内存可以自己释放, 但是内核区的PCB没有办法自己释放, 必须由父进程释放.

创建孤儿进程代码:

```cpp
#include <stdio.h>
#include <unistd.h>

int main() {

    pid_t pid = fork();

    if (pid > 0) {
        puts("I'm father");
    }
    else {
        // 子进程睡眠, 让父进程运行
        sleep(1);
        printf("Child PID: %d, Father PID: %d\n", getpid(), getppid());
    }
}
```

输出结果:

```
I'm father
Child PID: 78608, Father PID: 1
```



## 僵尸进程

> 僵尸进程是怎么产生的?

* 子进程先于父进程结束, 子进程的PCB等内核态资源需要父进程释放.
* 如果父进程由于某些原因无法释放, 那么子进程就成了僵尸进程 (进程结束了, 但是PCB等内核资源没办法释放).

> 产生僵尸进程代码?

```cpp
#include <stdio.h>
#include <unistd.h>

int main() {

    pid_t pid = fork();

    if (pid > 0) {
        // 父进程一直休眠
        while (1) {
            sleep(1);
        }
    }
    else {
        // 子进程直接退出, 成为僵尸进程
        printf("Child PID: %d, Father PID: %d\n", getpid(), getppid());
    }
}
```

> 杀死僵尸进程的方法?

* 杀死僵尸进程的父进程即可.
* 用`kill`无法杀死僵尸进程.



## 进程回收`wait`



### `wait`

为了避免僵尸进程的产生, 需要让父进程回收子进程资源, 这个回收方式分为两种类型:

* 阻塞回收.
* 非阻塞回收.

用`wait`函数可以实现对子进程的阻塞回收:

* 头文件: `#include <sys/wait.h>`

* 函数: `pid_t wait(int *status)`.
  * `status`是你传进去的变量, 这个变量会保存子进程退出的状态码, 如果不需要直接传`NULL`.
  * 返回值: 被回收的子进程的pid, 如果失败/没有子进程可以等待了, 就返回-1.

* 注意: 这个函数被调用一次, 只能回收一个子进程资源, 如果有多个子进程, 需要调用多次.

例子:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {

    pid_t pid;
    for (int i = 0; i < 5; i ++) {
      // 创建5个子进程
        pid = fork();
        if (pid == 0) {
            break;
        }
    }

    if (pid > 0) {
        while (1) {
            pid_t ret = wait(NULL);
          // 如果成功回收
            if (ret > 0) {
                printf("Child %d PCB freed\n", ret);
            }
          // 如果没子进程可等了, 就退出
            else {
                printf("Child process freed error\n");
                break;
            }
        }
    }
    else if (pid == 0) {
        printf("I m father process\n");
    }
    return 0;
}
```



### `waitpid`

* `waitpid`是`wait`的升级版本, 可以对回收行为做进一步的控制.
* `waitpid`也是在`<sys/wait.h>`中:
  * 函数签名: `pid_t waitpid(pid_t pid, int *status, int options)`:
    * `pid`: 
      * 小于`-1`: 回收进程组ID是`|pid|`的所有子进程.
      * `-1`: 和`wait()`一样, 无差别回收.
      * `0`: 回收当前进程组所有的子进程.
      * `> 0`: 指定回收进程ID是`pid`的进程.
    * `status`: 和`wait`一样.
    * `options`: 控制函数是否阻塞当前进程:
      * `0`: 阻塞.
      * `WNOHANG`: 非阻塞.
  * 返回值:
    * `0`: 函数非阻塞, 子进程还在运行.
    * `-1`: 回收失败.
    * 成功会得到回收的子进程的`pid`.



> 例子: 通过`waitpid`非阻塞回收子进程资源

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {

    pid_t pid;
    for (int i = 0; i < 5; i ++) {
      // 创建5个子进程
        pid = fork();
        if (pid == 0) {
            break;
        }
    }

    if (pid > 0) {
        while (1) {
            pid_t ret = waitpid(-1, NULL, WNOHANG);
          // 成功回收
            if (ret > 0) {
                printf("Child %d PCB freed\n", ret);
            }
          // 子进程还没退出
          	else if (ret == 0) {
              
            }
          // 回收失败
            else {
                printf("Child process freed error\n");
                break;
            }
        }
    }
    else if (pid == 0) {
        printf("I m father process\n");
    }
    return 0;
}
```

