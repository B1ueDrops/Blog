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
* https://www.acwing.com/problem/content/428/

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



同类题: https://www.acwing.com/problem/content/1021/



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



## 混合背包问题

> https://www.acwing.com/problem/content/7/

在混合背包问题中, 对于一个物品:

* 它可能最多用一次(01背包).
* 可能可以用无数次(完全背包).
* 可能最多用`s`次(多重背包).

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
        int v, w, s;
        
        cin >> v >> w >> s;
        
        if (s == -1)
        {
            for (int j = m; j >= v; j --)
            {
                f[j] = max(f[j], f[j - v] + w);
            }
        }
        else if (s == 0)
        {
            for (int j = v; j <= m; j ++)
            {
                f[j] = max(f[j], f[j - v] + w);
            }
        }
        else
        {
            for (int k = 1; k <= s; k *= 2)
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
                {
                    f[j] = max(f[j], f[j - s * v] + s * w);
                }
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

同类题: 二维费用背包(https://www.acwing.com/problem/content/8/)



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



## 潜水员

> https://www.acwing.com/problem/content/1022/

这个题的背景是:

* 二维费用.
* 体积要求的是不少于, 而不是不超过.
* 求的是价值的最小值.

需要的改变是:

* 初始化时, 需要用`0x3f`初始化`f`数组, 并且把初始状态`f[0][0]`设置成0.
* 转移状态时用的是`min`.
* 注意: 体积为负数的时候(例如`j - v1 < 0`), 状态其实是有意义的, 因为要求体积大于等于一个负值是有意义的, 但是, 因为题目中能够确定每一个体积都是大于0的, 因此如果一个状态要求体积大于等于负数, 那么就等价于要求体积大于等于0.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 110;

int n, m1, m2;
int f[N][N];

int main()
{
    cin >> m1 >> m2;
    cin >> n;
    
    memset(f, 0x3f, sizeof f);
    f[0][0] = 0;
    
    for (int i = 0; i < n; i ++)
    {
        int v1, v2, w;
        cin >> v1 >> v2 >> w;
        
        for (int j = m1; j >= 0; j --)
        {
            for (int k = m2; k >= 0; k --)
            {
                f[j][k] = min(f[j][k], f[max(0, j - v1)][max(0, k - v2)] + w);
            }
        }
    }
    
    cout << f[m1][m2] << endl;
    return 0;
}
```

从这个题中, 可以总结出:

* 如果题目要求体积最多是`j`:  那么`f`初始化都是0, 要求`j - v`必须大于等于0.
* 如果题目要求体积恰好是`j`: 那么`f`初始化为`+∞`, `f[0] = 0`, 要求`j - v`必须大于等于0.
* 如果题目要求体积至少是`j`: 那么`f`初始化为`+∞`, `f[0] = 0`, `j - v`可以小于0.



## 机器分配

> https://www.acwing.com/problem/content/1015/

这个题目第一问是分组背包问题, 第二问是分组背包问题求方案数.

在这个题中, 求方案数目等价于:

* 已知`f[n][m]`, 求最优方案中, 每一个机器的体积是多少?

假设在最优方案中, 第`i`个机器的体积是`v`, 那么有如下特征:

* `f[i][m] = f[i - 1][m - v] + w`.

因此, 我只需要从将`i`从后向前遍历, 把体积`v`从前向后遍历, 只要这个体积`v`有`f[i][m] = f[i-1][m-v]+w`, 那么这个`v`就是最小体积.

* 之后再将体积减去`m -= v`即可.

```cpp
#include <iostream>

using namespace std;

const int N = 110;

int n, m;
int v[N][N], w[N][N];
// 这里二维是要求方案
int f[N][N];

int main()
{
    cin >> n >> m;
    
    for (int i = 1; i <= n; i ++)
    {
        // 注意这里一定要是1, 和一般的分组背包不同
        for (int j = 1; j <= m; j ++)
        {
            cin >> w[i][j];
            v[i][j] = j;
        }
    }
    
    for (int i = 1; i <= n; i ++)
    {
        // 注意j和k一定要从0开始, 因为设备数量是0也有意义
        for (int j = m; j >= 0; j --)
        {
            for (int k = 0; k <= m; k ++)
            {
                if (j >= v[i][k])
                {
                    f[i][j] = max(f[i][j], f[i - 1][j - v[i][k]] + w[i][k]);
                }
            }
        }
    }
    
    cout << f[n][m] << endl;
    
    int k = m;
    
    for (int i = n; i >= 1; i --)
    {
        for (int j = 0; j <= m; j ++)
        {
            if (k >= v[i][j])
            {
                if (f[i][k] == f[i - 1][k - v[i][j]] + w[i][j])
                {
                    cout << i << ' ' << v[i][j] << endl;
                    k -= v[i][j];
                    break;
                }
            }
        }
    }
    return 0;
}
```



## 01背包求方案(字典序最小)

> https://www.acwing.com/problem/content/description/12/

思路和上一题类似, 只需要倒过来递推就可以保证字典许最小.

```cpp
#include <iostream>

using namespace std;

const int N = 1010;

int n, m;
int v[N], w[N];
int f[N][N];

int main()
{
    cin >> n >> m;
    
    for (int i = 1; i <= n; i ++) cin >> v[i] >> w[i];
    
    for (int i = n; i >= 1; i --)
    {
        // 朴素写法
        for (int j = 0; j <= m; j ++)
        {
            if (j < v[i]) f[i][j] = f[i + 1][j];
            else f[i][j] = max(f[i + 1][j], f[i + 1][j - v[i]] + w[i]);
        }
    }
    
    int k = m;
    for (int i = 1; i <= n; i ++)
    {
        if (k >= v[i] && f[i][k] == f[i + 1][k - v[i]] + w[i])
        {
            cout << i << ' ';
            k -= v[i];
        }
    }
    return 0;
}
```



## 01背包求方案数

> https://www.acwing.com/problem/content/11/

本质上, DP问题都可以转化为最短路问题, 而求DP问题最优方案的方案数, 可以转化为求最短路的条数. 

在最短路问题中, 对于一个节点, 假设它有一个属性叫做`cnt[i]`, 表示到这个点`i`的最短路的条数.

那么对于下一个点`j`, 只需要遍历它的相邻节点`i`, 然后累加: `cnt[j] += cnt[i]`即可.

```cpp
#include <iostream>

using namespace std;

const int N = 1010, mod = 1e9 + 7;

int n, m;
int f[N], g[N];

int main()
{
    cin >> n >> m;
    
    g[0] = 1;
    
    for (int i = 1; i <= n; i ++)
    {
        int v, w;
        cin >> v >> w;
        
        for (int j = m; j >= v; j --)
        {
        
            int maxv = max(f[j], f[j - v] + w);
        
            int cnt = 0;
            if (maxv == f[j]) cnt += g[j];
            if (maxv == f[j - v] + w) cnt += g[j - v];
        
            g[j] = cnt % mod;
            f[j] = maxv;
        }
    }
    
    int res = 0;
    
    for (int i = 0; i <= m; i ++)
        if (f[i] == f[m])
            res = (res + g[i]) % mod;
    
    cout << res << endl;
    
    return 0;
}
```



## 有依赖的背包问题

> https://www.acwing.com/problem/content/10/

状态表示:

* $f(u, j)$: 所有以$u$为根(包含$u$)的子树中选, 且总体积不超过$j$​的方案.
* 价值的最大值.

状态计算:

* 首先循环每一个子树, 对于一个子树, 用体积划分, 我给他分配一个体积`k`, 那么这颗子树能给我贡献的价值是多少, 累加起来.
* $f(u, j) = max(f(u, j), f(u, j - k) + f(son, k))$

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 110;

int n, m;
int v[N], w[N];
int h[N], e[N], ne[N], idx;
int f[N][N];

void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void dfs(int u)
{
    for (int i = h[u]; i != -1; i = ne[i])
    {
        int son = e[i];
        dfs(son);
        
        // 注意体积从m - v[u]开始
        for (int j = m - v[u]; j >= 0; j --)
            for (int k = 0; k <= j; k ++)
                f[u][j] = max(f[u][j], f[u][j - k] + f[son][k]);
    }
    
    // 刚才没遍历物品u, 现在加进去
    for (int j = m; j >= v[u]; j --) f[u][j] = f[u][j - v[u]] + w[u];
    for (int j = 0; j < v[u]; j ++) f[u][j] = 0;
}

int main()
{
    memset(h, -1, sizeof h);
    int root;
    
    cin >> n >> m;
    for (int i = 1; i <= n; i ++)
    {
        int p;
        cin >> v[i] >> w[i] >> p;
        
        if (p == -1) root = i;
        else add(p, i);
    }
    
    dfs(root);
    
    cout << f[root][m] << endl;
    
    return 0;
}
```



## 金明的预算方案

> https://www.acwing.com/problem/content/description/489/

这是另一种有依赖的背包问题.

这种问题有两个特征:

* 有很多个物品, 对于每一个物品来说, 它有很多子树依赖, 但是不同物品之间的依赖独立, 也就是说它们的依赖之间不连通.
* 对于一个物品来说, 它的子树个数不多, 可以枚举.

那么可以将一个物品的依赖看作一个物品组, 比如买物品`p`, 它有`i, j`两个依赖(如果买`i, j`必须买`p`), 那么这个物品组就包含:

* `p`.
* `p, i`.
* `p, j`.
* `p, i, j`.

并且这四种选择之间互相独立, 就可以转化为分组背包问题. 

枚举物品组时可以使用二进制枚举.

```cpp
#include <iostream>
#include <vector>

using namespace std;

const int N = 32010;

typedef pair<int, int> PII;

int n, m;
int f[N];

PII master[N];
vector<PII> slave[N];

int main()
{
    cin >> m >> n;
    
    for (int i = 1; i <= n; i ++)
    {
        int v, p, q;
        cin >> v >> p >> q;
        
        p *= v;
        
        if (q == 0) master[i] = {v, p};
        else slave[q].push_back({v, p});
    }
    
    for (int i = 1; i <= n; i ++)
    {
        for (int j = m; j >= 0; j --)
        {
            // 枚举每一个物品组
            for (int k = 0; k < 1 << slave[i].size(); k ++)
            {
                int v = master[i].first;
                int w = master[i].second;
                for (int u = 0; u < slave[i].size(); u ++)
                {
                    if (k >> u & 1) {
                        v += slave[i][u].first;
                        w += slave[i][u].second;
                    }
                }
                if (j >= v) f[j] = max(f[j], f[j - v] + w);
            }
        }
    }
    
    cout << f[m] << endl;
    return 0;
}

```







## 多重背包问题(单调队列)

使用单调队列可以在$O(nm)$的时间复杂度内解决多重背包问题.
