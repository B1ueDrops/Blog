---
title: 数据结构系列
categories: 算法
mathjax: true
---

[toc]

## 栈/单调栈

### 用栈计算表达式

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

### 单调栈

> https://www.acwing.com/problem/content/90/

再维护一个专门存最小值的栈即可.

```cpp
class MinStack {
public:
    /** initialize your data structure here. */
    stack<int> stk, stk_min;
    MinStack() {
        
    }
    
    void push(int x) {
        stk.push(x);
        if (stk_min.size()) x = min(x, stk_min.top());
        stk_min.push(x);
    }
    
    void pop() {
        stk.pop();
        stk_min.pop();
    }
    
    int top() {
        return stk.top();
    }
    
    int getMin() {
        return stk_min.top();
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.getMin();
 */
```



## 队列



### 单调队列

> https://www.acwing.com/problem/content/156/

* 单调队列需要满足如下几个性质:
  * 插入元素后, 其中的元素从队头弹出的顺序按照某个函数递增/递减.
* 

```cpp
#include <iostream>

using namespace std;
const int N = 100010;
int n, k, a[N];
int hh = 0, tt = -1;
int q[N];

int main() {

	cin >> n >> k;
	for (int i = 0; i < n; i ++) cin >> a[i];

	for (int i = 0; i < n; i ++) {
		if (i - k + 1 > q[hh]) hh ++;
		while (hh <= tt && a[i] <= a[q[tt]]) tt --;
		q[ ++ tt] = i;
		if (i - k + 1 >= 0) cout << a[q[hh]] << ' ';
	}
	hh = 0, tt = -1;
	cout << endl;
	for (int i = 0; i < n; i ++) {
		if (i - k + 1 > q[hh]) hh ++;
		while (hh <= tt && a[i] >= a[q[tt]]) tt --;
		q[ ++ tt] = i;
		if (i - k + 1 >= 0) cout << a[q[hh]] << ' ';
	}
	return 0;
}
```



### 最大子序和
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

## 哈希表

### 手写哈希表

> https://www.acwing.com/problem/content/842/

拉链法: 将要存储的元素$x$, 哈希之后的下标是$k$, 如果有冲突, 就在数组的基础上, 拉成一个链表.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 100003;

int n;
int h[N], e[N], ne[N], idx;

void insert(int x)
{
    int k = (x % N + N) % N;
    e[idx] = x;
    ne[idx] = h[k];
    h[k] = idx ++;
}

bool find(int x)
{
    int k = (x % N + N) % N;

    for (int i = h[k]; i != -1; i = ne[i])
        if (e[i] == x) return true;
        
    return false;
}

int main()
{
    cin >> n;
    memset(h, -1, sizeof h);
    
    string op;
    int x;
    
    while (n --)
    {
        cin >> op >> x;
        if (op == "I") insert(x);
        else
        {
            if (find(x)) puts("Yes");
            else puts("No");
        }
    }
    return 0;
}
```

### 两数之和

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



### 字母异位词分组

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



### 无重复字符的最长子串

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





### 最长连续序列

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

同类题: 找到字符串中所有的字母异位词

> https://leetcode.cn/problems/find-all-anagrams-in-a-string/

* 核心: 在`s`串中维护一个长度是`p.size()`的滑动窗口, 如果窗口内字符的种类和**出现次数**和`p`串一致, 那么就符合题意.

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        unordered_map<char, int> hash;
      // 存储p中每个字符出现的次数
        for (auto c: p) hash[c] ++;
      // tot表示p中出现的字符种类
        int tot = hash.size();

        vector<int> res;
      // sa表示有多少种字符符合题意
        for (int i = 0, j = 0, sa = 0; i < s.size(); i ++) {
          // 遍历每一个字符, 窗口右侧在哈希表中减, 如果减到0, sa++
            if (-- hash[s[i]] == 0) sa ++;
          // 维护窗口大小
            if (i - j + 1 > p.size()) {
              // 如果原先s[j]是符合题意的字符, 窗口移走了, 符合题意的字符就少一个
                if (hash[s[j]] == 0) sa --;
                hash[s[j ++]] ++;
            }
            if (sa == tot) res.push_back(j);
        }
        return res;
    }
};
```



## 堆



### 数据流中的第k大数

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

### 堆排序模板题

> https://www.acwing.com/problem/content/840/

堆是一颗完全二叉树, 根节点小于等于左右两个子节点的值, 然后递归定义.

这个题考察堆的下沉操作.

注意, 把一个元素个数是$n$的数组变成堆的时间复杂度是$O(n)$, 直接从$\frac{n}{2}$下沉到1即可.

* 完全二叉树除了最后一层, 上面所有层的元素个数是$\frac{n}{2}$, 那么根据这个关系, 倒数第二层的元素个数是$\frac{n}{4}$.
* 倒数第二层元素个数是$\frac{n}{4}$, 需要下沉1层.
* 倒数第三层的元素个数是$\frac{n}{8}$, 需要下沉2层
* 倒数第四层元素个数是$\frac{n}{16}$, 需要下沉3层...

把这些加起来, 会发现它们其实和$O(n)$一个级别.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int n, m;
/* s是堆中的元素个数 */
int heap[N], s;

/* 下沉操作 */
void down(int u)
{
    int t = u;
    if (2 * u <= s && heap[2 * u] < heap[t]) t = 2 * u;
    if (2 * u + 1 <= s && heap[2 * u + 1] < heap[t]) t = 2 * u + 1;
    
    if (t != u)
    {
        swap(heap[t], heap[u]);
        down(t);
    }
}

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) cin >> heap[i];
    
    s = n;
    
    /* O(n)建堆 */
    for (int i = n / 2; i >= 1; i --) down(i);
    
    while (m --)
    {
        cout << heap[1] << ' ';
        heap[1] = heap[s --];
        down(1);
    }
    return 0;
}
```





### 索引优先队列模板题

> https://www.acwing.com/problem/content/841/

堆的一些操作的实现方式:

* 插入: 插入到最后然后上浮.
* 删除最小值: 让最后一个元素和第一个元素交换, 然后`size --`, 然后让第一个元素下沉.
* 

索引优先队列可以让堆支持以$O(1)$的时间复杂度修改第$k$个插入的数这种操作, 本质上是用空间换时间.

维护两个数组:

* `ph[i] = k`: 第$i$个插入的点, 在堆中的下标是$k$.
* `hp[k] = i`: 在堆中下标是$k$的点, 是第$i$个插入的点.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

/* idx为插入次序 */
int n, s, idx;
int h[N], hp[N], ph[N];

/* a, b是元素在堆中的下标 */
void heap_swap(int a, int b)
{
    swap(ph[hp[a]], ph[hp[b]]);
    swap(hp[a], hp[b]);
    swap(h[a], h[b]);
}

/* 下沉 */
void down(int u)
{
    int t = u;
    if (2 * u <= s && h[2 * u] < h[t]) t = 2 * u;
    if (2 * u + 1 <= s && h[2 * u + 1] < h[t]) t = 2 * u + 1;
    
    if (t != u)
    {
        heap_swap(t, u);
        down(t);
    }
}

/* 上浮 */
void up(int u)
{
    while (u / 2 && h[u] < h[u / 2])
    {
        heap_swap(u, u / 2);
        u >>= 1;
    }
}

int main()
{
    cin >> n;
    
    string op;
    int k, x;
    
    
    while (n --)
    {
        cin >> op;
        if (op == "I")
        {
            cin >> x;
            s ++;
            idx ++;
            h[s] = x;
            ph[idx] = s;
            hp[s] = idx;
            up(s);
        }
        else if (op == "PM") cout << h[1] << endl;
        else if (op == "DM")
        {
            heap_swap(1, s);
            s --;
            down(1);
        }
        else if (op == "D")
        {
            cin >> k;
            k = ph[k];
            heap_swap(k, s);
            s --;
            up(k);
            down(k);
        }
        else if (op == "C")
        {
            cin >> k >> x;
            k = ph[k];
            h[k] = x;
            up(k);
            down(k);
        }
    }
    return 0;
}
```



### 最小的k个数

> https://www.acwing.com/problem/content/49/

直接用一个大根堆动态维护即可.

```cpp
class Solution {
public:
    vector<int> getLeastNumbers_Solution(vector<int> input, int k) {
        
        priority_queue<int> heap;
        
        for (auto x : input)
        {
            if (heap.size() < k || heap.top() > x) heap.push(x);
            if (heap.size() > k) heap.pop();
        }
        
        vector<int> res;
        while (heap.size()) res.push_back(heap.top()), heap.pop();
        
        reverse(res.begin(), res.end());
        return res;
    }
};
```

### 串连所有单词的子串

> https://leetcode.cn/problems/substring-with-concatenation-of-all-words/

#### 暴力做法

首先用一个哈希表`hash`预处理每个单词出现的次数.

之后枚举给定字符串`s`的每一个起点, 对于每一个起点, 从后看每一个单词, 然后用另一个哈希表`cur_hash`处理遍历过程中每一个单词出现的次数, 假设现在遍历到的单词是`cur`:

* 如果`cur_hash[cur] <= hash[cur]`, 正常, 继续遍历
* 如果`cur_hash[cur] > hash[cur]`或者`cur`在`hash`中根本没有出现: 那么不符合题意, 当前枚举的`s`起点是无效的.

之后, 如果遍历单词的个数达到了所有单词的数量, 那么就成功了.

时间复杂度是$O(nm)$, 其中$n$是字符串`s`的长度, $m$是单词的个数.

```cpp
class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        unordered_map<string, int> hash;
        vector<int> ans;

        int n = s.size(), m = words.size(), len = words[0].size();

        /* 统计每个单词出现的次数 */
        for (auto word: words) hash[word] ++;

        for (int i = 0; i <= n - m * len; i ++) {
            
            unordered_map<string, int> cur_hash;
            int cnt = 0;

            for (int j = i; j <= n - len; j += len) {

                string cur = s.substr(j, len);

                /* 如果cur没有在words中出现过, 直接失败 */
                if (!hash.count(cur)) break;
            
                cur_hash[cur] ++;
                if (cur_hash[cur] > hash[cur]) break;
                else if (cur_hash[cur] <= hash[cur]) cnt ++;
                if (cnt == words.size()) {
                    ans.push_back(i);
                    break;
                }

            }
        }
        return ans;
    }
};
```



#### 双指针算法

首先, 还是用哈希表预处理每一个单词出现的次数.

假设每一个单词的长度是`len`.

之后, 对于给定的字符串`s`, 将它切分成一段一段, 每一段长度是`len`, 这些段可以组成一个原串的单词序列, 不同的序列一共有`len`中, 它们的起点分别是`0, 1, 2, ..., len - 1`, 其中以`len`为起点的序列和以`0`为起点的序列是等价的, 因为是子串关系.

之后, 遍历每一个从`s`中得到的序列, 用双指针算法得到由`words`中单词组成的连续的最长的序列, 看这个序列中的单词个数是否是`m`, 如果是`m`, 那就成功, 这个双指针算法可以参考[LeetCode 3](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

一共有`len`个序列, 枚举每个序列需要$O(n)$的时间复杂度, 那么总的时间复杂度就是$O(nlen)$

```cpp
class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        unordered_map<string, int> hash;
        vector<int> ans;

        int n = s.size(), m = words.size(), len = words[0].size();

        /* 统计每个单词出现的次数 */
        for (auto word: words) hash[word] ++;

        for (int i = 0; i < len; i ++) {
            vector<string> ws;
            int j = i;
            while (j + len - 1 < s.size()) {
                ws.push_back(s.substr(j, len));
                j += len;
            }

            unordered_map<string, int> cur_hash;
            for (int p = 0, q = 0; p < ws.size(); p ++) {
                cur_hash[ws[p]] ++;
                while (cur_hash[ws[p]] > hash[ws[p]]) cur_hash[ws[q ++]] --;
                if (p - q + 1 == m) ans.push_back(i + q * len);
            }
        }
        return ans;
    }
};
```


### 字符串前缀哈希

> https://www.acwing.com/problem/content/843/

按照字符的ASCII码, 可以将字符串映射到一个$P$进制的数, 可以实现在$O(1)$的时间内判断字符串的两个子串是否相等.

```cpp
#include <iostream>

using namespace std;

const int N = 100010, P = 131;

typedef unsigned long long ULL;

ULL h[N], p[N];
char str[N];

int n, m;

ULL get(int l, int r) {
    return h[r] - h[l - 1] * (r - l + 1);
}

int main() {
    
    scanf("%d%d", &n, &m);
    scanf("%s", str + 1);
    
    p[0] = 1;
    
    for (int i = 1; i <= n; i ++) {
        h[i] = h[i - 1] * P + str[i];
        p[i] = p[i - 1] * P;
    }
    
    while (m --) {
        int l1, r1, l2, r2;
        scanf("%d%d%d%d", &l1, &r1, &l2, &r2);
        if (get(l1, r1) == get(l2, r2)) puts("Yes");
        else puts("No");
    }
    return 0;
}
```

### 字母异位词分组

> https://leetcode.cn/problems/group-anagrams/

如果两个单词在同一个组, 那么它们排序后的字符串相同.

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> hashs;
        
        for (auto &str: strs) {
            string nstr = str;
            sort(nstr.begin(), nstr.end());
            hashs[nstr].push_back(str);
        }

        vector<vector<string>> ans;
      /* 遍历哈希表的方式 */
        for (auto &item: hashs) ans.push_back(item.second);
        return ans;
    }
};
```

## 字典树

### 字典树模板题

> https://www.acwing.com/problem/content/837/

字典树能够在$O(k)$的时间复杂度内查找一个字符串在集合中出现的次数, 其中$k$是字符串的长度.

```cpp
#include <iostream>

using namespace std;

/* 这个N表示字典树中可能有多少个节点, 等于所有可能插入的字符串的长度之和 */
const int N = 100010;

int idx, cnt[N], son[N][26];

char str[N];

/* 向字典树中插入str */
void insert()
{
    int p = 0;
    for (int i = 0; str[i]; i ++)
    {
        int u = str[i] - 'a';
        if (!son[p][u]) son[p][u] = ++idx;
        p = son[p][u];
    }
    cnt[p] ++;
}

/* 查找str出现的次数 */
int query()
{
    int p = 0;
    for (int i = 0; str[i]; i ++)
    {
        int u = str[i] - 'a';
        if (!son[p][u]) return 0;
        p = son[p][u];
    }
    return cnt[p];
}

int main()
{
    int n;
    cin >> n;
    
    while (n --)
    {
        char op[2];
        scanf("%s", op);
        
        if (*op == 'I')
        {
            scanf("%s", str);
            insert();
        }
        else
        {
            scanf("%s", str);
            cout << query() << endl;
        }
    }
    return 0;
}
```



### 最大异或对

> https://www.acwing.com/problem/content/145/

在给定的一个数组中, 选取两个数, 使得它们的异或最大.

```cpp
#include <iostream>

using namespace std;

const int N = 31 * 100010;

int idx, son[N][2];

int n, a[N / 31];

void insert(int x)
{
    int p = 0;
    for (int i = 30; i >= 0; i --)
    {
        int u = x >> i & 1;
        if (!son[p][u]) son[p][u] = ++idx;
        p = son[p][u];
    }
}

int query(int x)
{
    int p = 0;
    int res = 0;
    for (int i = 30; i >= 0; i --)
    {
        int u = x >> i & 1;
        if (son[p][!u])
        {
            res += (1 << i);
            p = son[p][!u];
        }
        else p = son[p][u];
    }
    return res;
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) 
    {
        cin >> a[i];
        insert(a[i]);
    }
    
    int res = -1;
    for (int i = 0; i < n; i ++) res = max(res, query(a[i]));
    
    cout << res << endl;
    return 0;
}
```



## 并查集

### 并查集模板题

> https://www.acwing.com/problem/content/838/

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int n, m;
int p[N];

int find(int x)
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int main()
{
    
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) p[i] = i;
    
    string op;
    int a, b;
    
    while (m --)
    {
        cin >> op >> a >> b;
        if (op == "M") p[find(a)] = find(b);
        else
        {
            if (find(a) == find(b)) puts("Yes");
            else puts("No");
        }
    }
    
    return 0;
}
```



### 连通块中点的个数

> https://www.acwing.com/problem/content/839/

本题是利用并查集来维护集合中元素的个数.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int n, m;
int p[N], s[N];

int find(int x)
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int main()
{
    cin >> n >> m;
    
    for (int i = 1; i <= n; i ++) p[i] = i, s[i] = 1;
    
    string op;
    int a, b;
    
    while (m --)
    {
        cin >> op;
        if (op == "C")
        {
            cin >> a >> b;
            a = find(a), b = find(b);
            if (a != b) p[a] = b, s[b] += s[a];
        }
        else if (op == "Q1")
        {
            cin >> a >> b;
            a = find(a), b = find(b);
            if (a == b) puts("Yes");
            else puts("No");
        }
        else if (op == "Q2")
        {
            cin >> a;
            cout << s[find(a)] << endl;
        }
    }
    return 0;
}
```



### 食物链

> https://www.acwing.com/problem/content/242/

本题是利用并查集维护一种环形的逻辑关系.

假设$X$吃$Y$, 并且$Y$吃$Z$, 那么根据题目要求, $Z$肯定吃$X$.

那么规定, $Z$是第0代.

* 因为$Y$吃$Z$, 所以规定$Y$是第1代.
* 因为$X$吃$Y$, 所以规定$X$是第2代.
* 那么第3代肯定和$Z$是同类

因此, 按照这个规定:

* 如果一个点到根节点的距离为1, 那么这个点吃根节点.
* 如果一个点到根节点的距离为2, 那么这个点被根节点吃.
* 如果一个点到根节点的距离在模3的意义下为0, 那么这个点和根节点同类.

这个距离可以用并查集维护.

```cpp
#include <iostream>

using namespace std;

const int N = 50010;

int n, k;
int p[N], d[N];

int find(int x)
{
    if (p[x] != x)
    {
        /* 先找到根节点, 并且压缩后面的路径 */
        int t = find(p[x]);
        /* 维护距离 */
        d[x] += d[p[x]];
        p[x] = t;
    }
    return p[x];
}


int main()
{
    cin >> n >> k;
    for (int i = 1; i <= n; i ++) p[i] = i;
    
    int res = 0;
    while (k --)
    {
        int t, x, y;
        cin >> t >> x >> y;
        
        /* 假话标准2 */
        if (x > n || y > n)
        {
            res ++;
            continue;
        }
        /* 如果给出的是同类的关系 */
        if (t == 1)
        {
            /* 找到x和y的父节点 */
            int px = find(x), py = find(y);
            /* 如果px == py, 那么x和y的关系一定已知 */
            if (px == py)
            {
                /* 如果d[x]和d[y]在模3的意义下距离不一样, 那么就不是同类 */
                if ((d[x] - d[y]) % 3 != 0)
                {
                    res ++;
                    continue;
                }
            }
            /* 如果px != py, 那么就需要维护x和y的关系 */
            else
            {
                p[px] = py;
                /* 距离有(d[x] + d[px] - d[y]) % 3 == 0 */
                d[px] = d[y] - d[x];
            }
        }
        /* 如果给出的是吃与被吃的关系 */
        else
        {
            int px = find(x), py = find(y);
            if (px == py)
            {
                /* x吃y, 如果是真话, 那么x肯定比y距离多1 */
                if ((d[x] - d[y] - 1) % 3 != 0)
                {
                    res ++;
                    continue;
                }
            }
            else
            {
                p[px] = py;
                /* (d[x] + d[px] - 1 - d[y]) % 3 == 0 */
                d[px] = d[y] + 1 - d[x];
            }
        }
    }
    cout << res << endl;
    return 0;
}
```
