---
title: LeetCode-1000题题解
categories: 算法
---

> *是LeetCode Hot 100中的题

[toc]




## 1. *两数之和(easy)

* 哈希表算法

  * 思路: 要遍历一次数组, 判断`target`减去当前值是否在数组中. 遍历时, 用哈希表把之前数组的值的信息保存在哈希表就可以.
  * 时间复杂度: $O(n)$

  ```cpp
  class Solution {
  public:
      vector<int> twoSum(vector<int>& nums, int target) {
        vector<int> ans;
        unordered_map<int, int> hash;
  
        for (int i = 0; i < nums.size(); i ++) {
          if (hash.count(target - nums[i])) {
              ans.push_back(i);
              ans.push_back(hash[target - nums[i]]);
              return ans;
          }
          hash[nums[i]] = i;
        }
        return ans;
      }
  };
  ```



## 2. *两数相加 (easy)

* 两个数位相加后, `t % 10`就是数位的值, 进位的值就是`t / 10`.

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        auto dummy = new ListNode(-1);
        auto cur = dummy;
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

* 注意: 子串要求元素连续, 子序列不要求连续.
* 用一个哈希表维护从前到后, 元素出现的次数.
* 从前到后遍历, 如果发现`hash[s[i]] > 1`, 说明出现了重复字符, 然后就把指针`j`向前移动, 直到没有重复字符, 最后统计子串的长度.

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
       int res = 0;
       unordered_map<int, int> hash;
       for (int i = 0, j = 0; i < s.size(); i ++) {
            hash[s[i]] ++;
            while (j < i && hash[s[i]] > 1) hash[s[j ++]] --;
            res = max(res, i - j + 1);
       }
       return res;
    }
};
```





## 11. *盛水最多的容器(medium)

* 双指针:

  * 思路: 假设两根棍子`i, j`, 它们之间的盛水量就是`min(height[i], height[j]) * (j - i)`, 只需要枚举`i, j`即可.
  * 如果`height[i] < height[j]`, 那么只需要`i++`即可, 因为只有棍子变大, 盛水量才能更多, 因此不需要用$O(n^2)$枚举i, j.

  ```cpp
  class Solution {
  public:
      int maxArea(vector<int>& height) {
          int res = 0;
          for (int i = 0, j = height.size() - 1; i < j; ) {
              res = max(res, min(height[i], height[j]) * (j - i));
              if (height[i] < height[j]) i ++;
              else j --;
          }
          return res;
      }
  };
  ```



## 15. *三数之和(medium)

* 双指针.
* 思路: 首先把数组排序(不要忘), 然后枚举第一个数, 再从后面根据双指针枚举第二个和第三个数.
* 枚举第二/第三个数时, 数组具有单调性, 第二个指针在前, 第三个指针在后, 根据`nums[i] + nums[l] + nums[r]`和`target`的大小关系前后调整指针即可.

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
       vector<vector<int>> ans;
       sort(nums.begin(), nums.end());
       for (int i = 0; i < nums.size(); i ++) {
            if (i && nums[i] == nums[i - 1]) continue;
            int l = i + 1, r = nums.size() - 1;
            while (l < r) {
                int sum = nums[i] + nums[l] + nums[r];
                if (sum == 0) {
                    ans.push_back({nums[i], nums[l], nums[r]});
                    do { l ++; } while (l < r && nums[l] == nums[l - 1]);
                    do { r --; } while (l < r && nums[r] == nums[r + 1]);
                }
                else if (sum < 0) l ++;
                else r --;
            }
       }
       return ans;
    }
};
```



## 17. *电话号码的字母组合(medium)

* 枚举每一个数字对应的字符集选哪一个字符即可.

```cpp
class Solution {
public:
    string str[10] = {
        "", "", "abc",
        "def", "ghi", "jkl",
        "mno", "pqrs", "tuv",
        "wxyz"
    };
    vector<string> ans;
    string path = "";
    vector<string> letterCombinations(string digits) {
        if (digits.size() == 0) return ans;
        dfs(digits, 0);
        return ans;
    }
    void dfs(string digits, int u) {
        if (u == digits.size()) {
            ans.push_back(path);
            return ;
        }
        for (auto c: str[digits[u] - '0']) {
            path.push_back(c);
            dfs(digits, u + 1);
            path.pop_back();
        }
    }
};

```





## 19. *删除链表的倒数第n个节点(medium)

* 首先, 链表头可能被删除, 需要虚拟头节点.

* 假设链表一共有`N`个节点:
  * 倒数第n个节点, 就是正数第`N - n + 1`个节点, 那么从虚拟头节点跳到这里, 需要跳`N - n + 1`次.
  * 但是我要删除这个节点, 需要跳到这个节点的前一个节点, 就需要从虚拟头节点跳`N - n`次.
* 如果需要遍历一次, 那么就需要搞两个指针.
  * 加上虚拟头节点, 一共能跳`N`次.
  * 先让一个指针跳`n`次, 然后再让另一个指针开始动, 直到前一个指针跳到最后一个节点.

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        auto dummy = new ListNode(-1);
        dummy->next = head;
        ListNode *p = dummy, *q = dummy;
        while (n --) p = p->next;
        while (p->next) {
            p = p->next, q = q->next;
        }
        q->next = q->next->next;
        return dummy->next;
    }
};

```



## 20. *有效的括号(easy)

* 遇到左括号就压栈, 遇到右括号就弹出.

```cpp
class Solution {
public:
    bool isValid(string s) {
        stack<char> stk;
        for (auto c : s) {
            if (c == '(' || c == '{' || c == '[') stk.push(c);
            else {
                if (stk.size() == 0) return false;
                if (c == ')' && stk.top() == '(') stk.pop();
                else if (c == '}' && stk.top() == '{') stk.pop();
                else if (c == ']' && stk.top() == '[') stk.pop();
                else return false;
            }
        }
        return stk.size() == 0;
    }
};
```





## 21. *合并两个有序的链表 (easy)

* 和归并排序合并的过程差不多.
* 注意, 不要忘记合并剩下的链表, 因为肯定有一个链表先走完.

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        auto dummy = new ListNode(-1);
        auto cur = dummy;
        auto l1 = list1, l2 = list2;
        while (l1 && l2) {
            if (l1->val <= l2->val) {
                cur = cur->next = new ListNode(l1->val);
                l1 = l1->next;
            }
            else {
                cur = cur->next = new ListNode(l2->val);
                l2 = l2->next;
            }
        }
        if (l1) cur->next = l1;
        if (l2) cur->next = l2;
        
        return dummy->next;
    }
};

```



## 22. *括号生成(medium)

* 一个括号序列是合法的充要条件是:

  * 序列中, 左右括号数量相等.
  * 任意前缀, 左括号数量大于等于有括号数量.

* 给定了n对括号, 那么序列中左右括号的数量一定相等, 那么对于括号序列的这`2n`个位置:

  * 如果左括号的数量小于n, 那么就可以填`(`.
  * 如果右括号数量小于n, 并且右括号数量小于左括号数量, 那么就可以填`)`.

  * 递归即可.

```cpp
class Solution {
public:
    int n;
    vector<string> ans;
    vector<string> generateParenthesis(int _n) {
        n = _n;
        dfs(0, 0, "");
        return ans;
    }
    void dfs(int lc, int rc, string s) {
        if (lc == n && rc == n) {
            ans.push_back(s);
            return ;
        }
        if (lc < n) dfs(lc + 1, rc, s + '(');
        if (rc < n && rc < lc) dfs(lc, rc + 1, s + ')');
    }
};

```





## 23. *合并K个升序链表(hard)

* 链表的k路归并问题, 可以采用堆排序解决.
  * 将每一个链表头节点插入堆.
  * 然后从堆中找到最小元素.
  * 之后再将下一个头节点插入堆.

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

* 首先, 链表头节点可能没有, 所以需要虚拟头节点.
* 其次, 如果要交换`p->a->b->b.next`中的节点`a`和`b`:
  * 首先, 将`a, b`看成一个整体, 动外面的指针: `p->next = b`, `a->next = b->next`.
  * 然后, 再动`a, b`内部的指针`b->next = a`.

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        
        auto dummy = new ListNode(-1);
        dummy->next = head;

        for (auto p = dummy; p->next && p->next->next;) {
            auto a = p->next, b = a->next;
            p->next = b;
            a->next = b->next;
            b->next = a;
            p = a;
        }

        return dummy->next;
    }
};
```



## 25. *K个一组反转链表(hard)

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

       for (auto p = dummy;; ) {
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



## 34. *二分插入(medium)

* 整数二分边界问题.

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

* 整数二分的边界: 假设左右两侧性质分别为`A`和`~A`
  * 左边界: 
    * `mid = l + (r - l) / 2 + 1`
    * `A`: `l = mid`
    * `~A`: `r = mid - 1`
  * 右边界:
    * `mid = l + (r - l) / 2`
    * `A`: `l = mid + 1`
    * `~A`: `r = mid`
* 注意, 如果插入位置在最后, 那么需要特判, 因为二分只能二分到数组内合法的索引.

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





## 39. *组合总和(medium)

* 虽然每一个元素可以任意选择, 但是由于有总和是`target`的限制, 可以选择的个数也是有限的, 枚举每一个元素以及这个元素的选择次数即可.

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





## 41. *缺失的第一个整数(medium)

* 思路:
  * 将所有数值“归位”, 就是让`nums[i] = i`, 然后遍历数组, 如果有不符合的, 那么就找到了最小没出现的正整数.

```cpp
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int n = nums.size();
        for (int i = 0; i < n; i ++) {
            if (nums[i] != INT_MIN) nums[i] --;
        }
        for (int i = 0; i < n; i ++) {
            while (nums[i] >= 0 && nums[i] < n && nums[nums[i]] != nums[i]) {
                swap(nums[i], nums[nums[i]]);
            }
        }
        for (int i = 0; i < n; i ++) {
            if (nums[i] != i) return i + 1;
        }
        return n + 1;
    } 
};

```





## 42. *接雨水(hard)

* 思路: 假设`l[i], r[i]`分别代表格子`h[i]`左右范围 (包括`h[i]`)的最大值, 那么这个格子对雨水的贡献量就是`min(l[i], r[i]) - h[i]`

```cpp
class Solution {
public:
    int trap(vector<int>& height) {
       const int N = 20010;
       int l[N], r[N];

       int maxv = 0;
       for (int i = 0; i < height.size(); i ++) {
            maxv = max(maxv, height[i]);
            l[i] = maxv;
       }
       maxv = 0;
       for (int i = height.size() - 1; i >= 0; i --) {
            maxv = max(maxv, height[i]);
            r[i] = maxv;
       }
       int res = 0;
       for (int i = 0; i < height.size(); i ++) {
            res += min(l[i], r[i]) - height[i];
       }
       return res;
    }
};
```



## 45. *跳跃游戏II(medium)

* 动态规划问题: 基于先验知识作出的最优选择, 也是全局视角下的最优选择.
* 假设`f[i]`表示跳到`i`位置的最小步数.
  * 那么`i`位置怎么才能从一个位置`j`一步跳到, 并且`j`离`i`最远呢.
  * 就是找最小的`j`, 但是`j + nums[j] >= i`的.

```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int n = nums.size();
        vector<int> f(n);

        for (int i = 1, j = 0; i < n; i ++) {
            while (j + nums[j] < i) j ++;
            f[i] = f[j] + 1;
        }
        return f[n - 1];
    }
};
```





## 46. *全排列(medium)

* 本题是无重复元素的数组求全排列.
* dfs时, 对于全排列的每一个位置, 枚举给定数组中的每一个元素, 看用哪一个元素放在这个位置, 选定位置后, 就递归到下一个位置.

```cpp
class Solution {
public:
    vector<bool> st;
    vector<int> path;
    vector<vector<int>> ans;
    vector<vector<int>> permute(vector<int>& nums) {
        int n = nums.size();
        st.resize(n);
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
            st[i] = true;
            path.push_back(nums[i]);
            dfs(nums, u + 1);
            st[i] = false;
            path.pop_back();
        }
    }
};
```





## 48. *旋转图像(medium)

* 顺时针90度: 主对角线对称(左上-右下), 竖直轴线对称.
* 逆时针90度: 水平轴线对称.
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
            for (int j = 0, k = n - 1; j < k; j ++, k --)
                swap(matrix[i][j], matrix[i][k]);
        
    }
};
```





## 49. *字母异位词(medium)

* 哈希表:

  * 思路: 一个单词的字母异位词, 指的是和它字母相同, 但是字母排列不同的所有单词组成的集合. 题目中要求集合中的单词必须在给定的数组中.

  ```cpp
  class Solution {
  public:
      vector<vector<string>> groupAnagrams(vector<string>& strs) {
          unordered_map<string, vector<string>> hash;
          for (auto &str: strs) {
              string nstr = str;
              sort(nstr.begin(), nstr.end());
              hash[nstr].push_back(str);
          }
          vector<vector<string>> ans;
          for (auto &item: hash) {
              ans.push_back(item.second);
          }
          return ans;
      }
  };
  ```



## 51. *N皇后 (hard)

* 用`col`, `dg`, `udg`分别记录每一列, 对角线/反对角线上是否有`Q`.
* 遍历时先遍历每一行`u`, 然后遍历每一列`i`, 如果一个位置`(u, i)`在列, 对角线, 反对角线上都没有棋子, 那么就可以放下棋子, 然后递归到下一个位置. 

```cpp
class Solution {
public:
    vector<bool> col;
    vector<bool> dg;
    vector<bool> udg;
    vector<string> g;
    vector<vector<string>> ans;
    vector<vector<string>> solveNQueens(int n) {
        col = vector<bool>(n, false);
        dg = vector<bool>(n, false);
        udg = vector<bool>(n, false);
        g = vector<string>(n, string(n, '.'));
        dfs(0);
        return ans;
    }
    void dfs(int u) {
        int n = g.size();
        if (u == n) {
            ans.push_back(g);
            return ;
        }
        for (int i = 0; i < n; i ++) {
            if (col[i] || dg[u + i] || udg[n + u - i]) continue;
            g[u][i] = 'Q';
            col[i] = dg[u + i] = udg[n + u - i] = true;
            dfs(u + 1);
            g[u][i] = '.';
            col[i] = dg[u + i] = udg[n + u - i] = false;
        }
    }
};
```






## 53. *最大子数组和(medium)

* 思路:
  * 对于这样一个连续和, 要么一直累加, 要么就从一个元素开始重开, 直接遍历即可.

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
       int ans = nums[0];
       int k = 0;
       for (auto x : nums) {
            k = max(k + x, x);
            ans = max(ans, k);
       }
       return ans;
    }
};
```



## 54. *螺旋矩阵(medium)

* 按照`dx, dy`向量方法遍历矩阵比较简单.

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int> ans;
        if (matrix.empty()) return ans;

        int n = matrix.size(), m = matrix[0].size();
        vector<vector<bool>> st(n, vector<bool>(m, false));

        int dx[] = { 0, 1, 0, -1 };
        int dy[] = { 1, 0, -1, 0 };

        int x = 0, y = 0, d = 0;
        for (int k = 0; k < n * m; k ++) {
            ans.push_back(matrix[x][y]);
            st[x][y] = true;
            int a = x + dx[d], b = y + dy[d];
            if (a < 0 || a >= n || b < 0 || b >= m || st[a][b])
                d = (d + 1) % 4;
            x = x + dx[d], y = y + dy[d];
        } 
        return ans;
    }
};

```



## 55. *跳跃游戏(medium)

* 枚举数组中的每一个数`nums[i]`, 它能够跳到的最大的位置就是`i + nums[i]`.
* 那么, 当遍历到`i`时, 如果之前能够跳到的最大位置`j < i`, 那么就无法到达这个位置, 因此, 就无法到达终点.

```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int range = 0;
        for (int i = 0; i < nums.size(); i ++) {
            if (range < i) return false;
            range = max(range, i + nums[i]);
        }
        return true;
    }
};
```





## 56. *合并区间(medium)

* 该题的意思是, 对于给定的任意两个区间, 如果有交集, 那么就合并成一个区间, 最后返回剩下的区间.
  * 首先, 所有区间按照左端点排序.
  * 之后, 维护一个区间`[st, ed]`, 这个区间是右端点最大的区间, 然后遍历所有区间`[l, r]`:
    * 如果`l <= ed`, 那么当前区间和维护的区间有交集, 直接合并`ed = max(ed, r)`
    * 如果`l > ed`, 那么当前区间和维护的区间没有交集, 需要更新`st = l, ed = r`.

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end());
        vector<vector<int>> ans;

        int st = -2e9, ed = -2e9;
        for (auto ra: intervals) {
            int l = ra[0], r = ra[1];
            if (l <= ed) ed = max(ed, r);
            else {
                if (ed != -2e9) ans.push_back({st, ed});
                st = l, ed = r;
            }
        }
      // 不要忘记最后还要push_back一下
        ans.push_back({st, ed});
        return ans;
    }
};

```



## 70. *爬楼梯 (easy)

* 注意: 需要考虑Fib数列的值是否会爆`int`.
* Fib数列的第0项是1, 第1项也是1.

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
                if (word1[i] != word2[j]) f[i][j] = min(f[i][j], f[i - 1][j - 1] + 1);
                else f[i][j] = min(f[i][j], f[i - 1][j - 1]);
            }
        return f[n][m];
    }
};
```





## 73. *矩阵置零(medium)

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
        
        bool row = 0, col = 0;
        for (int i = 0; i < n; i ++)
            if (!matrix[i][0]) col = 1;
        for (int i = 0; i < m; i ++)
            if (!matrix[0][i]) row = 1;

        for (int i = 1; i < n; i ++) {
            for (int j = 1; j < m; j ++) {
                if (!matrix[i][j])
                    matrix[i][0] = matrix[0][j] = 0;
            }
        }
        for (int i = 1; i < n; i ++) {
            if (!matrix[i][0]) {
                for (int j = 0; j < m; j ++)
                    matrix[i][j] = 0;
            }
        }
        for (int j = 1; j < m; j ++) {
            if (!matrix[0][j]) {
                for (int i = 0; i < n; i ++)
                    matrix[i][j] = 0;
            }
        }
        if (col) {
            for (int i = 0; i < n; i ++)
                matrix[i][0] = 0;
        }
        if (row) {
            for (int i = 0; i < m; i ++)
                matrix[0][i] = 0;
        }
    }
};
```



## 74. 搜索二维矩阵

* 二维矩阵和一维数组没有什么区别, 一位数组的下标`idx`分别除以/模矩阵列数就是在矩阵中的坐标`(idx / m, idx % m)`.

```cpp
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int n = matrix.size(), m = matrix[0].size();
        int s = n * m;
        int l = 0, r = s - 1;
        while (l < r) {
            int mid = l + (r - l) / 2;
            int x = mid / m, y = mid % m;
            if (matrix[x][y] < target) l = mid + 1;
            else r = mid;
        }
        return matrix[l / m][l % m] == target;
    }
};
```





## 76. *最小覆盖子串(hard)

* 双指针:
  * 思路: 向右移动指针, 直到完全覆盖`t`中的所有字符.
  * 然后向左移动指针, 找到长度最小的, 也可以完全覆盖`t`中字符的最小子串.
    * `hash[s[i]]`表示`t`中字符在`s`中出现的次数, 如果`hash[s[j]] + 1 > 0`, 那么表示如果`j`再向前移动, 就无法覆盖`t`中所有字符了.

```cpp
class Solution {
public:
    string minWindow(string s, string t) {
        int n = s.size(), m = t.size();
        unordered_map<char, int> hash;
        for (auto c: t) hash[c] ++;
        int tot = hash.size();

        int k = 0;
        int len = n;
        string ans;
        for (int i = 0, j = 0; i < n; i ++) {
            if (hash.count(s[i]) && --hash[s[i]] == 0) k ++;
            if (k == tot) {
                while (j < i) {
                    if (hash.count(s[j])) {
                        if (hash[s[j]] + 1 > 0) {
                            break;
                        }
                        hash[s[j]] ++;
                    }
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

* 从每一个格子位置开始DFS即可, 注意边界.

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

* 首先思考如何枚举每一个矩形, 假设某个矩形的高度是`h`, 那么从左边, 从右边第一个比`h`小的位置的右侧/左侧就是边界.
* 这个边界可以用单调栈预处理.

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
       root->left = dfs(preorder, inorder, pl + 1, pl + 1 + k - 1 - il + 1 - 1, il, k - 1);
       root->right = dfs(preorder, inorder, pl + 1 + k - 1 - il + 1, pr, k + 1, ir);
       return root;
    }
};

```





## 108. *将有序数组转换为二叉搜索树(medium)

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

* 从直观上来看, 需要把一个节点的左子树, 归并到这个节点和右节点之间.
* 左子树有两个关键的节点:
  * 左子树根节点, 当前节点的右指针要指向左子树根节点.
  * 当前节点中序遍历的前驱, 前驱的右节点需要只想右子树根节点.

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

* 注意: 杨辉三角每一行的第一个数和最后一个数都是1.

```cpp
class Solution {
public:
    vector<vector<int>> generate(int numRows) {
        vector<vector<int>> f;
        for (int i = 0; i < numRows; i ++) {
            vector<int> line(i + 1);
            line[0] = line[i] = 1;
            for (int j = 1; j < i; j ++)
                line[j] = f[i - 1][j - 1] + f[i - 1][j];
            f.push_back(line);
        }
        return f;
    }
};
```





## 121. *买卖股票的最佳时机(easy)

* 直接倒序遍历的过程中, 统计股票价格最大值即可.

  ```cpp
  class Solution {
  public:
      int maxProfit(vector<int>& prices) {
          int v = -1;
          int n = prices.size();
          int ans = 0;
          for (int i = n - 1; i >= 0; i --) {
              if (v != -1) {
                  ans = max(ans, v - prices[i]);
                  v = max(v, prices[i]);
              }
              else v = prices[i];
          }
          return ans;
      }
  };
  ```

  



## 124. *二叉树中的最大路径和(hard)

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

* 哈希表:
  * 思路: 
    * 首先, 将数组中所有数字插入哈希表.
    * 然后, 枚举每一个数字作为起点, 看最连续能到达哪里, 统计最大长度即可.
  * 时间复杂度: $O(n)$

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
       unordered_set<int> hash;
       for (auto x : nums) {
            hash.insert(x);
       }
       int ans = 0;
       for (auto x : nums) {
            if (!hash.count(x - 1)) {
                int y = x;
                while (hash.count(y + 1)) y ++;
                ans = max(ans, y - x + 1);
            }
       }
       return ans;
    }
};
```



## 131. *分割回文串(medium)

* 首先, 如果一个字符串是`aaaaa..`, 那么枚举分割方案的时间复杂度是$O(2^n)$, 所以这个问题是一个爆搜问题.
* 其次, 枚举分割方案的方法是, 枚举一个起点`u`, 然后从`u`向后 (包括`u`), 枚举终点`i`, 枚举终点后, 递归到下一个起点`i + 1`.
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

        for (int j = 0; j < n; j ++) {
            for (int i = j; i >= 0; i --) {
                if (i == j) f[i][j] = true;
                else if (i + 1 > j - 1) f[i][j] = (s[i] == s[j]);
                else f[i][j] = (s[i] == s[j]) && f[i + 1][j - 1];
            }
        }
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





## 138. *随机链表的复制

* 首先, 对于旧链表中的每一个节点, 在后面插入一个新的节点, 那么我就可以通过旧链表的位置相对关系, 推导出新链表的位置相对关系.
* 之后, 如果要复制`random`边, 只需要让旧链表中的节点`p`, 让`p->next->random = p->random->next`.
  * 其中`p->next`是新链表中的对应节点, `p->random->next`就是`p->random`在新链表中的相对位置.
* 之后, 再把新链表节点从原链表拆出来就可以.

```cpp
class Solution {
public:
    Node* copyRandomList(Node* head) {
       for (auto p = head; p ; p = p->next->next) {
            auto q = new Node(p->val);
            q->next = p->next;
            p->next = q;
       }
       for (auto p = head; p; p = p->next->next) {
            if (p->random)
                p->next->random = p->random->next;
            else
                p->next->random = NULL;
       }
       auto dummy = new Node(-1);
       dummy->next = head;
       auto cur = dummy;
       for (auto p = head; p; p = p->next) {
            auto q = p->next;
            cur = cur->next = q;
            p->next = p->next->next;
       }
       return dummy->next;
    }
};

```





## 141. *环形链表(easy)

* 检测链表中有没有环:
  * 首先特判链表为空, 或者只有一个节点的情况.
  * 之后, 用快慢指针, 快指针一次两个, 慢指针一次一个, 最终相遇, 就证明有环.

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if (!head || !head->next) return false;

        ListNode *p = head, *q = head;        
        while (p && q) {
            p = p->next;
            q = q->next;
            if (q) q = q->next;
            if (p == q) return true;
        }
        return false;
    }
};

```



## 142. *环形链表II (medium)

* 继上一题, 如果要找环形链表的环入口:
  * 当快慢指针第一次相遇时, 让慢指针退后到起始点.
  * 然后快指针和慢指针同时向后移动一次, 最终相遇点就是环的入口.

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if (!head || !head->next) return NULL;
        auto p = head, q = head;
        while (p && q) {
            p = p->next;
            q = q->next;
            if (q) q = q->next;
            if (q && p == q) break;
        }
        if (!q) return NULL;
        p = head;
        while (p != q) {
            p = p->next, q = q->next;
        }
        return p;
    }
};
```



## 144. 二叉树的前序遍历(medium)

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
        int key, val;
        Node *left, *right;
        Node(int _key, int _val): key(_key), val(_val), left(NULL), right(NULL) {}
    } *L, *R;
    unordered_map<int, Node*> hash;

    LRUCache(int capacity) {
        n = capacity;
        L = new Node(-1, -1);
        R = new Node(-1, -1);
        L->right = R;
        R->left = L;
    }

    void remove(Node *p) {
        p->right->left = p->left;
        p->left->right = p->right;
    }
    void insert(Node *p) {
        p->right = L->right;
        p->left = L;
        L->right->left = p;
        L->right = p;
    }
    
    int get(int key) {
        if (!hash.count(key)) return -1;
        auto p = hash[key];
        remove(p);
        insert(p);
        return p->val;
    }
    
    void put(int key, int value) {
        if (hash.count(key)) {
            auto p = hash[key];
            p->val = value;
            remove(p);
            insert(p);
        }
        else {
            if (hash.size() == n) {
                auto t = R->left;
                remove(t);
                hash.erase(t->key);
                delete t;
            }
            hash[key] = new Node(key, value);
            insert(hash[key]);
        }
    }
};
```



## 153. *寻找旋转排序数组中的最小值(medium)

* 假设数组的第一个元素是`nums[0]`.
* 数组的前半部分满足`nums[i] >= nums[0]`, 后半部分满足`nums[i] < nums[0]`, 以此来二分.
* 如果数组完全单调递增, 那么最终二分出来的`nums[i] >= nums[0]`, 此时直接返回`nums[0]`.

```cpp
class Solution {
public:
    int findMin(vector<int>& nums) {
        int n = nums.size();
        int t = nums[0];
        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] >= t) l = mid + 1;
            else r = mid;
        }
        if (nums[l] >= nums[0]) return nums[0];
        else return nums[l];
    }

};
```







## 155. 最小栈(medium)

* 直接开另外一个栈来维护最小值即可:

```cpp
class MinStack {
public:
    stack<int> stk;
    stack<int> stk_min;
    MinStack() {
        
    }
    
    void push(int val) {
        if (stk_min.empty() || val <= stk_min.top()) stk_min.push(val);
        stk.push(val);
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

* 思路: 
  * 让两个指针同时向前走一步, 如果有一个指针走到尽头, 那么就把它放到第二个指针的头部继续走.
  * 当两个指针相同时, 如果不为NULL, 那么就是相遇点, 否则就不是.

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        auto cur1 = headA, cur2 = headB;
        while (cur1 != cur2) {
            if (cur1) cur1 = cur1->next;
            else cur1 = headB;
            
            if (cur2) cur2 = cur2->next;
            else cur2 = headA;
        }
        if (cur1 == NULL) return NULL;
        return cur1;
    }
};
```





## 189 *轮转数组(medium)

* 思路: 反转数组的不同部分即可, 注意轮转的次数`k`要对数组的长度取模.

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

* 如何判断一个问题是否是DP?
  * 首先, 是求最优解.
  * 其次, 可能的方案数是指数级别.
* 状态机DP: `f[i][0]`表示第`i`个房子不打劫, `f[i][1]`表示打劫, 从后向前递推:

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> f(n, vector<int>(2, 0));

        f[0][0] = 0;
        f[0][1] = nums[0];
        for (int i = 1; i < nums.size(); i ++) {
            f[i][0] = max(f[i - 1][0], f[i - 1][1]);
            f[i][1] = f[i - 1][0] + nums[i];
        }
        return max(f[n - 1][0], f[n - 1][1]);
    }
};
```





## 199. *二叉树的右视图(medium)

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

* Flood Fill算法:
  * 每次遇到一块陆地, 就从这个陆地为起点进行搜索, 从这个起点向四周扩散, 如果再次遇到陆地, 就递归搜索.

```cpp
class Solution {
public:
    vector<vector<char>> g;
    int dx[4] = {-1, 0, 1, 0};
    int dy[4] = {0, 1, 0, -1};
    int numIslands(vector<vector<char>>& grid) {
        g = grid;
        int ans = 0;
        for (int i = 0; i < g.size(); i ++)
            for (int j = 0; j < g[i].size(); j ++)
                if (g[i][j] == '1')
                    dfs(i, j), ans ++;
        return ans;
    }
    void dfs(int x, int y) {
        g[x][y] = '0';
        for (int i = 0; i < 4; i ++) {
            int a = x + dx[i], b = y + dy[i];
            if (a >= 0 && a < g.size() && b >= 0 && b < g[0].size() && g[a][b] == '1')
                dfs(a, b);
        }
    }
};
```





## 206. *反转链表(easy)

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

* 从先修课程到后置课程连接一条有向边.
* 然后用拓扑排序, 所有入度为0的节点先入队, 然后BFS, 每次扩展一层就把入度-1, 然后再次把入度为0的点入队.
* 如果拓扑排序能把所有点遍历完全, 那么就证明符合要求.

```cpp
class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        int n = numCourses;
        vector<vector<int>> g(n);
        vector<int> d(n);

        for (auto &e: prerequisites) {
            int a = e[1], b = e[0];
            g[a].push_back(b);
            d[b] ++;
        }
        queue<int> q;
        for (int i = 0; i < n; i ++)
            if (d[i] == 0)
                q.push(i);

        int cnt = 0;
        while (q.size()) {
            auto t = q.front();
            q.pop();
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



## 230. *二叉搜索树中第K小的元素

* 中序遍历的过程中记录遍历节点的次数即可.

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





## 234. *回文链表

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



## 236. *二叉树的最近公共祖先

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



## 240. *搜索二维矩阵

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





## 283. *移动0(medium)

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



## 295. *数据流的中位数(hard)

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





## 347. *前k个高频元素(medium)

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



## 437. *路径总和 (medium)

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

