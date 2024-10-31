---
title: AcWing算法基础课题解
categories: 算法
mathjax: true
---



## 动态规划



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

