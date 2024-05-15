---
title: 递归系列
categories: 算法
mathjax: true
---





## 寻找两个正序数组的中位数

> https://leetcode.cn/problems/median-of-two-sorted-arrays/

考虑这样一个问题:

> 在两个排序的数组之间, 寻找第k小数.

如果这个问题能够解决, 那么中位数就是第`(n + m) / 2`小的数.

假设两个正序数组分别是$A$和$B$, 考虑$A[\frac{k}{2}]$和$B[\frac{k}{2}]$两个数:

* 如果$A[\frac{k}{2}] < B[\frac{k}{2}]$, 那么$A, B$数组中, 小于等于$A[\frac{k}{2}]$的数的个数就小于$k$, 此时两个数组中第$k$小的数肯定不会在$A$数组第$\frac{k}{2}$个元素的左边.
* 如果$A[\frac{k}{2}] > B[\frac{k}{2}]$, 那么$A, B$数组中, 小于等于$B[\frac{k}{2}]$的数的个数就小于$k$, 此时两个数组中第$k$小的数肯定不会在$B$数组第$\frac{k}{2}$个元素的左边.
* 如果$A[\frac{k}{2}] = B[\frac{k}{2}]$, 那么$A, B$数组中, 小于等于$A[\frac{k}{2}]$或$B[\frac{k}{2}]$的数的个数就等于$k$, 此时$A[\frac{k}{2}]$或者$B[\frac{k}{2}]$就是第$k$小的数.

根据以上分析, 定义一个函数, 函数的签名为`find(A, i, B, j, k)`, 表示给定两个数组$A, B$, 下标从$i, j$开始, 找到两个数组中第$k$小的数, 那么就可以用这个函数递归解决.

时间复杂度是$O(logk)$, 由于$k$最多是$m + n$, 那么时间复杂度就是$O(log(m + n))$

```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {

        int tot = nums1.size() + nums2.size();
        if (tot % 2 == 0) {
            int left = find(nums1, 0, nums2, 0, tot / 2);
            int right = find(nums1, 0, nums2, 0, tot / 2 + 1);
            return (left + right) / 2.0;
        }
        else {
            return (double)find(nums1, 0, nums2, 0, tot / 2 + 1);
        }
    }

    int find(vector<int> &nums1, int i, vector<int> &nums2, int j, int k) {
        
        /* 让nums1作为元素个数较少的数组 */
        if (nums1.size() - i > nums2.size() - j) return find(nums2, j, nums1, i, k);

        /* nums1为空的话, 直接返回nums2第k小的元素 */
        if (nums1.size() == i) return nums2[j + k - 1];

        /* k = 1时, 递归终止 */
        if (k == 1) return min(nums1[i], nums2[j]);
				
      	/* 找a[k/2]和b[k/2] */
        int si = min((int)(nums1.size() - 1), i + k / 2 - 1);
        int sj = j + (k - k / 2) - 1;

        if (nums1[si] > nums2[sj]) 
            return find(nums1, i, nums2, sj + 1, k - (sj - j + 1));
        else
            return find(nums1, si + 1, nums2, j, k - (si - i + 1));
    }
};
```
