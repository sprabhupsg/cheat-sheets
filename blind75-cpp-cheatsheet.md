# Blind 75 — C++ Cheat Sheet

A compact, interview-ready reference for the **Blind 75** LeetCode problems with idiomatic C++ solutions, time/space complexity, and the pattern behind each. Problems are numbered **1–75**.

**Coverage: 75 / 75 problems.** Five LeetCode-premium problems (Alien Dictionary, Meeting Rooms I & II, Graph Valid Tree, Number of Connected Components) are included as approach summaries plus a reusable Union-Find snippet rather than full boilerplate.

> Conventions used in snippets: `using namespace std;` is assumed. Vectors are passed by `const&` unless mutated. `int` overflow guarded with `long long` where noted.

---

## Problem count by category

| # | Category | Problems | Numbers |
|---|---|---|---|
| 2 | Arrays & Hashing | 8 | 1–8 |
| 3 | Two Pointers | 4 | 9–12 |
| 4 | Sliding Window | 4 | 13–16 |
| 5 | Stack | 1 | 17 |
| 6 | Binary Search | 2 | 18–19 |
| 7 | Linked List | 6 | 20–25 |
| 8 | Trees | 11 | 26–36 |
| 9 | Tries | 3 | 37–39 |
| 10 | Heap / Priority Queue | 1 | 40 |
| 11 | Backtracking | 2 | 41–42 |
| 12 | Graphs | 7 | 43–49 |
| 13 | Dynamic Programming | 13 | 50–62 |
| 14 | Intervals | 5 | 63–67 |
| 15 | Bit Manipulation | 5 | 68–72 |
| 16 | Math & Matrix | 3 | 73–75 |
| | **Total** | **75** | **1–75** |

---

## Table of Contents

1. [C++ STL & Pattern Primer](#cpp-primer)
2. [Arrays & Hashing](#arrays)
3. [Two Pointers](#two-pointers)
4. [Sliding Window](#sliding-window)
5. [Stack](#stack)
6. [Binary Search](#binary-search)
7. [Linked List](#linked-list)
8. [Trees](#trees)
9. [Tries](#tries)
10. [Heap / Priority Queue](#heap)
11. [Backtracking](#backtracking)
12. [Graphs](#graphs)
13. [Dynamic Programming](#dp)
14. [Intervals](#intervals)
15. [Bit Manipulation](#bits)
16. [Math & Matrix](#matrix)

---

<a id="cpp-primer"></a>
## 1. C++ STL & Pattern Primer

### Containers you reach for constantly

```cpp
unordered_map<int,int> mp;        // O(1) avg hash map
unordered_set<int> seen;          // O(1) avg hash set
map<int,int> ordered;             // O(log n), sorted keys
set<int> os;                      // sorted unique
vector<int> v(n, 0);              // dynamic array
deque<int> dq;                    // double-ended queue (sliding window, BFS)
stack<int> st;                    // LIFO
queue<int> q;                     // FIFO (BFS)
priority_queue<int> maxHeap;      // max-heap by default
priority_queue<int, vector<int>, greater<int>> minHeap;
```

### High-value idioms

```cpp
// Iterate map
for (auto& [k, val] : mp) { ... }

// Count frequencies
unordered_map<char,int> cnt;
for (char c : s) cnt[c]++;

// Sort with custom comparator
sort(v.begin(), v.end(), [](auto& a, auto& b){ return a[0] < b[0]; });

// Min/max in place
res = max(res, cur);

// Pair / tuple
priority_queue<pair<int,int>> pq;       // sorts by .first then .second
auto [a, b] = pq.top();

// INT bounds
INT_MAX, INT_MIN, LLONG_MAX            // <climits>
```

### Pattern decision guide

| Signal in the prompt | Likely pattern |
|---|---|
| "sorted array", "find target/boundary" | Binary Search |
| "contiguous subarray/substring", "window" | Sliding Window |
| "pair/triple summing to X" in sorted data | Two Pointers |
| "next greater", "valid parentheses", "monotonic" | Stack |
| "all combinations/permutations/subsets" | Backtracking |
| "grid/islands", "dependencies", "shortest path" | BFS/DFS/Graph |
| "min/max ways", "longest/optimal subsequence" | DP |
| "k largest/smallest", "median of stream" | Heap |
| "prefix matching", "dictionary of words" | Trie |
| "overlapping ranges" | Intervals + sort |

---

<a id="arrays"></a>
## 2. Arrays & Hashing

### 1. Two Sum — `O(n)` / `O(n)`
Hash value→index; check complement before inserting.
```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int,int> seen;
    for (int i = 0; i < nums.size(); i++) {
        int need = target - nums[i];
        if (seen.count(need)) return {seen[need], i};
        seen[nums[i]] = i;
    }
    return {};
}
```

### 2. Contains Duplicate — `O(n)` / `O(n)`
```cpp
bool containsDuplicate(vector<int>& nums) {
    unordered_set<int> s;
    for (int x : nums) if (!s.insert(x).second) return true;
    return false;
}
```

### 3. Valid Anagram — `O(n)` / `O(1)`
```cpp
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;
    int cnt[26] = {0};
    for (char c : s) cnt[c-'a']++;
    for (char c : t) if (--cnt[c-'a'] < 0) return false;
    return true;
}
```

### 4. Group Anagrams — `O(n·k log k)` / `O(n·k)`
Key = sorted string (or 26-count signature).
```cpp
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    unordered_map<string, vector<string>> mp;
    for (auto& s : strs) { string k = s; sort(k.begin(), k.end()); mp[k].push_back(s); }
    vector<vector<string>> res;
    for (auto& [k, v] : mp) res.push_back(v);
    return res;
}
```

### 5. Top K Frequent Elements — `O(n)` / `O(n)`
Bucket sort by frequency.
```cpp
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> cnt;
    for (int x : nums) cnt[x]++;
    vector<vector<int>> bucket(nums.size()+1);
    for (auto& [v, c] : cnt) bucket[c].push_back(v);
    vector<int> res;
    for (int c = bucket.size()-1; c >= 0 && res.size() < k; c--)
        for (int v : bucket[c]) { res.push_back(v); if (res.size()==k) break; }
    return res;
}
```

### 6. Product of Array Except Self — `O(n)` / `O(1)` extra
Prefix then suffix pass.
```cpp
vector<int> productExceptSelf(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, 1);
    for (int i = 1; i < n; i++) res[i] = res[i-1] * nums[i-1];
    int suf = 1;
    for (int i = n-1; i >= 0; i--) { res[i] *= suf; suf *= nums[i]; }
    return res;
}
```

### 7. Encode and Decode Strings — `O(n)` / `O(1)`
Length-prefix each chunk: `"len#payload"`.
```cpp
string encode(vector<string>& strs) {
    string r;
    for (auto& s : strs) r += to_string(s.size()) + "#" + s;
    return r;
}
vector<string> decode(string s) {
    vector<string> res; int i = 0;
    while (i < s.size()) {
        int j = i; while (s[j] != '#') j++;
        int len = stoi(s.substr(i, j-i));
        res.push_back(s.substr(j+1, len));
        i = j + 1 + len;
    }
    return res;
}
```

### 8. Longest Consecutive Sequence — `O(n)` / `O(n)`
Only start counting from sequence beginnings.
```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> s(nums.begin(), nums.end());
    int best = 0;
    for (int x : s) {
        if (s.count(x-1)) continue;          // not a start
        int len = 1;
        while (s.count(x+len)) len++;
        best = max(best, len);
    }
    return best;
}
```

---

<a id="two-pointers"></a>
## 3. Two Pointers

### 9. Valid Palindrome — `O(n)` / `O(1)`
```cpp
bool isPalindrome(string s) {
    int l = 0, r = s.size()-1;
    while (l < r) {
        while (l < r && !isalnum(s[l])) l++;
        while (l < r && !isalnum(s[r])) r--;
        if (tolower(s[l++]) != tolower(s[r--])) return false;
    }
    return true;
}
```

### 10. Two Sum II (sorted) — `O(n)` / `O(1)`
```cpp
vector<int> twoSum(vector<int>& a, int target) {
    int l = 0, r = a.size()-1;
    while (l < r) {
        int sum = a[l] + a[r];
        if (sum == target) return {l+1, r+1};
        sum < target ? l++ : r--;
    }
    return {};
}
```

### 11. 3Sum — `O(n^2)` / `O(1)`
Sort; fix one, two-pointer the rest; skip duplicates.
```cpp
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> res;
    int n = nums.size();
    for (int i = 0; i < n-2; i++) {
        if (i && nums[i] == nums[i-1]) continue;
        int l = i+1, r = n-1;
        while (l < r) {
            int s = nums[i] + nums[l] + nums[r];
            if (s < 0) l++;
            else if (s > 0) r--;
            else {
                res.push_back({nums[i], nums[l], nums[r]});
                while (l < r && nums[l] == nums[l+1]) l++;
                while (l < r && nums[r] == nums[r-1]) r--;
                l++; r--;
            }
        }
    }
    return res;
}
```

### 12. Container With Most Water — `O(n)` / `O(1)`
Move the shorter wall inward.
```cpp
int maxArea(vector<int>& h) {
    int l = 0, r = h.size()-1, best = 0;
    while (l < r) {
        best = max(best, (r-l) * min(h[l], h[r]));
        h[l] < h[r] ? l++ : r--;
    }
    return best;
}
```

---

<a id="sliding-window"></a>
## 4. Sliding Window

### 13. Best Time to Buy and Sell Stock — `O(n)` / `O(1)`
Track min price so far.
```cpp
int maxProfit(vector<int>& prices) {
    int minP = INT_MAX, best = 0;
    for (int p : prices) { minP = min(minP, p); best = max(best, p - minP); }
    return best;
}
```

### 14. Longest Substring Without Repeating Characters — `O(n)` / `O(1)`
```cpp
int lengthOfLongestSubstring(string s) {
    int last[128]; fill(last, last+128, -1);
    int start = 0, best = 0;
    for (int i = 0; i < s.size(); i++) {
        if (last[s[i]] >= start) start = last[s[i]] + 1;
        last[s[i]] = i;
        best = max(best, i - start + 1);
    }
    return best;
}
```

### 15. Longest Repeating Character Replacement — `O(n)` / `O(1)`
Window valid while `(len - maxFreq) <= k`.
```cpp
int characterReplacement(string s, int k) {
    int cnt[26] = {0}, l = 0, maxF = 0, best = 0;
    for (int r = 0; r < s.size(); r++) {
        maxF = max(maxF, ++cnt[s[r]-'A']);
        while ((r - l + 1) - maxF > k) cnt[s[l++]-'A']--;
        best = max(best, r - l + 1);
    }
    return best;
}
```

### 16. Minimum Window Substring — `O(n)` / `O(1)`
Expand to satisfy, then shrink to minimize.
```cpp
string minWindow(string s, string t) {
    int need[128] = {0}, required = t.size();
    for (char c : t) need[c]++;
    int l = 0, bestLen = INT_MAX, bestL = 0;
    for (int r = 0; r < s.size(); r++) {
        if (need[s[r]]-- > 0) required--;
        while (required == 0) {
            if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestL = l; }
            if (++need[s[l++]] > 0) required++;
        }
    }
    return bestLen == INT_MAX ? "" : s.substr(bestL, bestLen);
}
```

---

<a id="stack"></a>
## 5. Stack

### 17. Valid Parentheses — `O(n)` / `O(n)`
```cpp
bool isValid(string s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') st.push(c);
        else {
            if (st.empty()) return false;
            char t = st.top(); st.pop();
            if ((c==')'&&t!='(')||(c==']'&&t!='[')||(c=='}'&&t!='{')) return false;
        }
    }
    return st.empty();
}
```

> Other classic stack problems (Min Stack, Daily Temperatures, Car Fleet) use the same monotonic-stack idea: keep a stack that stays increasing/decreasing and pop when the invariant breaks.

---

<a id="binary-search"></a>
## 6. Binary Search

### 18. Find Minimum in Rotated Sorted Array — `O(log n)` / `O(1)`
```cpp
int findMin(vector<int>& a) {
    int l = 0, r = a.size()-1;
    while (l < r) {
        int m = l + (r-l)/2;
        if (a[m] > a[r]) l = m + 1;   // min is to the right
        else r = m;
    }
    return a[l];
}
```

### 19. Search in Rotated Sorted Array — `O(log n)` / `O(1)`
```cpp
int search(vector<int>& a, int target) {
    int l = 0, r = a.size()-1;
    while (l <= r) {
        int m = l + (r-l)/2;
        if (a[m] == target) return m;
        if (a[l] <= a[m]) {                       // left half sorted
            if (a[l] <= target && target < a[m]) r = m-1; else l = m+1;
        } else {                                  // right half sorted
            if (a[m] < target && target <= a[r]) l = m+1; else r = m-1;
        }
    }
    return -1;
}
```

> Generic template: use `l + (r-l)/2` to avoid overflow; decide `l = m+1` vs `r = m` (or `r = m-1`) based on whether `m` can still be the answer.

---

<a id="linked-list"></a>
## 7. Linked List

`struct ListNode { int val; ListNode* next; };`

### 20. Reverse Linked List — `O(n)` / `O(1)`
```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    while (head) { ListNode* nxt = head->next; head->next = prev; prev = head; head = nxt; }
    return prev;
}
```

### 21. Merge Two Sorted Lists — `O(n+m)` / `O(1)`
```cpp
ListNode* mergeTwoLists(ListNode* a, ListNode* b) {
    ListNode dummy, *tail = &dummy;
    while (a && b) {
        if (a->val <= b->val) { tail->next = a; a = a->next; }
        else { tail->next = b; b = b->next; }
        tail = tail->next;
    }
    tail->next = a ? a : b;
    return dummy.next;
}
```

### 22. Linked List Cycle — `O(n)` / `O(1)`
Floyd's fast/slow.
```cpp
bool hasCycle(ListNode* head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next; fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

### 23. Remove Nth Node From End — `O(n)` / `O(1)`
```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy; dummy.next = head;
    ListNode *fast = &dummy, *slow = &dummy;
    while (n--) fast = fast->next;
    while (fast->next) { fast = fast->next; slow = slow->next; }
    slow->next = slow->next->next;
    return dummy.next;
}
```

### 24. Reorder List — `O(n)` / `O(1)`
Find middle → reverse second half → merge alternately.
```cpp
void reorderList(ListNode* head) {
    if (!head || !head->next) return;
    ListNode *slow = head, *fast = head;
    while (fast->next && fast->next->next) { slow = slow->next; fast = fast->next->next; }
    ListNode *second = slow->next, *prev = nullptr; slow->next = nullptr;
    while (second) { ListNode* nxt = second->next; second->next = prev; prev = second; second = nxt; }
    ListNode *first = head; second = prev;
    while (second) {
        ListNode *t1 = first->next, *t2 = second->next;
        first->next = second; second->next = t1;
        first = t1; second = t2;
    }
}
```

### 25. Merge K Sorted Lists — `O(N log k)` / `O(k)`
Min-heap of list heads.
```cpp
ListNode* mergeKLists(vector<ListNode*>& lists) {
    auto cmp = [](ListNode* a, ListNode* b){ return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
    for (auto l : lists) if (l) pq.push(l);
    ListNode dummy, *tail = &dummy;
    while (!pq.empty()) {
        ListNode* n = pq.top(); pq.pop();
        tail->next = n; tail = n;
        if (n->next) pq.push(n->next);
    }
    return dummy.next;
}
```

---

<a id="trees"></a>
## 8. Trees

`struct TreeNode { int val; TreeNode *left, *right; };`

### 26. Maximum Depth — `O(n)` / `O(h)`
```cpp
int maxDepth(TreeNode* root) {
    return root ? 1 + max(maxDepth(root->left), maxDepth(root->right)) : 0;
}
```

### 27. Same Tree — `O(n)` / `O(h)`
```cpp
bool isSameTree(TreeNode* p, TreeNode* q) {
    if (!p || !q) return p == q;
    return p->val == q->val && isSameTree(p->left,q->left) && isSameTree(p->right,q->right);
}
```

### 28. Invert Binary Tree — `O(n)` / `O(h)`
```cpp
TreeNode* invertTree(TreeNode* root) {
    if (!root) return nullptr;
    swap(root->left, root->right);
    invertTree(root->left); invertTree(root->right);
    return root;
}
```

### 29. Subtree of Another Tree — `O(n·m)` / `O(h)`
```cpp
bool isSame(TreeNode* a, TreeNode* b){
    if(!a||!b) return a==b;
    return a->val==b->val && isSame(a->left,b->left) && isSame(a->right,b->right);
}
bool isSubtree(TreeNode* root, TreeNode* sub) {
    if (!root) return false;
    return isSame(root, sub) || isSubtree(root->left, sub) || isSubtree(root->right, sub);
}
```

### 30. Binary Tree Level Order Traversal — `O(n)` / `O(n)`
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        int sz = q.size(); vector<int> lvl;
        while (sz--) {
            TreeNode* n = q.front(); q.pop();
            lvl.push_back(n->val);
            if (n->left) q.push(n->left);
            if (n->right) q.push(n->right);
        }
        res.push_back(lvl);
    }
    return res;
}
```

### 31. Validate BST — `O(n)` / `O(h)`
Pass down (min, max) bounds.
```cpp
bool valid(TreeNode* n, long lo, long hi) {
    if (!n) return true;
    if (n->val <= lo || n->val >= hi) return false;
    return valid(n->left, lo, n->val) && valid(n->right, n->val, hi);
}
bool isValidBST(TreeNode* root){ return valid(root, LONG_MIN, LONG_MAX); }
```

### 32. Kth Smallest in BST — `O(h+k)` / `O(h)`
In-order traversal.
```cpp
int kthSmallest(TreeNode* root, int k) {
    stack<TreeNode*> st; TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }
        cur = st.top(); st.pop();
        if (--k == 0) return cur->val;
        cur = cur->right;
    }
    return -1;
}
```

### 33. Lowest Common Ancestor of BST — `O(h)` / `O(1)`
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    while (root) {
        if (p->val < root->val && q->val < root->val) root = root->left;
        else if (p->val > root->val && q->val > root->val) root = root->right;
        else return root;
    }
    return nullptr;
}
```

### 34. Construct Tree from Preorder & Inorder — `O(n)` / `O(n)`
```cpp
TreeNode* build(vector<int>& pre, int& pi, unordered_map<int,int>& idx, int l, int r) {
    if (l > r) return nullptr;
    TreeNode* root = new TreeNode(pre[pi]);
    int m = idx[pre[pi++]];
    root->left = build(pre, pi, idx, l, m-1);
    root->right = build(pre, pi, idx, m+1, r);
    return root;
}
TreeNode* buildTree(vector<int>& pre, vector<int>& in) {
    unordered_map<int,int> idx;
    for (int i = 0; i < in.size(); i++) idx[in[i]] = i;
    int pi = 0;
    return build(pre, pi, idx, 0, in.size()-1);
}
```

### 35. Binary Tree Maximum Path Sum — `O(n)` / `O(h)`
```cpp
int best = INT_MIN;
int gain(TreeNode* n) {
    if (!n) return 0;
    int l = max(0, gain(n->left)), r = max(0, gain(n->right));
    best = max(best, n->val + l + r);
    return n->val + max(l, r);
}
int maxPathSum(TreeNode* root){ best = INT_MIN; gain(root); return best; }
```

### 36. Serialize and Deserialize — `O(n)` / `O(n)`
Pre-order with null markers.
```cpp
void ser(TreeNode* n, string& s){
    if(!n){ s += "#,"; return; }
    s += to_string(n->val) + ","; ser(n->left,s); ser(n->right,s);
}
string serialize(TreeNode* root){ string s; ser(root,s); return s; }
TreeNode* des(int& i, string& s){
    if (s[i] == '#'){ i += 2; return nullptr; }
    int j = s.find(',', i); int val = stoi(s.substr(i, j-i)); i = j+1;
    TreeNode* n = new TreeNode(val);
    n->left = des(i,s); n->right = des(i,s); return n;
}
TreeNode* deserialize(string data){ int i = 0; return des(i, data); }
```

---

<a id="tries"></a>
## 9. Tries

### 37. Implement Trie — insert/search/startsWith `O(L)`
```cpp
struct Trie {
    Trie* ch[26] = {};
    bool end = false;
    void insert(string w) {
        Trie* n = this;
        for (char c : w) { if (!n->ch[c-'a']) n->ch[c-'a'] = new Trie(); n = n->ch[c-'a']; }
        n->end = true;
    }
    Trie* find(const string& w) {
        Trie* n = this;
        for (char c : w) { if (!n->ch[c-'a']) return nullptr; n = n->ch[c-'a']; }
        return n;
    }
    bool search(string w){ Trie* n = find(w); return n && n->end; }
    bool startsWith(string p){ return find(p); }
};
```

### 38. Add and Search Word (`.` wildcard) — `O(26^k)` worst
```cpp
struct WordDictionary {
    WordDictionary* ch[26] = {}; bool end = false;
    void addWord(string w){
        WordDictionary* n = this;
        for (char c : w){ if(!n->ch[c-'a']) n->ch[c-'a']=new WordDictionary(); n=n->ch[c-'a']; }
        n->end = true;
    }
    bool search(string w){ return dfs(w, 0, this); }
    bool dfs(const string& w, int i, WordDictionary* n){
        if (!n) return false;
        if (i == w.size()) return n->end;
        if (w[i] == '.') {
            for (int c = 0; c < 26; c++) if (dfs(w, i+1, n->ch[c])) return true;
            return false;
        }
        return dfs(w, i+1, n->ch[w[i]-'a']);
    }
};
```

### 39. Word Search II — Trie + grid DFS
Build a trie of words, DFS the board pruning by trie edges; mark found words at terminal nodes to dedupe.

---

<a id="heap"></a>
## 10. Heap / Priority Queue

### 40. Find Median from Data Stream — add `O(log n)`, find `O(1)`
Max-heap (low half) + min-heap (high half), kept balanced.
```cpp
class MedianFinder {
    priority_queue<int> lo;                              // max-heap
    priority_queue<int, vector<int>, greater<int>> hi;   // min-heap
public:
    void addNum(int num) {
        lo.push(num);
        hi.push(lo.top()); lo.pop();
        if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
    }
    double findMedian() {
        return lo.size() > hi.size() ? lo.top() : (lo.top() + hi.top()) / 2.0;
    }
};
```

---

<a id="backtracking"></a>
## 11. Backtracking

General shape: choose → recurse → undo.

### 41. Combination Sum (reuse allowed) — `O(2^t)`
```cpp
void dfs(vector<int>& c, int start, int target, vector<int>& cur, vector<vector<int>>& res){
    if (target == 0) { res.push_back(cur); return; }
    for (int i = start; i < c.size(); i++) {
        if (c[i] > target) continue;
        cur.push_back(c[i]);
        dfs(c, i, target - c[i], cur, res);   // i, not i+1 → reuse
        cur.pop_back();
    }
}
vector<vector<int>> combinationSum(vector<int>& cand, int target){
    sort(cand.begin(), cand.end());
    vector<vector<int>> res; vector<int> cur;
    dfs(cand, 0, target, cur, res); return res;
}
```

### 42. Word Search — `O(m·n·4^L)`
```cpp
bool dfs(vector<vector<char>>& b, int r, int c, string& w, int k){
    if (k == w.size()) return true;
    if (r < 0 || c < 0 || r >= b.size() || c >= b[0].size() || b[r][c] != w[k]) return false;
    char tmp = b[r][c]; b[r][c] = '#';
    bool found = dfs(b,r+1,c,w,k+1) || dfs(b,r-1,c,w,k+1)
              || dfs(b,r,c+1,w,k+1) || dfs(b,r,c-1,w,k+1);
    b[r][c] = tmp;
    return found;
}
bool exist(vector<vector<char>>& b, string w){
    for (int r = 0; r < b.size(); r++)
        for (int c = 0; c < b[0].size(); c++)
            if (dfs(b, r, c, w, 0)) return true;
    return false;
}
```

> Same template covers Subsets, Permutations, Combinations, Palindrome Partitioning, Letter Combinations.

---

<a id="graphs"></a>
## 12. Graphs

### 43. Number of Islands — BFS/DFS flood fill — `O(m·n)`
```cpp
void sink(vector<vector<char>>& g, int r, int c){
    if (r<0||c<0||r>=g.size()||c>=g[0].size()||g[r][c]!='1') return;
    g[r][c] = '0';
    sink(g,r+1,c); sink(g,r-1,c); sink(g,r,c+1); sink(g,r,c-1);
}
int numIslands(vector<vector<char>>& g){
    int cnt = 0;
    for (int r = 0; r < g.size(); r++)
        for (int c = 0; c < g[0].size(); c++)
            if (g[r][c] == '1') { cnt++; sink(g, r, c); }
    return cnt;
}
```

### 44. Clone Graph — `O(V+E)`
```cpp
Node* cloneGraph(Node* node) {
    if (!node) return nullptr;
    unordered_map<Node*, Node*> mp;
    function<Node*(Node*)> dfs = [&](Node* n) {
        if (mp.count(n)) return mp[n];
        Node* copy = new Node(n->val);
        mp[n] = copy;
        for (Node* nb : n->neighbors) copy->neighbors.push_back(dfs(nb));
        return copy;
    };
    return dfs(node);
}
```

### 45. Course Schedule — cycle detection (Kahn's topo sort) — `O(V+E)`
```cpp
bool canFinish(int n, vector<vector<int>>& pre) {
    vector<vector<int>> adj(n);
    vector<int> indeg(n, 0);
    for (auto& p : pre) { adj[p[1]].push_back(p[0]); indeg[p[0]]++; }
    queue<int> q;
    for (int i = 0; i < n; i++) if (!indeg[i]) q.push(i);
    int done = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop(); done++;
        for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
    }
    return done == n;
}
```

### 46. Pacific Atlantic Water Flow — `O(m·n)`
DFS inward from each ocean's border; answer = cells reachable from both.

### 47. Number of Connected Components (premium) — Union-Find `O(E·α)`
Count distinct roots after uniting every edge (see DSU below).

### 48. Graph Valid Tree (premium) — Union-Find `O(E·α)`
Valid tree iff `edges == n-1` **and** no union ever merges two already-connected nodes (no cycle).

```cpp
struct DSU {
    vector<int> p;
    DSU(int n): p(n) { iota(p.begin(), p.end(), 0); }
    int find(int x){ return p[x]==x ? x : p[x]=find(p[x]); }
    bool unite(int a, int b){ a=find(a); b=find(b); if(a==b) return false; p[a]=b; return true; }
};
// #47 components: start count at n, decrement on each successful unite().
// #48 valid tree: edges == n-1 AND every unite() returns true.
```

### 49. Alien Dictionary (premium) — topological sort `O(C)`
Build a graph from the first differing character between adjacent words, then topo-sort the characters (Kahn's / DFS). Detect a cycle ⇒ invalid ordering; watch the prefix edge case (`"abc"` before `"ab"` is invalid).

---

<a id="dp"></a>
## 13. Dynamic Programming

### 50. Climbing Stairs — `O(n)` / `O(1)`
```cpp
int climbStairs(int n) {
    int a = 1, b = 1;
    for (int i = 2; i <= n; i++) { int c = a + b; a = b; b = c; }
    return b;
}
```

### 51. House Robber — `O(n)` / `O(1)`
```cpp
int rob(vector<int>& nums) {
    int prev = 0, cur = 0;
    for (int x : nums) { int t = max(cur, prev + x); prev = cur; cur = t; }
    return cur;
}
```

### 52. House Robber II (circular) — `O(n)` / `O(1)`
Rob `[0..n-2]` or `[1..n-1]`, take the max.
```cpp
int robLine(vector<int>& a, int l, int r){
    int prev = 0, cur = 0;
    for (int i = l; i <= r; i++){ int t = max(cur, prev + a[i]); prev = cur; cur = t; }
    return cur;
}
int rob(vector<int>& n){
    if (n.size() == 1) return n[0];
    return max(robLine(n, 0, n.size()-2), robLine(n, 1, n.size()-1));
}
```

### 53. Coin Change (min coins) — `O(n·amount)`
```cpp
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount+1, amount+1);
    dp[0] = 0;
    for (int a = 1; a <= amount; a++)
        for (int c : coins) if (c <= a) dp[a] = min(dp[a], dp[a-c] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
}
```

### 54. Longest Increasing Subsequence — `O(n log n)`
Patience sorting via `lower_bound`.
```cpp
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    for (int x : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) tails.push_back(x);
        else *it = x;
    }
    return tails.size();
}
```

### 55. Longest Common Subsequence — `O(n·m)`
```cpp
int longestCommonSubsequence(string a, string b) {
    int n = a.size(), m = b.size();
    vector<vector<int>> dp(n+1, vector<int>(m+1, 0));
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            dp[i][j] = a[i-1]==b[j-1] ? dp[i-1][j-1]+1 : max(dp[i-1][j], dp[i][j-1]);
    return dp[n][m];
}
```

### 56. Word Break — `O(n^2)`
```cpp
bool wordBreak(string s, vector<string>& dict) {
    unordered_set<string> w(dict.begin(), dict.end());
    vector<bool> dp(s.size()+1, false); dp[0] = true;
    for (int i = 1; i <= s.size(); i++)
        for (int j = 0; j < i; j++)
            if (dp[j] && w.count(s.substr(j, i-j))) { dp[i] = true; break; }
    return dp[s.size()];
}
```

### 57. Combination Sum IV (count orderings) — `O(n·target)`
```cpp
int combinationSum4(vector<int>& nums, int target) {
    vector<unsigned> dp(target+1, 0); dp[0] = 1;
    for (int t = 1; t <= target; t++)
        for (int x : nums) if (x <= t) dp[t] += dp[t-x];
    return dp[target];
}
```

### 58. Decode Ways — `O(n)` / `O(1)`
```cpp
int numDecodings(string s) {
    if (s[0] == '0') return 0;
    int prev = 1, cur = 1;
    for (int i = 1; i < s.size(); i++) {
        int t = 0;
        if (s[i] != '0') t += cur;
        int two = (s[i-1]-'0')*10 + (s[i]-'0');
        if (two >= 10 && two <= 26) t += prev;
        prev = cur; cur = t;
    }
    return cur;
}
```

### 59. Unique Paths — `O(m·n)` / `O(n)`
```cpp
int uniquePaths(int m, int n) {
    vector<int> dp(n, 1);
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++) dp[j] += dp[j-1];
    return dp[n-1];
}
```

### 60. Jump Game — greedy `O(n)`
```cpp
bool canJump(vector<int>& nums) {
    int reach = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (i > reach) return false;
        reach = max(reach, i + nums[i]);
    }
    return true;
}
```

### 61. Maximum Subarray (Kadane) — `O(n)`
```cpp
int maxSubArray(vector<int>& nums) {
    int cur = nums[0], best = nums[0];
    for (int i = 1; i < nums.size(); i++) {
        cur = max(nums[i], cur + nums[i]);
        best = max(best, cur);
    }
    return best;
}
```

### 62. Maximum Product Subarray — `O(n)`
Track both max and min (negatives flip).
```cpp
int maxProduct(vector<int>& nums) {
    int best = nums[0], hi = nums[0], lo = nums[0];
    for (int i = 1; i < nums.size(); i++) {
        if (nums[i] < 0) swap(hi, lo);
        hi = max(nums[i], hi * nums[i]);
        lo = min(nums[i], lo * nums[i]);
        best = max(best, hi);
    }
    return best;
}
```

---

<a id="intervals"></a>
## 14. Intervals

### 63. Merge Intervals — `O(n log n)`
```cpp
vector<vector<int>> merge(vector<vector<int>>& iv) {
    sort(iv.begin(), iv.end());
    vector<vector<int>> res;
    for (auto& i : iv) {
        if (!res.empty() && i[0] <= res.back()[1]) res.back()[1] = max(res.back()[1], i[1]);
        else res.push_back(i);
    }
    return res;
}
```

### 64. Insert Interval — `O(n)`
```cpp
vector<vector<int>> insert(vector<vector<int>>& iv, vector<int> nw) {
    vector<vector<int>> res; int i = 0, n = iv.size();
    while (i < n && iv[i][1] < nw[0]) res.push_back(iv[i++]);
    while (i < n && iv[i][0] <= nw[1]) { nw[0]=min(nw[0],iv[i][0]); nw[1]=max(nw[1],iv[i][1]); i++; }
    res.push_back(nw);
    while (i < n) res.push_back(iv[i++]);
    return res;
}
```

### 65. Non-overlapping Intervals (min removals) — `O(n log n)`
Greedy by earliest end.
```cpp
int eraseOverlapIntervals(vector<vector<int>>& iv) {
    sort(iv.begin(), iv.end(), [](auto&a, auto&b){ return a[1] < b[1]; });
    int end = INT_MIN, removed = 0;
    for (auto& i : iv) {
        if (i[0] >= end) end = i[1];
        else removed++;
    }
    return removed;
}
```

### 66. Meeting Rooms (premium) — can attend all?
Sort by start; if any `start < prev end` → false.

### 67. Meeting Rooms II (premium) — min rooms
Sort starts & ends separately; two pointers, increment rooms on start, decrement on end; track max.

---

<a id="bits"></a>
## 15. Bit Manipulation

### 68. Sum of Two Integers (no `+`) — `O(1)`
```cpp
int getSum(int a, int b) {
    while (b) {
        unsigned carry = (unsigned)(a & b) << 1;
        a = a ^ b;
        b = carry;
    }
    return a;
}
```

### 69. Number of 1 Bits — `O(#bits)`
Brian Kernighan: `n & (n-1)` clears lowest set bit.
```cpp
int hammingWeight(uint32_t n) {
    int c = 0;
    while (n) { n &= (n - 1); c++; }
    return c;
}
```

### 70. Counting Bits — `O(n)`
```cpp
vector<int> countBits(int n) {
    vector<int> dp(n+1, 0);
    for (int i = 1; i <= n; i++) dp[i] = dp[i >> 1] + (i & 1);
    return dp;
}
```

### 71. Missing Number — `O(n)` / `O(1)`
XOR all indices and values.
```cpp
int missingNumber(vector<int>& nums) {
    int x = nums.size();
    for (int i = 0; i < nums.size(); i++) x ^= i ^ nums[i];
    return x;
}
```

### 72. Reverse Bits — `O(1)`
```cpp
uint32_t reverseBits(uint32_t n) {
    uint32_t r = 0;
    for (int i = 0; i < 32; i++) { r = (r << 1) | (n & 1); n >>= 1; }
    return r;
}
```

---

<a id="matrix"></a>
## 16. Math & Matrix

### 73. Set Matrix Zeroes — `O(m·n)` / `O(1)`
Use first row/col as markers.
```cpp
void setZeroes(vector<vector<int>>& m) {
    int R = m.size(), C = m[0].size();
    bool col0 = false;
    for (int r = 0; r < R; r++) {
        if (m[r][0] == 0) col0 = true;
        for (int c = 1; c < C; c++) if (m[r][c] == 0) m[r][0] = m[0][c] = 0;
    }
    for (int r = R-1; r >= 0; r--) {
        for (int c = C-1; c >= 1; c--) if (m[r][0] == 0 || m[0][c] == 0) m[r][c] = 0;
        if (col0) m[r][0] = 0;
    }
}
```

### 74. Spiral Matrix — `O(m·n)`
```cpp
vector<int> spiralOrder(vector<vector<int>>& m) {
    vector<int> res;
    int top = 0, bot = m.size()-1, left = 0, right = m[0].size()-1;
    while (top <= bot && left <= right) {
        for (int c = left; c <= right; c++) res.push_back(m[top][c]);
        top++;
        for (int r = top; r <= bot; r++) res.push_back(m[r][right]);
        right--;
        if (top <= bot) { for (int c = right; c >= left; c--) res.push_back(m[bot][c]); bot--; }
        if (left <= right) { for (int r = bot; r >= top; r--) res.push_back(m[r][left]); left++; }
    }
    return res;
}
```

### 75. Rotate Image (90° clockwise) — `O(n^2)` / `O(1)`
Transpose then reverse each row.
```cpp
void rotate(vector<vector<int>>& m) {
    int n = m.size();
    for (int i = 0; i < n; i++)
        for (int j = i+1; j < n; j++) swap(m[i][j], m[j][i]);
    for (auto& row : m) reverse(row.begin(), row.end());
}
```

---

## Final-stretch reminders

- **Always state complexity** before coding; interviewers expect it.
- **Clarify constraints**: input size hints the target complexity (n ≤ 20 → exponential/backtracking OK; n ≤ 1e5 → need `O(n log n)` or better).
- **Edge cases**: empty input, single element, all duplicates, negatives, overflow (`long long`).
- **Hash map / set** is the default first thought for "have I seen this / count this".
- **Sort first** unlocks two-pointers, greedy intervals, and dedupe.
- **Recursion on trees**: define what the function returns for a subtree, handle `nullptr` base case.
- **Dry-run** a tiny example out loud before declaring done.

