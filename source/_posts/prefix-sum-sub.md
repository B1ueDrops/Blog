---
title: 前缀和差分系列
categories: 算法
---



## 和为K的子数组

> https://leetcode.cn/problems/subarray-sum-equals-k/

要统计数组中有多少个子区间之和是$k$, 等价于在前缀和数组中, 假设固定$i$, 求有多少个$j \in [0, 1, ..., i - 1]$, 使得$s[i] - s[j] = k$, 也就是求有多少个$s[j]$, 使得$s[j] = s[i] - k$, 这个可以用哈希表来做.

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> s(n + 1);

        for (int i = 1; i <= n; i ++) s[i] = s[i - 1] + nums[i - 1];

        int res = 0;
        unordered_map<int, int> hash;
        // s[0] = 0出现1次
        hash[0] = 1;
        
        for (int i = 1; i <= n; i ++) {
            res += hash[s[i] - k];
            hash[s[i]] ++;
        }
        return res;
    }
};
```

