---
title: 线性动态规划系列
categories: 算法
mathjax: true
---



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



