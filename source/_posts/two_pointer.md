---
title: 双指针系列
categories: 算法
mathjax: true
---



## 移动零

> https://leetcode.cn/problems/move-zeroes/

* 时间复杂度是$O(n)$.

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int k = 0;
        for (auto x : nums) {
            if (x) nums[k ++] = x;
        }
        while (k < nums.size()) nums[k ++] = 0;
    }
};
```



## 盛水最多的容器

> https://leetcode.cn/problems/container-with-most-water/

* 首先, 对于两个水柱`h[i]`和`h[j]`来说(`i < j`), 它们之间的盛水容量是`min(h[i], h[j]) * (j - i)`.

* 那么, 只需要遍历所有合法的`(i, j)`就可以求出答案, 但是时间复杂度是$O(n^2)$.

* 木桶容量由短板决定, 移动长板的话, 水面高度不可能再上升, 而宽度变小了, 所以只有通过移动短板, 才有可能使水位上升.

  * 因此, 只需要将`height`小的一方移动即可.

  * 时间复杂度是$O(n)$.

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        
        int res = 0;
        for (int i = 0, j = height.size() - 1; i < j; ) {
            res = max(res, min(height[i], height[j]) * (j - i));
            if (height[i] > height[j]) j --;
            else i ++;
        }
        return res;
    }
};
```



## 三数之和

> https://leetcode.cn/problems/3sum/

模板题, 时间复杂度是$O(n^2)$.

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {

        int n = nums.size();
        vector<vector<int>> res;
        sort(nums.begin(), nums.end());

        for (int i = 0; i < n; i ++) {
            if (i != 0 && nums[i] == nums[i - 1]) continue;

            int l = i + 1, r = n - 1;
            while (l < r) {
                int t = nums[i] + nums[l] + nums[r];
                if (t > 0) {
                    r --;
                    continue;
                }
                else if (t < 0) {
                    l ++;
                    continue;
                }
                else res.push_back({ nums[i], nums[l], nums[r] });

                do { l ++; } while (l < r && nums[l] == nums[l - 1]);
                do { r --; } while (l < r && nums[r] == nums[r + 1]);
            }
        }
        return res;
    }
};
```





## 最长回文子串

> https://leetcode.cn/problems/longest-palindromic-substring/description/

* 这个题的数据范围在1000, 可以用$O(n^2)$的算法.
* 回文串分为两种类型, 一类是奇数长度的, 一类是偶数长度的.
  * 假设现在遍历到了`i`位置字符, 那么奇数长度就要从`i - 1, i + 1`向外扩展.
  * 偶数长度就要从`[i, i + 1]`向外扩展.

```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        string res;

        for (int i = 0; i < s.size(); i ++) {

            int l = i - 1, r = i + 1;
            while (l >= 0 && r < s.size() && s[l] == s[r]) l --, r ++;
            if (r - l - 1 > res.size()) res = s.substr(l + 1, r - l - 1);
            
            l = i, r = i + 1;
            while (l >= 0 && r < s.size() && s[l] == s[r]) l --, r ++;
            if (r - l - 1 > res.size()) res = s.substr(l + 1, r - l - 1);
        }
        return res;
    }
};
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

