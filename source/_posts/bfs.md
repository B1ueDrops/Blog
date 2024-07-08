---
title: 广度优先搜索
categories: 算法
mathjax: true
---

[toc]

## 走迷宫

> https://www.acwing.com/problem/content/description/846/

```cpp
#include <iostream>
#include <cstring>
#include <queue>

using namespace std;

const int N = 110;

typedef pair<int, int> PII;

int n, m;
int g[N][N], dist[N][N];

int dx[4] = {-1, 0, 1, 0};
int dy[4] = {0, 1, 0, -1};

int bfs()
{
    memset(dist, -1, sizeof dist);
    dist[0][0] = 0;
    
    queue<PII> q;
    q.push({0, 0});
    
    while (q.size())
    {
        auto t = q.front(); q.pop();
        
        for (int i = 0; i < 4; i ++)
        {
            int x = t.first + dx[i], y = t.second + dy[i];
            if (x >= 0 && x < n && y >= 0 && y < m && dist[x][y] == -1 && g[x][y] == 0)
            {
                dist[x][y] = dist[t.first][t.second] + 1;
                q.push({x, y});
            }
        }
    }
    return dist[n - 1][m - 1];
}

int main()
{
    cin >> n >> m;
    for (int i = 0; i < n; i ++)
        for (int j = 0; j < m; j ++)
            cin >> g[i][j];
    
    cout << bfs() << endl;
    return 0;
}
```



## 八数码

> https://www.acwing.com/problem/content/847/



```cpp
#include <iostream>
#include <queue>
#include <unordered_map>

using namespace std;

unordered_map<string, int> dist;

int dx[] = {-1, 0, 1, 0};
int dy[] = {0, 1, 0, -1};

int bfs(string start)
{
    string end = "12345678x";
    
    queue<string> q;
    q.push(start);
    dist[start] = 0;
    
    while (q.size())
    {
        string t = q.front(); q.pop();
        int distance = dist[t];
        
        if (t == end) return dist[t];
        int k = t.find('x');
        
        int x = k / 3, y = k % 3;
        
        for (int i = 0; i < 4; i ++)
        {
            int a = x + dx[i], b = y + dy[i];
            if (a >= 0 && a < 3 && b >= 0 && b < 3)
            {
                swap(t[3 * a + b], t[k]);
                if (!dist.count(t))
                {
                    dist[t] = distance + 1;
                    q.push(t);
                }
                swap(t[3 * a + b], t[k]);
            }
        }
    }
    return -1;
}

int main()
{
    string str = "";
    char c;
    
    while (cin >> c) str += c;
    
    cout << bfs(str) << endl;
    
    return 0;
}
```



## 图中点的层次

> https://www.acwing.com/problem/content/description/849/

当图中边权都是1时, 可以用bfs求最短距离.

```cpp
#include <iostream>
#include <cstring>
#include <queue>

using namespace std;

const int N = 100010;

int n, m;
int h[N], e[N], ne[N], idx;
int dist[N];

void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int bfs()
{
    memset(dist, -1, sizeof dist);
    dist[1] = 0;
    
    queue<int> q;
    q.push(1);
    
    while (q.size())
    {
        auto t = q.front(); q.pop();
        
        for (int i = h[t]; ~i; i = ne[i])
        {
            int j = e[i];
            if (dist[j] != -1) continue;
            dist[j] = dist[t] + 1;
            q.push(j);
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
        int a, b;
        cin >> a >> b;
        add(a, b);
    }
    cout << bfs() << endl;
    return 0;
}
```



## 机器人的运动范围

> https://www.acwing.com/problem/content/description/22/

注意题目中“不能进入行坐标和列坐标的数位之和大于 k的格子”, 是数位之和, 不是行坐标和列坐标之和.

```cpp
class Solution {
public:
    
    int get_sum(int x)
    {
        int cnt = 0;
        while (x)
        {
            cnt += x % 10;
            x /= 10;
        }
        return cnt;
    }
    int movingCount(int threshold, int rows, int cols)
    {
        typedef pair<int, int> PII;
        
        int n = rows, m = cols, k = threshold;
        if (!n || !m) return 0;
        if (!k) return 1;
        
        int dx[] = {-1, 0, 1, 0};
        int dy[] = {0, 1, 0, -1};
        
        queue<PII> q;
        vector<vector<bool>> st(n, vector<bool>(m, false));
        int cnt = 1;
        
        q.push({0, 0});
        st[0][0] = true;
        
        while (q.size())
        {
            auto t = q.front();
            q.pop();
            
            int x = t.first, y = t.second;
            for (int i = 0; i < 4; i ++)
            {
                int a = x + dx[i], b = y + dy[i];
                if (a >= 0 && a < n && b >= 0 && b < m && !st[a][b] && get_sum(a) + get_sum(b) <= k)
                {
                    st[a][b] = true;
                    cnt ++;
                    q.push({a, b});
                }
            }
            
        }
        return cnt;
    }
};
```

