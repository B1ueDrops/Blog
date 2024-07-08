---
title: 深度优先搜索
categories: 算法
---



[toc]



## 矩阵中的路径

> https://www.acwing.com/problem/content/description/21/

遍历每一个起点, 从起点开始搜索即可.

```cpp
class Solution {
public:

    bool dfs(vector<vector<char>> &matrix, string &str, int x, int y, int u) {
        if (matrix[x][y] != str[u]) return false;
        if (u == str.size() - 1) return true;
        
        int dx[] = {-1, 0, 1, 0};
        int dy[] = {0, 1, 0, -1};
        
        char t = matrix[x][y];
      	/* 用*把遍历过的(x, y)标记一下 */
        matrix[x][y] = '*';
        for (int i = 0; i < 4; i ++) {
            int a = x + dx[i], b = y + dy[i];
            if (a >= 0 && a < matrix.size() && b >= 0 && b < matrix[0].size() && matrix[a][b] != '*') {
                if (dfs(matrix, str, a, b, u + 1)) return true;
            }
        }
        matrix[x][y] = t;
        return false;
    }
    bool hasPath(vector<vector<char>>& matrix, string &str) {
        int n = matrix.size();
        if (!n) return false;
        int m = matrix[0].size();
        if (!m) return false;
        
        /* 枚举每一个起点 */
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j ++) {
                if (dfs(matrix, str, i, j, 0)) return true;
            }
        }
        return false;
    }
};
```



## 解数独

> https://leetcode.cn/problems/sudoku-solver/

```cpp
class Solution {
public:
    /* row[i][j]表示第i行j数字是否出现 */
    bool row[9][9], col[9][9], cell[3][3][9];
    void solveSudoku(vector<vector<char>>& board) {

        memset(row, 0, sizeof row);
        memset(col, 0, sizeof col);
        memset(cell, 0, sizeof cell);

        for (int i = 0; i < 9; i ++) {
            for (int j = 0; j < 9; j ++) {
                if (board[i][j] == '.') continue;
                int t = board[i][j] - '1';
                row[i][t] = col[j][t] = cell[i / 3][j / 3][t] = true;
            }
        }

        dfs(board, 0, 0);
    }

    bool dfs(vector<vector<char>> &board, int x, int y) {
        if (y == 9) x ++, y = 0;
        if (x == 9) return true;

        if (board[x][y] != '.') return dfs(board, x, y + 1);

        for (int i = 0; i < 9; i ++) {
            if (!row[x][i] && !col[y][i] && !cell[x / 3][y / 3][i]) {
                board[x][y] = i + '1';
                row[x][i] = col[y][i] = cell[x / 3][y / 3][i] = true;
                if (dfs(board, x, y + 1)) return true;
                board[x][y] = '.';
                row[x][i] = col[y][i] = cell[x / 3][y / 3][i] = false;
            }
        }
        return false;
    }
};
```



## 组合总和

> https://leetcode.cn/problems/combination-sum/description/

遍历所有物品, 枚举每一个物品选择的数量即可.

```cpp
class Solution {
public:
    vector<int> path;
    vector<vector<int>> ans;

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        dfs(candidates, 0, target);
        return ans;
    }

    void dfs(vector<int> &candidates, int u, int target) {
        if (target == 0) {
            ans.push_back(path);
            return ;
        }
        if (u == candidates.size()) return ;

        for (int i = 0; candidates[u] * i <= target; i ++) {
            dfs(candidates, u + 1, target - candidates[u] * i);
            path.push_back(candidates[u]);
        }

        for (int i = 0; candidates[u] * i <= target; i ++) {
            path.pop_back();
        }
    }
};
```



## 组合总和II

> https://leetcode.cn/problems/combination-sum-ii/

在上一个题目的基础上, 数组有重复元素, 并且每个物品只能用一次, 重复元素可以采用先将数组排序, 然后用双指针算法定位重复元素的范围, 然后枚举重复元素选几个.

```cpp
class Solution {
public:
    vector<int> path;
    vector<vector<int>> ans;

    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        dfs(candidates, 0, target);
        return ans;
    }

    void dfs(vector<int> &candidates, int u, int target) {
        if (target == 0) {
            ans.push_back(path);
            return ;
        }
        if (u == candidates.size()) return ;

        int k = u + 1;
        while (k < candidates.size() && candidates[k] == candidates[u]) k ++;

        int cnt = k - u;

        for (int i = 0; i <= cnt && candidates[u] * i <= target; i ++) {
            dfs(candidates, k, target - candidates[u] * i);
            path.push_back(candidates[u]);
        }
        for (int i = 0; i <= cnt && candidates[u] * i <= target; i ++) {
            path.pop_back();
        }
        
    }
};
```



## n皇后

> https://www.acwing.com/problem/content/845/

```cpp
#include <iostream>

using namespace std;

const int N = 10;

int n;
char g[N][N];
bool col[N], dg[N * 2], udg[N * 2];

/* u是每一行 */
void dfs(int u)
{
    if (u == n)
    {
        for (int i = 0; i < n; i ++) puts(g[i]);
        puts("");
        return ;
    }
    
    for (int i = 0; i < n; i ++)
    {
        if (!col[i] && !dg[u + i] && !udg[n + u - i])
        {
            g[u][i] = 'Q';
            col[i] = dg[u + i] = udg[n + u - i] = true;
            dfs(u + 1);
            g[u][i] = '.';
            col[i] = dg[u + i] = udg[n + u - i] = false;
        }
    }
}


int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++)
        for (int j = 0; j < n; j ++)
            g[i][j] = '.';
            
    dfs(0);
    return 0;
}
```



## n皇后的方案数

> https://leetcode.cn/problems/n-queens-ii/

和上一个题n皇后做法完全一样, 不用记录方案, 只需要中途统计一下数量即可.

```cpp
class Solution {
public:
    int n;
    vector<bool> col, dg, udg;

    int totalNQueens(int _n) {
        n = _n;
        col = vector<bool>(n);
        dg = vector<bool>(n * 2);
        udg = vector<bool>(n * 2);
        
        return dfs(0);
    }

    int dfs(int u) {
        if (u == n) return 1;

        int res = 0;
        for (int i = 0; i < n; i ++)
        {
            if (!col[i] && !dg[u + i] && !udg[n + i - u])
            {
                col[i] = dg[u + i] = udg[n + i - u] = true;
                res += dfs(u + 1);
                col[i] = dg[u + i] = udg[n + i - u] = false;
            }
        }
        return res;
    }
};
```



## 树的重心

> https://www.acwing.com/problem/content/848/

首先, 这个图有$n$个点, 有$n - 1$条边, 那么肯定是一个树.

可以用递归的方法, 求出一个节点$u$的所有相邻节点连通块中点的个数, 那么在递归过程中求出所有连通块点数量的最大值, 就可以解决问题.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 100010, M = N * 2;

int n;
int h[N], e[M], ne[M], idx;
bool st[N];

void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int ans = 0x3f3f3f3f;

/* 这个函数返回节点u周围所有子树的节点之和, 包含节点u */
int dfs(int u)
{
    st[u] = true;
    
    int sum = 1, res = 0;
    
    for (int i = h[u]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (!st[j])
        {
            int s = dfs(j);
            sum += s;
            res = max(res, s);
        }
    }
    res = max(res, n - sum);
    ans = min(ans, res);
    
    return sum;
}

int main()
{
    cin >> n;
    memset(h, -1, sizeof h);
    
    for (int i = 0; i < n - 1; i ++)
    {
        int a, b;
        cin >> a >> b;
        add(a, b), add(b, a);
    }
    
    dfs(1);
    
    cout << ans << endl;
    return 0;
}
```



## 排列组合枚举

### 排列型枚举I

> https://www.acwing.com/problem/content/844/

$1$到$n$按照字典序输出所有排列.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 10;

int n;
bool st[N];
int path[N];

void dfs(int u) {
    if (u == n) {
        for (int i = 0; i < n; i ++) printf("%d ", path[i]);
        puts("");
        return ;
    }
    for (int i = 1; i <= n; i ++) {
        if (st[i]) continue;
        path[u] = i;
        st[i] = true;
        dfs(u + 1);
        st[i] = false;
    }
}

int main() {
    cin >> n;
    dfs(0);
    return 0;
}
```



同类型的题: https://leetcode.cn/problems/permutations/

给定一个不包含重复元素的数组, 输出所有可能的排列方式.

```cpp
class Solution {
public:
    vector<bool> st;
    vector<int> path;
    vector<vector<int>> ans;

    vector<vector<int>> permute(vector<int>& nums) {
        int n = nums.size();
        st.resize(n);
        path.resize(n);

        dfs(nums, 0);
        return ans;
    }

    void dfs(vector<int> &nums, int u)
    {
        if (u == nums.size())
        {
            ans.push_back(path);
            return ;
        }

        for (int i = 0; i < nums.size(); i ++)
        {
            if (!st[i])
            {
                st[i] = true;
                path[u] = nums[i];
                dfs(nums, u + 1);
                st[i] = false;
            }
        }
    }
};
```



### 排列型枚举II

> https://leetcode.cn/problems/permutations-ii/
>
> https://www.acwing.com/problem/content/description/47/

给定一个可能包含重复元素的数组, 输出所有可能的排列方式.

```cpp
class Solution {
public:
    vector<int> path;
    vector<vector<int>> ans;
    
    vector<vector<int>> permutation(vector<int>& nums) {
        
        int n = nums.size();
        if (n == 0) return ans;
        path.resize(n);
        sort(nums.begin(), nums.end());
        dfs(nums, 0, 0, 0);
        return ans;
    }
    /* 这个函数用来描述nums[u]放到path的哪个坑上 */
    void dfs(vector<int> &nums, int start, int state, int u) {
        if (u == nums.size()) {
            ans.push_back(path);
            return ;
        }
        if (!u || nums[u] != nums[u - 1]) start = 0;
        
        for (int i = start; i < nums.size(); i ++) {
            if (!(state >> i & 1)) {
                path[i] = nums[u];
                dfs(nums, i + 1, state + (1 << i), u + 1);
            }
        }
    }
};
```



### 组合型枚举I

> https://www.acwing.com/problem/content/description/95/

从$1$到$n$这些数中选取$m$个, 枚举所有的选择方式, 并且按照字典序输出.

```cpp
#include <iostream>
using namespace std;

const int N = 30;

int n, m;
bool st[N];


void dfs(int u, int cnt) {
    
    if (cnt == m) {
        for (int i = 1; i <= n; i ++)
            if (st[i]) printf("%d ", i);
        puts("");
        return ;
    }
    
    if (u > n) return ;
    
    st[u] = true;
    
    dfs(u + 1, cnt + 1);
    
    st[u] = false;
    
    dfs(u + 1, cnt);
}


int main() {
    cin >> n >> m;
    dfs(1, 0);
    return 0;
}
```



### 组合型枚举II

> https://www.acwing.com/problem/content/1575/

给定一个长度为$n$的, 可包含重复数字的数组, 随机选取$m$个数字, 输出可能的方案.

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 30;

int n, m;
int a[N];
bool st[N];

void dfs(int u, int cnt) {
    if (cnt == m) {
        for (int i = 0; i < n; i ++) {
            if (st[i]) printf("%d ", a[i]);
        }
        puts("");
        return ;
    }
    
    if (u == n) return ;
    
  	/* 枚举重复元素选择多少个 */
    int k = u;
    
    while (k < n && a[k] == a[u]) {
        st[k ++ ] = true;
        cnt ++;
    }
  
  // 重复元素全选
    dfs(k, cnt);
    
  // 重复元素选择若干个
    for (int i = k - 1; i >= u; i --) {
        st[i] = false;
        cnt --;
        dfs(k, cnt);
    }
}


int main() {
    
    cin >> n >> m;
    for (int i = 0; i < n; i ++) cin >> a[i];
    sort(a, a + n);
    dfs(0, 0);
    
    return 0;
}
```



### 电话号码的字母组合

> https://leetcode.cn/problems/letter-combinations-of-a-phone-number/

```cpp
class Solution {
public:
    vector<string> res;
    string str[10] = {
        "", "", "abc", "def",
        "ghi", "jkl", "mno",
        "pqrs", "tuv", "wxyz"
    };
    vector<string> letterCombinations(string digits) {
            if (digits.size() == 0) return res;
            dfs(digits, 0, "");
            return res;
    }
    void dfs(string &digits, int u, string path) {
        if (u == digits.size()) res.push_back(path);
        else {
            for (auto c : str[digits[u] - '0']) {
                dfs(digits, u + 1, path + c);
            } 
        }
    }
};
```
