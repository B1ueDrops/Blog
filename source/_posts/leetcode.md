---
title: LeetCode-1000题题解
categories: 算法
---

> *是LeetCode Hot 100中的题

[toc]



## 1. *两数之和(easy)

> https://leetcode.cn/problems/two-sum/

* 最无脑的做法: 遍历两个数的所有可能性, 判断和是否是`target`.

* 如果要优化成$O(n)$的做法, 就需要扫描一边, 扫描的过程中用之前积累的先验知识做.

  * 扫描的时候, 把之前遍历的数据存储到哈希表中, 每次扫描到一个数, 就判断`target - nums[i]`是否在哈希表中.
  * 时间复杂度: $O(n)$

  ```cpp
  class Solution {
  public:
      vector<int> twoSum(vector<int>& nums, int target) {
         vector<int> ans;
         unordered_map<int, int> hash;
         for (int i = 0; i < nums.size(); i ++) {
              if (hash.count(target - nums[i])) return {hash[target - nums[i]], i};
              hash[nums[i]] = i;
         }
         return ans;
      }
  };
  ```



## 2. *两数相加 (easy)

> https://leetcode.cn/problems/add-two-numbers/

* 数位从后向前枚举, 两个数位相加后, `t % 10`就是数位的值, 进位的值就是`t / 10`.

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        auto dummy = new ListNode(-1), cur = dummy;
        int t = 0;
        while (l1 || l2 || t) {
            if (l1) t += l1->val, l1 = l1->next;
            if (l2) t += l2->val, l2 = l2->next;
            cur = cur->next = new ListNode(t % 10);
            t /= 10;
        }
        return dummy->next;
    }
};
```



## 3. *无重复字符的最长子串(medium)

> https://leetcode.cn/problems/longest-substring-without-repeating-characters/

* 双指针算法的一个pattern在于, 当我遍历时, 移动一个指针`i`时, 我可以借助题目中的某些性质, 让另一个指针`j`不必走某些位置.
* 在这个题中, 从前到后遍历字符串, 并且统计每一个字符出现的次数`hash[s[i]]`:
  * 如果`hash[s[i]] > 1`, 证明`[j, i]`范围内的子串出现了重复, 这个时候`j`需要向前移动, 直到`hash[s[i]] == 1`.


```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int ans = 0;
        unordered_map<int, int> hash;
        for (int i = 0, j = 0; i < s.size(); i ++) {
            hash[s[i]] ++;
            while (j < i && hash[s[i]] > 1) hash[s[j ++]] --;
            ans = max(ans, i - j + 1);
        }
        return ans;
    }
};
```



## 4. 寻找两个正序数组的中位数(hard)

> https://leetcode.cn/problems/median-of-two-sorted-arrays/

考虑这样一个问题:

> 在两个排序的数组之间, 寻找第k小数.

如果这个问题能够解决, 那么中位数就是第`(n + m) / 2`小的数.

假设两个正序数组分别是$A$和$B$, 考虑$A[\frac{k}{2}]$和$B[\frac{k}{2}]$两个数:

* 如果$A[\frac{k}{2}] < B[\frac{k}{2}]$, 那么$A, B$数组中, 小于等于$A[\frac{k}{2}]$的数的个数就小于$k$, 此时两个数组中第$k$小的数肯定不会在$A$数组第$\frac{k}{2}$个元素的左边.
* 如果$A[\frac{k}{2}] > B[\frac{k}{2}]$, 那么$A, B$数组中, 小于等于$B[\frac{k}{2}]$的数的个数就小于$k$, 此时两个数组中第$k$小的数肯定不会在$B$数组第$\frac{k}{2}$个元素的左边.
* 如果$A[\frac{k}{2}] = B[\frac{k}{2}]$, 那么$A, B$数组中, 小于等于$A[\frac{k}{2}]$或$B[\frac{k}{2}]$的数的个数就等于$k$, 此时$A[\frac{k}{2}]$或者$B[\frac{k}{2}]$就是第$k$小的数.

根据以上分析, 定义一个函数, 函数的签名为`find(A, B, i, j, k)`, 表示给定两个数组$A, B$, 下标从$i, j$开始, 找到两个数组中第$k$小的数, 那么就可以用这个函数递归解决.

时间复杂度是$O(logk)$, 由于$k$最多是$m + n$, 那么时间复杂度就是$O(log(m + n))$

```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int n = nums1.size(), m = nums2.size(), k = (n + m) / 2;
        if ((n + m) % 2 == 0) {
            int left = find(nums1, nums2, 0, 0, k);
            int right = find(nums1, nums2, 0, 0, k + 1);
            return (left + right) / 2.0;
        }
        else return find(nums1, nums2, 0, 0, k + 1);
    }
    int find(vector<int> &nums1, vector<int> &nums2, int i, int j, int k) {
        int n = nums1.size(), m = nums2.size();
        // nums1默认作为较小的数组
        if (n - i > m - j) return find(nums2, nums1, j, i, k);
        // 如果nums1为空
        if (i == nums1.size()) return nums2[j + k - 1];
        // 如果k == 1
        if (k == 1) return min(nums1[i], nums2[j]);

        int si = min((int)(nums1.size() - 1), i + k / 2 - 1);
        int sj = j + (k - k / 2) - 1;
        if (nums1[si] <= nums2[sj]) return find(nums1, nums2, si + 1, j, k - (si - i + 1));
        else return find(nums1, nums2, i, sj + 1, k - (sj - j + 1));
    }
};
```



## 5. 最长回文子串(medium)

> https://leetcode.cn/problems/longest-palindromic-substring/

* 要枚举一个字符串中的所有回文串:
  * 首先, 遍历字符串中所有的字符`s[i]`.
  * 然后, 回文串可以分为两种, 奇数长度和偶数长度, 奇数长度从`i - 1`和`i + 1`进行扩展, 偶数长度从`i`和`i + 1`进行扩展.

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





## 11. *盛水最多的容器(medium)

> https://leetcode.cn/problems/container-with-most-water/

* 思路: 假设两根棍子`i, j`, 它们之间的盛水量就是`min(height[i], height[j]) * (j - i)`.
* 最无脑的做法: 枚举所有可能的`i, j`.
* 如果我要让盛水量最大, 我应该移动短板, 也就是当`height[i] <= height[j]`时, 我应该让`i++`才有机会让水量变大.

```cpp
class Solution {
public:
    int maxArea(vector<int>& h) {
        int res = 0;
        for (int i = 0, j = h.size() - 1; i < j; ) {
            res = max(res, min(h[i], h[j]) * (j - i));
            if (h[i] <= h[j]) i ++;
            else j --;
        }
        return res;
    }
};
```



## 15. *三数之和(medium)

> https://leetcode.cn/problems/3sum/

* 首先将数组排序.
* 之后, 从前到后, 枚举数组中的第一个数`nums[i]`.
  * 然后, 从`i + 1`和`nums.size() - 1`分别枚举第二个数和第三个数, 然后根据三个数的加和调整指针.

* 注意: 数组中可能有重复元素, 需要在枚举每一个指针的过程中去掉重复.

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        for (int i = 0; i < nums.size(); i ++) {
            if (i && nums[i] == nums[i - 1]) continue;
            int j = i + 1, k = nums.size() - 1;
            while (j < k) {
                int sum = nums[i] + nums[j] + nums[k];
                if (sum < 0) j ++;
                else if (sum > 0) k --;
                else {
                    res.push_back({nums[i], nums[j], nums[k]});
                    do {j ++;} while (j < k && nums[j] == nums[j - 1]);
                    do {k --;} while (j < k && nums[k] == nums[k + 1]);
                }
            }
        }
        return res;
    }
};
```



## 16. 最接近的三数之和(medium)

> https://leetcode.cn/problems/3sum-closest/

* 和三数之和完全一致, 只需要在求和`sum`的时候维护一个离`target`最近的`sum`值即可.

```cpp
class Solution {
public:
    typedef long long LL;
    int threeSumClosest(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());
        int ans = 0x3f3f3f3f;
        for (int i = 0; i < nums.size(); i ++) {
            if (i && nums[i] == nums[i - 1]) continue;
            int l = i + 1, r = nums.size() - 1;
            while (l < r) {
                LL sum = (LL)nums[i] + nums[l] + nums[r];
                if (abs(sum - target) < abs(ans - target)) ans = sum;
                if (sum < target) l ++;
                else if (sum > target) r --;
                else {
                    do {l ++;} while (l < r && nums[l] == nums[l - 1]);
                    do {r --;} while (l < r && nums[r] == nums[r + 1]);
                }
            }
        }
        return ans;
    }
};
```





## 17. *电话号码的字母组合(medium)

> https://leetcode.cn/problems/letter-combinations-of-a-phone-number/

* 搜索顺序是: 对于每一个坑位, 枚举这个坑位上能够放置的所有可能性, 放置之后, 就递归到下一个位置, 注意恢复现场.
* 注意: 如果输入是空字符串的话, 需要在搜索的时候特判一下, 不要把空的`path`放到`ans`中.

```cpp
class Solution {
public:
    string path;
    vector<string> ans;
    string books[10] = {
        "", "", "abc", "def",
        "ghi", "jkl", "mno",
        "pqrs", "tuv", "wxyz"
    };
    vector<string> letterCombinations(string digits) {
        dfs(digits, 0);
        return ans;    
    }
    void dfs(string &digits, int u) {
        if (digits.size() && u == digits.size()) {
            ans.push_back(path);
            return ;
        }
        for (auto c: books[digits[u] - '0']) {
            path += c;
            dfs(digits, u + 1);
            path.pop_back();
        }
    }
};
```



## 18. 四数之和 (medium)

> https://leetcode.cn/problems/4sum/description/

* 和三数之和完全一致的做法, 只不过多了一层循环.
* 注意: 1. 数组先要排序. 2. `sum`需要用`long long`来存储.

```cpp
class Solution {
public:
    typedef long long LL;
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> ans;
        for (int i = 0; i < nums.size(); i ++) {
            if (i && nums[i] == nums[i - 1]) continue;
            for (int j = i + 1; j < nums.size(); j ++) {
                if (j != i + 1 && nums[j] == nums[j - 1]) continue;
                int l = j + 1, r = nums.size() - 1;
                while (l < r) {
                    LL sum = (LL)nums[i] + nums[j] + nums[l] + nums[r];
                    if (sum < target) l ++;
                    else if (sum > target) r --;
                    else {
                        ans.push_back({nums[i], nums[j], nums[l], nums[r]});
                        do { l ++; } while (l < r && nums[l] == nums[l - 1]);
                        do { r --; } while (l < r && nums[r] == nums[r + 1]);
                    }
                }
            }
        }
        return ans;
    }
};
```





## 19. *删除链表的倒数第n个节点(medium)

> https://leetcode.cn/problems/remove-nth-node-from-end-of-list/

* 由于链表头节点可能被删除, 因此需要虚拟头节点.

* 准备两个快慢指针, 初始放在dummy, 快指针从dummy开始走n步, 之后慢指针和快指针同时走, 快指针走到最后一个节点的时候, 慢指针就会停到倒数第n个节点的前一个节点, 直接删除即可.


```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
       auto dummy = new ListNode(-1), cur = dummy;
       dummy->next = head;
       auto p = dummy;
       for (int i = 0; i < n; i ++) p = p->next;
       auto q = dummy;
       // 注意, 这里要跳到要删除节点的前一个节点
       while (p->next) p = p->next, q = q->next;
       q->next = q->next->next;
       return dummy->next;
    }
};
```



## 20. *有效的括号(easy)

> https://leetcode.cn/problems/valid-parentheses/

* 遇到左括号就压栈.
* 如果遇到右括号需要注意几点:
  * 第一, 只有栈顶的左括号和有括号成功匹配, 才弹出.
  * 第二, 如果遇到右括号, 但是栈顶没有元素, 直接不匹配, 例如`]`.
  * 第三, 如果遇到右括号, 但是栈顶不匹配, 那么直接就不匹配, 例如`(]`.
  * 第二和第三需要特殊判断.


```cpp
class Solution {
public:
    bool isValid(string s) {
       stack<char> stk;
       for (auto c : s) {
            if (c == '(' || c == '{' || c == '[') stk.push(c);
            else {
                if (stk.empty()) return false;
                if (c == ')' && stk.top() == '(') stk.pop();
                else if (c == '}' && stk.top() == '{') stk.pop();
                else if (c == ']' && stk.top() == '[') stk.pop();
                else return false;
            }
       }
       return stk.empty();
    }
};
```





## 21. *合并两个有序的链表 (easy)

> https://leetcode.cn/problems/merge-two-sorted-lists/

* 和归并排序合并区间的过程一样.
* 首先需要枚举两个链表指针都存在的情况, 也就是`l1 && l2`.
* 之后再枚举某一个链表没走完的情况, 也就是`while (l1) while (l2)`.

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        auto dummy = new ListNode(-1), cur = dummy;
        while (l1 && l2) {
            if (l1->val <= l2->val) cur = cur->next = new ListNode(l1->val), l1 = l1->next;
            else cur = cur->next = new ListNode(l2->val), l2 = l2->next;
        }
        while (l1) cur = cur->next = new ListNode(l1->val), l1 = l1->next;
        while (l2) cur = cur->next = new ListNode(l2->val), l2 = l2->next;
        return dummy->next;
    }
};
```



## 22. *括号生成(medium)

> https://leetcode.cn/problems/generate-parentheses/description/

* 一个括号序列是合法的充要条件是:

  * 序列中, 左右括号数量相等.
  * 任意前缀, 左括号数量大于等于有括号数量.

* 给定了n对括号, 那么就有2n个坑位可以放置`(`和`)`, 那么搜索的顺序就是这样:
* 只要之前左括号数量小于`n`, 那么当前位置就可以放左括号.
  * 只要左括号数量大于右括号数量, 那么当前位置就可以放右括号.
  * 总而言之


```cpp
class Solution {
public:
    int n;
    string path;
    vector<string> ans;
    vector<string> generateParenthesis(int _n) {
        n = _n;
        dfs(0, 0);
        return ans;
    }
    void dfs(int lc, int rc) {
        if (lc == n && rc == n) {
            ans.push_back(path);
            return ;
        }
        if (lc < n) {
            path += '(';
            dfs(lc + 1, rc);
            path.pop_back();
        }
        if (rc < n && lc > rc) {
            path += ')';
            dfs(lc, rc + 1);
            path.pop_back();
        }
    }
};
```





## 23. *合并K个升序链表(hard)

> https://leetcode.cn/problems/merge-k-sorted-lists/

* 链表的k路归并问题, 可以采用堆排序解决.
  * 将每一个链表头节点插入堆.
  * 然后从堆中找到最小元素.
  * 之后再将最小元素的下一个头节点插入堆.
* 注意: 一开始将链表头节点插入堆时, 链表头可能为空, 需要注意判断.

```cpp
class Solution {
public:
    struct cmp {
        bool operator() (ListNode *a, ListNode *b) {
            return a->val > b->val;
        }
    };
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto dummy = new ListNode(-1);
        dummy->next = NULL;
        auto cur = dummy;
        priority_queue<ListNode*, vector<ListNode*>, cmp> heap;

        for (auto l: lists)
            if (l) heap.push(l);
            
        while (heap.size()) {
            auto t = heap.top(); heap.pop();
            cur = cur->next = new ListNode(t->val);
            if (t->next) heap.push(t->next);
        }
        return dummy->next;
    }
};

```





## 24. *两两交换链表中的节点(medium)

> https://leetcode.cn/problems/swap-nodes-in-pairs/

* 首先, 链表头节点可能没有, 所以需要虚拟头节点.
* 其次, 如果要两两交换链表中的节点, 需要在链表中, 把每两个节点看作一个整体, 假设这个整体的第一个节点是`r`, 最后一个节点是`q`.
  * `r`的前一个节点是`p`.
  * 那么首先需要调整整体的指针, `p->next = q, r->next = q->next`.
  * 然后再调整整体内部的指针: `q->next = r`.
  * 然后更新`p`, 指向下一个整体的前一个节点: `p = r`.
  

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
       auto dummy = new ListNode(-1);
       dummy->next = head;

       for (auto p = dummy; p->next && p->next->next; ) {
            auto q = p;
            for (int i = 0; i < 2 && q; i ++) q = q->next;
            auto u = p->next;
            auto a = p->next, b = p->next->next;
            p->next = q;
            a->next = q->next;
            while (a != q) {
                auto c = b->next;
                b->next = a;
                a = b, b = c;
            }
            p = u;
       }
       return dummy->next;
    }
};
```



## 25. *K个一组翻转链表(hard)

> https://leetcode.cn/problems/reverse-nodes-in-k-group/

* 首先, 头节点会被改变, 因此要加上虚拟头节点.
* 其次, 先找到这k个节点组成的一个集团, 以及这个集团的前一个节点`p`, 和最后的节点`q`.
* 之后, 先动边界节点, 然后修改集团内部的节点.
* 最后, 注意指针`p`的移动.

```cpp
class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        auto dummy = new ListNode(-1);
        dummy->next = head;

        for (auto p = dummy; p->next && p->next->next;) {
            auto q = p;
            for (int i = 0; i < k && q; i ++) q = q->next;
            if (!q) break;
            auto u = p->next;
            auto a = p->next, b = p->next->next;
            p->next = q;
            a->next = q->next;
            while (a != q) {
                auto c = b->next;
                b->next = a;
                a = b, b = c;
            }
            p = u;
        }
        return dummy->next;
    }
};
```



## 31. *下一个排列 (medium)

> https://leetcode.cn/problems/next-permutation/

* 从后向前遍历, 找到第一个逆序点`k`, 也就是`nums[k - 1] < nums[k]`.
  * `[k, nums.size() - 1]`这一段是反向升序的.
  * 那么在`[k, nums.size() - 1]`这一段, 找到最小的, 但是大于`nums[k - 1]`的数, 交换到`nums[k - 1]`的位置上.
  * 然后将`[k, nums.size() - 1]`这一段逆序 (也就是变成升序).
* 直觉上讲, 这种操作相当于让序列的第一个转折点变大, 那么字典序就是变大, 让后面的序列变成正序, 后面的字典序最小, 也就是下一个字典序的位置.

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int k = nums.size() - 1;
        while (k > 0 && nums[k - 1] >= nums[k]) k --;
        if (k <= 0) {
            reverse(nums.begin(), nums.end());
            return ;
        }
        else {
            int t = k;
            while (t < nums.size() && nums[t] > nums[k - 1]) t ++;
            swap(nums[k - 1], nums[t - 1]);
            reverse(nums.begin() + k, nums.end());
        }
    }
};
```





## 34. *在排序数组中查找元素的第一个和最后一个位置(medium)

> https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/

* 整数二分的问题, 能够通过一个数组中的元素是否满足某个性质, 把数组分成两个部分.
* 这两个部分有两个边界:
  * 如果要求左边界: `int mid = l + (r - l) / 2 + 1`
    * 满足左侧性质: `l = mid`
    * 满足右侧性质: `r = mid - 1`

  * 如果要求右边界: `int mid = l + (r - l) / 2`
    * 满足左侧性质: `l = mid + 1`
    * 满足右侧性质: `r = mid`

* 注意实现时需要特判数组是空的情况.

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int n = nums.size();
        if (!n) return {-1, -1};

        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] < target) l = mid + 1;
            else r = mid;
        }
        if (nums[l] != target) return {-1, -1};
        int left = l;
        l = 0, r = n - 1;
        while (l < r) {
            int mid = l + (r - l) / 2 + 1;
            if (nums[mid] <= target) l = mid;
            else r = mid - 1;
        }
        if (nums[l] != target) return {-1, -1};
        int right = l;
        return {left, right};
    }
};
```



## 35. *搜索插入位置(easy)

> https://leetcode.cn/problems/search-insert-position/

* 假设二分的性质是: `nums[i] < x`, 和`nums[i] >= x`, 那么插入的位置就是右边界.
* 注意: 如果插入的位置是数组的末尾, 那么答案应该是`n`, 但是二分不会得到这个答案, 需要特判.

```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int n = nums.size();
        if (nums[n - 1] < target) return n;
        
        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] < target) l = mid + 1;
            else r = mid;
        } 
        return l;
    }
};

```



## 38. 外观数列(medium)

> https://leetcode.cn/problems/count-and-say/description/

* 本质上, 这个题就是在统计字符串中, 每一个连续相同字符的序列的长度, 直接用双指针统计即可.

```cpp
class Solution {
public:
    string countAndSay(int n) {
        if (n == 1) return "1";
        string s = countAndSay(n - 1);

        string res;
        for (int i = 0; i < s.size(); i ++) {
            int j = i + 1;
            while (j < s.size() && s[j] == s[i]) j ++;
            int cnt = j - i;
            res += cnt + '0';
            res += s[i];
            i = j - 1;
        }
        return res;
    }
};
```





## 39. *组合总和(medium)

> https://leetcode.cn/problems/combination-sum/

* 搜索的顺序是: 对于`candidates`数组中的每一个数, 枚举这个数被选择的次数.

```cpp
class Solution {
public:
    vector<int> path;
    vector<vector<int>> ans;
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        dfs(candidates, target, 0);
        return ans;
    }
    void dfs(vector<int> &nums, int sum, int u) {
        if (u == nums.size() || sum == 0) {
            if (sum == 0)
                ans.push_back(path);
            return ;
        }
        for (int i = 0; nums[u] * i <= sum; i ++) {
            dfs(nums, sum - nums[u] * i, u + 1);
            path.push_back(nums[u]);
        }
        for (int i = 0; nums[u] * i <= sum; i ++)
            path.pop_back();
    }
};

```





## 41. *缺失的第一个正数(medium)

> https://leetcode.cn/problems/first-missing-positive/

* 如果让数组中所有的正整数元素满足`nums[i] == i`, 那么从前到后遍历正整数, 第一个不满足`nums[i] != i`的就是缺失的第一个整数.

  * 如果所有的正整数都满足, 那么缺失的第一个正整数就是`n + 1`.

* 对于这段代码:

  ```cpp
  while (nums[i] >= 0 && nums[i] < n && nums[i] != nums[nums[i]])
  	swap(nums[i], nums[nums[i]]);
  ```

  * 每一次交换, 就相当于把`nums[i]`这个值, 放到了以`nums[i]`为下标的位置, 每一次都可以排好一个值.

```cpp
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int n = nums.size();
        for (int i = 0; i < n; i ++)
            if (nums[i] != INT_MIN) nums[i] --;

        for (int i = 0; i < n; i ++)
            while (nums[i] >= 0 && nums[i] < n && nums[nums[i]] != nums[i])
                swap(nums[i], nums[nums[i]]);
                
        for (int i = 0; i < n; i ++)
            if (nums[i] != i)
                return i + 1;
        return n + 1;
    }
};
```





## 42. *接雨水(hard)

> https://leetcode.cn/problems/trapping-rain-water/

* 一个格子`h[i]`对整体雨水的贡献量是: `min(l[i], r[i]) - h[i]`.
  * `l[i], r[i]`分别表示格子`i`左右两侧(包括`h[i]`), 的最大的格子高度.


```cpp
class Solution {
public:
    int trap(vector<int>& h) {
        int n = h.size();
        vector<int> l(n), r(n);
        
        int maxv = 0;
        for (int i = 0; i < n; i ++) {
            maxv = max(maxv, h[i]);
            l[i] = maxv;
        }
        maxv = 0;
        for (int i = n - 1; i >= 0; i --) {
            maxv = max(maxv, h[i]);
            r[i] = maxv;
        }
        int res = 0;
        for (int i = 0; i < n; i ++)
            res += min(l[i], r[i]) - h[i];
        return res;
    }
};
```



## 45. *跳跃游戏II(medium)

> https://leetcode.cn/problems/jump-game-ii/

* 假设`f[i]`表示到达位置`i`所需要的最小跳跃次数.
* 那么`f[i] = f[j] + 1`, 其中`j`是最小的, 能够一步跳到`i`的位置.
* `i, j`的维护逻辑类似于双指针算法.
* 注意, 这里`i`要从1开始, 因为`f[0] = 0`是一种特殊情况, 不能参与到循环中, 一旦进入循环就会被更新.

```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int n = nums.size();
        vector<int> f(n);

        for (int i = 1, j = 0; i < n; i ++) {
            while (j < i && j + nums[j] < i) j ++;
            f[i] = f[j] + 1;
        }
        return f[n - 1];
    }
};
```



## 46. *全排列(medium)

> https://leetcode.cn/problems/permutations/

* 本题是无重复元素的数组求全排列.
* 搜索顺序是: 对于每一个全排列上的坑位, 枚举这个坑位填哪一个数, 如果一个数被填了就需要记录, 防止重复填.

```cpp
class Solution {
public:
    vector<bool> st;
    vector<int> path;
    vector<vector<int>> ans;
    vector<vector<int>> permute(vector<int>& nums) {
        st.resize(nums.size()); path.resize(nums.size());
        dfs(nums, 0);
        return ans;
    }
    void dfs(vector<int> &nums, int u) {
        if (u == nums.size()) {
            ans.push_back(path);
            return ;
        }
        for (int i = 0; i < nums.size(); i ++) {
            if (st[i]) continue;
            path[u] = nums[i]; st[i] = true;
            dfs(nums, u + 1);
            st[i] = false;
        }
    }
};

```



## 48. *旋转图像(medium)

> https://leetcode.cn/problems/rotate-image/

* 顺时针90度: 主对角线对称(左上-右下), 竖直轴线对称.
* 逆时针90度: 主对角线对称(左上-右下), 水平轴线对称.
* 顺时针/逆时针180度: 主对角线对称 + 副对角线对称.

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
       int n = matrix.size(), m = matrix[0].size();
       for (int i = 0; i < n; i ++)
            for (int j = 0; j < i; j ++)
                swap(matrix[i][j], matrix[j][i]);
        for (int i = 0; i < n; i ++)
            for (int j = 0, k = m - 1; j < k; j ++, k --)
                swap(matrix[i][j], matrix[i][k]);
    }
};
```





## 49. *字母异位词分组(medium)

> https://leetcode.cn/problems/group-anagrams/

* 这个题的题意是: 给定一个字符串的数组, 如果将这个数组中的元素分为若干个group, 每一个group中的所有字符串经过排序后都能得到同样的结果.
* 那么直接用哈希表, key作为排序后的结果, value就是所有符合题意的字符串即可.

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> hash;
        for (auto &str: strs) {
            auto nstr = str;
            sort(nstr.begin(), nstr.end());
            hash[nstr].push_back(str);
        }
        vector<vector<string>> ans;
        for (auto [_, s]: hash) ans.push_back(s);
        return ans;
    }
};
```



## 51. *N皇后 (hard)

> https://leetcode.cn/problems/n-queens/

* 用`col`, `dg`, `udg`分别记录每一列, 对角线/反对角线上是否有`Q`.
* 遍历时先遍历每一行`u`, 然后遍历每一列`i`, 如果一个位置`(u, i)`在列, 对角线, 反对角线上都没有棋子, 那么就可以放下棋子, 然后递归到下一个位置. 

```cpp
class Solution {
public:
    int n;
    vector<bool> col, dg, udg;
    vector<string> path;
    vector<vector<string>> ans;
    vector<vector<string>> solveNQueens(int _n) {
        n = _n;
        col = vector<bool>(n, false);
        dg = vector<bool>(n, false);
        udg = vector<bool>(n, false);
        path = vector<string>(n, string(n, '.'));
        dfs(0);
        return ans;
    }
    void dfs(int u) {
        if (u == n) {
            ans.push_back(path);
            return ;
        }
        for (int i = 0; i < n; i ++) {
            if (col[i] || dg[u + i] || udg[n + i - u]) continue;
            col[i] = dg[u + i] = udg[n + i - u] = true;
            path[u][i] = 'Q';
            dfs(u + 1);
            path[u][i] = '.';
            col[i] = dg[u + i] = udg[n + i - u] = false;
        }
    }
};

```



## 53. *最大子数组和(medium)

> https://leetcode.cn/problems/maximum-subarray/

* 假设`f[i]`表示以`i`结尾的所有子数组中, 和的最大值.
* 那么状态转移就是: `f[i] = max(nums[i], f[i - 1] + nums[i])`.
* 最后的答案就是求所有`f[i]`的最大值.

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int f = nums[0];
        int ans = nums[0];
        for (int i = 1; i < nums.size(); i ++) {
            f = max(f + nums[i], nums[i]);
            ans = max(ans, f);
        }
        return ans;
    }
};
```



## 54. *螺旋矩阵(medium)

> https://leetcode.cn/problems/spiral-matrix/

* 遍历时, 一旦遇到下标越界/边界已经搜索过的情况, 就把`dx, dy`数组的方向改变一下即可.
* 注意: x的正方向是下, y的正方向是右.

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int> ans;
        int dx[] = {0, 1, 0, -1};
        int dy[] = {1, 0, -1, 0};
        int n = matrix.size(), m = matrix[0].size();
        vector<vector<bool>> st(n, vector<bool>(m, false));
        int x = 0, y = 0, d = 0;
        for (int k = 0; k < n * m; k ++) {
            ans.push_back(matrix[x][y]);
            st[x][y] = true;
            int a = x + dx[d], b = y + dy[d];
            if (a < 0 || a >= n || b < 0 || b >= m || st[a][b])
                d = (d + 1) % 4;
            x += dx[d], y += dy[d];
        }
        return ans;
    }
};
```



## 55. *跳跃游戏(medium)

> https://leetcode.cn/problems/jump-game/

* 对于数组中的每一个数`nums[i]`, 它能够跳到的最大的位置就是`i + nums[i]`.
* 那么, 当遍历到`i`时, 如果之前能够跳到的最大位置`j < i`, 那么就无法到达这个位置, 因此, 就无法到达终点.

```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int f = 0 + nums[0];
        for (int i = 1; i < nums.size(); i ++) {
            if (f < i) return false;
            f = max(f, i + nums[i]);
        }
        return true;
    }
};
```





## 56. *合并区间(medium)

> https://leetcode.cn/problems/merge-intervals/

* 首先, 区间按照左端点排序.
* 然后, 用`[st, ed]`维护当前右端点最大的区间, 然后依次枚举当前区间`[l, r]`:
  * 如果`l <= ed`, 有交集, 那么直接更新`ed = max(ed, r)`.
  * 如果`l > ed`, 没交集, 那么需要先把`[st, ed]`放在答案中, 然后更新区间`st = l, ed = r`.

* 注意: 枚举完之后, 最后一个区间`[st, ed]`也要放到答案中.

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end());
        vector<vector<int>> ans;
        int st = -1, ed = -1;
        for (auto &range: intervals) {
            int l = range[0], r = range[1];
            if (l <= ed)
                ed = max(ed, r);
            else {
                if (ed != -1) ans.push_back({st, ed});
                st = l, ed = r;
            }
        }
        if (ed != -1) ans.push_back({st, ed});
        return ans;
    }
};
```



## 62. *不同路径(medium)

> https://leetcode.cn/problems/unique-paths/

* 设`f[i][j]`表示从起点走到`(i, j)`的路径数量.
* 首先初始化起点, 路径数量是1: `f[0][0] = 1`.
* 之后更新状态: `f[i][j] += f[i - 1][j]`, 以及`f[i][j] += f[i][j - 1]`.

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> f(m, vector<int>(n, 0));
        for (int i = 0; i < m; i ++) {
            for (int j = 0; j < n; j ++) {
                if (!i && !j) f[i][j] = 1;
                else {
                    if (i) f[i][j] += f[i - 1][j];
                    if (j) f[i][j] += f[i][j - 1];
                }
            }
        }
        return f[m - 1][n - 1];
    }
};
```



## 64. *最小路径和(medium)

> https://leetcode.cn/problems/minimum-path-sum/

* 设`f[i][j]`表示从起点走到`(i, j)`的最小路径和.
* 首先初始化所有位置的`f[i][j]`为`INF`, 然后将起点`f[0][0] = grid[0][0];`
* 之后更新状态: `f[i][j] = min(f[i - 1][j], f[i][j - 1]) + grid[i][j];`

```cpp
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        vector<vector<int>> f(n, vector<int>(m, 0x3f3f3f3f));

        for (int i = 0; i < n; i ++) {
            for (int j = 0; j < m; j ++) {
                if (!i && !j) f[i][j] = grid[i][j];
                else {
                    if (i) f[i][j] = min(f[i][j], f[i - 1][j] + grid[i][j]);
                    if (j) f[i][j] = min(f[i][j], f[i][j - 1] + grid[i][j]);
                }
            }
        }
        return f[n - 1][m - 1];
    }
};
```





## 70. *爬楼梯 (easy)

> https://leetcode.cn/problems/climbing-stairs/

* 斐波那契数列的第0项是1, 第1项也是1.

```cpp
class Solution {
public:
    int climbStairs(int n) {
        typedef long long LL;
        LL a = 1, b = 1;

        while (n --) {
            LL c = a + b;
            a = b, b = c;
        }
        return a;
    }
};
```



## 72. *编辑距离(medium)

> https://leetcode.cn/problems/edit-distance/description/

* 假设`f[i][j]`表示从`word1[1:i]`转换为`word2[1:j]`的最小编辑次数, 针对`word1[i]`的操作, 状态转移分为三种情况:
  * 增加`word1[i]`: 那么就要求`word[1:i]`和`word2[1:j-1]`匹配.
  * 删除`word1[i]`: 要求`word[1:i-1]`和`word2[1:j]`匹配
  * 替换`word1[i]`: 要求`word[1:i-1]`和`word2[1:j-1]`匹配, 并且`word1[i] == word2[j]`.
    * 如果`word1[i] == word2[j]`, 那么就可以直接从`f[i - 1][j - 1]`转移.
* 注意初始化`f[i][0]`和`f[0][i]`为`i`.

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.size(), m = word2.size();
        word1 = ' ' + word1;
        word2 = ' ' + word2;
        vector<vector<int>> f(n + 1, vector<int>(m + 1, 0));
        for (int i = 0; i <= n; i ++) f[i][0] = i;
        for (int i = 0; i <= m; i ++) f[0][i] = i;
        for (int i = 1; i <= n; i ++)
            for (int j = 1; j <= m; j ++) {
                f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1);
                if (word1[i] == word2[j])
                    f[i][j] = min(f[i][j], f[i - 1][j - 1]);
                else
                    f[i][j] = min(f[i][j], f[i - 1][j - 1] + 1);
            }
        return f[n][m];
    }
};
```





## 73. *矩阵置零(medium)

> https://leetcode.cn/problems/set-matrix-zeroes/

* 原地算法:
  * 首先记录一下第0行和第0列是否需要清0.
  * 然后从第1行/第1列开始对矩阵进行判断, 如果第i行包含0元素, 就在`matrix[i][0]`处标记为0.
    * 原来的`matrix[i][0]`可能是0, 也可能不是0, 这个时候如果标记为0, 那么原来的值就不知道, 那么后面就无法判断第0行和第0列是否需要清0, 所以需要预先处理一下第0行和第0列.
  * 然后根据标记的`matrix[i][0]`和`matrix[0][i]`进行清理.
  * 之后清理第0行和第0列.

```cpp
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int n = matrix.size(), m = matrix[0].size();
        bool row = false, col = false;
        for (int i = 0; i < n; i ++)
            if (!matrix[i][0])
                col = true;
        for (int i = 0; i < m; i ++)
            if (!matrix[0][i])
                row = true;
        for (int i = 1; i < n; i ++)
            for (int j = 1; j < m; j ++)
                if (!matrix[i][j]) {
                    matrix[i][0] = 0;
                    matrix[0][j] = 0;
                }
        // 注意这里要从1开始遍历
        for (int i = 1; i < n; i ++)
            if (!matrix[i][0]) {
                for (int j = 1; j < m; j ++)
                    matrix[i][j] = 0;
            }
        for (int i = 1; i < m; i ++)
            if (!matrix[0][i]) {
                for (int j = 1; j < n; j ++)
                    matrix[j][i] = 0;
            }
        if (row) {
            for (int i = 0; i < m; i ++)
                matrix[0][i] = 0;
        }
        if (col) {
            for (int i = 0; i < n; i ++)
                matrix[i][0] = 0;
        }
    }
};
```



## 74. *搜索二维矩阵(medium)

> https://leetcode.cn/problems/search-a-2d-matrix/

* 二维矩阵和一维数组没有什么区别, 一位数组的下标`idx`分别除以/模矩阵列数就是在矩阵中的坐标`(idx / m, idx % m)`.

```cpp
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int n = matrix.size(), m = matrix[0].size();
        int l = 0, r = n * m - 1;
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (matrix[mid / m][mid % m] < target) l = mid + 1;
            else r = mid;
        }
        return matrix[l / m][l % m] == target;
    }
};
```



## 75. *颜色分类(medium)

> https://leetcode.cn/problems/sort-colors/

* 核心思想是扫描数组的时候, 维护三个区域, 一个区域全是0, 一个区域全是1, 另一个区域全是2.

* 维护三个指针`i, j, k`, `j`扫描数组:
  * 如果`nums[j] == 0`, 那么`0`就放置到`i`处, 然后让`i`指针移动.
  * 如果`nums[j] == 1`, 那么直接`j ++`.
  * 如果`nums[j] == 2`, 那么直接交换到`k`指针处, 然后`k --`.


```cpp
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int i = 0, j = 0, k = nums.size() - 1;
        while (j <= k) {
            if (nums[j] == 0) swap(nums[i ++], nums[j ++]);
            else if (nums[j] == 1) j ++;
            else if (nums[j] == 2) swap(nums[j], nums[k --]);
        }
    }
};
```



## 76. *最小覆盖子串(hard)

> https://leetcode.cn/problems/minimum-window-substring/

* 思路: 当指针`i`向前移动时, 向后移动指针`j`, 直到子串完全覆盖`t`中的所有字符.
* 然后向左移动指针, 找到长度最小的, 也可以完全覆盖`t`中字符的最小子串.
  * `hash[s[i]]`表示`t`中字符在`s`中出现的次数, 如果`hash[s[j]] + 1 > 0`, 那么表示如果`j`再向前移动, 就无法覆盖`t`中所有字符了.

```cpp
class Solution {
public:
    string minWindow(string s, string t) {
        int n = s.size();
        unordered_map<char, int> hash;
        for (auto c: t) hash[c] ++;
        int tot = hash.size();

        int k = 0, len = n;
        string ans;
        for (int i = 0, j = 0; i < n; i ++) {
            if (hash.count(s[i]) && --hash[s[i]] == 0) k ++;
            if (k == tot) {
                while (j < i) {
                    if (hash.count(s[j]) && hash[s[j]] + 1 > 0) break;
                    if (hash.count(s[j])) hash[s[j]] ++;
                    j ++;
                }
                if (i - j + 1 <= len) {
                    len = i - j + 1;
                    ans = s.substr(j, i - j + 1);
                }
            }
        }
        return ans;
    }
};
```



## 78. *子集(medium)

> https://leetcode.cn/problems/subsets/

* 递归写法:
  * 枚举数组中的每个数选/不选即可.

```cpp
class Solution {
public:
    vector<int> path;
    vector<vector<int>> ans;
    vector<vector<int>> subsets(vector<int>& nums) {
        dfs(nums, 0);
        return ans;
    }
    void dfs(vector<int> &nums, int u) {
        if (u == nums.size()) {
            ans.push_back(path);
            return ;
        }
        dfs(nums, u + 1);
        path.push_back(nums[u]);
        dfs(nums, u + 1);
        path.pop_back();
    }
};

```

* 迭代写法: 用二进制枚举子集

```cpp
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> ans;
        int n = nums.size();
        for (int i = 0; i < 1 << n; i ++) {
            vector<int> path;
            for (int j = 0; j < n; j ++)
                if (i >> j & 1)
                    path.push_back(nums[j]);
            ans.push_back(path);
        }
        return ans;
    }
};
```



## 79. *单词搜索(medium)

> https://leetcode.cn/problems/word-search/

* 遍历网格中的每一个格子, 从每一个格子开始进行搜索.
  * 对于每一个格子, 如果它与当前单词字符相等, 就把格子设置为`.` (标记为已搜索过). 然后扩展到其他合法格子.

```cpp
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
       int n = board.size(), m = board[0].size();
       for (int i = 0; i < n; i ++)
            for (int j = 0; j < m; j ++)
                if (dfs(board, word, 0, i, j))
                    return true;
        return false;
    }
    bool dfs(vector<vector<char>> &board, string word, int u, int x, int y) {
        if (board[x][y] != word[u]) return false;
        if (u == word.size() - 1) return true;

        char t = board[x][y];
        board[x][y] = '.';
        int n = board.size(), m = board[0].size();
        int dx[] = {-1, 0, 1, 0};
        int dy[] = {0, 1, 0, -1};
        for (int i = 0; i < 4; i ++) {
            int a = x + dx[i], b = y + dy[i];
            if (a < 0 || a >= n || b < 0 || b >= m || board[a][b] == '.') continue;
            if (dfs(board, word, u + 1, a, b)) return true;
        }
        board[x][y] = t;
        return false;
    }
};
```



## 84. *柱状图中最大的矩形(hard)

> https://leetcode.cn/problems/largest-rectangle-in-histogram/

* 对于每个柱子`h[i]`, 找到它左边, 右边第一个比它小的位置下标`l, r`.
* 那么这个矩形的面积就是`(r - l + 1) * h[i]`, 枚举这个值就可以.

```cpp
class Solution {
public:
    int largestRectangleArea(vector<int>& h) {
        vector<int> left, right;

        stack<int> stk;
        for (int i = 0; i < h.size(); i ++) {
            while (stk.size() && h[i] <= h[stk.top()]) stk.pop();
            if (stk.empty()) left.push_back(0);
            else left.push_back(stk.top() + 1);
            stk.push(i);
        }

        stk = stack<int>();
        for (int i = h.size() - 1; i >= 0; i --) {
            while (stk.size() && h[i] <= h[stk.top()]) stk.pop();
            if (stk.empty()) right.push_back(h.size() - 1);
            else right.push_back(stk.top() - 1);
            stk.push(i);
        }
        reverse(right.begin(), right.end());
        
        int ans = -1;
        for (int i = 0; i < h.size(); i ++)
            ans = max(ans, h[i] * (right[i] - left[i] + 1));

        return ans;
    }
};
```





## 94. *二叉树的中序遍历(easy)

> https://leetcode.cn/problems/binary-tree-inorder-traversal/

* 递归写法:

  ```cpp
  class Solution {
  public:
      vector<int> ans;
      vector<int> inorderTraversal(TreeNode* root) {
          dfs(root);
          return ans; 
      }
      void dfs(TreeNode *root) {
          if (!root) return ;
          dfs(root->left);
          ans.push_back(root->val);
          dfs(root->right);
      }
  };
  
  ```

* 迭代写法:

  ```cpp
  class Solution {
  public:
      vector<int> ans;
      stack<TreeNode *> stk;
      vector<int> inorderTraversal(TreeNode* root) {
        // 这里root和stk.size()是两个判断条件.
        // root用来判断上一个处理节点是否有右节点, 如果有就递归到右节点
        // stk.size()用来判断上一个节点如果没有右节点, 就从栈中弹出上一个节点处理
          while (root || stk.size()) {
              while (root) {
                  stk.push(root);
                  root = root->left;
              }
              root = stk.top(); stk.pop();
              ans.push_back(root->val);
              root = root->right;
          }
          return ans;
      }
  };
  
  ```



## 98. *验证二叉搜索树(medium)

> https://leetcode.cn/problems/validate-binary-search-tree/

* dfs函数返回一个节点所在子树的元素最小值和最大值.
* 递归向上比较左子树最大值, 右子树最小值和当前节点值的关系即可验证.

```cpp
class Solution {
public:
    bool ans = true;
    bool isValidBST(TreeNode* root) {
        if (!root) return true;
        dfs(root);
        return ans;
    }
    vector<int> dfs(TreeNode *root) {
        int minv = root->val, maxv = root->val;
        if (root->left) {
            auto t = dfs(root->left);
            if (t[1] >= root->val) ans = false;
            minv = min(minv, t[0]);
            maxv = max(maxv, t[1]);
        }
        if (root->right) {
            auto t = dfs(root->right);
            if (t[0] <= root->val) ans = false;
            minv = min(minv, t[0]);
            maxv = max(maxv, t[1]);
        }
        return {minv, maxv};
    }
};

```





## 101. *对称二叉树(easy)

> https://leetcode.cn/problems/symmetric-tree/

* 递归判断一个节点的左节点和右节点是否相等即可.

  ```cpp
  class Solution {
  public:
      bool isSymmetric(TreeNode* root) {
          if (!root) return true;
          return dfs(root->left, root->right); 
      }
      bool dfs(TreeNode *p, TreeNode *q) {
          if (!p || !q) return !p && !q;
          if (p->val != q->val) return false;
          return dfs(p->left, q->right) && dfs(p->right, q->left);
      }
  };
  
  ```



## 102. *二叉树的层序遍历

> https://leetcode.cn/problems/binary-tree-level-order-traversal/

* 注意: 层序遍历的话, 一定要首先特判根节点是否为`NULL`.

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ans;
        if (!root) return ans;

        queue<TreeNode *> q;
        q.push(root);
        while (q.size()) {
          // 这里的q.size()就是一层的长度
            int len = q.size();
            vector<int> level;
            while (len --) {
                auto t = q.front(); q.pop();
                level.push_back(t->val);
                if (t->left) q.push(t->left);
                if (t->right) q.push(t->right);
            }
            ans.push_back(level);
        }
        return ans;
    }
};

```





## 104. *二叉树的最大深度(easy)

> https://leetcode.cn/problems/maximum-depth-of-binary-tree/

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) return 0;
        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
};
```



## 105. *从前序与中序遍历序列构造二叉树 (medium)

> https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

```cpp
class Solution {
public:
    unordered_map<int, int> hash;
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        int n = inorder.size();
        for (int i = 0; i < n; i ++) hash[inorder[i]] = i;
        return dfs(preorder, inorder, 0, n - 1, 0, n - 1);
    }
    TreeNode *dfs(vector<int> &preorder, vector<int> &inorder, int pl, int pr, int il, int ir) {
       if (pl > pr) return NULL; 
       int r = preorder[pl];
       int k = hash[r];
       auto root = new TreeNode(r);
      // 注意这里是(k - 1) - il + 1
       root->left = dfs(preorder, inorder, pl + 1, pl + 1 + k - 1 - il + 1 - 1, il, k - 1);
       root->right = dfs(preorder, inorder, pl + 1 + k - 1 - il + 1, pr, k + 1, ir);
       return root;
    }
};

```





## 108. *将有序数组转换为二叉搜索树(medium)

> https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/

* 从中点开始创造节点, 然后左子树和右子树从中点前后范围递归构建.

```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return dfs(nums, 0, nums.size() - 1);
    }
    TreeNode *dfs(vector<int> &nums, int l, int r) {
        if (l > r) return NULL;
        int mid = l + (r - l) / 2;
        auto root = new TreeNode(nums[mid]);
        root->left = dfs(nums, l, mid - 1);
        root->right = dfs(nums, mid + 1, r);
        return root;
    }
};
```



## 114. *二叉树展开为链表(medium)

> https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/

* 从直观上来看, 需要把一个节点的左子树, 归并到这个节点和右节点之间.
* 左子树有两个关键的节点:
  * 左子树根节点, 当前节点的右指针要指向左子树根节点.
  * 当前节点中序遍历的前驱, 前驱的右节点需要指向右子树根节点.

```cpp
class Solution {
public:
    void flatten(TreeNode* root) {
        auto cur = root;
        while (cur) {
            if (cur->left) {
                auto prev = cur->left;
                while (prev->right) prev = prev->right;
                prev->right = cur->right;
                cur->right = cur->left;
                cur->left = NULL;
            }
            cur = cur->right;
        }
    }
};
```



## 118. *杨辉三角(easy)

> https://leetcode.cn/problems/pascals-triangle/

* 注意: 杨辉三角每一行的第一个数和最后一个数都是1.

```cpp
class Solution {
public:
    vector<vector<int>> generate(int numRows) {
        vector<vector<int>> f;
        for (int i = 0; i < numRows; i ++) {
            vector<int> level(i + 1);
            level[0] = level[i] = 1;
            for (int j = 1; j < i; j ++)
                level[j] = f[i - 1][j] + f[i - 1][j - 1];
            f.push_back(level);
        }
        return f;
    }
};
```





## 121. *买卖股票的最佳时机(easy)

> https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/

* 直接倒序遍历的过程中, 统计股票价格最大值即可.

  ```cpp
  class Solution {
  public:
      int maxProfit(vector<int>& prices) {
          int v = -1, n = prices.size();
          int ans = 0;
          for (int i = n - 1; i >= 0; i --) {
              if (v == -1) v = prices[i];
              else {
                  ans = max(ans, v - prices[i]);
                  v = max(v, prices[i]);
              }
          }
          return ans;
      }
  };
  ```
  




## 124. *二叉树中的最大路径和(hard)

> https://leetcode.cn/problems/binary-tree-maximum-path-sum/

* `dfs`函数是从当前节点出发, 伸到子树中所有节点的单向路径的最大权值之和.
* 那么对于一条路径, 他有三种情况:
  * 从左节点向下延伸.
  * 从右节点向下延伸.
  * 从左右节点同时向下延伸.
* 只需要对这三种情况取一个max即可.

```cpp
class Solution {
public:
    int ans = INT_MIN;
    int maxPathSum(TreeNode* root) {
        dfs(root);
        return ans;
    }
    int dfs(TreeNode *root) {
        if (!root) return 0;
        int l = dfs(root->left), r = dfs(root->right);
        int maxv = root->val;
        maxv = max(maxv, root->val + l);
        maxv = max(maxv, root->val + r);
        int c = maxv;
        maxv = max(maxv, l + r + root->val);
        ans = max(ans, maxv);
        return c;
    }
};
```





## 128. *最长连续序列(medium)

> https://leetcode.cn/problems/longest-consecutive-sequence/

* 首先, 将数组中所有数字插入哈希表.
* 然后, 枚举数组中的每一个数字`x`, 将这个数字作为起点 (起点的意思就是`!hash.count(x - 1)`), 然后枚举`x + 1, x + 2, ...`, 如果一直在哈希表中, 那么序列就可以延伸, 直到有一个元素不在哈希表.

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> hash;
        for (auto x : nums) hash.insert(x);
        int ans = 0;
        for (auto x : nums) {
            if (hash.count(x - 1)) continue;
            int y = x + 1;
            while (hash.count(y)) y ++;
            ans = max(ans, y - x);
        }
        return ans;
    }
};
```



## 131. *分割回文串(medium)

> https://leetcode.cn/problems/palindrome-partitioning/

* 首先, 如果一个字符串是`aaaaa..`, 那么枚举分割方案的时间复杂度是$O(2^n)$, 所以这个问题是一个爆搜问题.
* 其次, 枚举**分割方案**的方法是, 枚举一个起点`u`, 然后从`u`向后 (包括`u`), 枚举终点`i`, 枚举终点后, 递归到下一个起点`i + 1`.
* 可以用一个`f[i][j]`预处理`s[i, j]`是否是回文串, 递推式是:
  * `f[i][j] = f[i + 1][j - 1] && s[i] == s[j]`.
  * 注意由于要满足拓扑序, `i`要从后向前, `j`要从前向后.

```cpp
class Solution {
public:
    vector<string> path;
    vector<vector<bool>> f;
    vector<vector<string>> ans;
    vector<vector<string>> partition(string s) {
        int n = s.size();
        f = vector<vector<bool>>(n, vector<bool>(n, false));
        for (int j = 0; j < n; j ++)
            for (int i = n - 1; i >= 0; i --)
                if (i == j) f[i][j] = true;
                else if (i + 1 > j - 1) f[i][j] = (s[i] == s[j]);
                else f[i][j] = f[i + 1][j - 1] && (s[i] == s[j]);
        dfs(s, 0);
        return ans;    
    }
    void dfs(string &s, int u) {
        if (u == s.size()) {
            ans.push_back(path);
            return ;
        }
        for (int i = u; i < s.size(); i ++) {
            if (!f[u][i]) continue;
            path.push_back(s.substr(u, i - u + 1));
            dfs(s, i + 1);
            path.pop_back();
        }
    }
};
```



## 136. *只出现一次的数字(easy)

> https://leetcode.cn/problems/single-number/

* 直接对数组中所有元素进行异或就可以.

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
       int ans = 0;
       for (auto x: nums) ans = ans ^ x;
       return ans; 
    }
};
```



## 139. *单词拆分(medium)

> https://leetcode.cn/problems/word-break/

* 设`f[i]`表示以`s[i]`结尾, 是否存在划分方式.
* 那么假设`k < i`, 并且`s[k:i]`是在字典中出现的, 那么`f[i] = f[k - 1]`.
  * 如果要判断`s[k:i]`是否在字典中出现过, 可以用字符串哈希, 做到$O(1)$的时间复杂度.

```cpp
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        const int P = 131;
        typedef unsigned long long ULL;
        unordered_set<ULL> hash;

        for (auto &word: wordDict) {
            ULL h = 0;
            for (auto c : word) h = h * P + c;
            hash.insert(h);
        }
        int n = s.size();
        vector<bool> f(n + 1);
        s = ' ' + s;
        f[0] = true;
        for (int i = 0; i < n; i ++) {
            if (!f[i]) continue;
            ULL h = 0;
          // 枚举f[i]能更新到哪个位置
            for (int j = i + 1; j <= n; j ++) {
                h = h * P + s[j];
                if (hash.count(h)) f[j] = true;
            }
        }
        return f[n];
    }
};
```





## 138. *随机链表的复制

> https://leetcode.cn/problems/copy-list-with-random-pointer/

* 首先, 对于旧链表中的每一个节点, 在后面插入一个新的节点, 那么我就可以通过旧链表的位置相对关系, 推导出新链表的位置相对关系.
* 之后, 如果要复制`random`边, 只需要让旧链表中的节点`p`, 让`p->next->random = p->random->next`.
  * 其中`p->next`是新链表中的对应节点, `p->random->next`就是`p->random`在新链表中的相对位置.
* 之后, 再把新链表节点从原链表拆出来就可以.

```cpp
class Solution {
public:
    Node* copyRandomList(Node* head) {
        for (auto p = head; p; p = p->next->next) {
            auto q = new Node(p->val);
            q->next = p->next;
            p->next = q;
        }
        for (auto p = head; p; p = p->next->next) {
            if (p->random) p->next->random = p->random->next;
            else p->next->random = NULL;
        }
        auto dummy = new Node(-1), cur = dummy;
        for (auto p = head; p; p = p->next) {
            cur = cur->next = p->next;
            p->next = p->next->next;
        }
        return dummy->next;
    }
};
```





## 141. *环形链表(easy)

> https://leetcode.cn/problems/linked-list-cycle/

* 用快慢指针, 快指针一次两个, 慢指针一次一个, 最终相遇, 就证明有环.
* 注意, 如果链表只有一个节点, 那么当两个指针相等的时候可能都是`NULL`, 这个要特判一下.

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        auto p = head, q = head;
        while (p && q) {
            p = p->next;
            q = q->next;
            if (q) q = q->next;
            if (q && p == q) return true;
        }
        return false;
    }
};
```



## 142. *环形链表II (medium)

> https://leetcode.cn/problems/linked-list-cycle-ii/

* 继上一题, 如果要找环形链表的环入口:
  * 当快慢指针第一次相遇时, 让慢指针退后到起始点.
  * 然后快指针和慢指针同时向后移动一次, 最终相遇点就是环的入口.

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        auto p = head, q = head;
        while (p && q) {
            p = p->next;
            q = q->next;
            if (q) q = q->next;
            if (q && p == q) break;
        }
        if (!q) return NULL;
        p = head;
        while (p != q) p = p->next, q = q->next;
        return p;
    }
};
```



## 144. 二叉树的前序遍历(medium)

> https://leetcode.cn/problems/binary-tree-preorder-traversal/

* 递归写法:

```cpp
class Solution {
public:
    vector<int> ans;
    vector<int> preorderTraversal(TreeNode* root) {
        dfs(root);
        return ans;
    }
    void dfs(TreeNode *root) {
        if (!root) return ;
        ans.push_back(root->val);
        dfs(root->left);
        dfs(root->right);
    }
};

```

* 迭代写法:

```cpp
class Solution {
public:
    vector<int> ans;
    stack<TreeNode *> stk;
    vector<int> preorderTraversal(TreeNode* root) {
        while (root || stk.size()) {
            while (root) {
                ans.push_back(root->val);
                stk.push(root);
                root = root->left;
            }
            root = stk.top(); stk.pop();
            root = root->right;
        }
        return ans;
    }
};

```



## 146. *LRU缓存(medium)

> https://leetcode.cn/problems/lru-cache/

* LRU缓存需要用一个双向链表和一个哈希表实现.
  * 双向链表: 存储实际的`key`和`value`.
  * 哈希表: 用于快速通过`key`获取双向链表节点的位置.
* 构造函数逻辑:
  * 主要初始化双向链表的哨兵节点.
* `get`函数逻辑:
  * 首先通过`key`, 在哈希表中找到节点 (如果节点不存在就直接返回`NULL`).
  * 然后把这个节点从双向链表中删除, 放到头节点.
* `put`函数逻辑:
  * 如果哈希表原来存在, 那么就修改, 从双链表中删除, 放到头节点.
  * 如果没有:
    * 如果容量满了, 那么就要删除双向链表的尾部节点, 并且从哈希表中清除记录.
    * 从哈希表中创建节点, 然后放到双链表最左侧.

```cpp
class LRUCache {
public:
    int n;
    struct Node {
        int key, value;
        Node *left, *right;
        Node(int _key, int _value): key(_key), value(_value), left(NULL), right(NULL) {}
    } *L, *R;
    unordered_map<int, Node*> hash;

    void remove(Node *u) {
        u->right->left = u->left;
        u->left->right = u->right;
    }

    void insert(Node *u) {
        u->left = L;
        u->right = L->right;
        L->right->left = u;
        L->right = u;
    }

    LRUCache(int capacity) {
        n = capacity;
        L = new Node(-1, -1);
        R = new Node(-1, -1);
        L->right = R, R->left = L;
    }
    
    int get(int key) {
        if (!hash.count(key)) return -1;
        auto t = hash[key];
        remove(t); insert(t);
        return t->value;
    }
    
    void put(int key, int value) {
        if (hash.count(key)) {
            auto t = hash[key];
            t->value = value;
            remove(t); insert(t);
        }
        else {
            if (hash.size() == n) {
                auto p = R->left;
                hash.erase(p->key);
                remove(p);
                delete p;
            }
            auto t = new Node(key, value);
            hash[key] = t;
            insert(t);
        }
    }
};
```



## 148. *排序链表(medium)

> https://leetcode.cn/problems/sort-list/

* 如果要求空间复杂度是常数, 那么只能用非递归的归并排序.

```cpp
class Solution {
public:
    ListNode* sortList(ListNode* head) {
       // 统计链表节点数目
       int n = 0;
       for (auto p = head; p ; p = p->next) n ++;

       // 第一层枚举两个归并区间中每个区间的长度
       // 一共做n-1层, 最终合并到n个元素
       for (int i = 1; i < n; i *= 2) {
            auto dummy = new ListNode(-1), cur = dummy;
            // 第二层以枚举由所有的两个归并区间
         		// 注意这里一定是j <= n, 因为j = n证明区间里面只有一个元素, 也要单独做一层.
            for (int j = 1; j <= n; j += i * 2) {
                // head存储归并区间开头
                auto p = head, q = p;
                // 把q放到下一个归并区间开头, 归并区间长度很可能不足i
                for (int k = 0; k < i && q; k ++) q = q->next;
                // o存储下一个归并区间的开头, 方便更新head
                auto o = q;
                for (int k = 0; k < i && o; k ++) o = o->next;
                int l = 0, r = 0;
                while (l < i && r < i && p && q)
                    if (p->val <= q->val) cur = cur->next = p, p = p->next, l ++;
                    else cur = cur->next = q, q = q->next, r ++;
                while (l < i && p) cur = cur->next = p, p = p->next, l ++;
                while (r < i && q) cur = cur->next = q, q = q->next, r ++;
                head = o;
            }
            // 做完一层, 最后的节点next是NULL
            cur->next = NULL;
            head = dummy->next;
       }
       return head;
    }
};

```





## 152. *乘积最大子数组(medium)

> https://leetcode.cn/problems/maximum-product-subarray/

* 设`f[i]`, `g[i]`分别表示以`i`结尾的, 连续子数组乘积的最大值和最小值.
  * 存储最小值的原因是因为乘法具有负负得正的特性.
* 那么`f[i] = max(nums[i], f[i - 1] * nums[i], g[i - 1] * nums[i])`.
* 并且`g[i] = min(nums[i], f[i - 1] * nums[i], g[i - 1] * nums[i])`

```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int ans = nums[0], f = nums[0], g = nums[0];
        for (int i = 1; i < nums.size(); i ++) {
            int fp = f, gp = g;
            f = max(nums[i], max(nums[i] * fp, nums[i] * gp));
            g = min(nums[i], min(nums[i] * fp, nums[i] * gp));
            ans = max(ans, f);
        }
        return ans;
    }
};
```





## 153. *寻找旋转排序数组中的最小值(medium)

> https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/

* 假设数组的第一个元素是`nums[0]`.
* 数组的前半部分满足`nums[i] >= nums[0]`, 后半部分满足`nums[i] < nums[0]`, 以此来二分.
* 如果数组完全单调递增, 那么最终二分出来的`nums[i] >= nums[0]`, 此时直接返回`nums[0]`.

```cpp
class Solution {
public:
    int findMin(vector<int>& nums) {
        int n = nums.size();
        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] >= nums[0]) l = mid + 1;
            else r = mid;
        }
        if (nums[l] >= nums[0]) return nums[0];
        else return nums[l];
    }
};
```





## 155. 最小栈(medium)

> https://leetcode.cn/problems/min-stack/

* 直接单独开一个栈, 专门用来存储最小值.
* push时, 如果这个最小栈是空的, 或者要插入的值value小于最小栈栈顶, 那么就要向最小栈中插入.
* pop时, 如果最小栈的栈顶等于当前栈要pop出去的元素, 那么最小栈就pop.
* 对于`stack`来说, 要调用`top()`一定要保证栈不是空的!!

```cpp
class MinStack {
public:
    stack<int> stk;
    stack<int> stk_min;
    MinStack() {
        
    }
    
    void push(int val) {
        stk.push(val);
        if (stk_min.empty() || val <= stk_min.top()) stk_min.push(val);
    }
    
    void pop() {
       if (stk_min.top() == stk.top()) stk_min.pop();
       stk.pop();
    }
    
    int top() {
       return stk.top(); 
    }
    
    int getMin() {
       return stk_min.top(); 
    }
};
```





## 160. *相交链表(easy)

> https://leetcode.cn/problems/intersection-of-two-linked-lists/

* 让两个指针同时向前走一步, 如果有一个指针走到尽头, 那么就把它放到第二个指针的头部继续走.
* 当两个指针相同时, 如果不为NULL, 那么就是相遇点, 否则就不是.

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        auto p = headA, q = headB;
        while (p != q) {
            if (p) p = p->next;
            else p = headB;
            if (q) q = q->next;
            else q = headA;
        }
        if (!p) return NULL;
        return p;
    }
};
```



## 169. *多数元素(easy)

> https://leetcode.cn/problems/majority-element/

* 摩尔投票算法: 使用$O(n)$时间复杂度, $O(1)$的空间复杂度, 找到一个数组中的众数.
  * 从前到后遍历数组中的元素`x`, 并且维护一个当前候选人`r`和投票数`c`.
  * 如果某个元素`x`等于`r`, 那么投票数`c ++`.
  * 如果不等于`r`, 那么投票数`c --`.
  * 如果投票数减到0, 那么`r`就要换人.
  * 最终的`r`就是赢家.


```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int r = 0, c = 0;
        for (auto x : nums) {
            if (!c) r = x;
            if (x == r) c ++;
            else c --;
        }
        return r;
    }
};
```





## 189 *轮转数组(medium)

> https://leetcode.cn/problems/rotate-array/

* 这种题一般是把一个数组/序列的一部分移动到最前面的位置, 通用做法是:
  * 先反转整个数组, 再分别反转数组的前面/后面的位置.

* 注意: `k`可能是一个很大的异常值, 需要模上数组长度.

```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        k = k % nums.size();
        reverse(nums.begin(), nums.end());
        reverse(nums.begin(), nums.begin() + k);
        reverse(nums.begin() + k, nums.end());
    }
};
```



## 198. *打家劫舍 (medium)

> https://leetcode.cn/problems/house-robber/

* 状态机DP: `f[i][0]`表示第`i`个房子不打劫, `f[i][1]`表示打劫, 从后向前递推即可.

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> f(n, vector<int>(2, 0));

        f[0][0] = 0;
        f[0][1] = nums[0];

        for (int i = 1; i < n; i ++) {
            f[i][0] = max(f[i - 1][0], f[i - 1][1]);
            f[i][1] = f[i - 1][0] + nums[i];
        }
        return max(f[n - 1][0], f[n - 1][1]);
    }
};
```





## 199. *二叉树的右视图(medium)

> https://leetcode.cn/problems/binary-tree-right-side-view/

* 右视图序列就是层序遍历中每一层最后一个节点组成的序列.

```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> ans;
        if (!root) return ans;

        queue<TreeNode*> q;
        q.push(root);

        while (q.size()) {
            int len = q.size();
            for (int i = 0; i < len; i ++) {
                auto t = q.front();
                q.pop();
                if (i == len - 1) ans.push_back(t->val);
                if (t->left) q.push(t->left);
                if (t->right) q.push(t->right);
            }
        }
        return ans;
    }
};

```



## 200. *岛屿数量 (medium)

> https://leetcode.cn/problems/number-of-islands/

* Flood Fill算法:
  * 每次遇到一块陆地, 就从这个陆地为起点进行搜索, 从这个起点向四周扩散, 如果再次遇到陆地, 就递归搜索.

```cpp
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        int ans = 0;
        int n = grid.size(), m = grid[0].size();
        for (int i = 0; i < n; i ++)
            for (int j = 0; j < m; j ++)
                if (grid[i][j] == '1')
                    dfs(grid, i, j), ans ++;
        return ans;
    }
    void dfs(vector<vector<char>> &grid, int x, int y) {
        grid[x][y] = '0';
        int n = grid.size(), m = grid[0].size();
        int dx[] = {-1, 0, 1, 0};
        int dy[] = {0, 1, 0, -1};
        for (int i = 0; i < 4; i ++) {
            int a = x + dx[i], b = y + dy[i];
            if (a < 0 || a >= n || b < 0 || b >= m || grid[a][b] == '0')
                continue;
            dfs(grid, a, b);
        }
    }
};
```





## 206. *反转链表(easy)

> https://leetcode.cn/problems/reverse-linked-list/

* 递归方法: 注意先判断链表是否是空/只有一个节点.

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head || !head->next) return head;
        auto tail = reverseList(head->next);
        head->next->next = head;
        head->next = NULL;
        return tail;
    }
};
```

* 迭代方法:

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode *p = NULL;
        ListNode *q = head;
        while (q != NULL) {
            auto r = q->next;
            q->next = p;
            p = q, q = r;
        }
        return p;
    }
};
```



## 207. *课程表(medium)

> https://leetcode.cn/problems/course-schedule/

* 从先修课程到后置课程连接一条有向边, 组成一个有向图.
* 然后用拓扑排序, 所有入度为0的节点先入队, 然后BFS, 每次扩展一层就把入度-1, 然后再次把入度为0的点入队.
* 如果一个图有环, 那么当遍历到环的入口节点之后, 入口节点入度-1后还是1, 永远无法变成0, 那么这种节点就永远无法进入队列, 因此只要进入/弹出队列的节点数目小于`n`, 就证明有环.

```cpp
class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        int n = numCourses;
        vector<int> d(n);
        vector<vector<int>> g(n);
        for (auto &e: prerequisites) {
            int a = e[1], b = e[0];
            g[a].push_back(b); d[b] ++;
        }
        queue<int> q;
        for (int i = 0; i < n; i ++)
            if (!d[i]) q.push(i);
        
        int cnt = 0;
        while (q.size()) {
            auto t = q.front(); q.pop();
            for (auto u: g[t])
                if (-- d[u] == 0)
                    q.push(u);
            cnt ++;
        }
        return cnt == n;
    }
};
```



## 208. *实现Trie(前缀树)

> https://leetcode.cn/problems/implement-trie-prefix-tree/

* 前缀树模板题.
* 前缀树存储的元素需要有固定的字符集进行编码, 假设字符集中的元素个数为n, 那么前缀树就是一个n叉树.
* n叉树的节点有两个成员变量:
  * 一个用来存储所有的儿子.
  * 另一个用来存储这个节点是否是某个元素的结尾.
* 查询和插入都是$O(n)$的时间复杂度, 其中$n$是字符集元素的个数.

```cpp
class Trie {
public:
    struct Node {
        Node *son[26];
        bool is_end;

        Node() {
            for (int i = 0; i < 26; i ++) son[i] = NULL;
            is_end = false;
        }
    } *root;
    Trie() {
        root = new Node();
    }
    
    void insert(string word) {
        auto p = root;
        for (auto c: word) {
            int u = c - 'a';
            if (!p->son[u]) p->son[u] = new Node();
            p = p->son[u];
        }
        p->is_end = true;
    }
    
    bool search(string word) {
        auto p = root;
        for (auto c: word) {
            int u = c - 'a';
            if (!p->son[u]) return false;
            p = p->son[u];
        }
        return p->is_end;
    }
    
    bool startsWith(string prefix) {
        auto p = root;
        for (auto c: prefix) {
            int u = c - 'a';
            if (!p->son[u]) return false;
            p = p->son[u];
        }
        return true;
    }
};
```



## 215. *数组中的第K个最大元素(medium)

> https://leetcode.cn/problems/kth-largest-element-in-an-array/

* 首先注意题目要求是第k个最大, 还是第k个最小.
* 本题是快速选择算法.

```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
       return quick_sort(nums, 0, nums.size() - 1, k); 
    }
    int quick_sort(vector<int> &nums, int l, int r, int k) {
        if (l >= r) return nums[l];
        int i = l - 1, j = r + 1, x = nums[l + (r - l) / 2];
        while (i < j) {
            while (nums[ ++ i] > x);
            while (nums[ --j] < x);
            if (i < j) swap(nums[i], nums[j]);
        }
        int sl = j - l + 1;
        if (k <= sl) return quick_sort(nums, l, j, k);
        else return quick_sort(nums, j + 1, r, k - sl);
    }
};

```





## 226. *翻转二叉树(easy)

> https://leetcode.cn/problems/invert-binary-tree/

* 首先递归翻转左子树和右子树, 然后将左子树和右子树进行交换即可.

```cpp
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (!root) return NULL;
        auto left = invertTree(root->left);
        auto right = invertTree(root->right);
        root->left = right;
        root->right = left;
        return root;
    }
};
```



## 230. *二叉搜索树中第K小的元素(medium)

> https://leetcode.cn/problems/kth-smallest-element-in-a-bst/

* 中序遍历的过程中, 每次遍历一个节点, 就将计数器+1, 计数到`k`的时候, 就找到了第k小的元素.

```cpp
class Solution {
public:
    int k, ans;
    int kthSmallest(TreeNode* root, int _k) {
        k = _k;
        dfs(root);
        return ans;
    }
    bool dfs(TreeNode *root) {
        if (!root) return false;
        if (dfs(root->left)) return true;
        if (-- k == 0) {
            ans = root->val;
            return true;
        }
        return dfs(root->right);
    }
};

```





## 234. *回文链表(easy)

> https://leetcode.cn/problems/palindrome-linked-list/

* 首先, 找到链表的中间节点.
* 之后, 将中间节点后面的链表反转.
* 然后, 将反转部分的链表和前半部分的链表进行对比即可.

```cpp
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        int cnt = 0;
        auto cur = head;
        while (cur != NULL) {
            cnt ++;
            cur = cur->next;
        }
        if (cnt == 1) return true;
        cnt = cnt % 2 ? cnt / 2 : cnt / 2 - 1;

        cur = head;
        while (cnt --) cur = cur->next;

        ListNode *p = NULL;
        ListNode *q = cur->next;
        while (q != NULL) {
            auto r = q->next;
            q->next = p;
            p = q, q = r;
        }
        auto l1 = head, l2 = p;
        while (l1 != NULL && l2 != NULL) {
            if (l1->val != l2->val) return false;
            l1 = l1->next, l2 = l2->next;
        }
        return true;
    }
};
```



## 236. *二叉树的最近公共祖先(medium)

> https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/

* 两个节点的最近公共祖先就是同时包含两个节点的, 并且深度最深的节点.

```cpp
class Solution {
public:
    TreeNode *ans = NULL;
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        dfs(root, p, q);
        return ans;
    }
    // root子树是否含有p, q, 都不含00, 有p: 01, 有q: 10, 都有11
    int dfs(TreeNode *root, TreeNode *p, TreeNode *q) {
        if (!root) return 0;
        int state = dfs(root->left, p, q);
        if (root == p) state |= 1;
        else if (root == q) state |= 2;
        state |= dfs(root->right, p, q);
        if (state == 3 && !ans) ans = root;
        return state;
    }
};

```





## 238. *除自身以外数组的乘积

> https://leetcode.cn/problems/product-of-array-except-self/

* 思路: 原地算法
  * 首先维护一个类似前缀和的数组, `s[i]`表示从`nums[0]`乘到`nums[i - 1]`.
  * 然后用一个`suffix`维护后缀积, 遍历乘一遍, 直接赋值到前缀和数组就可以.

```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n = nums.size();
        vector<int> s(n, 1);
        for (int i = 1; i < n; i ++) s[i] = s[i - 1] * nums[i - 1];
        
        int suffix = 1;
        for (int i = n - 1; i >= 0; i --) {
            s[i] *= suffix;
            suffix *= nums[i];
        }
        return s;
    }
};

```





## 239. *滑动窗口最大值(hard)

> https://leetcode.cn/problems/sliding-window-maximum/

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        const int N = 100010;
        int q[N], hh = 0, tt = -1;

        vector<int> ans;
        for (int i = 0, j = 0; i < nums.size(); i ++) {
            if (hh <= tt && i - k + 1 > q[hh]) hh ++;
            while (hh <= tt && nums[i] > nums[q[tt]]) tt --;
            q[ ++ tt] = i;
            if (i - k + 1 >= 0) ans.push_back(nums[q[hh]]);
        }
        return ans;
    }
};
```



## 240. *搜索二维矩阵II (medium)

> https://leetcode.cn/problems/search-a-2d-matrix-ii/

* 拿右上角/左下角的元素作为基准进行搜索即可.

```cpp
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int n = matrix.size(), m = matrix[0].size();
        int x = 0, y = m - 1;
        while (x >= 0 && x < n && y >= 0 && y < m) {
            if (matrix[x][y] == target) return true;
            else if (matrix[x][y] > target) y --;
            else x ++;
       }
        return false;
    }
};
```



## 279. *完全平方数(medium)

> https://leetcode.cn/problems/perfect-squares/

* 设`f[i]`表示能够拼凑出`i`的方案中, 完全平方数个数的最小值.

```cpp
class Solution {
public:
    int numSquares(int x) {
        int n = sqrt(x);
        vector<int> f(x + 1, 0x3f3f3f3f);
        f[0] = 0;
        for (int i = 1; i <= x; i ++) {
            for (int j = 1; j * j <= i; j ++) {
                f[i] = min(f[i], f[i - j * j] + 1);
            }
        }
        return f[x];
    }
};
```



## 283. *移动零(medium)

> https://leetcode.cn/problems/move-zeroes/

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



## 287. *寻找重复数(medium)

> https://leetcode.cn/problems/find-the-duplicate-number/

* 这个题等价于环型链表找环入口问题.
* 对于一个`i`, 从`i`向`nums[i]`连一条边.
* 如果有重复数, 一定存在环:
  * 首先, 环入口入度为2, 因为`nums[i]`会出现两次, 但是下标不同.
  * 其次, 环入口出度为1, 因为`nums[i]`在下标范围.

```cpp
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int p = nums[0], q = nums[nums[0]];
        while (p != q) {
            p = nums[p];
            q = nums[nums[q]];
        }
        p = 0;
        while (p != q) {
            p = nums[p];
            q = nums[q];
        }
        return p;
    }
};
```





## 295. *数据流的中位数(hard)

> https://leetcode.cn/problems/find-median-from-data-stream/

* 用`up`: 小根堆和`down`: 大根堆来维护.
* 如果一个数小于等于`down.top()`那么就插到`down`中, 否则就插到`up`中.
* 然后需要动态维护`down`和`up`的大小, 保证`down`和`up`大小相等, 或者`down.size() - up.size() = 1`.
* 这样就能维护中位数的边界.

```cpp
class MedianFinder {
public:
    priority_queue<int, vector<int>, greater<int>> up;
    priority_queue<int> down;

    MedianFinder() {
        
    }
    
    void addNum(int num) {
        if (down.empty() || num <= down.top()) {
            down.push(num);
            if (down.size() > up.size() + 1) {
                up.push(down.top());
                down.pop();
            }
        }
        else {
            up.push(num);
            if (up.size() > down.size()) {
                down.push(up.top());
                up.pop();
            }
        }
    }
    
    double findMedian() {
        int n = up.size() + down.size();
        if (n % 2) return down.top();
        else return (up.top() + down.top()) / 2.0;
    }
};
```



## 300. *最长递增子序列(medium)

> https://leetcode.cn/problems/longest-increasing-subsequence/

* 最长上升子序列模型.
* 注意一点, `q`数组的长度要初始化成`n + 1`, 因为最长上升子序列的长度最小值就是1, 没有0, 避免出现下标问题 (因为长度值要作为下标).

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();
        vector<int> q(n + 1);
        int len = 0;
        for (int i = 0; i < n; i ++) {
            int l = 0, r = len;
            while (l < r) {
                int mid = l + (r - l) / 2 + 1;
                if (q[mid] < nums[i]) l = mid;
                else r = mid - 1;
            }
            len = max(len, r + 1);
            q[r + 1] = nums[i];
        }
        return len;
    }
};
```



## 322. *零钱兑换(medium)

> https://leetcode.cn/problems/coin-change/

* 设`f[i][j]`表示从前`i`个硬币中选, 正好能凑成`amount`的方案中, 最少的硬币数量.
* 然后用完全背包问题即可解决.

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int n = coins.size(), m = amount;
        vector<int> f(m + 1, 0x3f3f3f3f);
        f[0] = 0;

        for (int i = 0; i < n; i ++) {
            for (int j = coins[i]; j <= m; j ++) {
                f[j] = min(f[j], f[j - coins[i]] + 1);
            }
        }
        int ans = f[amount] == 0x3f3f3f3f ? -1 : f[amount];
        return ans;
    }
};
```





## 347. *前k个高频元素(medium)

> https://leetcode.cn/problems/top-k-frequent-elements/

* 首先统计一下数组中各个元素出现的次数.
* 然后, 用计数排序的思想, 开一个`n + 1`长度的数组, 数组下标表示出现次数, 这个数组存储出现次数为`i`的元素有`nums[i]`种.
* 然后反向遍历这个计数排序的数组即可得到答案.

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> hash;
        for (auto x: nums) hash[x] ++;

        int n = nums.size();
        vector<int> cnt(n + 1, 0);
        for (auto [x, c]: hash) cnt[c] ++;

        int i = n, t = 0;
        while (t < k) t += cnt[i --];

        vector<int> ans;
        for (auto [x, c]: hash)
            if (c > i)
                ans.push_back(x);
        return ans;

    }
};
```





## 394. *字符串解码(medium)

> https://leetcode.cn/problems/decode-string/

* 这类题是一种前缀表达式的计算问题, 通用思路是用`dfs`.
  * `dfs`中, 用一个引用变量`u`记录处理到表达式的哪个位置.
  * 如果遇到算符, 就需要递归地把运算数计算出来.
  * 如果遇到运算数的一部分, 就需要把所有运算数进行拼接.

```cpp
class Solution {
public:
    string s;
    string decodeString(string _s) {
        s = _s;
        int u = 0;
        return dfs(u);
    }
    string dfs(int &u) {
        string ans;
        while (u < s.size() && s[u] != ']') {
            if (isdigit(s[u])) {
               int k = u;
               while (k < s.size() && isdigit(s[k])) k ++;
               int cnt = stoi(s.substr(u, k - u));
               u = k + 1;
               string res = dfs(u);
               u ++; // 过滤]
               for (int i = 0; i < cnt; i ++) ans += res; 
            }
            else if (s[u] >= 'a' && s[u] <= 'z' || s[u] >= 'A' && s[u] <= 'Z') ans += s[u ++];
        }
        return ans;
    }
};
```



## 416. *分割等和子集

> https://leetcode.cn/problems/partition-equal-subset-sum/

* 这个题本质上是一个背包问题:
  * 每一个物品的体积就是`a[i]`.
  * 总体积是数组和`sum / 2`.
  * 最终的结果是是否可以满足选的物品体积之和等于`sum / 2`.

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = 0, n = nums.size();
        for (auto x : nums) sum += x;
        if (sum % 2) return false;
        sum /= 2;

        vector<int> f(sum + 1);
        f[0] = 1;
        for (auto x : nums)
            for (int j = sum; j >= x; j --) {
                f[j] |= f[j - x];
            }
        return f[sum];
    }
};
```





## 437. *路径总和III (medium)

> https://leetcode.cn/problems/path-sum-iii/

* 首先, 这道题类似于前缀和:
  * 要求在前缀和数组`s`中找到`l, r`, 使得`s[r] - s[l - 1] = k`.
  * 也就是当遍历到`s[i]`时, 要在之前的`0<= j < i`中, 找到一个`s[j]`, 使得`s[j] = s[i] - k`.
  * 那么直接在之前用哈希表存储每一个`s[j]`就可以了, 当遍历到`s[i]`, 就查有没有符合题意的`s[j]`.
* 注意: 前缀和容易超出`int`范围, 需要用`long long`存储.

```cpp
class Solution {
public:
    typedef long long LL;
    int res = 0;
    unordered_map<LL, int> hash;
    int pathSum(TreeNode* root, int targetSum) {
        // 0是前缀和中的0位置的元素 
        hash[0]++;
        dfs(root, targetSum, (LL)0);
        return res;
    }
    void dfs(TreeNode *root, int sum, LL cur) {
        if (!root) return ;
        cur += root->val;
        res += hash[cur - sum];
        hash[cur] ++;
        dfs(root->left, sum, cur), dfs(root->right, sum, cur);
        hash[cur] --;
    }
};

```





## 438 *找到字符串中所有字母异位词(medium)

>  https://leetcode.cn/problems/find-all-anagrams-in-a-string/

* 思路:
  * 判断两个字符串是否是字母异位词的充要条件是: 两个字符串的字符出现种类, 以及次数相同.
  * 直接维护一个`p.size()`的滑动窗口, 判断滑动窗口内的子串是否和`p`是字母异位词即可.

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
       unordered_map<char, int> hash;
       for (auto c : p) hash[c] ++;
       int tot = hash.size(); // p中有多少种字符
       
       // 符合条件的字符种类数目
       int k = 0;
       vector<int> ans;
       for (int i = 0, j = 0; i < s.size(); i ++) {
         // 处理滑动窗口边界
            if (i - j + 1 > p.size()) {
                if (hash[s[j]] == 0) k --;
                hash[s[j ++]] ++;
            }
            if (--hash[s[i]] == 0) k ++;
            if (k == tot) ans.push_back(j);
       }
       return ans;
    }
};
```



## 543. *二叉树的直径(easy)

> https://leetcode.cn/problems/diameter-of-binary-tree/

```cpp
class Solution {
public:
    int ans = 0;
    int diameterOfBinaryTree(TreeNode* root) {
        dfs(root);
        return ans;
    }
  // dfs的意思是, 从root出发, 包括root, 到达底部的最长路径上节点的个数
    int dfs(TreeNode *root) {
        if (!root) return 0;
        int left = dfs(root->left), right = dfs(root->right);
      // 其实应该是left + right  + 1 - 1, +1表示加上root这个点, -1表示求边数
        ans = max(ans, left + right);
        return max(left, right) + 1;
    }
};
```





## 560. *和为k的子数组 (medium)

> https://leetcode.cn/problems/subarray-sum-equals-k/

* 思路: 假设前缀和数组是`s`, 问题就等价于前缀和数组中是否存在`s[r] - s[l - 1] = k`, 和两数之和本质相同.

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> s(n + 1);
        for (int i = 1; i <= n; i ++) s[i] = s[i - 1] + nums[i - 1];

        unordered_map<int, int> hash;
        int res = 0;
        for (int i = 0; i <= n; i ++) {
            res += hash[s[i] - k];
            hash[s[i]] ++;
        }
        return res;
    }
};
```



## 739. *每日温度(medium)

> https://leetcode.cn/problems/daily-temperatures/

* 单调栈问题可以用$O(n)$的时间复杂度求出一个元素左侧/右侧比他大/小, /具有单调性的最近的元素.
* 如果要找到右边第一个比他大的元素, 那么在遍历它之前, 就需要有右侧的先验知识, 因此从右向左维护单调栈.

```cpp
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& t) {
        stack<int> stk;
        vector<int> ans;
        
        for (int i = t.size() - 1; i >= 0; i --) {
            while (stk.size() && t[i] >= t[stk.top()]) stk.pop();
            if (stk.empty())
                ans.push_back(0);
            else
                ans.push_back(stk.top() - i);
            stk.push(i);
        }
        reverse(ans.begin(), ans.end());
        return ans;
    }
};
```



## 763. *划分字母区间(medium)

> https://leetcode.cn/problems/partition-labels/

* 这道题的本质是区间问题, 对于每一种字符, 都有一个开始和结束位置, 这个位置看成一个一个区间, 最终合并区间就是分割方案.
* 实际上只需维护一个所有区间能到达的最右范围`end`, 从前向后遍历, 如果最右范围`end = i`, 那么就说明`[0, i]`这一段已经和后面一段不可能产生交集, 因此就可以作为一个合法分割.

```cpp
class Solution {
public:
    vector<int> partitionLabels(string s) {
        int last[26];
        int n = s.size();
        for (int i = 0; i < n; i ++) last[s[i] - 'a'] = i;

        int start = 0, end = 0;
        vector<int> ans;
        for (int i = 0; i < n; i ++) {
            end = max(end, last[s[i] - 'a']);
            if (end == i) {
                ans.push_back(end - start + 1);
                start = end = i + 1;
            }
        }
        return ans;
    }
};
```





## 994. *腐烂的🍊 (medium)

> https://leetcode.cn/problems/rotting-oranges/

* 多源BFS问题:
  * 直接把腐烂的橘子放入队列中宽搜即可.
  * 注意一个逻辑, 在队列中统计的是BFS的层数, 最终的答案应该是层数 - 1, 而且注意, 这个层数- 1应该是建立在队列中一开始有东西的前提下, 如果一开始就没有东西, 那么直接返回0.

```cpp
#define x first
#define y second
typedef pair<int, int> PII;

class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        queue<PII> q;
        for (int i = 0; i < n; i ++)
            for (int j = 0; j < m; j ++)
                if (grid[i][j] == 2)
                    q.push({i, j});
        
        int res = 0;
        int dx[] = {-1, 0, 1, 0};
        int dy[] = {0, 1, 0, -1};
        if (q.size()) res --;
        while (q.size()) {
            int len = q.size();
            while (len --) {
                auto t = q.front(); q.pop();
                for (int i = 0; i < 4; i ++) {
                    int a = t.x + dx[i], b = t.y + dy[i];
                    if (a < 0 || a >= n || b < 0 || b >= m || grid[a][b] != 1)
                        continue;
                    grid[a][b] = 2;
                    q.push({a, b});
                }
            }
            res ++;
        }
        for (int i = 0; i < n; i ++)
            for (int j = 0; j < m; j ++)
                if (grid[i][j] == 1)
                    return -1;
        return res;
    }
};
```



## 1143. *最长公共子序列(medium)

> https://leetcode.cn/problems/longest-common-subsequence/description/

* 设`f[i][j]`表示`a[1:i]`, `b[1:j]`中所有公共子序列中长度的最大值.
* 那么`f[i][j]`可以由两种状态转移:
  * `a[i] == b[j]`, 那么`f[i][j] = f[i - 1][j - 1] + 1`.
  * `a[i] != b[j]`, 那么`f[i][j] = max(f[i - 1][j], f[i][j - 1])`.
    * 这种情况就是子序列中选`a[i]`还是选`b[j]`, 这两部分可能有重叠, 但是对于最大值来讲无妨.

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size(), m = text2.size();
        text1 = ' ' + text1;
        text2 = ' ' + text2;
        vector<vector<int>> f(n + 1, vector<int>(m + 1));

        for (int i = 1; i <= n; i ++)
            for (int j = 1; j <= m; j++) {
                if (text1[i] == text2[j])
                    f[i][j] = f[i - 1][j - 1] + 1;
                else
                    f[i][j] = max(f[i - 1][j], f[i][j - 1]);
            }
        return f[n][m];
    }
};
```

