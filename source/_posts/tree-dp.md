---
title: 树形DP
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



## 皇宫看守

> https://www.acwing.com/problem/content/1079/

* 状态表示:

  * $f(i, 0)$: 点$i$没有放警卫, 并且被父节点看到的所有方案的最小花费.
  * $f(i, 1)$: 点$i$没有放警卫, 并且被子节点看到的所有方案的最小花费.
  * $f(i, 2)$: 在点$i$上放警卫的所有方案的最小花费.

* 状态计算:

  * $f(i, 0) = \sum_j\ min\{f(j, 1), f(j, 2)\}$

    * 首先, 点$i$没有放警卫, 因此点$i$的子节点$j$一定不会被父节点看到, 也就是$f(j, 0)$没有.
    * 剩下的子节点的情况就可以不用在乎, 直接取`min`.

  * $f(i, 2) = \sum_j\ min\{f(j, 0), f(j, 1), f(j, 2)\}$

  * 对于$f(i, 1)$, 枚举所有子节点$k$:

    * 假设在$k$上放了警卫, 那么再枚举其他节点$j$, 取$f(j, 1)$和$f(j, 2)$的最小值.

    $$
    f(i, 1) = min_k\{f(k, 2) + min_{j\neq k}\{f(j, 1), f(j, 2)\} \}
    $$

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 1510;

int n;
bool st[N];
int h[N], e[N], ne[N], w[N], idx;

int f[N][3];

void dfs(int u) {

    // 如果我要在本节点放东西
    f[u][2] = w[u];

    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        // 求子节点j的f[j][0], f[j][1], f[j][2]
        dfs(j);
        f[u][0] += min(f[j][1], f[j][2]);
        f[u][2] += min(min(f[j][0], f[j][1]), f[j][2]);
    }

    f[u][1] = 0x3f3f3f3f;
    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        // f[u][0]记录了所有子节点j的min(f[j][1], f[j][2])之和
        // f[u][0] - min(f[j][1], f[j][2])就代表除了j之外的所有点min(f[j][1], f[j][2])的和
        f[u][1] = min(f[u][1], f[j][2] + f[u][0] - min(f[j][1], f[j][2]));
    }
}

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int main() {

    memset(h, -1, sizeof h);
    cin >> n;

    for (int i = 0; i < n; i ++) {
        int id, cost, m;
        cin >> id >> cost >> m;

        w[id] = cost;
        while (m --) {
            int ver;
            cin >> ver;
            add(id, ver);
            st[ver] = true;
        }
    }

    // 求根节点
    int root = 1;
    while (st[root]) root ++;

    dfs(root);

    cout << min(f[root][1], f[root][2]) << endl;

    return 0;
}
```

同类题: https://leetcode.cn/problems/binary-tree-cameras/

```cpp
class Solution {
public:
    const int INF = 0x3f3f3f3f;

    vector<int> dfs(TreeNode *root) {
        // 三个元素分别是0状态, 1状态, 2状态
        if (!root) return { 0, 0, INF };
        auto l = dfs(root->left), r = dfs(root->right);

        return {
            min(l[1], l[2]) + min(r[1], r[2]),
            min( l[2] + min(r[1], r[2]) , r[2] + min(l[1], l[2])),
            // 2状态, 自己当前位置要放一个摄像头
            min(l[0], min(l[1], l[2])) + min(r[0], min(r[1], r[2])) + 1
        };
    }
    int minCameraCover(TreeNode* root) {
        auto f = dfs(root);
        return min(f[1], f[2]);
    }
};
```

