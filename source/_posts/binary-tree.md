---
title: 二叉树全系列
categories: 算法
mathjax: true
---



## 中序遍历

> https://leetcode.cn/problems/binary-tree-inorder-traversal/

中序遍历是左, 中, 右.



### 递归写法

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> ans;
    vector<int> inorderTraversal(TreeNode* root) {
        dfs(root);
        return ans;
    }
    void dfs(TreeNode *root) {
        if (!root) return ;
        dfs(root->left);
        ans.push_back(root->val);
        dfs(root->right);
    }
};
```

### 迭代写法

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:

    vector<int> ans;
    stack<TreeNode*> stk;
    vector<int> inorderTraversal(TreeNode* root) {
        while (root || stk.size()) 
        {
            while (root)
            {
                stk.push(root);
                root = root->left;
            }

            root = stk.top();
            stk.pop();
            ans.push_back(root->val);
            root = root->right;
        }
        return ans;
    }
};
```

## 前序遍历

> https://leetcode.cn/problems/binary-tree-preorder-traversal/



前序遍历是根, 左, 右.

### 递归写法

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> ans;
    vector<int> preorderTraversal(TreeNode* root) {
        dfs(root);
        return ans;
    }
    void dfs(TreeNode *root) {
        if (!root) return ;
        ans.push_back(root->val);
        dfs(root->left);
        dfs(root->right);
    }
};
```

### 迭代写法

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> ans;
    stack<TreeNode *> stk;
    vector<int> preorderTraversal(TreeNode* root) {
        while (root || stk.size())
        {
            while (root)
            {
                ans.push_back(root->val);
                stk.push(root);
                root = root->left;
            }

            root = stk.top();
            stk.pop();
            root = root->right;
        }
        return ans;
    }
};
```

## 后序遍历

> https://leetcode.cn/problems/binary-tree-postorder-traversal/



后序遍历是左, 右, 根.

### 递归写法



```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> ans;
    vector<int> postorderTraversal(TreeNode* root) {
        dfs(root);
        return ans;
    }
    void dfs(TreeNode *root) {
        if (!root) return ;
        dfs(root->left);
        dfs(root->right);
        ans.push_back(root->val);
    }
};
```

### 迭代写法

 先按照根右左的顺序遍历, 然后反向即可.

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> ans;
    stack<TreeNode *> stk;
    vector<int> postorderTraversal(TreeNode* root) {
        while (root || stk.size()) {
            while (root) {
                ans.push_back(root->val);
                stk.push(root);
                root = root->right;
            }
            root = stk.top();
            stk.pop();
            root = root->left;
        }
        reverse(ans.begin(), ans.end());
        return ans;
    }
};
```



## 层序遍历

> https://leetcode.cn/problems/binary-tree-level-order-traversal
>
> https://www.acwing.com/problem/content/description/41/
>
> https://www.acwing.com/problem/content/42/

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ans;
        if (!root) return ans;

        queue<TreeNode *> q;
        q.push(root);

        while (q.size()) {
            vector<int> level;
            int len = q.size();
						
          /* 每一层单独放 */
            while (len --) {
                auto t = q.front();
                q.pop();
                level.push_back(t->val);
                if (t->left) q.push(t->left);
                if (t->right) q.push(t->right);
            }
            ans.push_back(level);
        }

        return ans;
    }
};
```



## 之字形层序遍历

> https://www.acwing.com/problem/content/description/43/

在层序遍历的基础上用一个变量控制一下是否反转`level`即可.

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> printFromTopToBottom(TreeNode* root) {
        
        vector<vector<int>> ans;
        if (!root) return ans;
        
        queue<TreeNode *> q;
        q.push(root);
        
        bool rev = false;
        
        while (q.size())
        {
            vector<int> level;
            int len = q.size();
            
            while (len --)
            {
                auto t = q.front();
                q.pop();
                level.push_back(t->val);
                if (t->left) q.push(t->left);
                if (t->right) q.push(t->right);
            }
            if (rev) reverse(level.begin(), level.end());
            ans.push_back(level);
            rev = !rev;
        }
        return ans;
    }
};
```





## 前序中序恢复二叉树

> https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

首先, 前序遍历序列的第一个元素就是根节点, 通过哈希表可以找到第一个元素在中序遍历中的位置, 假设根节点在中序遍历中的下标是$k$.

假设前序遍历的下标范围是$[pl, pr]$, 中序遍历的下标范围是$[il, ir]$, 那么:

* 左子树在前序遍历的下标范围是$[pl + 1, pl + 1 + (k - 1 - il + 1) - 1]$
* 右子树在前序遍历的下标范围是$[pl + 1 +  (k - il + 1) - 1 + 1, pr]$
* 左子树在中序遍历的下标范围是$[il, k - 1]$
* 右子树在中序遍历的下标范围是$[k + 1, ir]$

然后递归建树即可.

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    unordered_map<int, int> hash;
    TreeNode *dfs(vector<int> &preorder, vector<int> &inorder, int pl, int pr, int il, int ir) {
        if (pl > pr) return NULL;
        int val = preorder[pl];
        int k = hash[val];
        TreeNode *root = new TreeNode(val);

        root->left = dfs(preorder, inorder, pl + 1, pl + 1 + k - 1 - il, il, k - 1);
        root->right = dfs(preorder, inorder, pl + 1 + k - 1 - il + 1, pr, k + 1, ir);

        return root;
    }
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        int n = inorder.size();
        for (int i = 0; i < n; i++) hash[inorder[i]] = i;
        return dfs(preorder, inorder, 0, n - 1, 0, n - 1);
    }
};
```

注意, 只有前序和后序不能唯一确定一个二叉树 



## 后序中序恢复二叉树

> https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/description

分析思路和上一题相同.

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    unordered_map<int, int> hash;

    TreeNode *dfs(vector<int> &inorder, vector<int> &postorder, int il, int ir, int pl, int pr) {
        if (pl > pr) return NULL;
        int val = postorder[pr];
        int k = hash[val];

        TreeNode *root = new TreeNode(val);

        root->left = dfs(inorder, postorder, il, k - 1, pl, pl + k - 1 - il);
        root->right = dfs(inorder, postorder, k + 1, ir, pl + k - 1 - il + 1, pr - 1);

        return root;
    }

    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        int n = inorder.size();
        for (int i = 0; i < n; i++) hash[inorder[i]] = i;
        return dfs(inorder, postorder, 0, n - 1, 0, n - 1); 
    }
};
```



## 二叉树中序遍历的下一个节点

> https://www.acwing.com/problem/content/description/31/

分两种情况:

* 第一: 如果给定节点有右子树, 那么下一个节点就是右子树最左边的节点, 对应左根右的右这一部分.
* 第二: 如果给定节点没有右子树, 那么它需要向上, 找到节点$p$, 使得$p.father.left = p$, 那么$p.father$就是下一个节点, 对应左根右的根这一部分.

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode *father;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL), father(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* inorderSuccessor(TreeNode* p) {
        if (p->right) {
            p = p->right;
            while (p->left) p = p->left;
            return p;
        }
        else {
            while (p->father && p->father->right == p) p = p->father;
            return p->father;
        }
    }
};
```



## 二叉搜索树的后序遍历

> https://www.acwing.com/problem/content/44/



```cpp
class Solution {
public:
    bool verifySequenceOfBST(vector<int> sequence) {
        int n = sequence.size();
        if (!n) return true;
        return dfs(0, n - 1, sequence);
    }
    
    bool dfs(int l, int r, vector<int> &sequence) {
        if (l >= r) return true;
        
        int index = l, root = sequence[r];
        
        while (index < r && sequence[index] < root) index ++;
        
        for (int i = index; i < r; i ++)
            if (sequence[i] < root)
                return false;
        return dfs(l, index - 1, sequence) && dfs(index, r - 1, sequence);
    }
};
```



## 二叉树中和是某一值的路径

> https://www.acwing.com/problem/content/description/45/

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    vector<int> path;
    vector<vector<int>> ans;
    
    vector<vector<int>> findPath(TreeNode* root, int sum) {
        
        if (!root) return ans;
        dfs(root, sum);
        return ans;
    }
    
    void dfs(TreeNode *root, int sum)
    {
        if (!root) return ;
        
        path.push_back(root->val);
        sum -= root->val;
        
        if (!root->left && !root->right && !sum)
        {
            ans.push_back(path);
            path.pop_back();
            sum += root->val;
            return ;
        }
        
        dfs(root->left, sum);
        dfs(root->right, sum);
        
        sum += root->val;
        path.pop_back();
        
        return ;
    }
};
```

> https://leetcode.cn/problems/path-sum/

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) return false;
        return dfs(root, targetSum);
    }

    bool dfs(TreeNode *root, int sum) {
        
        if (!root) return false;

        sum -= root->val;

        if (!root->left && !root->right && !sum) {
            sum += root->val;
            return true;
        }
        
        bool left = dfs(root->left, sum);
        bool right = dfs(root->right, sum);

        if (left || right) {
            sum += root->val;
            return true;
        }
        return false;
    }
};
```



## 树的子结构

> https://www.acwing.com/problem/content/35/

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    /* 判断r2子树是否是r1子树的子结构 */
    bool dfs(TreeNode *r1, TreeNode *r2) {
      // 注意, 这里有顺序, 一定要先判断r2是否为空
        if (!r2) return true;
        if (!r1) return false;
        if (r1->val != r2->val) return false;
        return dfs(r1->left, r2->left) && dfs(r1->right, r2->right);
    }
    
    bool hasSubtree(TreeNode* pRoot1, TreeNode* pRoot2) {
        /* 如果是有一个是NULL, 那么就不是子结构 */
        if (!pRoot1 || !pRoot2) return false;
        if (dfs(pRoot1, pRoot2)) return true;
        return hasSubtree(pRoot1->left, pRoot2) || hasSubtree(pRoot1->right, pRoot2);
    }
};
```



## 二叉树的镜像

> https://www.acwing.com/problem/content/description/37/

将二叉树变成镜像的方法就是递归地把左右节点交换即可.

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    void mirror(TreeNode* root) {
        
        if (!root) return ;
        mirror(root->left);
        mirror(root->right);
        
        auto t = root->left;
        root->left = root->right;
        root->right = t;
    }
};
```



## 对称的二叉树

> https://www.acwing.com/problem/content/description/38/

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    /* 判断p和q所在的子树是否对称 */
    bool dfs(TreeNode *p, TreeNode *q) {
        if (!p || !q) return !p && !q;
        if (p->val != q->val) return false;
        return dfs(p->left, q->right) && dfs(p->right, q->left);
    }
    bool isSymmetric(TreeNode* root) {
        if (!root) return true;
        return dfs(root->left, root->right);
    }
};
```



## 不同的二叉搜索树求数目

> https://leetcode.cn/problems/unique-binary-search-trees/

> 二叉搜索树的中序遍历一定是严格递增的.

假设$f(n)$表示长度为$n$, 元素为$[1, n]$的二叉搜索树的个数.

初始状态是`f[0] = 1`. 注意空子树这个形态是合法的, 因为一个状态会由它的左子树和右子树的状态转移而来, 而左子树和右子树可以为空.

从$[1, n]$进行枚举, 对于状态`i`, 再从$j \in [1, i]$进行枚举, $j$作为根节点, 那么左子树和右子树的个数就分别是`f[j - 1]`和`f[i - (j + 1) + 1]`, 进行状态转移即可.

```cpp
class Solution {
public:
    int numTrees(int n) {
        vector<int> f(n + 1);
        f[0] = 1;
        for (int i = 1; i <= n; i ++) {
            for (int j = 1; j <= i; j ++) {
                f[i] += f[j - 1] * f[i - j];
            }
        }
        return f[n];
    }
};
```

本题目也有通项公式, 是卡特兰数.



## 不同的二叉搜索树求方案

> https://leetcode.cn/problems/unique-binary-search-trees-ii/

常规的搜索问题.

```cpp
class Solution {
public:
    vector<TreeNode*> generateTrees(int n) {
        return dfs(1, n);
    }

    vector<TreeNode *> dfs(int l, int r) {
        if (l > r) return {NULL};
        vector<TreeNode *> res;

        // 枚举根节点的位置
        for (int i = l; i <= r; i ++) {
            auto left = dfs(l, i - 1), right = dfs(i + 1, r);
            for (auto p : left) {
                for (auto q: right) {
                    TreeNode *root = new TreeNode(i);
                    root->left = p, root->right = q;
                    res.push_back(root);
                }
            }
        }
        return res;
    }
};
```



## 验证二叉搜索树

> https://leetcode.cn/problems/validate-binary-search-tree/

方法一: 如果一个树是二叉搜索树, 等价于中序遍历严格递增.

方法二: 按照定义搜索, 需要求子树中所有元素的最小值和最大值.

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        if (!root) return true;
        return dfs(root)[0];
    }
    // 返回三个信息, 是否是二叉搜索树, 子树最大值, 子树最小值
    vector<int> dfs(TreeNode *root) {
        vector<int> res({ 1, root->val, root->val });
				
        if (root->left) {
            auto t = dfs(root->left);
            if (!t[0] || t[2] >= root->val) res[0] = 0;
            res[1] = min(root->val, t[1]);
            res[2] = max(root->val, t[2]);
        }
        if (root->right) {
            auto t = dfs(root->right);
            if (!t[0] || t[1] <= root->val) res[0] = 0;
            // 注意这里min里面是res[1], 而不是root->val, 因为取最小值要连上左子树一起比较
            res[1] = min(res[1], t[1]);
            res[2] = max(res[2], t[2]);
        }
        return res;
    }
};
```





