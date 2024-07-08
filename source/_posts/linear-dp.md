---
title: 线性DP
categories: 算法
mathjax: true
---



[toc]

## 最大子数组和

> https://leetcode.cn/problems/maximum-subarray/
>
> https://www.acwing.com/problem/content/description/50/

假设$f[i]$表示$[0, i]$中, 所有子数组的最大和, 那么状态转移为: $f[i] = max(f[i - 1] + nums[i], nums[i])$, 递推一遍, 时间复杂度是$O(n)$.

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int tmp = nums[0];
        int ans = nums[0];

        for (int i = 1; i < nums.size(); i ++) {
            tmp = max(tmp + nums[i], nums[i]);
            ans = max(ans, tmp);
        }
        return ans;
    }
};
```



## RMQ模板题

> https://www.acwing.com/problem/content/1275/

RMQ (Range Max/Min Query)问题可以用动态规划解决, 预处理时间是$O(nlogn)$, 查询时间是$O(1)$.

设$f(i, j)$表示以$i$开头, 长度是$2^j$的区间中的最大值, 那么状态转移方程是:
$$
f(i, j) = max\{f(i, j - 1), f(i + 2^{j-1}, j-1)\}
$$
假设查询的区间是$[l, r]$:

* 首先我需要找到一个$k$, 使得$2^{k} \leqslant r - l + 1$, 并且 $ 2^{k+1} > r - l + 1$.
* 然后, 要找的最大值就是$max\{f(l, k), f(r - 2^k + 1, k)\}$, 这两个区间一定能覆盖$[l, r]$​.
* $k = \lfloor log_{2}(r-l+1) \rfloor = \lfloor \frac{log_{10}(r - l + 1)}{log_{10}(2)} \rfloor $

```cpp
#include <iostream>
#include <cmath>

using namespace std;

// M = log_2(N)
const int N = 200010, M = 20;

int n, m;
int w[N];
int f[N][M];

void init() {
    for (int j = 0; j < M; j ++) {
        for (int i = 1; i + (1 << j) - 1 <= n; i ++) {
            if (!j) f[i][j] = w[i];
            else f[i][j] = max(f[i][j - 1], f[i + (1 << j - 1)][j - 1]);
        }
    }
}

int query(int l, int r) {
    int k = log(r - l + 1) / log(2);
    return max(f[l][k], f[r - (1 << k) + 1][k]);
}

int main() {

    cin >> n;
    for (int i = 1; i <= n; i ++) cin >> w[i];

    init();

    cin >> m;
    while (m --) {
        int l, r;
        cin >> l >> r;
        cout << query(l, r) << endl;
    }
    return 0;
}
```



## 斐波那契数列

### 斐波那契数列介绍

$f(0) = 1$

$f(1) = 1$

$f(n) = f(n - 1) + f(n - 2), n \in Z, n \geqslant 2$

注意: 

* 有些题目会把第0项定义成0.
* 爬楼梯问题, 爬0阶楼梯的方案数是1.



### 滚动变量

时间复杂度是$O(n)$, 空间复杂度是$O(1)$

```cpp
int fib(int n)
{
	int a = 1, b = 1;
  while (n --)
  {
    int c = a + b;
    a = b, b = c;
  }
  return a;
}
```



### 快速幂算法

> https://www.acwing.com/problem/content/19/
>
> https://leetcode.cn/problems/climbing-stairs/description/

快速幂可以在$O(logn)$的时间复杂度内求解斐波那契数列的第$n$项.

定义向量$X_n = [a_n, a_{n - 1}]$, 那么$X_1 = [a_1, a_0]$

可以找到矩阵
$$
A = \begin{bmatrix}
1 & 1 \\
1 & 0 \\
\end{bmatrix}
$$


使得$X_n = X_{n - 1} \times A$

那么$X_n = X_1 \times A^{n-1}$

那么, 求斐波那契数列的问题就转化成了求矩阵$A^{n-1}$的问题, 这个问题可以用快速幂求解.

注意, 这个方法第0项需要单独判断, 因为要保证$n - 1 \geqslant 0$.

```cpp
class Solution {
public:
    void mul(int a[2][2], int b[2][2], int c[2][2])
    {
        int tmp[2][2] = {{0, 0}, {0, 0}};
        for (int i = 0; i < 2; i ++)
            for (int j = 0; j < 2; j ++)
                for (int k = 0; k < 2; k ++)
                    tmp[i][j] += a[i][k] * b[k][j];
                    
        for (int i = 0; i < 2; i ++)
            for (int j = 0; j < 2; j ++)
                c[i][j] = tmp[i][j];
    }
    int Fibonacci(int n) {
        
        if (n == 0) return 0;
        /* 定义初始向量[a1, a0] */
        int x[2] = {1, 0};
        /* 定义矩阵A */
        int A[2][2] = {{1, 1}, {1, 0}};
        /* 要计算A的n - 1次幂 */
        int k = n - 1;
        
        int res[2][2] = {{1, 0}, {0, 1}};
        
        /* 快速幂 */
        while (k)
        {
            /* mul的意思是把res * A的值放到res中 */
            if (k & 1) mul(res, A, res);
            mul(A, A, A);
            k >>= 1;
        }
        /* 计算x和A^{n-1}的乘积的第一项 */
        return x[0] * res[0][0] + x[1] * res[1][0];
    }
};
```





## 跳跃游戏 II

> https://leetcode.cn/problems/jump-game-ii/

假设$f(i)$表示从下标为0的地方跳到下标为$i$的地方所需要的最小步数.

首先, $f(0) = 0$.

假设$j$是最小的, 能够一步跳到下标为$i$的位置, 那么$f(i) = f(j) + 1$.

因此, 每次找到最小的, 能够一步跳到下标为$i$的位置$j$, 更新状态即可.

```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
       
       int n = nums.size();
       vector<int> f(n);

       for (int i = 1, j = 0; i < n; i ++) {
         /* 找到最小的, 能够一步跳到下标为i的位置j */
           while (j + nums[j] < i) j ++;
           f[i] = f[j] + 1;
       }

       return f[n - 1];
    }
};
```



## 跳跃游戏

> https://leetcode.cn/problems/jump-game/

```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        for (int i = 0, j = 0; i < nums.size(); i ++) {
            if (j < i) return false;
            /* 维护一个j, 记录j前面的所有元素能够跳到的最大距离 */
            j = max(j, i + nums[i]);
        }
        return true;
    }
};
```



## 正则表达式匹配

> https://leetcode.cn/problems/regular-expression-matching/

* 首先需要注意, `*`是不能单独出现的, 它是和前面一个字符组成一个整体, 例如`a*`, 表示空字符串, 一个`a`, 两个`a`, 到任意多个`a`.
* 假设$f(i, j)$表示$s(1..i)$和$p(1...j)$的所有的匹配方案是否为空, 注意: 下标从1开始, 状态转移可以分为两种情况考虑:
  * 如果$p(j)$不是`*`, 那么$f(i, j) = f(i - 1, j - 1)\ \&\&\ (s(i) == p(j)\ ||\ p(j) == '.')$
  * 如果$p(j)$是`*`, 那么需要根据这个`*`代表多少个$p(j - 1)$字符进行分类讨论:
    * 如果代表0个$p(j - 1)$, 那么$f(i, j) = f(i, j - 2)$
    * 如果代表1个$p(j - 1)$, 那么$f(i, j) = f(i - 1, j - 2)\ \&\&\ (s(i) == p(j - 1)\ ||\ p(j - 1) == '.')$
    * 如果代表2个$p(j - 1)$, 那么$f(i, j) = f(i - 2, j - 2)\ \&\&\ (s(i-1) == p(j - 1)\ \&\&\ s(i) == p(j - 1)\ ||\ p(j - 1) == '.')$
    * ..., 最后的$f(i, j)$就是上面几种情况相与.
    * 可以发现, $f(i, j)$和$f(i - 1, j)$总是相差一个$s(i) == p(j - 1)\ ||\ p(j - 1) == '.'$(见下面的分析), 因此, 最终的状态转移是$f(i, j) = f(i,j-2)\ ||\ (f(i - 1, j)\ \&\&\ (s(i) == p(j - 1))\ ||\ (p(j - 1) == '.'))$.

状态转移优化的推导:

```cpp
f[i][j] = (f[i][j - 2]) || (f[i - 1][j - 2] && (s[i] == p[j - 1] || p[j - 1] == '.')) || (f[i - 2][j - 2] && ((s[i - 1] == p[j - 1] && s[i] == p[j - 1]) || p[j - 1] == '.')) || ...

f[i - 1][j] = (f[i - 1][j - 2]) || (f[i - 2][j - 2] && (s[i - 1] == p[j - 1] || p[j - 1] == '.')) || (f[i - 3][j - 2] && ((s[i - 2] == p[j - 1] && s[i - 1] == p[j - 1]) || p[j - 1] == '.')) || ...
```



```cpp
class Solution {
public:
    bool isMatch(string s, string p) {
        int n = s.size();
        int m = p.size();

        s = ' ' + s;
        p = ' ' + p;

        vector<vector<bool>> f(n + 1, vector<bool>(m + 1, false));
        f[0][0] = true;

        for (int i = 0; i <= n; i ++) {
            for (int j = 1; j <= m; j ++) {

                /* 如果p[j]不是*, 那么要求前面的s不是空 */
                if (i >= 1 && p[j] != '*') {
                        f[i][j] = f[i - 1][j - 1] && (s[i] == p[j] || p[j] == '.');
                }
                else if (p[j] == '*') {
                        /* 如果出现了*, 那么前面一定有字符, j >= 2 */
                        f[i][j] = f[i][j - 1] = f[i][j - 2] || ((i >= 1) && f[i - 1][j] && (s[i] == p[j - 1] || p[j - 1] == '.'));
                }
            }
        }

        return f[n][m];
    }
};
```



## 通配符匹配

> https://leetcode.cn/problems/wildcard-matching/

状态表示和上一题一样: $f(i, j)$表示$s(1..i)$和$p(1...j)$的所有的匹配方案是否为空.

* 如果$p(j)$不是`*`, 那么状态转移为:

  $f(i, j) = f(i - 1, j - 1)\ \&\&\ (s(i) == p(j)\ ||\ p(j) == '?')$

* 如果$p(j)$是`*`,  那么就需要分情况讨论:

  * 如果`*`表示空字符序列, 那么状态转移为:

    $f(i, j) = f(i, j - 1)$

  * 如果`*`表示一个字符, 那么状态转移为:

    $f(i, j) = f(i - 1, j - 1)$

  * 如果`*`表示两个字符, 那么状态转移为:

    $f(i, j) = f(i - 2, j - 1)$

  * ....

  总的状态转移就是上面几种情况与起来.

观察如下表达式

```
f[i][j] = f[i][j - 1] || f[i - 1][j - 1] || f[i - 2][j - 1] || ...
f[i - 1][j] = f[i - 1][j - 1] || f[i - 2][j - 1] || f[i - 3][j - 1] || ...
```

通过观察, 可以发现: $f(i, j) = f(i, j - 1) || f(i - 1, j)$, 这个优化类似于背包问题.

```cpp
class Solution {
public:
    bool isMatch(string s, string p) {
        int n = s.size(), m = p.size();
        s = ' ' + s, p = ' ' + p;

        vector<vector<bool>> f(n + 1, vector<bool>(m + 1));
        f[0][0] = true;

        for (int i = 0; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (i >= 1 && p[j] != '*') {
                    f[i][j] = f[i - 1][j - 1] && (s[i] == p[j] || p[j] == '?');
                }
                else if (p[j] == '*') {
                    f[i][j] = (j >= 1) && f[i][j - 1] || (i >= 1) && f[i - 1][j];
                }
            }
        }

        return f[n][m];
    }
};
```
