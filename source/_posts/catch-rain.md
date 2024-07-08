---
title: 接雨水问题
categories: 算法
mathjax: true
---



## 一维接雨水问题

> https://leetcode.cn/problems/trapping-rain-water/

维护两个数组:

* `l[i]`表示包含当前第`i`个元素, 和它的左边的所有元素的高度最大值.
* `r[i]`表示包含当前第`i`个元素, 和它的右边的所有元素的高度最大值.

那么第`i`个高度对于雨水的贡献就是`min(l[i], r[i]) - h[i]`, 直接累计即可.

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



## 二维接雨水问题

> https://leetcode.cn/problems/trapping-rain-water-ii/

首先需要明确, 对于一个格子$(i, j)$, 如何计算它能够达到的最大高度(贡献的雨水量就是这个最大高度和这个格子高度的差值), 假设这个最大高度定义为$f(i, j)$.

这个格子$(i, j)$有很多条路径通向边界, 对于每一条单独的路径而言, 它能达到的最大高度是这条路径上格子高度的最大值.

而对于所有路径而言, 就需要把这些所有路径的最大值再取一个最小值.

用数学公式来说的话:
$$
f(i, j) = min\{max(f(i-1, j), h(i, j)), max(f(i + 1, j), h(i, j)), max(f(i, j - 1), h(i, j)), max(f(i, j + 1), h(i, j)) \}
$$
但是这个是无法计算的, 因为状态转移不是拓扑图.

现在考虑这样一个问题, 对于一个边界格子, 它能够达到的最大高度是多少?

* 首先, 边界格子能够直接通向边界, 如果按这个路径来说, 它能够贡献的水量就是$h(i, j)$.
* 然后, 边界格子也可能通过走其他格子通向边界, 对于某一条具体的路径, 如果格子高度最大值$h \geqslant h(i, j)$, 那么最后一步取最小值时, 它最终贡献的水量还是$h(i, j)$.

因此, 边界格子$(i, j)$能够达到的最大高度就是$h(i, j)$.

那么现在的问题就是, 如何从边界格子$(i, j)$开始, 逐步计算内部格子能够贡献的雨水量$f(i, j)$.

直接用一个小根堆, 将边界能达到的最大高度放入小根堆, 每次枚举四个方向, 用这个最大高度更新内部格子的最大高度即可.

```cpp
class Solution {
public:

    struct Cell {
        /* h是Cell能达到的最大高度 */
        int h, x, y;
        bool operator < (const Cell &t) const {
            return h > t.h;
        }
    };
    int trapRainWater(vector<vector<int>>& h) {
            /* 判空 */
            if (h.empty() || h[0].empty()) return 0;

            int n = h.size(), m = h[0].size();
            
            /* 小根堆 */
            priority_queue<Cell> heap;
            /* 判重数组 */
            vector<vector<bool>> st(n, vector<bool>(m));

            /* 初始化左右边界 */
            for (int i = 0; i < n; i ++) {
                st[i][0] = st[i][m - 1] = true;
                heap.push({ h[i][0], i, 0 });
                heap.push({ h[i][m - 1], i, m - 1 });
            }

            /* 初始化上下边界 */
            for (int i = 1; i < m - 1; i ++) {
                st[0][i] = st[n - 1][i] = true;
                heap.push({ h[0][i], 0, i });
                heap.push({ h[n - 1][i], n - 1, i });
            }

            int res = 0;
            int dx[] = { -1, 0, 1, 0 };
            int dy[] = { 0, 1, 0, -1 };

            while (heap.size()) {
                auto t = heap.top();
                heap.pop();
                res += t.h - h[t.x][t.y];
                for (int i = 0; i < 4; i ++) {
                    int x = t.x + dx[i];
                    int y = t.y + dy[i];
                    if (x > 0 && x < n && y > 0 && y < m && !st[x][y]) {
                        st[x][y] = true;
                        heap.push({ max(h[x][y], t.h), x, y });
                    }
                }
            }
            return res;
    }
};
```



