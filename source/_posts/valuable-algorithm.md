---
title: 其他的算法题题解
categories: 算法
mathjax: true
---



## 字符串的split函数

* Split函数可以将一个字符串, 按照其中出现的某个字符`t`, 切割成若干的pieces.
* 实现可参考:

```cpp
vector<string> split(string str, char t) {
    str += t;
    vector<string> items;
    for (int i = 0; i < str.size(); i ++) {
      // 对于每一个i, 移动j, 直到移动到str[j] == t为止, 然后把扫过的部分放到数组中
        int j = i;
        string item;
        while (str[j] != t) item += str[j ++];
        i = j;
        items.push_back(item);
    }
    return items;
}
```



## 最大子序和

> https://www.acwing.com/problem/content/description/137/

* 这个题可以转化成在前缀和数组中找两个元素`s[j]`和`s[i]`, 在`0 < i - j <= m`的情况下, 求`min(s[i] - s[j])`.

* 如果是暴力算法的话, 时间复杂度是$O(nm)$.
* 如果要压缩成$O(n)$, 就需要遍历`s[i]`的时候, 在区间`[i - m, i - 1]`里, 一下子取出来最小值. 这个区间的长度是`m`.
* 注意: 一个细节, 一开始队列中已经有`s[0]`, 这是为什么?
  * 如果队列为空, 一开始从`0`遍历的话, `res = max(res, s[i] - s[q[hh]])`后, `res`的值是0, 如果原数组全是负数, 那么答案就不正确.
  * 根本原因, 在于遍历`s[i]`时, 要查找最小值的区间范围时`[i - m, i - 1]`, 这里的`i - 1`是严格小于`i`的, 如果是`0`元素, 就没有要查找最小值的区间, 因为

```cpp
#include <iostream>
#include <climits>
using namespace std;

const int N = 300010;

int n, m, s[N];
int hh = 0, tt = 0;
int q[N];

int main() {
    
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) cin >> s[i], s[i] += s[i - 1];
    
    int res = INT_MIN;
    for (int i = 1; i <= n; i ++) {
        if (i - m > q[hh]) hh ++;
        res = max(res, s[i] - s[q[hh]]);
        while (hh <= tt && s[i] <= s[q[tt]]) tt --;
        q[ ++ tt] = i;
    }
    cout << res << endl;
    return 0;
}
```

