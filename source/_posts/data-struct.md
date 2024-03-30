---
title: 数据结构系列
categories: 算法
mathjax: true
---



## 单调队列模板

> https://www.acwing.com/problem/content/156/

```cpp
#include <iostream>

using namespace std;

const int N = 1000010;

int n, k;
int hh = 0, tt = -1;
int a[N], q[N];

int main() {

    cin >> n >> k;
    for (int i = 0; i < n; i ++) cin >> a[i];

    // min value
    for (int i = 0; i < n; i ++) {
        if (i - k + 1 > q[hh]) hh ++;
        while (hh <= tt && a[q[tt]] >= a[i]) tt --;
        q[ ++ tt ] = i;
        if (i - k + 1 >= 0) printf("%d ", a[q[hh]]);
    }

    puts("");
    hh = 0, tt = -1;

    // max value
    for (int i = 0; i < n; i ++) {
        if (i - k + 1 > q[hh]) hh ++;
        while (hh <= tt && a[q[tt]] <= a[i]) tt --;
        q[ ++ tt ] = i;
        if (i - k + 1 >= 0) printf("%d ", a[q[hh]]);
    }
    return 0;
}
```



## 最大子序和(单调队列)
> https://www.acwing.com/problem/content/description/137/

首先要注意审题, 这个子序列是连续的.

将原数组变成前缀和后, 维护一个长度是$m$的单调队列(最小值), 对于下标`i`, 只需要从单调队列中找到一个最小值, 然后相减, 就得到**以下标i为结尾, 长度不超过m+1的连续子序列的最大值**, 然后遍历一遍求全局最大值就是答案.

时间复杂度是$O(n)$.

```cpp
#include <iostream>
#include <climits>

using namespace std;

const int N = 300010;

int n, m;
int hh = 0, tt = 0;
int s[N], q[N];

int main() {

    cin >> n >> m;

    for (int i = 1; i <= n; i ++) cin >> s[i], s[i] += s[i - 1];


    int res = INT_MIN;
    for (int i = 1; i <= n; i ++) {
        if (i - m > q[hh]) hh ++;
        res = max(res, s[i] - s[q[hh]]);
        while (hh <= tt && s[q[tt]] >= s[i]) tt --;
        q[ ++ tt ] = i;

    }

    cout << res << endl;
    return 0;
}
```

注意, 这段代码里面有三个细节:

* 首先, 对于前缀和来说, `[l, r]`里所有元素的和是`s[r] - s[l-1]`. 由于减去的是`l-1`, 我在维护单调队列时, 对于下标`i`来说, 我实际需要维护的单调队列长度是`m + 1`.
* 对于元素`s[1]`, 我首先需要向队列中添加一个0, 让`q[0] = 0`, 来让第一个元素也参与运算.
* `res = max(res, s[i] - s[q[hh]])`必须在用`s[q[tt]]`更新队列之前, 表明了这个单调队列维护的是**`s[i]`之前(不包含`s[i]`), 长度为`m + 1`的滑动窗口内的最小值**, 如果先用`s[i]`更新了队列, 然后求最小值, 那么最小值就一定会大于等于0, 显然不符合题意.



## 两数之和(哈希表)

> https://leetcode.cn/problems/two-sum/description/

```cpp
class Solution {
public:
    vector<int> ans;
    unordered_map<int, int> hash;
    vector<int> twoSum(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size(); i ++) {
            if (hash.count(target - nums[i])) {
                ans.push_back(i);
                ans.push_back(hash[target - nums[i]]);
                break;
            }
            hash[nums[i]] = i;
        }
        return ans;
    }
};
```



## 数据流中的第k大数(堆)

> https://leetcode.cn/problems/kth-largest-element-in-a-stream/

第k大数直接用小根堆维护, 如果插入的树超过k个就直接弹出最小值, 维护后的堆顶元素就是最小值.

```cpp
class KthLargest {
public:
    priority_queue<int, vector<int>, greater<int>> heap;
    int k;
    KthLargest(int _k, vector<int>& nums) {
        k = _k;
        for (auto num: nums) {
            heap.push(num);
            if (heap.size() > k) heap.pop();
        }
    }
    
    int add(int val) {
        heap.push(val);
        if (heap.size() > k) heap.pop();
        return heap.top();
    }
};
```

