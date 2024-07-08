---
title: 双指针
categories: 算法
mathjax: true
---



[toc]



## 字符串的split函数

```cpp
vector<string> split(string str, char t) {
    str += t;
    vector<string> items;
    for (int i = 0; i < str.size(); i ++) {
        int j = i;
        string item;
        while (str[j] != t) item += str[j ++];
        i = j;
        items.push_back(item);
    }
    return items;
}
```



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



## 三数之和系列

### 常规三数之和

> https://leetcode.cn/problems/3sum/

算法是排序+双指针算法.

将数组排序后, 枚举每一个数$c$, 维护两个指针$l, r$, 两个指针分别指向头和尾, 计算$c + nums[l] + nums[r]$的大小

* 如果大于`target`就将`r--`
* 如果小于`target`就将`l++`

时间复杂度为$O(n^2)$

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> ans;
        
        sort(nums.begin(), nums.end());

        for (int i = 0; i < nums.size(); i ++) {
          // 如果当前枚举的数和上一个枚举的数相同, 跳过
            if (i != 0 && nums[i] == nums[i - 1]) continue;

            int l = i + 1, r = nums.size() - 1;
            while (l < r) {
                int sum = nums[i] + nums[l] + nums[r];
                if (sum > 0) {
                    r --;
                    continue;
                }
                if (sum < 0) {
                    l ++;
                    continue;
                }
                if (sum == 0) {
                    ans.push_back({ nums[i], nums[l], nums[r] });
                }
              // 去重
                do { l ++; } while (l < r && nums[l] == nums[l - 1]);
                do { r --; } while (l < r && nums[r] == nums[r + 1]);
            }
        }
        return ans;
    }
};

```



### 常规四数之和

> https://leetcode.cn/problems/4sum/

和三数之和思路一样, 只是枚举的是选定的两个数, 时间复杂度是$O(n^3)$

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {

        typedef long long LL;

        vector<vector<int>> ans;

        sort(nums.begin(), nums.end());

        for (int i = 0; i < nums.size(); i ++) {

            if (i != 0 && nums[i] == nums[i - 1]) continue;

            for (int j = i + 1; j < nums.size(); j ++) {

                if (j != i + 1 && nums[j] == nums[j - 1]) continue;

                int l = j + 1, r = nums.size() - 1;

                while (l < r) {
                    LL sum = (LL)nums[i] + (LL)nums[j] + (LL)nums[l] + (LL)nums[r];

                    if (sum < target) {
                        l ++;
                        continue;
                    }
                    if (sum > target) {
                        r --;
                        continue;
                    }
                    ans.push_back({ nums[i], nums[j], nums[l], nums[r]});

                    do { l ++; } while (l < r && nums[l] == nums[l - 1]);
                    do { r --; } while (l < r && nums[r] == nums[r + 1] );
                }
            }
        }
        return ans;
    }
};
```



### 最接近的三数之和

> https://leetcode.cn/problems/3sum-closest

和三数之和一样, 只需要在遍历时维护一个最接近`target`的值即可.

```cpp
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {

        int ans = 0x3f3f3f3f;
        sort(nums.begin(), nums.end());


        for (int i = 0; i < nums.size(); i ++) {

            if (i != 0 && nums[i] == nums[i - 1]) continue;

            int l = i + 1, r = nums.size() - 1;

            while (l < r) {
                int sum = nums[i] + nums[l] + nums[r];
                
                if (abs(sum - target) < abs(ans - target)) ans = sum;

                if (sum < target) {
                    l ++;
                    continue;
                }
                if (sum > target) {
                    r --;
                    continue;
                }
                do { l ++ ;} while (l < r && nums[l] == nums[l - 1]);
                do { r --;} while (l < r && nums[r] == nums[r + 1]);
            }
        }

        return ans;
    }
};
```



### 数组元素的目标和

> https://www.acwing.com/problem/content/802/

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int n, m, x;
int a[N], b[N];

int main()
{
    
    cin >> n >> m >> x;
    for (int i = 0; i < n; i ++) cin >> a[i];
    for (int i = 0; i < m; i ++) cin >> b[i];
    /* b数组从后遍历, a数组从前遍历 */
    for (int i = 0, j = m - 1; i < n; i ++) {
        while (j >= 0 && a[i] + b[j] > x) j --;
        if (a[i] + b[j] == x) printf("%d %d\n", i, j);
    }
    
    return 0;
}
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



## 判断子序列

> https://www.acwing.com/problem/content/description/2818/

遍历较长的字符串, 然后如果长字符串中有一个和子序列字符串相等, 就+1, 看最后加到的数值是否是子序列字符串的长度.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int n, m;
int a[N], b[N];

int main() {

    cin >> n >> m;

    for (int i = 0; i < n; i++) cin >> a[i];
    for (int i = 0; i < m; i ++) cin >> b[i];

    int j = 0;
    for (int i = 0; i < m; i ++) {
        // 不要忘记 j < n
        if (j < n && a[j] == b[i]) j ++;
    }
    if (j == n) puts("Yes");
    else puts("No");

    return 0;
}
```

## 外观数列

> https://leetcode.cn/problems/count-and-say/description/

典型的双指针算法.

```cpp
class Solution {
public:
    string countAndSay(int n) {
        if (n == 1) return "1";
        string prev = countAndSay(n - 1);

        vector<pair<int, int>> group;

        string res = "";

        for (int i = 0; i < prev.size(); i ++) {
            int j = i + 1;
            while (j < prev.size() && prev[j] == prev[i]) j ++;
            int cnt = j - i;
            res += cnt + '0';
            res += prev[i];
            i = j - 1;
        }

        return res;
    }
};
```
