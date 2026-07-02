# Heaps / Priority Queues in C++ — Coding Interview Cheat Sheet

A **heap** is a complete binary tree giving O(log n) insert/pop and O(1) peek at the min or max. In C++ you almost always use `std::priority_queue`. The interview skill is **recognizing when "top-k / kth / streaming / merge / scheduling" means heap**.

---

## 1. `std::priority_queue` Essentials

```cpp
#include <queue>

priority_queue<int> maxHeap;                          // MAX-heap (default)
priority_queue<int, vector<int>, greater<int>> minHeap; // MIN-heap

maxHeap.push(5);          // insert           O(log n)
maxHeap.top();            // peek largest     O(1)
maxHeap.pop();            // remove largest   O(log n)
maxHeap.size();
maxHeap.empty();
```

**Remember:** default `priority_queue` is a **MAX-heap**. Use `greater<>` for a min-heap.

### Build from a vector — O(n) heapify

```cpp
vector<int> v = {3, 1, 4, 1, 5};
priority_queue<int> pq(v.begin(), v.end());   // O(n), not O(n log n)
```

---

## 2. Custom Comparators (the part everyone forgets)

### Comparator rule of thumb
`priority_queue`'s comparator is **"reverse" of what you'd expect**: if `cmp(a, b)` returns `true`, then `a` goes *below* `b`. So `greater<>` → min-heap.

### Pairs / tuples

```cpp
// min-heap of (dist, node)
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
pq.push({dist, node});
```

### Lambda comparator

```cpp
auto cmp = [](const Item& a, const Item& b) {
    return a.priority > b.priority;   // '>' → smallest priority on top (min-heap)
};
priority_queue<Item, vector<Item>, decltype(cmp)> pq(cmp);
```

### Struct with `operator<` (for a max-heap by that ordering)

```cpp
struct Task {
    int priority;
    bool operator<(const Task& o) const { return priority < o.priority; }
};
priority_queue<Task> pq;   // top() = highest priority
```

---

## 3. Signature Patterns

### 3a. Top-K / Kth largest → keep a MIN-heap of size k

Counter-intuitive but key: to find the **k largest**, keep a **min-heap of size k**; pop the smallest whenever size exceeds k. The heap ends up holding the k largest, with the kth-largest on top.

```cpp
int findKthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> minHeap;
    for (int x : nums) {
        minHeap.push(x);
        if (minHeap.size() > k) minHeap.pop();   // drop smallest
    }
    return minHeap.top();                        // kth largest
}
```
Time O(n log k), space O(k). (For k smallest → keep a **max-heap** of size k.)

### 3b. Merge k sorted sequences → heap of "current front" of each

```cpp
// see LC 23 solution in section 6
```

### 3c. Two heaps → running median

Max-heap for the lower half, min-heap for the upper half; keep sizes balanced.

### 3d. Scheduling / greedy by "next available" → min-heap of end times

Meeting rooms, task scheduling, CPU intervals — push end time, pop when freed.

---

## 4. Heapsort & `std::make_heap` (raw heap API)

When you need heap operations on an existing container:

```cpp
vector<int> v = {3, 1, 4, 1, 5, 9};
make_heap(v.begin(), v.end());     // max-heap in place, O(n)
push_heap(v.begin(), v.end());     // after push_back, sift up
pop_heap(v.begin(), v.end());      // moves max to back; then v.pop_back()
sort_heap(v.begin(), v.end());     // O(n log n), ascending (destroys heap)
```

---

## 5. Complexity Quick Reference

| Operation                        | Time        | Space |
|----------------------------------|-------------|-------|
| push / pop                       | O(log n)    | —     |
| top (peek)                       | O(1)        | —     |
| build from n elements (heapify)  | O(n)        | O(n)  |
| Kth largest (size-k heap)        | O(n log k)  | O(k)  |
| Merge k lists (N total nodes)    | O(N log k)  | O(k)  |
| Heapsort                         | O(n log n)  | O(1)  |
| Running median (two heaps)       | O(log n)/op | O(n)  |

**Heap vs sorting:** if you only need the top-k (not full order) or a *stream*, a heap (O(n log k)) beats sorting (O(n log n)) and handles unbounded input.

---

## 6. Must-Practice LeetCode Problems

| # | Problem | Pattern |
|---|---------|---------|
| 215 | Kth Largest Element | size-k min-heap |
| 347 | Top K Frequent Elements | freq map + heap |
| 703 | Kth Largest in a Stream | size-k min-heap (online) |
| 973 | K Closest Points to Origin | size-k max-heap |
| 23 | Merge k Sorted Lists | heap of fronts |
| 621 | Task Scheduler | greedy + heap |
| 253 | Meeting Rooms II | min-heap of end times |
| 295 | Find Median from Data Stream | two heaps |
| 692 | Top K Frequent Words | heap + tie-break |
| 1046 | Last Stone Weight | max-heap |
| 378 | Kth Smallest in Sorted Matrix | min-heap / binary search |
| 767 | Reorganize String | max-heap by count |

---

## 7. Full Solutions

Each is LeetCode-ready (method inside a `Solution` class). Assume `#include <queue>`, `<vector>`, `<unordered_map>`, `<string>`.

### 215. Kth Largest Element in an Array

```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        priority_queue<int, vector<int>, greater<int>> minHeap;
        for (int x : nums) {
            minHeap.push(x);
            if (minHeap.size() > (size_t)k) minHeap.pop();
        }
        return minHeap.top();
    }
};
```

### 347. Top K Frequent Elements

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int,int> freq;
        for (int x : nums) freq[x]++;
        // min-heap of (count, value), keep size k
        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
        for (auto& [val, cnt] : freq) {
            pq.push({cnt, val});
            if (pq.size() > (size_t)k) pq.pop();
        }
        vector<int> res;
        while (!pq.empty()) { res.push_back(pq.top().second); pq.pop(); }
        return res;
    }
};
```

### 703. Kth Largest Element in a Stream

```cpp
class KthLargest {
    priority_queue<int, vector<int>, greater<int>> minHeap;
    int k;
public:
    KthLargest(int k, vector<int>& nums) : k(k) {
        for (int x : nums) add(x);
    }
    int add(int val) {
        minHeap.push(val);
        if (minHeap.size() > (size_t)k) minHeap.pop();
        return minHeap.top();
    }
};
```

### 973. K Closest Points to Origin

```cpp
class Solution {
public:
    vector<vector<int>> kClosest(vector<vector<int>>& points, int k) {
        // max-heap of (distance, index), keep k closest
        priority_queue<pair<int,int>> pq;   // default max-heap
        for (int i = 0; i < points.size(); ++i) {
            int d = points[i][0]*points[i][0] + points[i][1]*points[i][1];
            pq.push({d, i});
            if (pq.size() > (size_t)k) pq.pop();   // drop farthest
        }
        vector<vector<int>> res;
        while (!pq.empty()) { res.push_back(points[pq.top().second]); pq.pop(); }
        return res;
    }
};
```

### 23. Merge k Sorted Lists

```cpp
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
        priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
        for (ListNode* l : lists) if (l) pq.push(l);
        ListNode dummy(0); ListNode* tail = &dummy;
        while (!pq.empty()) {
            ListNode* top = pq.top(); pq.pop();
            tail->next = top; tail = top;
            if (top->next) pq.push(top->next);
        }
        return dummy.next;
    }
};
```

### 621. Task Scheduler

```cpp
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        int freq[26] = {0};
        for (char t : tasks) freq[t - 'A']++;
        priority_queue<int> pq;
        for (int f : freq) if (f) pq.push(f);
        int time = 0;
        while (!pq.empty()) {
            vector<int> temp;
            int cycle = n + 1;
            while (cycle-- && !pq.empty()) {   // run up to n+1 distinct tasks
                int c = pq.top(); pq.pop();
                if (c - 1 > 0) temp.push_back(c - 1);
                time++;
            }
            for (int c : temp) pq.push(c);
            if (!pq.empty()) time += cycle + 1;  // idle for remaining slots
        }
        return time;
    }
};
```

### 253. Meeting Rooms II

```cpp
class Solution {
public:
    int minMeetingRooms(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end());   // by start
        priority_queue<int, vector<int>, greater<int>> endTimes;  // min-heap
        for (auto& iv : intervals) {
            if (!endTimes.empty() && endTimes.top() <= iv[0])
                endTimes.pop();          // reuse a freed room
            endTimes.push(iv[1]);
        }
        return endTimes.size();          // rooms in use = answer
    }
};
```

### 295. Find Median from Data Stream (two heaps)

```cpp
class MedianFinder {
    priority_queue<int> lo;                                  // max-heap (lower half)
    priority_queue<int, vector<int>, greater<int>> hi;       // min-heap (upper half)
public:
    void addNum(int num) {
        lo.push(num);
        hi.push(lo.top()); lo.pop();          // balance: move max of lo to hi
        if (hi.size() > lo.size()) {          // keep lo >= hi
            lo.push(hi.top()); hi.pop();
        }
    }
    double findMedian() {
        if (lo.size() > hi.size()) return lo.top();
        return (lo.top() + hi.top()) / 2.0;
    }
};
```

### 692. Top K Frequent Words

```cpp
class Solution {
public:
    vector<string> topKFrequent(vector<string>& words, int k) {
        unordered_map<string,int> freq;
        for (auto& w : words) freq[w]++;
        // min-heap: less frequent on top; ties → lexicographically larger on top
        auto cmp = [](const pair<int,string>& a, const pair<int,string>& b) {
            if (a.first != b.first) return a.first > b.first;   // min by count
            return a.second < b.second;                         // max by word
        };
        priority_queue<pair<int,string>, vector<pair<int,string>>, decltype(cmp)> pq(cmp);
        for (auto& [w, c] : freq) {
            pq.push({c, w});
            if (pq.size() > (size_t)k) pq.pop();
        }
        vector<string> res(k);
        for (int i = k - 1; i >= 0; --i) { res[i] = pq.top().second; pq.pop(); }
        return res;
    }
};
```

### 1046. Last Stone Weight

```cpp
class Solution {
public:
    int lastStoneWeight(vector<int>& stones) {
        priority_queue<int> pq(stones.begin(), stones.end());   // max-heap
        while (pq.size() > 1) {
            int a = pq.top(); pq.pop();
            int b = pq.top(); pq.pop();
            if (a != b) pq.push(a - b);
        }
        return pq.empty() ? 0 : pq.top();
    }
};
```

### 378. Kth Smallest Element in a Sorted Matrix

```cpp
class Solution {
public:
    int kthSmallest(vector<vector<int>>& matrix, int k) {
        int n = matrix.size();
        // min-heap of (value, row, col)
        priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<>> pq;
        for (int r = 0; r < min(n, k); ++r) pq.push({matrix[r][0], r, 0});
        int val = 0;
        while (k--) {
            auto [v, r, c] = pq.top(); pq.pop();
            val = v;
            if (c + 1 < n) pq.push({matrix[r][c + 1], r, c + 1});
        }
        return val;
    }
};
```

### 767. Reorganize String

```cpp
class Solution {
public:
    string reorganizeString(string s) {
        int freq[26] = {0};
        for (char c : s) freq[c - 'a']++;
        priority_queue<pair<int,char>> pq;   // max-heap by count
        for (int i = 0; i < 26; ++i)
            if (freq[i]) pq.push({freq[i], char('a' + i)});
        string res;
        while (pq.size() > 1) {
            auto [c1, ch1] = pq.top(); pq.pop();
            auto [c2, ch2] = pq.top(); pq.pop();
            res += ch1; res += ch2;          // place two different chars
            if (--c1 > 0) pq.push({c1, ch1});
            if (--c2 > 0) pq.push({c2, ch2});
        }
        if (!pq.empty()) {
            if (pq.top().first > 1) return "";   // impossible
            res += pq.top().second;
        }
        return res;
    }
};
```

---

## 8. Common Bugs & Interview Tips

- **Wrong heap direction**: default is MAX-heap; for min-heap you need `greater<>`. Say it out loud to avoid inverting the logic.
- **Comparator is inverted**: `cmp(a,b)==true` means `a` has *lower* priority (sinks). For a min-heap by value, use `a > b`.
- **Top-k direction**: k *largest* → min-heap of size k; k *smallest* → max-heap of size k. Easy to flip by accident.
- **`size()` is unsigned**: comparing `pq.size() > k` with signed `k` can warn/overflow; cast with `(size_t)k`.
- **Structured bindings** (`auto [v, r, c] = pq.top();`) need C++17.
- **Heapify is O(n)**: build from a range with the range constructor instead of n pushes when you have all data up front.
- **Stability**: heaps are *not* stable; add a tiebreak field to the key if order among equals matters (see LC 692).

---

**One-liner to remember:** *k largest → min-heap of size k; k smallest → max-heap of size k.* Default `priority_queue` is a MAX-heap.
