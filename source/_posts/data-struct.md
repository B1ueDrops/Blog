---
title: 数据结构系列
categories: 算法
mathjax: true
---





## 用数组模拟栈

> https://www.acwing.com/problem/content/description/

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int tt = -1;
int stk[N];

int main() {

    int m;
    cin >> m;

    while (m --) {

        string op;
        cin >> op;

        if (op == "push") {
            int x;
            cin >> x;
            stk[++ tt] = x;
        }
        else if (op == "pop") {
            -- tt;
        }
        else if (op == "empty") {
            if (tt < 0) puts("YES");
            else puts("NO");
        }
        else {
            printf("%d\n", stk[tt]);
        }
    }
    return 0;
}
```



## 表达式求值(栈)

> https://www.acwing.com/problem/content/description/3305/
>
> https://www.acwing.com/problem/content/description/456/

直接背模板:

```cpp
#include <iostream>
#include <stack>
#include <unordered_map>


using namespace std;

/* 将运算符映射到优先级上 */
unordered_map<char, int> pr = {
    { '+', 1 },
    { '-', 1 },
    { '*', 2 },
    { '/', 2 }
};

/* 存储运算数的栈 */
stack<int> nums;
/* 存储运算符的栈 */
stack<char> op;

/* 表达式 */
string str;

void eval()
{
    int b = nums.top(); nums.pop();
    int a = nums.top(); nums.pop();
    
    char c = op.top(); op.pop();
    
    if (c == '+') nums.push(a + b);
    else if (c == '-') nums.push(a - b);
    else if (c == '*') nums.push(a * b);
    else nums.push(a / b);
}

int main()
{
    cin >> str;
    
    for (int i = 0; i < str.size(); i ++)
    {
        /* 如果是左括号, 加入运算符的栈 */
        if (str[i] == '(') op.push(str[i]);
        /* 如果是右括号, 那么需要eval括号内表达式的值 */
        else if (str[i] == ')')
        {
            while (op.size() && op.top() != '(') eval();
            op.pop();
        }
        /* 如果是数位, 那么需要找到整个数, 然后放到nums栈中 */
        else if (isdigit(str[i]))
        {
            int x = 0, j = i;
          	/* 这个j ++不要忘记 */
            while (j < str.size() && isdigit(str[j])) x = 10 * x + str[j ++] - '0';
            i = j - 1;
            nums.push(x);
        }
        /* 如果是运算符, 需要先eval优先级高的运算符, 然后放到栈中 */
        else
        {
            while (op.size() && pr[op.top()] >= pr[str[i]]) eval();
            op.push(str[i]);
        }
    }
    while (op.size()) eval();
    
    cout << nums.top() << endl;
    return 0;
}
```







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
>
> https://www.acwing.com/problem/content/802/

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



## 字母异位词分组(哈希表)

> https://leetcode.cn/problems/group-anagrams/

这个题的题意是: 

* 如果一个单词能通过字符的重排, 变成另一个单词, 那么这两个单词就属于同一组.
* 需要把属于同一组的所有单词放到同一个列表中返回.

如果两个单词属于同一组, 那么它们字符排序后的结果一样.

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> hash;
        for (auto str : strs) {
            string nstr = str;
            sort(nstr.begin(), nstr.end());
            hash[nstr].push_back(str);
        }
        vector<vector<string>> ans;
        for (auto &item: hash) ans.push_back(item.second);
        return ans;
    }
};
```



## 无重复字符的最长子串(哈希表)

> https://leetcode.cn/problems/longest-substring-without-repeating-characters/

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





## 最长连续序列(哈希表)

> https://leetcode.cn/problems/longest-consecutive-sequence/
>
> https://www.acwing.com/problem/content/description/801/

* 首先, 把所有元素放到Hash Set中.
* 然后, 枚举每一个元素`x`, 如果元素`x - 1`不在hash set中, 证明序列不再连续, 需要单开一个起点.
* 然后, 从单开的起点`y = x`开始, 依次判断`y + 1`是否存在, 就能找到连续序列的终点.

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> hash;
        for (auto x : nums) hash.insert(x);

        int res = 0;
        for (auto x : nums) {
            if (!hash.count(x - 1)) {
                int y = x;
                while (hash.count(y + 1)) y ++;
                res = max(res, y - x + 1);
            }
        }
        return res;
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

