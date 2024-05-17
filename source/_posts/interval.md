---
title: 区间问题
categories: 算法
mathjax: true
---



## 交集区间合并

> https://www.acwing.com/problem/content/805/

给定$n$个区间, 要求合并所有有交集的区间, 返回合并完成后的区间个数.

首先按照区间左端点进行排序, 然后从前向后遍历每一个区间, 判断是否有交集, 模拟合并即可.

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

int n;
struct Range
{
    int l, r;
    bool operator < (const Range &W) const {
        return l < W.l;
    }
} range[N];

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> range[i].l >> range[i].r;
    
    sort(range, range + n);
    
    int st = -2e9, ed = -2e9, res = 0;
    
    for (int i = 0; i < n; i ++)
    {
        /* 如果没有交集 */
        if (ed < range[i].l)
        {
            res ++;
            st = range[i].l, ed = range[i].r;
        }
        /* 如果有交集 */
        else ed = max(ed, range[i].r);
    }
    
    cout << res << endl;
    return 0;
}
```



## 区间选点覆盖

> https://www.acwing.com/problem/content/907/

给定$n$个区间, 选择尽量少的点用来覆盖这$n$个区间, 求出选择点的个数.

按照区间右端点排序, 然后从前向后遍历每一个区间, 判断是否有交集, 如果没有交集, 那么就需要另外一个点覆盖新增加的区间.

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

int n;

struct Range
{
    int l, r;
    bool operator < (const Range &W) const
    {
        return r < W.r;
    }
} range[N];

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> range[i].l >> range[i].r;
    
    sort(range, range + n);
    
    int ed = -2e9, res = 0;
    for (int i = 0; i < n; i ++)
    {
      /* 如果当前区间和下一个区间没有交集/是第一个区间 */
        if (ed < range[i].l)
        {
            res ++;
            ed = range[i].r;
        }
    }
    cout << res << endl;
    return 0;
}

```



## 最大不相交区间数量

> https://www.acwing.com/problem/content/910/

这个题和区间选点覆盖本质上是一样的, 假设所有能被同一个点覆盖的区间组成一组, 那么对于这些组, 每一个组出一个区间, 那么就符合题目中区间互不相交的条件.



## 不相交区间分组

> https://www.acwing.com/problem/content/908/

给定$n$个区间, 要求将这些区间分成若干组, 组内的区间两两不能有交集, 组的数量尽可能少.

首先, 将区间按照左端点进行排序.

然后, 用一个小根堆维护当前所有组的右端点最小值, 然后从前向后遍历所有区间:

* 如果当前区间与右端点最小的组都有交集/小根堆是空的, 那么只能新开一个组.
* 如果当前区间和右端点最小的组没有交集, 那么就可以把这个区间放到右端点最小的组.

```cpp
#include <iostream>
#include <algorithm>
#include <queue>

using namespace std;

const int N = 100010;

int n;
struct Range
{
    int l, r;
    bool operator < (const Range &W) const
    {
        return l < W.l;
    }
} range[N];


int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> range[i].l >> range[i].r;
    
    sort(range, range + n);
    
    priority_queue<int, vector<int>, greater<int>> heap;
    
    for (int i = 0; i < n; i ++)
    {
        if (heap.empty() || range[i].l <= heap.top()) heap.push(range[i].r);
        else {
            heap.pop();
            heap.push(range[i].r);
        }
    }
    cout << heap.size() << endl;
    return 0;
}
```



## 选定区间覆盖指定区间

> https://www.acwing.com/problem/content/909/

给定$n$个区间和一个目标区间$[s, t]$, 要求使用尽量少的区间将$[s, t]$覆盖.

首先将区间按照左端点进行排序.

然后, 从前向后找到所有能够覆盖点$s$的区间中, 右端点最靠后的一个区间, 然后将$s$更新成这个右端点的值, 重复上述过程.

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

int n, s, t;

struct Range
{
    int l, r;
    bool operator < (const Range &W) const
    {
        return l < W.l;
    }
} range[N];


int main()
{
    cin >> s >> t;
    cin >> n;
    
    for (int i = 0; i < n; i ++) cin >> range[i].l >> range[i].r;
    
    sort(range, range + n);
    
    int res = 0;
    bool flag = false;
    
    for (int i = 0; i < n; i ++)
    {
        int j = i, ed = -2e9;
        while (j < n && range[j].l <= s)
        {
            ed = max(ed, range[j].r);
            j ++;
        }
        if (ed < s)
        {
            res = -1;
            break;
        }
        res ++;
        if (ed >= t)
        {
            flag = true;
            break;
        }
        s = ed;
        i = j - 1;
    }
    
    if (!flag) res = -1;
    printf("%d\n", res);
    
    return 0;
}
```



## 交集区间合并求最终区间

> https://leetcode.cn/problems/merge-intervals/

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        vector<vector<int>> ans;

        sort(intervals.begin(), intervals.end());

        int st = -2e9, ed = -2e9;
        for (int i = 0; i < intervals.size(); i ++)
        {
            int l = intervals[i][0], r = intervals[i][1];
            if (ed < l)
            {
                if (ed != -2e9) ans.push_back({st, ed});
                st = l, ed = r;
            }
            else 
                ed = max(ed, r);
        }
      /* 注意把最后一个区间包括进来 */
        ans.push_back({st, ed});
        return ans;
    }
};
```

