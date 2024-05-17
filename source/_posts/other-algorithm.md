---
title: 冷门算法/模拟题
categories: 算法
mathjax: true
---





## 保序离散化

保序离散化适合如下情况:

* 题目中的数值的范围过大, 例如在$[-10^9, 10^9]$这种, 但是出现次数比较少.

保序离散化是指将这些过大的数值映射到从0开始递增的自然数的过程.

模板代码:

```cpp
#include <vector>
#include <algorithm>

/* 定义离散化数组 */
vector<int> alls;

/* 找到x对应的离散化后的数值 */
/* <= x的最大值 */
int find(int x)
{
  int l = 0, r = alls.size() - 1;
  while (l < r)
  {
    int mid = l + (r - l) / 2 + 1;
    if (alls[mid] > x) r = mid - 1;
    else l = mid;
  }
  /* 如果要映射到从0开始的, 就return r, 如果从1开始就return r+1 */
  return r;
}

int main()
{
  /* other code */
  
  /* 将需要离散化的数值全部放入alls中 */
  /* 排序 */
  sort(alls.begin(), alls.end());
  /* 去重 */
  alls.erase(unique(alls.begin(), alls.end()), alls.end());
  
  
  return 0;
}
```



## 区间和

> https://www.acwing.com/problem/content/804/

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

//注意这里的倍数需要根据题目确定
const int N = 100010 * 3;

typedef pair<int, int> PII;

int n, m;

vector<int> alls;
vector<PII> add;
vector<PII> query;
int a[N];

int find(int x)
{
    int l = 0, r = alls.size() - 1;
    while (l < r)
    {
        int mid = l + (r - l) / 2 + 1;
        if (alls[mid] > x) r = mid - 1;
        else l = mid;
    }
  /* 注意这个题是前缀和, 要从1开始映射 */
    return r + 1;
}

int main()
{
    cin >> n >> m;
    
    for (int i = 0; i < n; i ++)
    {
        int x, c;
        cin >> x >> c;
        add.push_back({x, c});
        alls.push_back(x);
    }
    
    for (int i = 0; i < m; i ++)
    {
        int l, r;
        cin >> l >> r;
        query.push_back({l, r});
        alls.push_back(l);
        alls.push_back(r);
    }
    
    sort(alls.begin(), alls.end());
    alls.erase(unique(alls.begin(), alls.end()), alls.end());
    
    for (auto item: add)
    {
        int x = item.first, c = item.second;
        a[find(x)] += c;
    }
    
    for (int i = 1; i <= alls.size(); i ++) a[i] += a[i - 1];

    
    for (auto item: query)
    {
        int l = item.first, r = item.second;
      /* 注意这里不是a[find(l-1)] */
        cout << (a[find(r)] - a[find(l) - 1]) << endl;
    }
    
    return 0;
}
```





## 整数转罗马数字

> https://leetcode.cn/problems/integer-to-roman/

罗马数字的一些对应表如下:

```
1    2    3    4    5    6    7    8    9
I   II   III   IV   V   VI   VII   VIII IX

10   20   30   40   50   60   70   80   90
X    XX   XXX  XL    L   LX  LXX  LXXX  XC

100  200  300  400  500  600   700    800   900
C    CC   CCC   CD   D    DC   DCC    DCCC   CM

1000  2000  3000
 M     MM   MMM
```

基本单位是: `1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1`

对应的罗马数字的基本元素是`M, CM, D, CD, C, XC, L, XL, X, IX, V, IV, I `

直接对给定的数据求它们由多少个基本单位组成即可.

```cpp
class Solution {
public:
    string intToRoman(int num) {
        int values[] = {
            1000,
            900, 500, 400, 100,
            90, 50, 40, 10,
            9, 5, 4, 1
        };

        string resp[] = {
            "M",
            "CM", "D", "CD", "C",
            "XC", "L", "XL", "X",
            "IX", "V", "IV", "I",
        };

        string ans;

        for (int i = 0; i < 13; i ++) {
            while (num >= values[i]) {
                num -= values[i];
                ans += resp[i];
            }
        }
        return ans;
    }
};
```



## 罗马数字转整数

> https://leetcode.cn/problems/roman-to-integer/

```cpp
class Solution {
public:
    int romanToInt(string s) {
            unordered_map<char, int> hash;
            hash['I'] = 1;
            hash['V'] = 5;
            hash['X'] = 10;
            hash['L'] = 50;
            hash['C'] = 100;
            hash['D'] = 500;
            hash['M'] = 1000;

            int res = 0;
            for (int i = 0; i < s.size(); i ++) {
                if (i + 1 < s.size() && hash[s[i]] < hash[s[i + 1]]) {
                    res -= hash[s[i]];
                }
                else res += hash[s[i]];
            }
            return res;
    }
};
```



## 移除元素

> https://leetcode.cn/problems/remove-element/

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int k = 0;

        for (int i = 0; i < nums.size(); i ++) {
            if (nums[i] != val) nums[k ++] = nums[i];
        }
        return k;
    }
};
```



## 二维数组中的查找

> https://www.acwing.com/problem/content/16/

以最右上角的元素为基准, 如果最右上角的元素比`target`大, 那么就向左移动一格变小, 如果比`target`小, 那么就向右移动一格变大.

```cpp
class Solution {
public:
    bool searchArray(vector<vector<int>> array, int target) {
        
        int n = array.size();
        if (!n) return false;
        int m = array[0].size();
        
        int i = 0, j = m - 1;
        while (i < n && j >= 0) {
            if (target < array[i][j]) j --;
            else if (target > array[i][j]) i ++;
            else return true;
        }
        return false;
    }
};
```



## 用两个栈实现队列

> https://www.acwing.com/problem/content/description/36/

```cpp
class MyQueue {
public:
    stack<int> s1;
    stack<int> s2;
    /** Initialize your data structure here. */
    MyQueue() {
        
    }
    
    /** Push element x to the back of queue. */
    void push(int x) {
        s1.push(x);
    }
    
    /** Removes the element from in front of queue and returns that element. */
    int pop() {
        while (s1.size() > 1) {
            int t = s1.top(); s1.pop();
            s2.push(t);
        }
        int t = s1.top(); s1.pop();
        while (s2.size()) {
            int q = s2.top(); s2.pop();
            s1.push(q);
        }
        return t;
    }
    
    /** Get the front element. */
    int peek() {
        while (s1.size() > 1) {
            int t = s1.top(); s1.pop();
            s2.push(t);
        }
        int t = s1.top();
        while (s2.size()) {
            int q = s2.top(); s2.pop();
            s1.push(q);
        }
        return t;
    }
    
    /** Returns whether the queue is empty. */
    bool empty() {
        return s1.size() == 0;
    }
};

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue obj = MyQueue();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.peek();
 * bool param_4 = obj.empty();
 */
```



## 有效的数独

> https://leetcode.cn/problems/valid-sudoku/

枚举行列和每一个小九宫格, 判断即可.

```cpp
class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
      /* 判重数组 */
        bool st[9];

        /* 判断行 */
        for (int i = 0; i < 9; i ++) {
            memset(st, false, sizeof(st));
            for (int j = 0; j < 9; j ++) {
                if (board[i][j] == '.') continue;
                int t = board[i][j] - '1';
                if (st[t]) return false;
                st[t] = true;
            }
        }
        

        /* 判断列 */
        for (int i = 0; i < 9; i ++) {
            memset(st, false, sizeof(st));
            for (int j = 0; j < 9; j ++) {
                if (board[j][i] == '.') continue;
                int t = board[j][i] - '1';
                if (st[t]) return false;
                st[t] = true;
            }
        }
				/* 判断每一个小九宫格 */
        for (int i = 0; i < 9; i += 3) {
            for (int j = 0; j < 9; j += 3) {
                memset(st, false, sizeof(st));
                for (int x = i; x < i + 3; x ++) {
                    for (int y = j; y < j + 3; y ++) {
                        if (board[x][y] == '.') continue;
                        int t = board[x][y] - '1';
                        if (st[t]) return false;
                        st[t] = true;
                    }
                }
            }
        }
        return true;
    }
};
```





## 手写栈

> https://www.acwing.com/problem/content/830/

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

int tt;
int stk[N];

void init()
{
    tt = -1;
}

void push(int x)
{
    stk[++ tt] = x;
}

void pop()
{
    tt --;
}

bool empty()
{
    return tt < 0;
}

int top()
{
    return stk[tt];
}

int main()
{
    /* 不要忘了init */
    init();
    int m;
    scanf("%d", &m);
    
    while (m --)
    {
        string op;
        int x;
        
        cin >> op;
        
        if (op == "push")
        {
            cin >> x;
            push(x);
        }
        else if (op == "pop")
        {
            pop();
        }
        else if (op == "query")
        {
            cout << top() << endl;
        }
        else if (op == "empty")
        {
            if (empty()) puts("YES");
            else puts("NO");
        }
    }
    return 0;
}
```



## 模拟散列表

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



## 逆时针打印矩阵

> https://www.acwing.com/problem/content/description/39/

可以借助方向向量实现.

```cpp
class Solution {
public:
    vector<int> printMatrix(vector<vector<int> > matrix) {
        
        vector<int> res;
        if (matrix.empty()) return res;
        
        int n = matrix.size(), m = matrix[0].size();
        
        vector<vector<bool>> st(n, vector<bool>(m, false));
        
        int dx[4] = { 0, 1, 0, -1 };
        int dy[4] = { 1, 0, -1, 0 };
        
        int x = 0, y = 0, d = 0;
        
        for (int k = 0; k < n * m; k ++)
        {
            res.push_back(matrix[x][y]);
            st[x][y] = true;
            
            int a = x + dx[d], b = y + dy[d];
            if (a < 0 || a >= n || b < 0 || b >= m || st[a][b])
                d = (d + 1) % 4;
            
            a = x + dx[d], b = y + dy[d];
            x = a, y = b;
        }
        return res;
    }
};
```



## 栈的压入/弹出序列

> https://www.acwing.com/problem/content/description/40/

直接用一个栈真实模拟即可, 如果最后模拟完压入弹出序列之后栈为空, 那么就是一个合法的压入/弹出序列.

```cpp
class Solution {
public:
    bool isPopOrder(vector<int> pushV,vector<int> popV) {
        
        int n = pushV.size(), m = popV.size();
        if (n != m) return false;
        
        stack<int> stk;
        int index = 0;
        
        for (auto x : pushV)
        {
            stk.push(x);
            while (stk.size() && popV[index] == stk.top())
            {
                stk.pop();
                index ++;
            }
        }
        return stk.empty();
    }
};
```



## 验证IP地址

> https://leetcode.cn/problems/validate-ip-address/

将判断逻辑进行拆分再判断即可.

```cpp
class Solution {
public:

    vector<string> split(string str, char t) {
        str += t;
        vector<string> items;
        for (int i = 0; i < str.size(); i ++) {
            int j = i;
            string item;
            while (str[j] != t) item += str[j ++];
            items.push_back(item);
            i = j;
        }
        return items;
    }

    string check_ipv4(string ip) {
        vector<string> ip_items = split(ip, '.');
        // Check Length
        if (ip_items.size() != 4) return "Neither";
        // Check each component
        for (auto item: ip_items) {
            // check item length
            if (item.size() == 0 || item.size() > 3) return "Neither";
            // check item character
            for (auto c: item) {
                if (c < '0' || c > '9') return "Neither";
            }
            // check leading zeros
            if (item.size() > 1 && item[0] == '0') return "Neither";
            // check number range
            int num = stoi(item);
            if (num > 255) return "Neither";
        }
        return "IPv4";
    }

    bool check(char c) {
        if (c >= '0' && c <= '9') return true;
        if (c >= 'a' && c <= 'f') return true;
        if (c >= 'A' && c <= 'F') return true;
        return false;
    }

    string check_ipv6(string ip) {
        vector<string> ip_items = split(ip, ':');
      	// check Length
        if (ip_items.size() != 8) return "Neither";
				// check each component
        for (auto item: ip_items) {
          	// check component length
            if (item.size() < 1 || item.size() > 4) return "Neither";
          	// check component character
            for (auto c: item) {
                if (!check(c)) return "Neither";
            }
        }
        return "IPv6";
    }

    string validIPAddress(string ip) {
        if (ip.find(".") != -1 && ip.find(":") != -1) return "Neither";
        if (ip.find(".") != -1) return check_ipv4(ip);
        if (ip.find(":") != -1) return check_ipv6(ip);
        return "Neither";
    }
};
```



## Z字形变换

> https://leetcode.cn/problems/zigzag-conversion/description/

这个题本质上是一个找规律题, 观察如下N字形变换, 其中$n = 4$:

```
0      6       12
1    5 7    11 13
2  4   8  10   14
3      9       15
```

* 对于第一行($i = 0$)来说, 这个序列是一个等差数列, 首项是0, 公差是$2n - 2$.
* 对于最后一行 ($i = n - 1$)来说, 这个序列也是一个等差数列, 首项是$n - 1$, 公差是$2n - 2$.
* 对于中间的若干行 ($0 < i < n - 1$)来说, 由两部分组成:
  * 第一部分是首项是$i$, 公差是$2n - 2$的等差数列.
  * 第二部分是首项是$2n - 2 - i$, 公差是$2n - 2$的等差数列.

* 其中$n = 1$的情况要特殊判断, 因为如果$n = 1$, 那么公差就变成0了, 按照循环输出的话, 就会变成死循环.

```cpp
class Solution {
public:
    string convert(string s, int n) {
        string res;
        if (n == 1) return s;
				
      	// i是枚举行数
        for (int i = 0; i < n; i ++) {
            if (i == 0 || i == n - 1) {
                for (int j = i; j < s.size(); j += 2 * n - 2) res += s[j];
            }
            else {
                for (int j = i, k = 2 * n - 2 - i; j < s.size() || k < s.size(); j += 2 * n - 2, k += 2 * n - 2) {
                    if (j < s.size()) res += s[j];
                    if (k < s.size()) res += s[k];
                }
            }
        }

        return res;
    }
};
```



## 整数反转

> https://leetcode.cn/problems/reverse-integer/description/

* 注意C++中的负数取模运算:
  * 结果是负数.
  * 绝对值等于这个负数的绝对值取模.

```cpp
class Solution {
public:
    int reverse(int x) {
        int r = 0;

        while (x) {
          // 10 * r + x % 10 > INT_MAX
          // 不用乘法, 因为溢出
            if (r > 0 && r > (INT_MAX - x % 10) / 10) return 0;
            if (r < 0 && r < (INT_MIN - x % 10) / 10) return 0;

            r = 10 * r + x % 10;
            x /= 10;
        }
        return r;
    }
};
```



## 字符串转整数

> https://leetcode.cn/problems/string-to-integer-atoi/

```cpp
class Solution {
public:
    int myAtoi(string s) {
        int n = s.size();
				
      // 过滤空格
        int k = 0;
        while (k < n && s[k] == ' ') k ++;
        if (k == n) return 0;
			// 判断符号
        bool minus = false;
        if (s[k] == '-') minus = true;
        if (s[k] == '-' || s[k] == '+') k ++;

        int res = 0;
        while (k < n && s[k] >= '0' && s[k] <= '9') {
            int x = s[k] - '0';
            if (minus) x = -x;
            if (!minus && res > (INT_MAX - x) / 10) return INT_MAX;
            if (minus && res < (INT_MIN - x) / 10) return INT_MIN;
 
            res = 10 * res + x;
            k ++;
        }
        return res;
    }
};
```



## 回文数

> https://leetcode.cn/problems/palindrome-number/

这个题如果求这个数的逆序, 很可能会超过`INT_MAX`, 因此先求后一半的逆序, 然后反过来比较前面.

```cpp
class Solution {
public:
    bool isPalindrome(int x) {

        // 负数/最后一位是0(除了0)都不是
        if (x < 0 || x && x % 10 == 0) return false;
        
        // 算后一半的逆序, 判断和前一半是否相等
        int s = 0;

        while (s <= x) {
            s = 10 * s + x % 10;
            // 长度是奇数/偶数
            if (s == x || s == x / 10) return true;
            x /= 10;
        }
        return false;

    }
};
```

