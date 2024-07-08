---
title: 图论
categories: 算法
mathjax: true
---

[toc]

## 拓扑排序

> https://www.acwing.com/problem/content/description/850/

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 100010;

int n, m;
int h[N], e[N], ne[N], d[N], q[N], idx;

void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

bool topsort()
{
    int hh = 0, tt = -1;
    
    for (int i = 1; i <= n; i ++)
        if (!d[i]) q[++ tt] = i;
    
    while (hh <= tt)
    {
        int t = q[hh ++];
        
        for (int i = h[t]; ~i; i = ne[i])
        {
            int j = e[i];
            d[j] --;
            if (!d[j]) q[ ++tt] = j;
        }
    }
    
    return tt == n - 1;
}

int main()
{
    cin >> n >> m;
    memset(h, -1, sizeof h);
    
    while (m --)
    {
        int a, b;
        cin >> a >> b;
        d[b] ++;
        add(a, b);
    }
    
    
    if (topsort())
        for (int i = 0; i < n; i ++) cout << q[i] << ' ';
    else cout << -1 << endl;
    
    return 0;
}
```



## 朴素Dijkstra算法

> https://www.acwing.com/problem/content/851/

Dijkstra算法用来求边权大于0的图中的最短路, 可以有环.

朴素版Dijkstra算法的时间复杂度是$O(n^2)$, 其中$n$是点数, 适合稠密图(边的数量远大于点).

注意, Dijkstra算法需要用到邻接矩阵, 邻接矩阵的初始化和加边如下:

```cpp
memset(g, 0x3f, sizeof g);
for (int i = 1; i <= n; i ++) g[i][i] = 0;
for (int i = 0; i < m; i++)
{
    int a, b, c;
    cin >> a >> b >> c;
    g[a][b] = min(g[a][b], c);
}
```

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 510;

int n, m;
int g[N][N];

bool st[N];
int dist[N];

int dijkstra()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    
    for (int i = 0; i < n; i ++)
    {
        int t = -1;
        /* 选取没被更新过, 但是距离点1最小的点, 用这个点更新其他人 */
        for (int j = 1; j <= n; j ++)
            if (!st[j] && (t == -1 || dist[t] > dist[j])) t = j;
            
        st[t] = true;
        
        for (int j = 1; j <= n; j ++)
            dist[j] = min(dist[j], dist[t] + g[t][j]);
    }
    
    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}


int main()
{
    cin >> n >> m;
    memset(g, 0x3f, sizeof g);
    for (int i = 1; i <= n; i ++) g[i][i] = 0;
    
    while (m --)
    {
        int a, b, c;
        cin >> a >> b >> c;
        g[a][b] = min(g[a][b], c);
    }
    
    cout << dijkstra() << endl;
    return 0;
}
```



## 堆优化Dijkstra算法

> https://www.acwing.com/problem/content/852/

时间复杂度是$O(mlogn)$.

```cpp
#include <iostream>
#include <cstring>
#include <queue>

using namespace std;

const int N = 150010;

typedef pair<int, int> PII;

int n, m;
int h[N], e[N], ne[N], w[N], idx;
int dist[N];
bool st[N];

void add(int a, int b, int c)
{
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx ++;
}

int dijkstra()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    
    priority_queue<PII, vector<PII>, greater<PII>> q;
    q.push({0, 1});
    
    while (q.size())
    {
        auto t = q.top(); q.pop();
        
        int distance = t.first, ver = t.second;
        
        /* 为什么要加上这一句? 因为有自环的存在, 有了这一句, 子环的代码就不会向下运行 */
        if (st[ver]) continue;
        st[ver] = true;
        
        for (int i = h[ver]; ~i; i = ne[i])
        {
            int j = e[i];
            if (st[j]) continue;
            if (dist[j] > distance + w[i])
            {
                dist[j] = distance + w[i];
                q.push({dist[j], j});
            }
        }
    }
    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}

int main()
{
    cin >> n >> m;
    memset(h, -1, sizeof h);
    
    while (m --)
    {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
    }
    
    cout << dijkstra() << endl;
    return 0;
}
```



## Bellman-Ford算法

> https://www.acwing.com/problem/content/855/

用Bellman-Ford算法可以解决如下最短路问题:

* 边权有负数
* 要求最短路只经过$k$条边

时间复杂度是$O(mk)$

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 510, M = 10010;

int n, m, k;
int dist[N], backup[N];

struct Edge
{
    int a, b, c;
} edges[M];

int bellman_ford()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    /* 更新k次 */
    for (int i = 0; i < k; i ++)
    {
        memcpy(backup, dist, sizeof backup);
        /* 遍历所有m条 */
        for (int j = 0; j < m; j ++)
        {
            auto e = edges[j];
            dist[e.b] = min(dist[e.b], backup[e.a] + e.c);
        }
    }
    return dist[n];
}

int main()
{
    cin >> n >> m >> k;
    for (int i = 0; i < m; i ++)
    {
        int a, b, c;
        cin >> a >> b >> c;
        edges[i] = {a, b, c};
    }
    
    int t = bellman_ford();
    
    /* 由于有负权边的存在, 0x3f3f3f3f可能被更新成一个小于0x3f3f3f3f的值, 所以判断无穷大的方法要改 */
    if (t > 0x3f3f3f3f / 2) puts("impossible");
    else cout << t << endl;
    
    return 0;
}
```



## SPFA算法求最短路

> https://www.acwing.com/problem/content/853/

SPFA是优化版的Bellman-Ford算法, 能够求带有负权边的最短路, 但是不能求带有负权回路的最短路.

时间复杂度是$O(km)$, 其中$k$是一个常数, 最坏时间复杂度是$O(nm)$, 

```cpp
#include <iostream>
#include <cstring>
#include <queue>

using namespace std;

const int N = 100010;

int n, m;
int h[N], e[N], ne[N], w[N], dist[N], idx;
bool st[N];

void add(int a, int b, int c)
{
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx ++;
}

int spfa()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    queue<int> q;
    q.push(1);
    st[1] = true;
    
    while (q.size())
    {
        auto t = q.front(); q.pop();
        
        st[t] = false;
        
        for (int i = h[t]; ~i ; i = ne[i])
        {
            int j = e[i];
            /* 被更新的点需要入队, 用被更新的点更新所有相邻的点, 然后把相邻的, 被更新的点放到堆中更新其他点, 
            但是其他点有可能已经在堆中了, 就不用再加入堆了 */
            if (dist[j] > dist[t] + w[i])
            {
                dist[j] = dist[t] + w[i];
                if (st[j]) continue;
                st[j] = true;
                q.push(j);
            }
        }
    }
    
    return dist[n];
}

int main()
{
    cin >> n >> m;
    memset(h, -1, sizeof h);
    while (m --)
    {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
    }
    
    int t = spfa();
    if (t == 0x3f3f3f3f) puts("impossible");
    else cout << t << endl;
    
    return 0;
}
```



## SPFA判断负权回路

> https://www.acwing.com/problem/content/description/854/

首先, 创建一个虚拟原点, 然后这个虚拟原点向周围的每一个点连接一个边权是0的边.

之后, 统计一下每个点入队次数, 每个点只要一入队, 就说明虚拟原点到这个点的最短路多了一条边, 也就是说, 每个点的入队次数, 等于虚拟原点到这个点最短路的边的条数, 如果这个条数大于等于$n$, 就说明存在负权回路.

```cpp
#include <iostream>
#include <cstring>
#include <queue>

using namespace std;

const int N = 2010, M = 10010;

int n, m;
int h[N], e[M], ne[M], w[M], cnt[N], dist[N], idx;
bool st[N];

void add(int a, int b, int c)
{
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx ++;
}

bool spfa()
{
    memset(dist, 0x3f, sizeof dist);
    queue<int> q;
    
    for (int i = 1; i <= n; i ++)
    {
        dist[i] = 0;
        st[i] = true;
        q.push(i);
    }
    
    while (q.size())
    {
        auto t = q.front(); q.pop();
        
        st[t] = false;
        
        for (int i = h[t]; ~i; i = ne[i])
        {
            int j = e[i];
            if (dist[j] > dist[t] + w[i])
            {
                dist[j] = dist[t] + w[i];
                if (!st[j])
                {
                    st[j] = true;
                    cnt[j] = cnt[t] + 1;
                    if (cnt[j] >= n) return true;
                    q.push(j);
                }
            }
        }
    }
    return false;
}

int main()
{
    cin >> n >> m;
    memset(h, -1, sizeof h);
    
    while (m --)
    {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
    }
    
    if (spfa()) puts("Yes");
    else puts("No");
    
    return 0;
}
```



## Floyd算法求最短路

> https://www.acwing.com/problem/content/description/856/

Floyd算法可以求任意两点之间的, 可以有负权边的最短路.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 210;

int n, m, k;
int g[N][N];

int main()
{
    cin >> n >> m >> k;
    memset(g, 0x3f, sizeof g);
    for (int i = 1; i <= n; i ++) g[i][i] = 0;
    
    while (m --)
    {
        int a, b, c;
        cin >> a >> b >> c;
        g[a][b] = min(g[a][b], c);
    }
    
    /* 注意这里k需要在外面 */
    for (int k = 1; k <= n; k ++)
        for (int i = 1; i <= n; i ++)
            for (int j = 1; j <= n; j ++)
                g[i][j] = min(g[i][j], g[i][k] + g[k][j]);
    
    while (k --)
    {
        int x, y;
        cin >> x >> y;
        if (g[x][y] > 0x3f3f3f3f / 2) puts("impossible");
        else cout << g[x][y] << endl;
    }
    return 0;
}
```



## Prim算法求最小生成树

> https://www.acwing.com/problem/content/860/

朴素版Prim算法用邻接矩阵存储图.

只适用于稠密图, 时间复杂度是$O(n^2)$, 如果不是稠密图就用Kruskal算法.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 510;

int n, m;
int g[N][N], dist[N];
bool st[N];

int prim()
{
    memset(dist, 0x3f, sizeof dist);
    int res = 0;
    
    for (int i = 0; i < n; i ++)
    {
        int t = -1;
        for (int j = 1; j <= n; j ++)
            if (!st[j] && (t == -1 || dist[t] > dist[j])) t = j;
        
        st[t] = true;
        /* 当所有点不连通的时候, 不存在生成树 */
        if (i && dist[t] == 0x3f3f3f3f) return 0x3f3f3f3f;
        if (i) res += dist[t];
        
        for (int j = 1; j <= n; j ++) dist[j] = min(dist[j], g[t][j]);
    }
    return res;
}

int main()
{
    cin >> n >> m;
    memset(g, 0x3f, sizeof g);
    for (int i = 1; i <= n; i ++) g[i][i] = 0;
    
    while (m --)
    {
        int a, b, c;
        cin >> a >> b >> c;
        /* 双向边 */
        g[a][b] = g[b][a] = min(g[a][b], c);
    }
    
    int t = prim();
    if (t == 0x3f3f3f3f) puts("impossible");
    else cout << t << endl;
    
    return 0;
}
```



## Kruskal算法求最小生成树

> https://www.acwing.com/problem/content/861/

Kruskal算法的时间复杂度包括两部分:

* 按照边权排序: $O(mlogm)$
* 遍历所有边, 每次遍历一条边都要判断两个顶点是否在一个并查集中, 时间复杂度是$O(\alpha m)$, 其中$\alpha$可以看做常数.

总的时间复杂度是$O(mlogm)$.

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010, M = N * 2;

int n, m;
int p[N];

struct Edge
{
    int a, b, w;
    bool operator < (const Edge &W) const
    {
        return w < W.w;
    }
} edges[M];

int find(int x)
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) p[i] = i;
    
    for (int i = 0; i < m; i ++)
        cin >> edges[i].a >> edges[i].b >> edges[i].w;
    
    sort(edges, edges + m);
    
    int res = 0, cnt = 0;
    
    for (int i = 0; i < m; i ++)
    {
        int a = edges[i].a, b = edges[i].b, w = edges[i].w;
        a = find(a), b = find(b);
        if (a != b)
        {
            p[a] = b;
            res += w;
            cnt ++;
        }
    }
    if (cnt < n - 1) puts("impossible");
    else cout << res << endl;
    
    return 0;
}
```



## 染色法判断二分图

> https://www.acwing.com/problem/content/description/862/

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 100010, M = N * 2;

int n, m;
int h[N], e[M], ne[M], idx;
int color[N];

void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

bool dfs(int u, int c)
{
    color[u] = c;
    for (int i = h[u]; ~i; i = ne[i])
    {
        int j = e[i];
        if (!color[j])
        {
            if (!dfs(j, 3 - c)) return false;
        }
        else if (color[j] == c) return false;
    }
    return true;
}

int main()
{
    cin >> n >> m;
    memset(h, -1, sizeof h);
    
    while (m --)
    {
        int a, b;
        cin >> a >> b;
        add(a, b), add(b, a);
    }
    
    bool flag = true;
    for (int i = 1; i <= n; i ++)
        if (!color[i] && !dfs(i, 1))
        {
            flag = false;
            break;
        }
        
    if (flag) puts("Yes");
    else puts("No");
    
    return 0;
}
```



## 匈牙利算法

> https://www.acwing.com/problem/content/863/

匈牙利算法用来求二分图的一个最大匹配.

* 二分图的匹配: 匹配就是在二分图中选若干条边, 如果是匹配, 那么选出的边都不共用一个顶点, 也就是不会出现脚踏两条船的情况.
* 最大匹配就是选出边的数量最大.

匈牙利算法的思路如下:

* 从二分图的一个集合中遍历所有男生.
* 然后看每一个男生看上的所有女生.
  * 如果看上的女生目前单身, 那么匹配成功.
  * 如果看上的女生目前不是单身, 那么找到这个女生当前的男朋友, 如果这个男朋友有下家, 那么就让这个男朋友找到下家, 自己霸占这个女生.

时间复杂度是$O(nm)$, 其中$n, m$是二分图两个部分的点的数量.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 510 * 2, M = 100010;

int n1, n2, m;
int h[N], e[M], ne[M], idx;

bool st[N];
int match[N];

void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}


bool find(int x)
{
    for (int i = h[x]; ~i; i = ne[i])
    {
        int j = e[i];
        if (!st[j])
        {
            st[j] = true;
            if (match[j] == 0 || find(match[j]))
            {
                /* 注意match数组的下标是女生, 值是男生 */
                match[j] = x;
                return true;
            }
        }
    }
    return false;
}

int main()
{
    cin >> n1 >> n2 >> m;
    memset(h, -1, sizeof h);
    
    while (m --)
    {
        int a, b;
        cin >> a >> b;
        add(a, b);
    }
    
    int res = 0;
    for (int i = 1; i <= n1; i ++)
    {
        memset(st, false, sizeof st);
        if (find(i)) res ++;
    }
    
    cout << res << endl;
    return 0;
}
```

