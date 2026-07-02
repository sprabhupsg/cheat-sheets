# Binary Tree — C++ Cheat Sheet

A comprehensive, interview-ready reference for Binary Tree problems in C++.
Covers the node definition, traversals, construction, properties, views, paths,
BST operations, LCA, serialization, and common patterns — each with clean,
copy-pasteable code.

---

## Table of Contents

1. [Node Definition & Helpers](#1-node-definition--helpers)
2. [Traversals (Recursive & Iterative)](#2-traversals)
3. [Level Order / BFS](#3-level-order--bfs)
4. [Tree Properties](#4-tree-properties)
5. [Construction Problems](#5-construction-problems)
6. [Views of a Tree](#6-views-of-a-tree)
7. [Path Problems](#7-path-problems)
8. [Lowest Common Ancestor (LCA)](#8-lowest-common-ancestor-lca)
9. [Binary Search Tree (BST)](#9-binary-search-tree-bst)
10. [Serialization / Deserialization](#10-serialization--deserialization)
11. [Modify / Transform](#11-modify--transform)
12. [Misc & Advanced Patterns](#12-misc--advanced-patterns)
13. [Complexity Quick Reference](#13-complexity-quick-reference)

---

## 1. Node Definition & Helpers

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* l, TreeNode* r) : val(x), left(l), right(r) {}
};
```

---

## 2. Traversals

### 2.1 Recursive

```cpp
void preorder(TreeNode* root, vector<int>& out) {   // Root, Left, Right
    if (!root) return;
    out.push_back(root->val);
    preorder(root->left, out);
    preorder(root->right, out);
}

void inorder(TreeNode* root, vector<int>& out) {     // Left, Root, Right
    if (!root) return;
    inorder(root->left, out);
    out.push_back(root->val);
    inorder(root->right, out);
}

void postorder(TreeNode* root, vector<int>& out) {   // Left, Right, Root
    if (!root) return;
    postorder(root->left, out);
    postorder(root->right, out);
    out.push_back(root->val);
}
```

### 2.2 Iterative Preorder

```cpp
vector<int> preorderIter(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top(); st.pop();
        res.push_back(node->val);
        if (node->right) st.push(node->right);   // right first so left is processed first
        if (node->left)  st.push(node->left);
    }
    return res;
}
```

### 2.3 Iterative Inorder

```cpp
vector<int> inorderIter(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }
        cur = st.top(); st.pop();
        res.push_back(cur->val);
        cur = cur->right;
    }
    return res;
}
```

### 2.4 Iterative Postorder (two stacks)

```cpp
vector<int> postorderIter(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    stack<TreeNode*> s1, s2;
    s1.push(root);
    while (!s1.empty()) {
        TreeNode* node = s1.top(); s1.pop();
        s2.push(node);
        if (node->left)  s1.push(node->left);
        if (node->right) s1.push(node->right);
    }
    while (!s2.empty()) { res.push_back(s2.top()->val); s2.pop(); }
    return res;
}
```

### 2.5 Morris Inorder (O(1) space)

```cpp
vector<int> morrisInorder(TreeNode* root) {
    vector<int> res;
    TreeNode* cur = root;
    while (cur) {
        if (!cur->left) {
            res.push_back(cur->val);
            cur = cur->right;
        } else {
            TreeNode* pre = cur->left;
            while (pre->right && pre->right != cur) pre = pre->right;
            if (!pre->right) {            // create thread
                pre->right = cur;
                cur = cur->left;
            } else {                      // remove thread
                pre->right = nullptr;
                res.push_back(cur->val);
                cur = cur->right;
            }
        }
    }
    return res;
}
```

---

## 3. Level Order / BFS

### 3.1 Level Order (grouped by level)

```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level;
        for (int i = 0; i < sz; ++i) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}
```

### 3.2 Zigzag (Spiral) Level Order

```cpp
vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q; q.push(root);
    bool leftToRight = true;
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level(sz);
        for (int i = 0; i < sz; ++i) {
            TreeNode* node = q.front(); q.pop();
            int idx = leftToRight ? i : sz - 1 - i;
            level[idx] = node->val;
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        leftToRight = !leftToRight;
        res.push_back(level);
    }
    return res;
}
```

---

## 4. Tree Properties

### 4.1 Height / Max Depth

```cpp
int maxDepth(TreeNode* root) {
    if (!root) return 0;
    return 1 + max(maxDepth(root->left), maxDepth(root->right));
}
```

### 4.2 Min Depth

```cpp
int minDepth(TreeNode* root) {
    if (!root) return 0;
    if (!root->left)  return 1 + minDepth(root->right);
    if (!root->right) return 1 + minDepth(root->left);
    return 1 + min(minDepth(root->left), minDepth(root->right));
}
```

### 4.3 Count Nodes

```cpp
int countNodes(TreeNode* root) {
    if (!root) return 0;
    return 1 + countNodes(root->left) + countNodes(root->right);
}
```

### 4.4 Diameter (longest path between any two nodes)

```cpp
int diameterOfBinaryTree(TreeNode* root) {
    int diameter = 0;
    function<int(TreeNode*)> dfs = [&](TreeNode* node) -> int {
        if (!node) return 0;
        int l = dfs(node->left);
        int r = dfs(node->right);
        diameter = max(diameter, l + r);   // path through node (edge count)
        return 1 + max(l, r);
    };
    dfs(root);
    return diameter;
}
```

### 4.5 Balanced Check

```cpp
bool isBalanced(TreeNode* root) {
    function<int(TreeNode*)> height = [&](TreeNode* node) -> int {
        if (!node) return 0;
        int l = height(node->left);
        if (l == -1) return -1;
        int r = height(node->right);
        if (r == -1) return -1;
        if (abs(l - r) > 1) return -1;     // -1 signals imbalance
        return 1 + max(l, r);
    };
    return height(root) != -1;
}
```

### 4.6 Same Tree

```cpp
bool isSameTree(TreeNode* p, TreeNode* q) {
    if (!p && !q) return true;
    if (!p || !q || p->val != q->val) return false;
    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
}
```

### 4.7 Symmetric Tree

```cpp
bool isSymmetric(TreeNode* root) {
    function<bool(TreeNode*, TreeNode*)> mirror = [&](TreeNode* a, TreeNode* b) {
        if (!a && !b) return true;
        if (!a || !b || a->val != b->val) return false;
        return mirror(a->left, b->right) && mirror(a->right, b->left);
    };
    return !root || mirror(root->left, root->right);
}
```

### 4.8 Subtree of Another Tree

```cpp
bool isSameTree(TreeNode* a, TreeNode* b) {
    if (!a && !b) return true;
    if (!a || !b || a->val != b->val) return false;
    return isSameTree(a->left, b->left) && isSameTree(a->right, b->right);
}
bool isSubtree(TreeNode* root, TreeNode* sub) {
    if (!sub) return true;
    if (!root) return false;
    if (isSameTree(root, sub)) return true;
    return isSubtree(root->left, sub) || isSubtree(root->right, sub);
}
```

---

## 5. Construction Problems

### 5.1 Build from Preorder + Inorder

```cpp
TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
    unordered_map<int,int> idx;
    for (int i = 0; i < (int)inorder.size(); ++i) idx[inorder[i]] = i;
    int pre = 0;
    function<TreeNode*(int,int)> build = [&](int l, int r) -> TreeNode* {
        if (l > r) return nullptr;
        int rootVal = preorder[pre++];
        TreeNode* root = new TreeNode(rootVal);
        int mid = idx[rootVal];
        root->left  = build(l, mid - 1);
        root->right = build(mid + 1, r);
        return root;
    };
    return build(0, inorder.size() - 1);
}
```

### 5.2 Build from Inorder + Postorder

```cpp
TreeNode* buildTreePost(vector<int>& inorder, vector<int>& postorder) {
    unordered_map<int,int> idx;
    for (int i = 0; i < (int)inorder.size(); ++i) idx[inorder[i]] = i;
    int post = postorder.size() - 1;
    function<TreeNode*(int,int)> build = [&](int l, int r) -> TreeNode* {
        if (l > r) return nullptr;
        int rootVal = postorder[post--];
        TreeNode* root = new TreeNode(rootVal);
        int mid = idx[rootVal];
        root->right = build(mid + 1, r);   // right before left (post reversed)
        root->left  = build(l, mid - 1);
        return root;
    };
    return build(0, inorder.size() - 1);
}
```

---

## 6. Views of a Tree

### 6.1 Right Side View

```cpp
vector<int> rightSideView(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; ++i) {
            TreeNode* node = q.front(); q.pop();
            if (i == sz - 1) res.push_back(node->val);   // last node of level
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
    return res;
}
```

### 6.2 Top View

```cpp
vector<int> topView(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    map<int,int> cols;                          // horizontal distance -> value
    queue<pair<TreeNode*,int>> q;
    q.push({root, 0});
    while (!q.empty()) {
        auto [node, hd] = q.front(); q.pop();
        if (cols.find(hd) == cols.end()) cols[hd] = node->val;  // first seen
        if (node->left)  q.push({node->left,  hd - 1});
        if (node->right) q.push({node->right, hd + 1});
    }
    for (auto& [hd, val] : cols) res.push_back(val);
    return res;
}
```

### 6.3 Bottom View

```cpp
vector<int> bottomView(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    map<int,int> cols;
    queue<pair<TreeNode*,int>> q;
    q.push({root, 0});
    while (!q.empty()) {
        auto [node, hd] = q.front(); q.pop();
        cols[hd] = node->val;                   // overwrite -> keeps last (lowest)
        if (node->left)  q.push({node->left,  hd - 1});
        if (node->right) q.push({node->right, hd + 1});
    }
    for (auto& [hd, val] : cols) res.push_back(val);
    return res;
}
```

### 6.4 Vertical Order Traversal

```cpp
vector<vector<int>> verticalTraversal(TreeNode* root) {
    map<int, map<int, multiset<int>>> nodes;    // col -> row -> values
    function<void(TreeNode*,int,int)> dfs = [&](TreeNode* node, int row, int col) {
        if (!node) return;
        nodes[col][row].insert(node->val);
        dfs(node->left,  row + 1, col - 1);
        dfs(node->right, row + 1, col + 1);
    };
    dfs(root, 0, 0);
    vector<vector<int>> res;
    for (auto& [col, rows] : nodes) {
        vector<int> colVals;
        for (auto& [row, vals] : rows)
            colVals.insert(colVals.end(), vals.begin(), vals.end());
        res.push_back(colVals);
    }
    return res;
}
```

### 6.5 Boundary Traversal

```cpp
vector<int> boundaryTraversal(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    auto isLeaf = [](TreeNode* n){ return !n->left && !n->right; };
    if (!isLeaf(root)) res.push_back(root->val);

    // left boundary (top-down, exclude leaves)
    TreeNode* cur = root->left;
    while (cur) {
        if (!isLeaf(cur)) res.push_back(cur->val);
        cur = cur->left ? cur->left : cur->right;
    }
    // leaves (left-to-right)
    function<void(TreeNode*)> leaves = [&](TreeNode* n) {
        if (!n) return;
        if (isLeaf(n)) { res.push_back(n->val); return; }
        leaves(n->left); leaves(n->right);
    };
    leaves(root);
    // right boundary (bottom-up, exclude leaves)
    vector<int> rb;
    cur = root->right;
    while (cur) {
        if (!isLeaf(cur)) rb.push_back(cur->val);
        cur = cur->right ? cur->right : cur->left;
    }
    res.insert(res.end(), rb.rbegin(), rb.rend());
    return res;
}
```

---

## 7. Path Problems

### 7.1 Root-to-Leaf Path Sum (exists?)

```cpp
bool hasPathSum(TreeNode* root, int target) {
    if (!root) return false;
    if (!root->left && !root->right) return target == root->val;
    return hasPathSum(root->left,  target - root->val) ||
           hasPathSum(root->right, target - root->val);
}
```

### 7.2 All Root-to-Leaf Paths Equal to Sum

```cpp
vector<vector<int>> pathSum(TreeNode* root, int target) {
    vector<vector<int>> res;
    vector<int> path;
    function<void(TreeNode*,int)> dfs = [&](TreeNode* node, int rem) {
        if (!node) return;
        path.push_back(node->val);
        if (!node->left && !node->right && rem == node->val)
            res.push_back(path);
        else {
            dfs(node->left,  rem - node->val);
            dfs(node->right, rem - node->val);
        }
        path.pop_back();
    };
    dfs(root, target);
    return res;
}
```

### 7.3 Max Path Sum (any node to any node)

```cpp
int maxPathSum(TreeNode* root) {
    int best = INT_MIN;
    function<int(TreeNode*)> dfs = [&](TreeNode* node) -> int {
        if (!node) return 0;
        int l = max(0, dfs(node->left));
        int r = max(0, dfs(node->right));
        best = max(best, node->val + l + r);   // path through node
        return node->val + max(l, r);          // best downward path
    };
    dfs(root);
    return best;
}
```

### 7.4 All Root-to-Leaf Paths (as strings)

```cpp
vector<string> binaryTreePaths(TreeNode* root) {
    vector<string> res;
    function<void(TreeNode*, string)> dfs = [&](TreeNode* node, string cur) {
        if (!node) return;
        cur += to_string(node->val);
        if (!node->left && !node->right) { res.push_back(cur); return; }
        cur += "->";
        dfs(node->left, cur);
        dfs(node->right, cur);
    };
    dfs(root, "");
    return res;
}
```

### 7.5 Print Path to a Given Node

```cpp
bool findPath(TreeNode* root, int target, vector<int>& path) {
    if (!root) return false;
    path.push_back(root->val);
    if (root->val == target) return true;
    if (findPath(root->left, target, path) || findPath(root->right, target, path))
        return true;
    path.pop_back();
    return false;
}
```

---

## 8. Lowest Common Ancestor (LCA)

### 8.1 LCA in a Binary Tree

```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* l = lowestCommonAncestor(root->left, p, q);
    TreeNode* r = lowestCommonAncestor(root->right, p, q);
    if (l && r) return root;     // p and q split here
    return l ? l : r;
}
```

### 8.2 LCA in a BST

```cpp
TreeNode* lcaBST(TreeNode* root, TreeNode* p, TreeNode* q) {
    while (root) {
        if (p->val < root->val && q->val < root->val) root = root->left;
        else if (p->val > root->val && q->val > root->val) root = root->right;
        else return root;
    }
    return nullptr;
}
```

---

## 9. Binary Search Tree (BST)

### 9.1 Validate BST

```cpp
bool isValidBST(TreeNode* root) {
    function<bool(TreeNode*, long, long)> valid =
        [&](TreeNode* node, long lo, long hi) {
            if (!node) return true;
            if (node->val <= lo || node->val >= hi) return false;
            return valid(node->left, lo, node->val) &&
                   valid(node->right, node->val, hi);
        };
    return valid(root, LONG_MIN, LONG_MAX);
}
```

### 9.2 Search

```cpp
TreeNode* searchBST(TreeNode* root, int val) {
    while (root && root->val != val)
        root = val < root->val ? root->left : root->right;
    return root;
}
```

### 9.3 Insert

```cpp
TreeNode* insertIntoBST(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);
    if (val < root->val) root->left  = insertIntoBST(root->left, val);
    else                 root->right = insertIntoBST(root->right, val);
    return root;
}
```

### 9.4 Delete

```cpp
TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;
    if (key < root->val)      root->left  = deleteNode(root->left, key);
    else if (key > root->val) root->right = deleteNode(root->right, key);
    else {
        if (!root->left)  return root->right;
        if (!root->right) return root->left;
        TreeNode* succ = root->right;            // inorder successor
        while (succ->left) succ = succ->left;
        root->val = succ->val;
        root->right = deleteNode(root->right, succ->val);
    }
    return root;
}
```

### 9.5 Kth Smallest (inorder)

```cpp
int kthSmallest(TreeNode* root, int k) {
    stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }
        cur = st.top(); st.pop();
        if (--k == 0) return cur->val;
        cur = cur->right;
    }
    return -1;
}
```

### 9.6 Sorted Array → Balanced BST

```cpp
TreeNode* sortedArrayToBST(vector<int>& nums) {
    function<TreeNode*(int,int)> build = [&](int l, int r) -> TreeNode* {
        if (l > r) return nullptr;
        int mid = l + (r - l) / 2;
        TreeNode* root = new TreeNode(nums[mid]);
        root->left  = build(l, mid - 1);
        root->right = build(mid + 1, r);
        return root;
    };
    return build(0, nums.size() - 1);
}
```

### 9.7 Inorder Successor in BST

```cpp
TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
    TreeNode* succ = nullptr;
    while (root) {
        if (p->val < root->val) { succ = root; root = root->left; }
        else root = root->right;
    }
    return succ;
}
```

---

## 10. Serialization / Deserialization

```cpp
class Codec {
public:
    string serialize(TreeNode* root) {
        string s;
        function<void(TreeNode*)> dfs = [&](TreeNode* node) {
            if (!node) { s += "#,"; return; }
            s += to_string(node->val) + ",";
            dfs(node->left);
            dfs(node->right);
        };
        dfs(root);
        return s;
    }

    TreeNode* deserialize(string data) {
        stringstream ss(data);
        string token;
        function<TreeNode*()> build = [&]() -> TreeNode* {
            if (!getline(ss, token, ',')) return nullptr;
            if (token == "#") return nullptr;
            TreeNode* node = new TreeNode(stoi(token));
            node->left  = build();
            node->right = build();
            return node;
        };
        return build();
    }
};
```

---

## 11. Modify / Transform

### 11.1 Invert / Mirror Tree

```cpp
TreeNode* invertTree(TreeNode* root) {
    if (!root) return nullptr;
    swap(root->left, root->right);
    invertTree(root->left);
    invertTree(root->right);
    return root;
}
```

### 11.2 Flatten to Linked List (preorder, in place)

```cpp
void flatten(TreeNode* root) {
    TreeNode* cur = root;
    while (cur) {
        if (cur->left) {
            TreeNode* pre = cur->left;
            while (pre->right) pre = pre->right;
            pre->right = cur->right;
            cur->right = cur->left;
            cur->left = nullptr;
        }
        cur = cur->right;
    }
}
```

### 11.3 Connect Next Right Pointers (perfect tree)

```cpp
struct Node {
    int val; Node* left; Node* right; Node* next;
};

Node* connect(Node* root) {
    Node* leftmost = root;
    while (leftmost && leftmost->left) {
        Node* head = leftmost;
        while (head) {
            head->left->next = head->right;
            if (head->next) head->right->next = head->next->left;
            head = head->next;
        }
        leftmost = leftmost->left;
    }
    return root;
}
```

---

## 12. Misc & Advanced Patterns

### 12.1 All Nodes at Distance K from Target

```cpp
vector<int> distanceK(TreeNode* root, TreeNode* target, int k) {
    unordered_map<TreeNode*, TreeNode*> parent;
    function<void(TreeNode*, TreeNode*)> mapParents =
        [&](TreeNode* node, TreeNode* par) {
            if (!node) return;
            parent[node] = par;
            mapParents(node->left, node);
            mapParents(node->right, node);
        };
    mapParents(root, nullptr);

    unordered_set<TreeNode*> visited;
    queue<TreeNode*> q; q.push(target);
    visited.insert(target);
    int dist = 0;
    while (!q.empty()) {
        if (dist == k) break;
        int sz = q.size();
        for (int i = 0; i < sz; ++i) {
            TreeNode* node = q.front(); q.pop();
            for (TreeNode* nxt : {node->left, node->right, parent[node]}) {
                if (nxt && !visited.count(nxt)) {
                    visited.insert(nxt);
                    q.push(nxt);
                }
            }
        }
        ++dist;
    }
    vector<int> res;
    while (!q.empty()) { res.push_back(q.front()->val); q.pop(); }
    return res;
}
```

### 12.2 Count Nodes in Complete Binary Tree (O(log²n))

```cpp
int countNodesComplete(TreeNode* root) {
    if (!root) return 0;
    int lh = 0, rh = 0;
    for (TreeNode* n = root; n; n = n->left)  ++lh;
    for (TreeNode* n = root; n; n = n->right) ++rh;
    if (lh == rh) return (1 << lh) - 1;        // perfect subtree
    return 1 + countNodesComplete(root->left) + countNodesComplete(root->right);
}
```

### 12.3 Burning Tree / Time to Infect (Amazon favorite)

> Same template as 12.1: build parent map, BFS from target, the number of
> BFS levels until the queue empties is the answer.

### 12.4 Children Sum Property Check

```cpp
bool childrenSum(TreeNode* root) {
    if (!root || (!root->left && !root->right)) return true;
    int sum = 0;
    if (root->left)  sum += root->left->val;
    if (root->right) sum += root->right->val;
    return root->val == sum && childrenSum(root->left) && childrenSum(root->right);
}
```

### 12.5 Maximum Width of Binary Tree

```cpp
int widthOfBinaryTree(TreeNode* root) {
    if (!root) return 0;
    long long maxWidth = 0;
    queue<pair<TreeNode*, unsigned long long>> q;
    q.push({root, 0});
    while (!q.empty()) {
        int sz = q.size();
        unsigned long long first = q.front().second, last = first;
        for (int i = 0; i < sz; ++i) {
            auto [node, idx] = q.front(); q.pop();
            last = idx;
            unsigned long long base = idx - first;        // normalize to avoid overflow
            if (node->left)  q.push({node->left,  base * 2});
            if (node->right) q.push({node->right, base * 2 + 1});
        }
        maxWidth = max(maxWidth, (long long)(last - first + 1));
    }
    return (int)maxWidth;
}
```

---

## 13. Complexity Quick Reference

| Operation                         | Time        | Space (aux)      |
|-----------------------------------|-------------|------------------|
| Traversal (any)                   | O(n)        | O(h) rec / O(n) iter |
| Morris traversal                  | O(n)        | O(1)             |
| Level order / BFS                 | O(n)        | O(w) width       |
| Height / Diameter / Balanced      | O(n)        | O(h)             |
| Build from traversals             | O(n)        | O(n)             |
| LCA (binary tree)                 | O(n)        | O(h)             |
| BST search / insert / delete      | O(h)        | O(h)             |
| BST validate / kth smallest       | O(n)        | O(h)             |
| Serialize / Deserialize           | O(n)        | O(n)             |
| Nodes at distance K               | O(n)        | O(n)             |
| Count nodes (complete tree)       | O(log² n)   | O(log n)         |

> `n` = number of nodes, `h` = height (O(log n) balanced, O(n) skewed),
> `w` = max width of a level.

---

### Interview Tips

- **Default to recursion** for tree problems; convert to iterative only when the
  interviewer asks about stack overflow / O(1) space (then mention Morris).
- **Most "through node" problems** (diameter, max path sum) use the pattern:
  *return the best one-sided value, update a global with the two-sided value.*
- **BST problems** almost always reduce to an **inorder traversal** producing a
  sorted sequence.
- **"Distance / nearest / burning"** problems → convert the tree to an undirected
  graph via a parent map, then BFS.
- Watch for **integer overflow** in path-sum and width problems (use `long long`).

---

*Happy coding!*
