---
title: 排序算法
categories: 算法
mathjax: true
---

[toc]



## 快速排序模板

> https://www.acwing.com/problem/content/787/

快速排序的平均时间复杂度是$O(nlogn)$, 最坏时间复杂度是$O(n^2)$, 空间复杂度是$O(logn)$.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;


int n;
int q[N];

void quick_sort(int l, int r)
{
    if (l >= r) return ;
    
    int i = l - 1, j = r + 1, mid = l + (r - l) / 2;
    
    int x = q[mid];
    
    while (i < j)
    {
        while (q[ ++i] < x);
        while (q[ --j] > x);
        if (i < j) swap(q[i], q[j]);
    }
    // 注意这里是j, 不是mid
    quick_sort(l, j), quick_sort(j + 1, r);
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> q[i];
    
    quick_sort(0, n - 1);
    
    for (int i = 0; i < n; i ++) printf("%d ", q[i]);
    
    return 0;
}
```



## 快速选择算法

> https://www.acwing.com/problem/content/788/

快速选择算法是基于快速排序的, 可以在$O(n)$时间复杂度内找到一个数组的第$k$小的数.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int n, k;
int q[N];

int quick_sort(int l, int r, int k)
{
    if (l >= r) return q[l];
    
    int i = l - 1, j = r + 1, x = q[l + (r - l) / 2];
    
    while (i < j)
    {
        while (q[ ++i] < x);
        while (q[ --j] > x);
        if (i < j) swap(q[i], q[j]);
    }
    
    int sl = j - l + 1;
    if (k <= sl) return quick_sort(l, j, k);
    else return quick_sort(j + 1, r, k - sl);
    
}

int main()
{
    cin >> n >> k;
    for (int i = 0; i < n; i ++) cin >> q[i];
    
    printf("%d\n", quick_sort(0, n - 1, k));
    
    return 0;
}
```



## 归并排序模板

> https://www.acwing.com/problem/content/789/

归并排序的时间复杂度是$O(nlogn)$, 空间复杂度是$O(n)$.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int n;
int a[N], tmp[N];

void merge_sort(int l, int r)
{
    if (l >= r) return ;
    
    int mid = l + (r - l) / 2;
    
    merge_sort(l, mid), merge_sort(mid + 1, r);
    
    int k = 0, i = l, j = mid + 1;
    
    while (i <= mid && j <= r)
    {
        if (a[i] <= a[j]) tmp[k ++] = a[i ++];
        else tmp[k ++] = a[j ++];
    }
    while (i <= mid) tmp[k ++] = a[i ++];
    while (j <= r) tmp[k ++] = a[j ++];
    
    for (int i = l, j = 0; i <= r; i ++, j ++) a[i] = tmp[j];
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> a[i];
    
    merge_sort(0, n - 1);
    for (int i = 0; i < n; i ++) printf("%d ", a[i]);
    
    return 0;
}
```



## 归并排序求逆序数

> https://www.acwing.com/problem/content/790/
>
> https://www.acwing.com/problem/content/109/

注意, 一个有$n$个元素的数组, 它的逆序数对最多可能有$C_n^{2}$个, 需要用`long long`存储.

**注意: **冒泡排序的交换次数就等于原数组中逆序对的个数.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

typedef long long LL;

int n;
int a[N], tmp[N];

LL merge_sort(int l, int r)
{
    if (l >= r) return 0;
    
    int mid = l + (r - l) / 2;
    
    LL res = merge_sort(l, mid) + merge_sort(mid + 1, r);
    
    int i = l, j = mid + 1, k = 0;
    
    while (i <= mid && j <= r)
    {
        if (a[i] <= a[j]) tmp[k ++] = a[i ++];
        else {
            res += mid - i + 1;
            tmp[k ++] = a[j ++];
        }
    }
    
    while (i <= mid) tmp[k ++] = a[i ++];
    while (j <= r) tmp[k ++] = a[j ++];
    
    for (int i = l, j = 0; i <= r; i ++, j ++) a[i] = tmp[j];
    
    return res;
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> a[i];
    
    printf("%lld\n", merge_sort(0, n - 1));
    
    return 0;
}
```



## 调整数组顺序使奇数位于偶数前面

> https://www.acwing.com/problem/content/description/30/

```cpp
class Solution {
public:
    void reOrderArray(vector<int> &array) {
         
         int n = array.size();
         
         if (!n) return ;
         
         int i = -1, j = n;
         
         while (i < j)
         {
             while (array[++ i] % 2);
             while (array[-- j] % 2 == 0);
             if (i < j) swap(array[i], array[j]);
         }
    }
};
```
