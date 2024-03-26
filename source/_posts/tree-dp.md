---
title: 树形动态规划系列
categories: 算法
mathjax: true
---



## 没有上司的舞会

> https://www.acwing.com/problem/content/287/

设$f[u][0]$表示以$u$为根的子树中, 不选择$u$的所有方案中, happy指数之和的最大值, $f[u][1]$表示选择$u$.

那么状态转移方程是:

* $f[u][0] = \sum_j max(f[j][0], f[j][1])$, 其中$j$是$u$​的子节点.
* $f[u][1] = happy[u] + \sum_j f[j][0]$

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 6010;

int n;
int happy[N];
int f[N][2];
bool has_father[N];

int h[N], e[N], ne[N], idx;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void dfs(int u) {
    
    f[u][1] += happy[u];
    
    for (int i = h[u]; ~i; i = ne[i]) {
        
        int j = e[i];
        dfs(j);
        
        f[u][0] += max(f[j][0], f[j][1]);
        f[u][1] += f[j][0];
    }
}

int main() {
    
    cin >> n;
    memset(h, -1, sizeof h);
    
    for (int i = 1; i <= n; i ++) cin >> happy[i];
    
    for (int i = 1; i <= n - 1; i ++) {
        int a, b;
        cin >> a >> b;
        add(b, a), has_father[a] = true;
    }
    
    int root;
    for (int i = 1; i <= n; i ++)
        if (!has_father[i]) {
            root = i;
            break;
        }
    
    dfs(root);
    cout << max(f[root][0], f[root][1]) << endl;
    
    return 0;
}
```

同类题: https://leetcode.cn/problems/house-robber-iii/

```cpp
class Solution {
public:
    unordered_map<TreeNode*, unordered_map<int, int>> f;

    int rob(TreeNode * root) {
        dfs(root);
        return max(f[root][0], f[root][1]);    
    }

    void dfs(TreeNode * root) {
        if (!root) return ;
        dfs(root->left);
        dfs(root->right);
        f[root][0] = max(f[root->left][0], f[root->left][1]) + max(f[root->right][0], f[root->right][1]);
        f[root][1] = root->val + f[root->left][0] + f[root->right][0];
    }
};
```

