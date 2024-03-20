---
title: 线性动态规划系列
categories: 算法
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

