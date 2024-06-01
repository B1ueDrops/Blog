---
title: 二分系列
categories: 算法
mathjax: true
---

[TOC]



## 二分的本质

二分的本质在于, 假设对于区间$[l, r]$, 有一个分界点$mid$, 对于某一种性质来说, 区间$[l, mid]$上的数和区间$[mid +1,  r]$上的数只有一侧满足这个性质.

此时, 就出现了两个边界点, 一个是$mid$, 一个是$mid + 1$, 求这两个边节点, 就需要用到两种不同的二分模板.

用两个模板时, 只需要考虑两个问题:

* 性质是什么?
* 求哪个边界点?



## 二分求左边界点

```cpp
while (l < r) {
  int mid = l + (r - l) / 2 + 1;
  if (check(mid)) r = mid - 1;
  else l = mid;
}
```



## 二分求右边界点

```cpp
while (l < r) {
  int mid = l + (r - l) / 2;
  if (check(mid)) r = mid;
  else l = mid + 1;
}
```



## 数的范围

> https://www.acwing.com/problem/content/description/791/
>
> https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/

注意: <font color=red>需要提前检查数组是否为空!</font>


* 求左边位置:
  * 性质是大于等于给定的数$k$.
  * 边界是右边界
* 求右边位置:
  * 性质是大于给定的数$k$.
  * 边界是左边界

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 100010;

int n, q;
int a[N];

int main() 
{
    cin >> n >> q;
    for (int i = 0; i < n; i ++) cin >> a[i];
    
    while (q --) 
    {
        int k;
        cin >> k;
        
      // 求右边界
        int l = 0, r = n - 1;
        while (l < r)
        {
            int mid = l + (r - l) / 2;
            if (a[mid] >= k) r = mid;
            else l = mid + 1;
        }
        if (a[l] != k) 
        {
            cout << "-1 -1" << endl;
            continue;
        }
        int left = l;
        
      // 求左边界
        l = 0, r = n - 1;
        while (l < r)
        {
            int mid = l + (r - l) / 2 + 1;
            if (a[mid] > k) r = mid - 1;
            else l = mid;
        }
        if (a[l] != k) {
            cout << "-1 -1" << endl;
            continue;
        }
        cout << left << ' ' << l << endl;
        
    }
    return 0;
}
```



## 旋转数组的最小数字

> https://www.acwing.com/problem/content/description/20/

* 第一步, 从后向前, 删掉所有等于`nums[0]`的元素.
* 删掉之后, 可以发现数组满足二分性质, 边界点左边的位置满足`>= nums[0]`, 边界点右边的位置满足`< nums[0]`

```cpp
class Solution {
public:
    int findMin(vector<int>& nums) {
        
        int n = nums.size() - 1;
        if (n < 0) return -1;
        
        while (n >= 0 && nums[n] == nums[0]) n --;
        // 数组完全单调的情况, 删除后最后一个数还是大于nums[0]
        if (nums[0] < nums[n]) return nums[0];

        int l = 0, r = n;
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] < nums[0]) r = mid;
            else l = mid + 1;
        }
        
        return nums[l];
    }
};
```



## 搜索旋转排序数组

> https://leetcode.cn/problems/search-in-rotated-sorted-array/description/

思路和“旋转数组的最小数字”相同, 首先用二分找到数组中的最小数字, 然后用这个最小数字和`target`做比较, 然后在另一个范围中, 再用二分查找`target`是否出现.

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int n = nums.size();
        
        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] < nums[0]) r = mid;
            else l = mid + 1;
        }
        /* 单独处理完全单调的情况 */
        if (nums[l] > nums[0]) {
            l = 0, r = n - 1;
        }
        else {
            if (target < nums[0]) r = n - 1;
            else l = 0, r = r - 1;
        }
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] >= target) r = mid;
            else l = mid + 1;
        }
        if (nums[l] != target) return -1;
        return l;

    }
};
```



## 搜索插入位置

> https://leetcode.cn/problems/search-insert-position/description/

本质上是找第一个小于等于`target`的值, 注意, 如果`target`比数组的最大值还要大, 那么需要返回`n`, 其中`n`是数组的大小.

```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {

        int n = nums.size() - 1;
        if (target > nums[n]) return n + 1;
        int l = 0, r = nums.size() - 1;
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] >= target) r = mid;
            else l = mid + 1;
        }
        return l;
    }
};
```



## 浮点数二分

> https://www.acwing.com/problem/content/description/792/

* 浮点数二分一般用来求某个运算的数值, 本质上也就是在答案范围$[l, r]$查找运算数值等于某个目标值的数$x$, 需要这个运算具有单调性.
* 如果要求浮点数运算结果保留$n$位小数, 那么阈值`eps`一般设置成$10^{-(n+2)}$.

```cpp
#include <iostream>

using namespace std;

int main()
{
    
    double x;
    cin >> x;
    
    double l = -100.0, r = 100.0;
    /* 保留6位小数 */
    while (r - l > 1e-8)
    {
        double mid = (l + r) / 2;
        if (mid * mid * mid > x) r = mid;
        else l = mid;
    }
    printf("%.6lf", l);
    return 0;
}
```



## x的平方根

> https://leetcode.cn/problems/sqrtx/

这个题本质上是在区间$[0, x]$处, 找一个$y$, 其中$y$是性质$y^{2} > x$的左边界.

```cpp
class Solution {
public:
    int mySqrt(int x) {

        int l = 0, r = x;
        while (l < r)
        {
            int mid = l + (r - l) / 2 + 1;
            /* 避免溢出 */
            if (mid > x / mid) r = mid - 1;
            else l = mid;
        }
        return l;
    }
};
```

