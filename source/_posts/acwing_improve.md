---
title: AcWing算法提高课题解
categories: 算法
mathjax: true
---

[toc]



## 动态规划



### 背包模型

#### 采药

> https://www.acwing.com/problem/content/425/

* 01背包问题模板.

```cpp
#include <iostream>
using namespace std;
const int N = 1010;

int n, m, f[N];

int main() {
    
    cin >> m >> n;
    for (int i = 1; i <= n; i ++) {
        int v, w;
        cin >> v >> w;
        for (int j = m; j >= v; j --)
            f[j] = max(f[j], f[j - v] + w);
    }
    cout << f[m] << endl;
    return 0;
}
```



#### 装箱问题

> https://www.acwing.com/problem/content/description/1026/

* 01背包问题模板题.

```cpp
#include <iostream>
using namespace std;
const int N = 20010;

int n, m, f[N];

int main() {
    
    cin >> m >> n;
    for (int i = 1; i <= n; i ++) {
        int v;
        cin >> v;
        for (int j = m; j >= v; j --)
            f[j] = max(f[j], f[j - v] + v);
    }
    cout << m - f[m] << endl;
    return 0;
}
```



#### 宠物小精灵之收服

> https://www.acwing.com/problem/content/1024/

* 这个题考查多重限制下的01背包, 直接在第二重循环多加几个就可以.

```cpp
#include <iostream>
using namespace std;
const int N = 1010;

int n, m1, m2, f[N][N];

int main() {
    
    cin >> m1 >> m2 >> n;
    m2 --;
    for (int i = 1; i <= n; i ++) {
        int v1, v2;
        cin >> v1 >> v2;
        for (int j = m1; j >= v1; j --)
            for (int k = m2; k >= v2; k --)
                f[j][k] = max(f[j][k], f[j - v1][k - v2] + 1);
    }
    
    int k = m2;
    while (k >= 0 && f[m1][k] == f[m1][m2]) k --;
    k ++;
    

    cout << f[m1][m2] << ' ' << m2 - k + 1 << endl;
    return 0;
}
```



#### 数字组合

> https://www.acwing.com/problem/content/280/

* 首先大背景是01背包问题.

* 这个题有两个地方不同:
  * 第一, 要求与总体积相等.
  * 第二, 求选择方案, 不是求最大价值.
* 如果要求与总体积相等, 那么状态定义要发生改变, `f[i][j]`应该表示从前`i`个物品中选, 总体积恰好为`j`.
* 如果要求方案数, 那么状态转移方程是: `f[i][j] = f[i - 1][j] + f[i - 1][j - v[i]]`, 分别对应选/不选物品`i`.
  * 注意, 如果求方案数目, 需要初始化`f[0][0]`是1, 表示一个物品都不选, 总体积和为`0`有一种方案.

```cpp
#include <iostream>
using namespace std;
const int N = 10010;

int n, m, f[N];

int main() {
    
    cin >> n >> m;
    f[0] = 1;
    for (int i = 1; i <= n; i ++) {
        int v; cin >> v;
        for (int j = m; j >= v; j --)
            f[j] += f[j - v];
    }
    cout << f[m] << endl;
    return 0;
}
```





### 最长上升子序列模型

#### 怪盗基德的滑翔翼

