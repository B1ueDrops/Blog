---
title: 贪心算法
categories: 算法
---

[toc]

## 连续子数组的最大和

> https://www.acwing.com/problem/content/description/50/

如果是连续的子数组, 那么对于数组中的每一个数`nums[i]`, 都只有两个选项:

* 要么加上`nums[i]`
* 要么从`nums[i]`重新开始.

那么每一步直接去最大的一个就可以.

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        
        int n = nums.size();
        
        int res = nums[0];
        int ans = nums[0];
        
        for (int i = 1; i < n; i ++)
        {
            res = max(res + nums[i], nums[i]);
            ans = max(ans, res);
        }
        
        return ans;
    }
};
```



## 耍杂技的牛

> https://www.acwing.com/problem/content/127/

假设牛从上到下的排序是0, 1, 2, ...$n - 1$, 现在考虑将第$i$头牛和第$i + 1$头牛交换带来的影响.

| ..        | 交换前的危险系数                  | 交换后的危险系数                  |
| --------- | --------------------------------- | --------------------------------- |
| 第i头牛   | $w_1+w_2+...+w_{i-1}-s_i$         | $w_1+w_2+...+w_{i-1}+w_{i+1}-s_i$ |
| 第i+1头牛 | $w_1+w_2+...+w_{i-1}+w_i-s_{i+1}$ | $w_1+w_2+...+w_{i-1}-s_{i+1}$     |

然后将相同的因子$w_1 + w_2 + ... + w_{i-1}$删去, 得到:

| ..        | 交换前的危险系数 | 交换后的危险系数 |
| --------- | ---------------- | ---------------- |
| 第i头牛   | $-s_i$           | $w_{i+1}-s_i$    |
| 第i+1头牛 | $w_i-s_{i+1}$    | $-s_{i+1}$       |

如果要求危险系数最大值最小, 等价于:

$max(-s_i, w_i - s_{i+1}) \geqslant max(w_{i+1} - s_i, -s_{i+1})$

把这个四个数同时加上$s_i + s_{i+1}$, 等价于:

$max(s_{i+1}, w_i+s_i) \geqslant max(s_i, w_{i+1}+s_{i+1})$

因此, 如果第$i$和第$i + 1$头牛满足这个条件, 那么就可以将它们进行交换, 如果要用代码实现的话, 就可以按照$max(s_{i+1}, w_i + s_i) < max(s_i, w_{i+1}+s_{i+1})$进行排序.

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 50010;

int n;

struct Cow
{
    int w, s;
} cows[N];

bool cmp(Cow a, Cow b)
{
    return max(b.s, a.w + a.s) < max(a.s, b.w + b.s);
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> cows[i].w >> cows[i].s;
    
    sort(cows, cows + n, cmp);
    
    int sum = 0, res = -2e9;
    
    for (int i = 0; i < n; i ++)
    {
        res = max(res, sum - cows[i].s);
        sum += cows[i].w;
    }
    cout << res << endl;
    return 0;
}
```





## 国王游戏

> https://www.acwing.com/problem/content/116/

和上面的题完全一样, 将数组按照条件$max(s_{i+1}, w_is_i) \leqslant max(s_i, w_{i+1}s_{i+1})$进行排序.

本题涉及高精度乘法和除法.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

const int N = 1010;

int n;
int x, y;

struct People
{
    int w, s;
} p[N];

bool cmp(People a, People b)
{
  /* 注意要写成<, 不然排序会出现segmentation fault */
    return max(b.s, a.w * a.s) < max(a.s, b.w * b.s);
}

vector<int> mul(vector<int> A, int b)
{
    vector<int> C;
    int t = 0;
    
    for (int i = 0; i < A.size() || t; i ++)
    {
        if (i < A.size()) t += A[i] * b;
        C.push_back(t % 10);
        t /= 10;
    }
    while (C.size() > 1 && C.back() == 0) C.pop_back();
    
    return C;
}

vector<int> div(vector<int> A, int b)
{
    int r = 0;
    vector<int> C;
    
    for (int i = A.size() - 1; i >= 0; i --)
    {
        r = 10 * r + A[i];
        C.push_back(r / b);
        r %= b;
    }
    
    reverse(C.begin(), C.end());
    
    while (C.size() > 1 && C.back() == 0) C.pop_back();
    
    return C;
}

vector<int> vec_max(vector<int> A, vector<int> B)
{
    if (A.size() > B.size()) return A;
    if (A.size() < B.size()) return B;
    
    if (vector<int>(A.rbegin(), A.rend()) > vector<int>(B.rbegin(), B.rend()))
        return A;
    return B;
}


int main()
{
    cin >> n;
    cin >> x >> y;
    for (int i = 0; i < n; i ++) cin >> p[i].w >> p[i].s;

    sort(p, p + n, cmp);

    vector<int> product(1, x);
    vector<int> res(1, 0);
    
    for (int i = 0; i < n; i ++)
    {
        res = vec_max(res, div(product, p[i].s));
        product = mul(product, p[i].w);
    }
    
    for (int i = res.size() - 1; i >= 0; i --) cout << res[i];
    return 0;
}
```

