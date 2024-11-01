---
title: AcWing算法基础课题解
categories: 算法
mathjax: true
---

[toc]





## 动态规划

### 背包问题

#### 01背包问题

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

int n, m, f[N];

int main() {
    
    cin >> n >> m;
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



#### 完全背包问题

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

```cpp
#include <iostream>
using namespace std;
const int N = 1010;

int n, m, f[N];

int main() {
    
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) {
        int v, w;
        cin >> v >> w;
        for (int j = v; j <= m; j ++)
            f[j] = max(f[j], f[j - v] + w);
    }
    cout << f[m] << endl;
    return 0;
}
```



#### 多重背包问题

> https://www.acwing.com/problem/content/4/

* 朴素做法: 枚举物品, 体积和每个物品选的次数.

```cpp
#include <iostream>
using namespace std;
const int N = 110;

int n, m, f[N][N];

int main() {
    
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) {
        int v, w, s;
        cin >> v >> w >> s;
        for (int j = 0; j <= m; j ++) {
            for (int k = 0; k <= s && k * v <= j; k ++)
                f[i][j] = max(f[i][j], f[i - 1][j - k * v] + k * w);
        }
    }
    cout << f[n][m] << endl;
    return 0;
}
```



#### 多重背包问题II

> https://www.acwing.com/problem/content/5/

对于一个具体的背包, 假设它的个数是$s$.

那么, 这个$s$可以被不同的2的整数次幂进行表示, 也就是说$s = c_0 \times 2^{0} + c_1 \times 2^1 + ...$, 其中系数$c_0, c_1, ... \in \{0, 1\}$

那么, 我如果把这$s$个背包, 按照每2的整数幂进行一次打包, 那么完全背包问题就转化为了01背包问题.

时间复杂度是$O(nmlog(s))$

```cpp
#include <iostream>
using namespace std;
const int N = 3010;

int n, m, f[N];

int main() {
    
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) {
        int v, w, s;
        cin >> v >> w >> s;
        for (int k = 1; k <= s; k *= 2) {
            for (int j = m; j >= k * v; j --)
                f[j] = max(f[j], f[j - k * v] + k * w);
            s -= k;
        }
        if (s) {
            for (int j = m; j >= s * v; j --)
                f[j] = max(f[j], f[j - s * v] + s * w);
        }
    }
    cout << f[m] << endl;
    return 0;
}
```



#### 分组背包问题

> https://www.acwing.com/problem/content/description/9/

在分组背包问题中, 不再有$n$个物品, 而是有$n$组物品, 每组物品中只能选择一个物品.

本质上和01背包问题没有区别, 只是需要枚举一下$n$组物品中, 每一组需要选哪一个.

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
      	// s[i]表示第i组一共有多少个物体
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



### 线性DP



#### 最长上升子序列

> https://www.acwing.com/problem/content/897/

* 最长上升子序列的DP做法, 时间复杂度是$O(n^2)$.
* 假设`f[i]`表示以`a[i]`结尾的最长上升子序列的长度.
  * 那么首先, 一个元素也可以叫最长上升子序列, 那么初始化就是1.
  * 其次, 如果`j < i`并且`a[j] < a[i]`, 那么`f[i] = f[j] + 1`.

```cpp
#include <iostream>
using namespace std;
const int N = 1010;

int n, a[N], f[N];

int main() {
    
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> a[i];
    for (int i = 0; i < n; i ++) {
        f[i] = 1;
        for (int j = 0; j < i; j ++)
            if (a[j] < a[i])
                f[i] = max(f[i], f[j] + 1);
    }
    int res = 0;
    for (int i = 0; i < n; i ++)
        res = max(res, f[i]);
    
    cout << res << endl;
    return 0;
}
```



#### 最长上升子序列II

> https://www.acwing.com/problem/content/898/

* 最长上升子序列的优化做法, 时间复杂度是$O(nlogn)$.
* 在$O(n^2)$的做法中, 有一步是找到`a[j] < a[i]`, 以`a[j]`结尾的, 最长上升子序列的长度值.
* 用一个数组`q[len]`存储长度是`len`的最长上升子序列的最后一个元素.
* 每次用`a[i]`对`q`数组进行二分, 找到`q`值小于`a[i]`, 但是下标最大的这个下标`r`, 这个就是要转移到`f[i]`的前一个状态`f[j]`, 直接+1就可以.

```cpp
#include <iostream>
using namespace std;
const int N = 100010;

int n, a[N], q[N];

int main() {
    
    cin >> n;
    for (int i = 1; i <= n; i ++) cin >> a[i];
    
    int len = 0;
    for (int i = 1; i <= n; i ++) {
        int l = 0, r = len;
        while (l < r) {
            int mid = l + (r - l) / 2 + 1;
            if (q[mid] < a[i]) l = mid;
            else r = mid - 1;
        }
        len = max(len, r + 1);
        q[r + 1] = a[i];
    }
    cout << len << endl;
    return 0;
}
```





#### 最短编辑距离

> https://www.acwing.com/problem/content/description/904/

* 假设`f[i][j]`表示从`word1[1:i]`转换为`word2[1:j]`的最小编辑次数, 针对`word1[i]`的操作, 状态转移分为三种情况:
  * 增加`word1[i]`: 那么就要求`word[1:i]`和`word2[1:j-1]`匹配.
  * 删除`word1[i]`: 要求`word[1:i-1]`和`word2[1:j]`匹配
  * 替换`word1[i]`: 要求`word[1:i-1]`和`word2[1:j-1]`匹配, 并且`word1[i] == word2[j]`.
    * 如果`word1[i] == word2[j]`, 那么就可以直接从`f[i - 1][j - 1]`转移.
* 注意初始化`f[i][0]`和`f[0][i]`为`i`.

```cpp
#include <iostream>
using namespace std;
const int N = 1010;

int n, m, f[N][N];
char a[N], b[N];

int main() {
    
    cin >> n >> a + 1;
    cin >> m >> b + 1;
        
    for (int i = 0; i <= n; i ++) f[i][0] = i;
    for (int i = 0; i <= m; i ++) f[0][i] = i;
    
    for (int i = 1; i <= n; i ++)
        for (int j = 1; j <= m; j ++) {
            f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1);
            if (a[i] != b[j]) f[i][j] = min(f[i][j], f[i - 1][j - 1] + 1);
            else f[i][j] = min(f[i][j], f[i - 1][j - 1]);
        }
    cout << f[n][m] << endl;
    return 0;
}
```

