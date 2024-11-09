---
title: 剑指Offer题解
categories: 算法
mathjax: true
---



## 调整数组顺序使奇数位于偶数前面

> https://www.acwing.com/problem/content/description/30/

* 和快速排序中调整数组, 使得一部分`< x`, 一部分`> x`一样.

```cpp
class Solution {
public:
    void reOrderArray(vector<int> &array) {
        int n = array.size();
        if (!n) return ;
        int i = -1, j = n;
        while (i < j) {
            while (array[ ++ i] % 2);
            while (array[ -- j] % 2 == 0);
            if (i < j) swap(array[i], array[j]);
        }
    }
};
```

