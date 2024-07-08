---
title: 数学问题
categories: 算法
mathjax: true
---



## 排序不等式

> https://www.acwing.com/problem/content/description/915/

排序不等式的形式是: 顺序和 大于等于 乱序和 大于等于 逆序和

因此, 让等待时间$t$最小的人先接水, 整体的等待时间最小.

第一个人等待时间: 0

第二个人等待时间: $t_1$

第三个人等待时间: $t_1 + t_2$

...

第$n$个人等待时间: $t_1 + t_2 + ... + t_{n-1}$

等待时间之和是: $(n - 1)t_1 + (n - 2)t_2 + ... + t_{n-1}$, 正好是逆序和最小.

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

typedef long long LL;

int n;
int a[N];

int main()
{
    cin >> n;
    for (int i = 0; i < n; i++) cin >> a[i];
    
    sort(a, a + n);
    
    LL res = 0;
    
    for (int i = 0; i < n; i ++) res += a[i] * (n - i - 1);
    
    cout << res << endl;
    return 0;
}
```



## Huffman树

> https://www.acwing.com/problem/content/127/

给定一系列叶子结点, 以及这些节点对应的权值, 每个叶子结点的带权路径长度等于根节点到叶子结点的路径长度与叶子结点权值的乘积. 所有叶子结点带权路径长度之和最小的树就是Huffman树.

求Huffman树需要用到小根堆, 只需要每次把最小的两个数找到并且合并, 然后把合并后的数放到最小堆中, 重复这个过程就可以构造出Huffman树.

```cpp
#include <iostream>
#include <queue>

using namespace std;

const int N = 10010;

int n;
priority_queue<int, vector<int>, greater<int>> heap;

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++)
    {
        int x;
        cin >> x;
        heap.push(x);
    }
    
    int res = 0;
    
    while(heap.size() > 1)
    {
        int a = heap.top(); heap.pop();
        int b = heap.top(); heap.pop();
        res += a + b;
        heap.push(a + b);
    }
    cout << res << endl;
    return 0;
}
```



## 绝对值不等式

> https://www.acwing.com/problem/content/description/106/

假设货仓建在位置$x$, 那么这个货仓到所有商店的距离是 (假设$A_1 \leqslant A_2 \leqslant ... \leqslant A_n$):
$$
|x - A_1| + |x - A_2| + ... + |x - A_n|
$$
将这个式子进行分组:
$$
(|x - A_1| + |x - A_n|) + (|x - A_2| + |x - A_{n-1}|) + ...
$$
$x$只要建在$A_1$和$A_n$中间, 那么距离之和就是$A_n - A_1$, 那么这些如果把货仓建在所有商店的中间, 那么距离就是:
$$
(A_n - A_1) + (A_{n-1} - A_{2}) + ...
$$
$x$只要取中位数就可以.

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

typedef long long LL;

int n;
int a[N];

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++) cin >> a[i];
    
    sort(a, a + n);
    
    LL res = 0;
    for (int i = 0; i < n; i ++) res += abs(a[i] - a[n / 2]);
    
    cout << res << endl;
    return 0;
}
```



## 试除法判断质数

> https://www.acwing.com/problem/content/868/

时间复杂度是$O(\sqrt n)$.

```cpp
#include <iostream>

using namespace std;

/* 试除法判断质数 */
bool is_prime(int n)
{
    if (n < 2) return false;
    
    for (int i = 2; i <= n / i; i ++)
    {
        if (n % i == 0) return false;
    }
    return true;
}

int main()
{
    int n;
    cin >> n;
    
    while (n --)
    {
        int x;
        cin >> x;
        if (is_prime(x)) puts("Yes");
        else puts("No");
    }
    return 0;
}
```



## 质因数分解

> https://www.acwing.com/problem/content/869/

质因数分解的时间复杂度也是$O(\sqrt n)$

```cpp
#include <iostream>

using namespace std;

void divide(int n)
{
    for (int i = 2; i <= n / i; i ++)
    {
        int s = 0;
        while (n % i == 0)
        {
            n /= i;
            s ++;
        }
        if (s) printf("%d %d\n", i, s);
    }
    if (n > 1) printf("%d 1\n", n);
    puts("");
}

int main()
{
    int n;
    cin >> n;
    
    while (n --)
    {
        int x;
        cin >> x;
        divide(x);
    }
    return 0;
}
```

## 线性筛质数

> https://www.acwing.com/problem/content/870/

筛质数问题本质上是给定$n$, 求$[1, n]$中质数的个数.

线性筛质数的时间复杂度是$O(n)$, 它的思路是用对于每个合数, 需要用它的最小质因子筛掉它.

代码如下:

```cpp
/* 注意这里是n, 而不是n / i */
for (int i = 2; i <= n; i ++ )
{
    if (!st[i]) primes[cnt ++ ] = i;
    for (int j = 0; primes[j] <= n / i; j ++ )
    {
        st[primes[j] * i] = true;
      /* 由于primes[j]是从小到大枚举的, 因此当这个条件成立时, primes[j]一定是i的最小质因子, 同理primes[j]也是i * primes[j]的最小质因子 */
      /* 当这个条件不成立时, primes[j]一定比i的最小质因子海啸, primes[j] * i的最小质因子还是primes[j] */
        if (i % primes[j] == 0) break;
    }
}
```

这个筛法是$O(n)$的关键在于: 每一个合数只会被它的最小质因子筛去, 由于一个合数的最小质因子唯一, 所以每个合数只会被筛一次.

同类题: https://leetcode.cn/problems/count-primes/

## 试除法求约数

> https://www.acwing.com/problem/content/871/

求一个数$n$所有约数的时间复杂度是$O(\sqrt n)$.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

vector<int> get_divisors(int n)
{
    vector<int> divisors;
    /* 注意求约数从1开始, 最小的约数是1 */
    for (int i = 1; i <= n / i; i ++)
    {
        if (n % i == 0)
        {
            divisors.push_back(i);
            if (n / i != i) divisors.push_back(n / i);
        }
    }
    sort(divisors.begin(), divisors.end());
    return divisors;
}

int main()
{
    int n;
    cin >> n;
    
    while (n --)
    {
        int x;
        cin >> x;
        
        auto divisors = get_divisors(x);
        
        for (auto t : divisors)
            printf("%d ", t);
        puts("");
    }
    return 0;
}
```

## 求约数个数

> https://www.acwing.com/problem/content/872/

对于一个数$a$, 将它进行质因数分解: $a = p_{1}^{c_1}p_{2}^{c_2}...p_{n}^{c_n}$

那么约数个数就是$(c_1 + 1)(c_2 + 1)...(c_n + 1)$

```cpp
#include <iostream>
#include <unordered_map>

using namespace std;

const int mod = 1e9 + 7;

typedef long long LL;

/* 将一个质因子映射到它的次幂 */
unordered_map<int, int> prime;

int main()
{
    int n;
    cin >> n;
    
    while (n --)
    {
        int x;
        cin >> x;
        
        for (int i = 2; i <= x / i; i ++)
        {
            while (x % i == 0)
            {
                x /= i;
                prime[i] ++;
            }
        }
        if (x > 1) prime[x] ++;
    }
    
    LL res = 1;
    for (auto p: prime) res = res * (p.second + 1) % mod;
    
    cout << res << endl;
    
    return 0;
}
```



## 求约数之和

> https://www.acwing.com/problem/content/873/

对于一个数$a$, 将它进行质因数分解: $a = p_{1}^{c_1}p_{2}^{c_2}...p_{n}^{c_n}$

那么约数之和是$(p_1^{0} + p_1^1 + ...+p_1^{c_1})(p_2^{0} + p_2^1 + ...+p_2^{c_2})...(p_n^{0} + p_n^1 + ...+p_n^{c_n})$

对于多项式$\sum_{i=0}^{n}p^i$可以采用如下代码计算:

```cpp
int t = 1;
while (n --)
  t = t * p + 1;
```

```cpp
#include <iostream>
#include <unordered_map>

using namespace std;

const int mod = 1e9 + 7;

typedef long long LL;

unordered_map<int, int> primes;

int main()
{
    int n;
    cin >> n;
    
    while (n --)
    {
        int x;
        cin >> x;
        
        for (int i = 2; i <= x / i; i ++)
        {
            while (x % i == 0)
            {
                x /= i;
                primes[i] ++;
            }
        }
        if (x > 1) primes[x] ++;
    }
    
    LL res = 1;
    
    for (auto p: primes)
    {
        LL t = 1;
        int s = p.second;
        
        while (s --)
            t = (t * p.first + 1) % mod;
        
        res = res * t % mod;
    }
    
    cout << res << endl;
    return 0;
}
```



## 求最大公约数

> https://www.acwing.com/problem/content/874/

求最大公约数需要用到欧几里得算法, 即$gcd(a, b) = gcd(b, a \% b)$

证明:

首先, 有$gcd(a, b) = gcd(a, ka + b)$, 也就是将$a$和$b$进行线性组合之后, 对于公约数来说性质不变.

由于$a \%b = a - \lfloor \frac{a}{b} \rfloor \times b$, 本质上也是$a$和$b$的线性组合.

因此, 证明完毕.

计算的函数就是:

```cpp
int gcd(int a, int b)
{
  return b ? gcd(b, a % b) : a;
}
```



## 分解质因数求欧拉函数

> https://www.acwing.com/problem/content/875/

如果两个数的公约数只有1, 那么两个数互质.

对于一个正整数$n$, 欧拉函数定义为$[1, n]$中与$n$互质的数的个数.

如果对$n$进行质因数分解: $n = p_{1}^{c_1}p_{2}^{c_2}...p_{m}^{c_m}$

那么欧拉函数$\phi(n) = n(1 - \frac{1}{p_1})(1 - \frac{1}{p_2})...(1 - \frac{1}{p_m})$

证明需要用到容斥原理:

* 首先从$[1, n]$中去掉$p_1, p_2, ..., p_m$的所有倍数, 因为质因子的倍数和$n$肯定不互质.

  $n - \frac{n}{p_1} - \frac{n}{p_2} - ... - \frac{n}{p_m}$

* 然后加上所有$p_ip_j, 1 \leqslant i, j \leqslant m$的倍数, 然后减去$p_ip_jp_k$的倍数, 以此类推, 经过整理, 这个表达式就是欧拉函数的公式.

```cpp
#include <iostream>

using namespace std;

int n;

int main()
{
    cin >> n;
    
    while (n --)
    {
        int x;
        cin >> x;
        int res = x;
        
        for (int i = 2; i <= x / i; i ++)
        {
            if (x % i == 0) res = res / i * (i - 1);
            while (x % i == 0) x /= i;
        }
        if (x > 1) res = res / x * (x - 1);
        
        cout << res << endl;
    }
    
    return 0;
}
```



## 筛法求欧拉函数

> https://www.acwing.com/problem/content/876/

筛法求欧拉函数可以在$O(n)$的时间复杂度内计算出$[1, n]$中每一个数的欧拉函数.

在线性筛法中:

* 一个质数$p$的欧拉函数$\phi(p) = p - 1$.
* 当`i % primes[j] == 0`时, $\phi(i \times p_j) = p_j \phi(i)$
* 当`i % primes[j] != 0`时, $\phi(i \times p_j) = p_j(1 - \frac{1}{p_j})\phi(i) = (p_j- 1)\phi(i)$

那么在进行线性筛法的时候, 顺便计算一下欧拉函数即可.

```cpp
#include <iostream>

using namespace std;

const int N = 1000010;

typedef long long LL;

int n, cnt;
bool st[N];
int primes[N], phi[N];

void get_eulers(int n)
{
    phi[1] = 1;
    for (int i = 2; i <= n; i ++)
    {
        if (!st[i]) 
        {
            primes[cnt ++] = i;
            phi[i] = i - 1;
        }
        
        for (int j = 0; primes[j] <= n / i; j ++)
        {
            st[primes[j] * i] = true;
            if (i % primes[j] == 0)
            {
                phi[primes[j] * i] = primes[j] * phi[i];
                break;
            }
            phi[primes[j] * i] = (primes[j] - 1) * phi[i];
        }
    }
}

int main()
{
    cin >> n;
    
    LL res = 0;
    
    get_eulers(n);
    
    for (int i = 1; i <= n; i ++) res += phi[i];
    
    cout << res << endl;
    return 0;
}
```



## 利用快速幂求乘法逆元

> https://www.acwing.com/problem/content/878/

乘法逆元的定义: 假设$b$是整数, 且$b$和$M$互质,  如果$bx \equiv 1\ (mod\ M)$, 那么$x$叫做$b$在模$M$意义下的乘法逆元.

根据费马小定理: 如果$p$是质数, 并且$b$和$p$互质, 那么$b^{p-1} \equiv 1\ (mod\ p)$

那么$b \times b^{p-2} \equiv 1\ (mod\ p)$, 本质上就是求$b^{p-2}$, 可以用快速幂求.

```cpp
#include <iostream>

using namespace std;

const int N = 100010;

typedef long long LL;

int qmi(int a, int k, int p)
{
    int res = 1;
    
    while (k)
    {
        if (k & 1) res = (LL)res * a % p;
        a = (LL)a * a % p;
        k >>= 1;
    }
    return res;
}

int main()
{
    int n;
    cin >> n;
    
    while (n --)
    {
        int a, p;
        cin >> a >> p;
        if (a % p == 0) puts("impossible");
        else cout << qmi(a, p - 2, p) << endl;
    }
    return 0;
}
```



## 扩展欧几里得算法

> https://www.acwing.com/problem/content/879/

裴蜀定理(Bezout定理): 对于任意的正整数$a, b$, 一定存在整数$x, y$, 使得$ax + by = gcd(a, b)$

扩展欧几里得算法用来求$x, y$.

扩展欧几里得算法代码如下:

```cpp
int exgcd(int a, int b, int &x, int &y)
{
  if (!b)
  {
    x = 1, y = 0;
    return a;
  }
  int d = exgcd(b, a % b, y, x);
  y -= a / b * x;
  return d;
}
```

* 首先, 如果`b`等于0, 那么`gcd(a, b) = a`, 那么`a = 1 * a + 0 * b`, 系数就可以求出来了.
* 然后, 用`exgcd(b, a % b, y, x)`可以求出使得`gcd(a, b) = gcd(b, a % b) = y * b + x * (a % b)`

* 然后, 将`a % b = a - (a / b) * b`带入, 就可以得到: `ax + b(y - (a / b)x) = gcd(a, b)`.



## 求解线性同余方程

> https://www.acwing.com/problem/content/880/

线性同余方程问题是, 给定$a, b, m$, 求解一个$x$, 使得: $ax \equiv b\ (mod\ m)$.

这个方程有解等价于$ax - b$是$m$的倍数, 也就是$ax - b = ym$, 也就等价于$ax + my = b$.

因此, 这个方程有解的等价条件就是$gcd(a, m)\ |\ b$.

因此, 先用扩展欧几里得算法求出一组解$x_0, y_0$使得$ax_0 + my_0 = gcd(a, m)$,  真正解$x, y$就是$x_0, y_0$再乘一个$\frac{b}{gcd(a, m)}$ , 然后再模$m$.

```cpp
#include <iostream>

using namespace std;

typedef long long LL;

int exgcd(int a, int b, int &x, int &y)
{
    if (!b)
    {
        x = 1, y = 0;
        return a;
    }
    
    int d = exgcd(b, a % b, y, x);
    
    y -= a / b * x;
    return d;
}

int main()
{
    int n;
    cin >> n;
    
    while (n --)
    {
        int a, b, m;
        cin >> a >> b >> m;
        
        int x, y;
        
        int d = exgcd(a, m, x, y);
        
        if (b % d != 0) puts("impossible");
        else
            cout << (LL) x * b / d % m << endl;
        
    }
    return 0;
}
```



## 扩展中国剩余定理

> https://www.acwing.com/problem/content/206/

扩展中国剩余定理解决的是这样一个问题:

* 给定$n$个整数: $m_1, m_2, ..., m_n$.
* 给定$n$个整数: $a_1, a_2, ..., a_n$.

求解以下方程组:
$$
\begin{cases}
	x \equiv a_1\ (mod\ m_1) \\
	x \equiv a_2\ (mod\ m_2) \\
	x \equiv a_3\ (mod\ m_3) \\
	...\\
	x \equiv a_n \ (mod\ m_n) \\
\end{cases}
$$

求解过程如下:

首先看前两个方程:
$$
\begin{cases}
 	x \equiv a_1\ (mod\ m_1) \\
	x \equiv a_2\ (mod\ m_2) \\
\end{cases}
$$
如果前两个方程有解, 那么就等价于存在整数$k_1, k_2$, 使得:
$$
\begin{cases}
x = k_1m_1 + a_1\\
x = k_2m_2 + a_2
\end{cases}
$$
那么就有: $k_1m_1 - k_2m_2 = a_2 - a_1$.

根据Bezout定理, 这个$k_1$和$k_2$存在的条件是$gcd(m_1, m_2)\ |\ (a_2 - a_1)$.

根据扩展欧几里得算法, 可以求出一组$k_1$和$k_2$, 使得$k_1m_1 - k_2m_2 = gcd(m_1, m_2)$, 那么根据这个式子也可以求出$k_1, k_2$, 假设现在已经求出来了$k_1$和$k_2$.

* 那么一组通解就是$(k_1 + k\frac{m_2}{gcd(m_1, m_2)}, k_2 + k\frac{m_1}{gcd(m_1, m_2)})$, 其中$k$是任意整数.

将这个通解代入$x$的表达式, 就可以得到$x$的通解形式:
$$
x = k_1m_1 + a_1 + k\frac{m_1m_2}{gcd(m_1, m_2)}
$$
$k_1m_1 + a_1$, 这个是一个常量.

这个形式和原来方程组中$x = k_1m_1 + a_1$这种形式完全一样.

此时, 我们就把原来方程组中的两个方程合并成一个方程, 对于有$n$个方程的方程组, 合并了$n-1$次就可以将其合并成一个方程. 最后的方程取$k = 0$就可以得到最小的非负整数解.

代码模板如下:

```cpp
#include <iostream>

using namespace std;

typedef long long LL;

LL exgcd(LL a, LL b, LL &x, LL &y)
{
    if (!b)
    {
        x = 1, y = 0;
        return a;
    }
    
    int d = exgcd(b, a % b, y, x);
    
    y -= a / b * x;
    
    return d;
}

int main()
{
    int n;
    cin >> n;
    
    LL m1, a1;
    
    cin >> m1 >> a1;
    
    bool has_answer = true;
    
    for (int i = 0; i < n - 1; i ++)
    {
        LL m2, a2;
        cin >> m2 >> a2;
        
        LL k1, k2;
        LL d = exgcd(m1, m2, k1, k2);
        // 判断是否有解
        if ( (a2 - a1) % d )
        {
            has_answer = false;
            break;
        }
        // 扩展欧求的是k1m1 - k2m2 = gcd(m1, m2), 需要处理一下才能得到真正的k1
        k1 *= (a2 - a1) / d;
        
        LL t = m2 / d;
        // 防止k1溢出
        k1 = (k1 % t + t) % t;
        
        a1 = k1 * m1 + a1;
        // 注意防止溢出
        m1 = abs(m1 / d * m2);
    }
    
    if (!has_answer) puts("-1");
    else cout << (a1 % m1 + m1) % m1 << endl;
    return 0;
}
```



## 高斯消元

> https://www.acwing.com/problem/content/885/

高斯消元可以在$O(n^3)$之内求解一个线性方程组, 这里假设线性方程组的系数矩阵是$n \times n$的方阵.

有三种初等行变换:

* 把某一行乘上一个不等于0的数$c$.
* 交换某两行.
* 把某一行的若干倍加到另一行上.

对增广矩阵进行上述三种初等行列变换后, 变换后的线性方程组和原来的线性方程组解是等价的.

通过这三种变换, 可以把增广矩阵变成上三角的形式.

线性方程组的解有三种情况:

* 唯一解: 增广矩阵是完美的阶梯型.
* 无解: 出现了“0 = 1”这种情况, 也就是系数矩阵都是0, 但是增广矩阵最后一列有非0.
* 无穷解: 出现了0 = 0的情况.

高斯消元的算法可以这样写:

* 枚举每一列, 假设当前列是$c$.
  * 找到当前这一列绝对值最大的元素对应的一行.
  * 把这一行换到第一行.
  * 交换后, 将这一行的第一个数变成1.
  * 将下面所有行的当前列的元素消成0.

