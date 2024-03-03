---
title: 背包问题的模板总结
categories: 算法
mathjax: true
---



## 01背包问题

> https://www.acwing.com/problem/content/2/

01背包问题是指一个背包只能是选/不选.

状态表示: $f(i, j)$: 只考虑前$i$个物品, 总体积是$j$的情况下, 总重量的最大值.

状态计算:

* 不选第$i$个物品: $f(i, j) = f(i - 1, j)$
* 选第$i$个物品: $f(i, j) = f(i - 1, j - v[i]) + w[i]$

那么状态转移是: $f(i, j) = max(f(i-1, j), f(i-1, j-v[i]) + w[i])$

初始化: $f(0, 0) = 0$.

时间复杂度是$O(nm)$, 其中$n$是物品个数, $m$是物品体积.

```cpp
#include <iostream>

using namespace std;

const int N = 1010;

int n, m;
int f[N][N];

int main()
{
    cin >> n >> m;
    
    for (int i = 1; i <= n; i ++)
    {
        int v, w;
        cin >> v >> w;
        for (int j = 0; j <= m; j ++)
        {
            if (j < v) f[i][j] = f[i - 1][j];
            else f[i][j] = max(f[i - 1][j], f[i - 1][j - v] + w);
        }
    }
    
    cout << f[n][m] << endl;
    
    return 0;
}
```

优化:

```cpp
#include <iostream>

using namespace std;

const int N = 1010;

int n, m;
int f[N];

int main()
{
    cin >> n >> m;
    
    for (int i = 1; i <= n; i ++)
    {
        int v, w;
        cin >> v >> w;
        
        for (int j = m; j >= v; j --)
            f[j] = max(f[j], f[j - v] + w);
    }
    
    cout << f[m] << endl;
    
    return 0;
}
```

同类题:

* https://www.acwing.com/problem/content/425/

## 完全背包问题

> https://www.acwing.com/problem/content/3/

完全背包问题是指, 一个背包可以选任意多个.

状态表示: $f(i, j)$, 只从前$i$个物品中选, 总体积不超过$j$的方案的集合.

状态计算:

* 不选: $f(i-1, j)$
* 选一个: $f(i-1, j-v) + w$
* 选两个: $f(i-1, j-2v) + 2w$
* ....

$f(i, j)$就是上述所有状态取一个max.

优化: 观察如下两个等式:

```
f[i][j] =   max(f[i-1][j], f[i-1][j-v]+w, f[i-1][j-2v]+2w, ...)
f[i][j-v] = max(           f[i-1][j-v],   f[i-1][j-2v]+w,  ...)
```

因此, $f(i, j) = max(f(i-1, j), f(i, j - v)+w)$

时间复杂度还是$O(nm)$.

优化: 

```cpp
#include <iostream>

using namespace std;

const int N = 1010;

int n, m;
int f[N];

int main()
{
    cin >> n >> m;
    
    for (int i = 1; i <= n; i ++)
    {
        int v, w;
        cin >> v >> w;
        for (int j = v; j <= m; j ++)
        {
            f[j] = max(f[j], f[j - v] + w);
        }
    }
    
    cout << f[m] << endl;
    return 0;
}
```



## 多重背包问题



### 朴素做法

> https://www.acwing.com/problem/content/4/

时间复杂度是$O(nms)$

```cpp
#include <iostream>

using namespace std;

const int N = 110;

int n, m;
int f[N][N];

int main()
{
    cin >> n >> m;
    
    for (int i = 1; i <= n; i ++)
    {
        int v, w, s;
        cin >> v >> w >> s;
        for (int j = 0; j <= m; j ++)
        {
            for (int k = 0; k <= s && k * v <= j; k ++)
            {
                f[i][j] = max(f[i][j], f[i - 1][j - k * v] + k * w);
            }
        }
    }
    
    cout << f[n][m] << endl;
    
    return 0;
}
```



### 二进制优化

> https://www.acwing.com/problem/content/5/

对于一个具体的背包, 假设它的个数是$s$.

那么, 这个$s$可以被不同的2的整数次幂进行表示, 也就是说$s = c_0 \times 2^{0} + c_1 \times 2^1 + ...$, 其中系数$c_0, c_1, ... \in \{0, 1\}$

那么, 我如果把这$s$个背包, 按照每2的整数幂进行一次打包, 那么完全背包问题就转化为了01背包问题.

时间复杂度是$O(nmlog(s))$

```cpp
#include <iostream>

using namespace std;

const int N = 2010;

int n, m;
int f[N];

int main()
{
    cin >> n >> m;
    
    for (int i = 0; i < n; i ++)
    {
        int v, w, s;
        cin >> v >> w >> s;
        
        for (int k = 1; k < s; k *= 2)
        {
            for (int j = m; j >= k * v; j --)
            {
                f[j] = max(f[j], f[j - k * v] + k * w);
            }
            s -= k;
        }
        
        if (s)
        {
            for (int j = m; j >= s * v; j --)
                f[j] = max(f[j], f[j - s * v] + s * w);
        }
    }
    
    cout << f[m] << endl;
    return 0;
}
```



## 分组背包问题

> https://www.acwing.com/problem/content/description/9/

在分组背包问题中, 不再有$n$个物品, 而是有$n$组物品, 每组物品中只能选择一个物品.
时间复杂度是$O(nms)$, 其中`s`是一个组中的物品个数.
```cpp
#include <iostream>

using namespace std;

const int N = 110;

int n, m;
int v[N][N], w[N][N], s[N];
int f[N];

int main()
{
    cin >> n >> m;
    
    for (int i = 1; i <= n; i ++)
    {
        cin >> s[i];
        for (int j = 0; j < s[i]; j ++)
            cin >> v[i][j] >> w[i][j];
    }
    
    for (int i = 1; i <= n; i ++)
    {
        for (int j = m; j >= 0; j --)
        {
            // 注意, 这里k一定要从0开始
            for (int k = 0; k < s[i]; k ++)
            {
                if (j >= v[i][k])
                    f[j] = max(f[j], f[j - v[i][k]] + w[i][k]);
            }
        }
    }
    
    cout << f[m] << endl;
    return 0;
}
```



## 装箱问题

> https://acwing.com/problem/content/1026/

这个问题中, 背包的价值等于体积, 也就是说, 在背包的总体积不超过$V$的情况下, 总体积之和最大.

```cpp
#include <iostream>

using namespace std;

const int N = 20010;

int n, m;
int f[N];

int main()
{
    cin >> m;
    cin >> n;
    
    for (int i = 1; i <= n; i ++)
    {
        int v;
        cin >> v;
        for (int j = m; j >= v; j --)
        {
            f[j] = max(f[j], f[j - v] + v);
        }
    }
    
    cout << m - f[m] << endl;
    return 0;
}
```



## 宠物小精灵之收服

> https://www.acwing.com/problem/content/1024/

在本题中有两种限制:

* 收服精灵用的精灵球小于等于上限.
* 扣的血必须小于上限, 注意这里不一样, 必须给皮卡丘留1点血.

在这两种限制之下要求能够收服的精灵数的最大值.

```cpp
#include <iostream>

using namespace std;

const int N = 1010;

int n, m1, m2;
int f[N][N];

int main()
{
    cin >> m1 >> m2 >> n;
    
    for (int i = 1; i <= n; i ++)
    {
        int v1, v2;
        cin >> v1 >> v2;
        
        for (int j1 = m1; j1 >= v1; j1 --)
        {
            for (int j2 = m2; j2 >= v2; j2 --)
            {
                f[j1][j2] = max(f[j1][j2], f[j1 - v1][j2 - v2] + 1);
            }
        }
    }
    /* 注意这里皮卡丘的血必须大于0 */
    int ans = f[m1][m2 - 1];
    cout << ans << ' ';
    
    int index = m2 - 1;
    while (index >= 0 && f[m1][index] == ans) index --;
    index ++;
    
    cout << m2 - index << endl;
    return 0;
}
```



## 数字组合

> https://www.acwing.com/problem/content/280/

本题是01背包问题求方案数.

状态表示: $f(i, j)$表示只考虑前$i$个物品, 物品体积之和正好是$j$的方案数.

状态计算:

* 不考虑第$i$个物品: `f[i][j] += f[i - 1][j]`.
* 考虑第$i$个物品: `f[i][j] += f[i - 1][j - v[i]]`

本质上和原来的01背包问题相同:

```cpp
#include <iostream>

using namespace std;

const int N = 110, M = 10010;

int n, m;
int f[M];

int main()
{
    cin >> n >> m;
    // 在前0个数中, 选择和为0, 方案是1 
    f[0] = 1;
    
    for (int i = 1; i <= n; i ++)
    {
        int v;
        cin >> v;
        
        for (int j = m; j >= v; j --)
            f[j] += f[j - v];
    }
    
    cout << f[m] << endl;
    return 0;
}
```



## 买书

> https://www.acwing.com/problem/content/1025/

本题是完全背包求方案数.

注意, 求方案数一般都需要使用`long long`.

``` cpp
#include <iostream>

using namespace std;

typedef long long LL;

const int N = 1010;

int n;
int a[4] = { 10, 20, 50, 100 };

LL f[N];

int main()
{
    cin >> n;
    
    f[0] = 1;
    
    for (int i = 0; i < 4; i ++)
    {
        for (int j = a[i]; j <= n; j ++)
        {
            f[j] += f[j - a[i]];
        }
    }
    
    cout << f[n] << endl;
    
    return 0;
}
```

同类题: 货币系统: https://www.acwing.com/problem/content/description/1023/

```cpp
#include <iostream>

using namespace std;

const int N = 3010;

typedef long long LL;

int n, m;
LL f[N];

int main()
{
    cin >> n >> m;
    
    f[0] = 1;
    
    for (int i = 1; i <= n; i ++)
    {
        int v;
        cin >> v;
        
        for (int j = v; j <= m; j ++)
        {
            f[j] += f[j - v];
        }
    }
    
    cout << f[m] << endl;
    return 0;
}
```



## 货币系统

> https://www.acwing.com/problem/content/534/

题目中说, 货币种数是$n$, 面额数值是$a[1..n]$的货币系统是$(n, a)$.

本质上就是说, 这个货币系统有$n$个基向量, 这$n$个基向量形成了一个空间, 但是要求这$n$个基向量的系数应该大于等于0.

现在这个题, 本质上就是求解最大无关向量组的个数.

其实这个题, 可以转化为完全背包问题求方案数:

* 总体积可以设置为$a$中的最大值.
* 然后利用完全背包问题求方案数, 可以求出当体积为一个定值$v$的时候, 有多少种方式可以表示.
* 之后, 我可以看数组$a$, 如果数组$a$中的一个元素只能有一种方式表示(方案数是1), 那么这个就是一个独立的基向量.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 110, M = 25010;

typedef long long LL;

int n, m;
int a[N];
LL f[M];

int main()
{
    int T;
    
    scanf("%d", &T);
    
    while (T --)
    {
        memset(f, 0, sizeof f);
        memset(a, 0, sizeof a);
        
        scanf("%d", &n);
        
        m = -1;
        for (int i = 1; i <= n; i ++)
        {
            scanf("%d", &a[i]);
            m = max(m, a[i]);
        }
        
        f[0] = 1;
        
        for (int i = 1; i <= n; i ++)
        {
            for (int j = a[i]; j <= m; j ++)
            {
                f[j] += f[j - a[i]];
            }
        }
        
        int ans = 0;
        for (int i = 1; i <= n; i ++)
            if (f[a[i]] == 1) ans ++;
        
        cout << ans << endl;
    }
    return 0;
}
```

