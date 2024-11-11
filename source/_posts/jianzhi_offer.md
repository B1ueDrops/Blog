---
title: 《剑指Offer》题解
categories: 算法
mathjax: true
---

## 在O(1)时间删除链表节点

> https://www.acwing.com/problem/content/85/

* 在单链表中, **除了尾节点**, 都可以在$O(1)$的时间复杂度内删除.

```cpp
class Solution {
public:
    void deleteNode(ListNode* node) {
        node->val = node->next->val;
        node->next = node->next->next;
    }
};
```



## 删除链表中的重复节点

> https://acwing.com/problem/content/27/

```cpp
class Solution {
public:
    ListNode* deleteDuplication(ListNode* head) {
        auto dummy = new ListNode(-1);
        dummy->next = head;
        
        auto p = dummy;
        while (p->next) {
            auto q = p->next;
            while (q && q->val == p->next->val) q = q->next;
            if (q == p->next->next) p = p->next;
            else p->next = q;
        }
        return dummy->next;
    }
};
```





## 调整数组顺序使奇数位于偶数前面

> https://www.acwing.com/problem/content/description/30/

* 和快速排序中调整数组, 使得一部分`< x`, 一部分`> x`一样.

```cpp
class Solution {
public:
    void reOrderArray(vector<int> &array) {
        int n = array.size();
        if (!n) return ;
        int i = -1, j = n;
        while (i < j) {
            while (array[ ++ i] % 2);
            while (array[ -- j] % 2 == 0);
            if (i < j) swap(array[i], array[j]);
        }
    }
};
```



## 包含min函数的栈

> https://www.acwing.com/problem/content/90/

```cpp
class MinStack {
public:
    stack<int> stk;
    stack<int> stk_min;
    /** initialize your data structure here. */
    MinStack() {
        
    }
    
    void push(int x) {
        if (stk_min.empty() || x <= stk_min.top()) stk_min.push(x);
        stk.push(x);
    }
    
    void pop() {
        if (stk_min.size() && stk_min.top() == stk.top()) stk_min.pop();
        stk.pop();
    }
    
    int top() {
        return stk.top();
    }
    
    int getMin() {
        return stk_min.top();
    }
};
```

