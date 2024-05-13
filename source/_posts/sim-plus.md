---
title: 高精度加减乘除
categories: 算法
---




## 高精度加法

> https://www.acwing.com/problem/content/793/

直接背模板, 注意要求两个数不含前导0

```cpp
#include <iostream>
#include <cstring>
#include <vector>

using namespace std;

vector<int> add(vector<int> &A, vector<int> &B) {
    
    vector<int> C;
    int t = 0;
    
    for (int i = 0; i < A.size() || i < B.size() || t; i ++) {
        
        if (i < A.size()) t += A[i];
        if (i < B.size()) t += B[i];
        C.push_back(t % 10);
        t /= 10;
    }
    
    return C;
}

int main() {
    
    string a, b;
    
    cin >> a >> b;
    vector<int> A, B;
    
    for (int i = a.size() - 1; i >= 0; i --) A.push_back(a[i] - '0');
    for (int i = b.size() - 1; i >= 0; i --) B.push_back(b[i] - '0');
    
    auto C = add(A, B);
    
    for (int i = C.size() - 1; i >= 0; i --) printf("%d", C[i]);
}
```



## 高精度减法

> https://www.acwing.com/problem/content/description/794/

直接背模板, 注意两个数不含前导0.

```cpp
#include <iostream>
#include <vector>

using namespace std;


bool cmp(vector<int> &A, vector<int> &B) {
    if (A.size() != B.size()) return A.size() > B.size();
    for (int i = A.size() - 1; i >= 0; i --) {
        if (A[i] != B[i]) return A[i] > B[i];
    }
    return true;
}

vector<int> sub(vector<int> &A, vector<int> &B) {
    
    vector<int> C;
    int t = 0;
    
    for (int i = 0; i < A.size(); i ++) {
        t = A[i] - t;
        if (i < B.size()) t -= B[i];
        C.push_back((t + 10) % 10);
        if (t < 0) t = 1;
        else t = 0;
    }
    /* 去除前导0 */
    while (C.back() == 0 && C.size() > 1) C.pop_back();
    return C;
}

int main() {
    
    string a, b;
    cin >> a >> b;
    
    vector<int> A, B;
    
    for (int i = a.size() - 1; i >= 0; i --) A.push_back(a[i] - '0');
    for (int i = b.size() - 1; i >= 0; i --) B.push_back(b[i] - '0');
    
    vector<int> C;
    
    if (cmp(A, B)) C = sub(A, B);
    else {
        C = sub(B, A);
        cout << '-';
    }
    for (int i = C.size() - 1; i >= 0; i --) cout << C[i];
    cout << endl;
    
    return 0;
}
```



## 高精度乘法

> https://leetcode.cn/problems/multiply-strings/

```cpp
class Solution {
public:
    string multiply(string num1, string num2) {

        int n = num1.size(), m = num2.size();
        vector<int> A, B;

        for (int i = n - 1; i >= 0; i --) A.push_back(num1[i] - '0');
        for (int i = m - 1; i >= 0; i --) B.push_back(num2[i] - '0');

        vector<int> C(n + m);
        for (int i = 0; i < n; i ++)
            for (int j = 0; j < m; j ++)
                C[i + j] += A[i] * B[j];

        for (int i = 0, t = 0; i < C.size(); i ++) {
            t += C[i];
            C[i] = t % 10;
            t /= 10;
        }

        while (C.size() > 1 && C.back() == 0) C.pop_back();

        string res;
        int k = C.size() - 1;
        while (k >= 0) res += C[k --] + '0';
        
        return res;
    }
};
```

另一个版本是一个`vector`乘一个较小的整数: https://www.acwing.com/problem/content/795/

```cpp
#include <iostream>
#include <vector>

using namespace std;

vector<int> mul(vector<int> &A, int b)
{
    int t = 0;
    vector<int> C;
    
    for (int i = 0; i < A.size() || t; i ++)
    {
        if (i < A.size()) t += A[i] * b;
        C.push_back(t % 10);
        t /= 10;
    }
    
    while (C.back() == 0 && C.size() > 1) C.pop_back();
    
    return C;
}

int main()
{
    string a;
    int b;
    
    cin >> a >> b;
    
    vector<int> A;
    for (int i = a.size() - 1; i >= 0; i --) A.push_back(a[i] - '0');
    
    auto C = mul(A, b);
    
    for (int i = C.size() - 1; i >= 0; i --) cout << C[i];
    
    return 0;
}
```



## 高精度除法

> https://www.acwing.com/problem/content/description/796/

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

vector<int> div(vector<int> &A, int b, int &r) {
    
    r = 0;
    vector<int> C;
    
    for (int i = A.size() - 1; i >= 0; i --) {
        r = r * 10 + A[i];
        C.push_back(r / b);
        r %= b;
    }
  // 注意反向
    reverse(C.begin(), C.end());
    while (C.size() > 1 && C.back() == 0) C.pop_back();
    
    return C;
}

int main() {
    
    string a;
    int b;
    
    cin >> a >> b;
    
    vector<int> A;
    for (int i = a.size() - 1; i >= 0; i --) A.push_back(a[i] - '0');
    
    int r;
    
    auto C = div(A, b, r);
    
    for (int i = C.size() - 1; i >= 0; i --) cout << C[i];
    cout << endl;
    cout << r << endl;
}
```

