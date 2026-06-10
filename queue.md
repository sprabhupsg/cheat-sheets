# Queue Pattern — C++ Cheat Sheet

Pattern for problems involving BFS traversal, sliding windows, scheduling, level-order processing, and stream-based operations.

---

## Core Concepts

**Queue Properties:** FIFO (First In, First Out) · O(1) push/pop/front · `std::queue<T>` in C++

**Key Variants:**

| Type | Header | Use Case |
|---|---|---|
| `queue<T>` | `<queue>` | Standard FIFO — BFS, level-order |
| `deque<T>` | `<deque>` | Double-ended — sliding window max/min |
| `priority_queue<T>` | `<queue>` | Max-heap by default — top-k, scheduling |

```cpp
#include <queue>
#include <deque>

// Standard queue
queue<int> q;
q.push(42);         // enqueue at back
q.front();           // peek front element
q.back();            // peek back element
q.pop();             // dequeue from front
q.empty();           // check if empty
q.size();            // number of elements

// Double-ended queue
deque<int> dq;
dq.push_back(1);     // add to back
dq.push_front(2);    // add to front
dq.pop_back();       // remove from back
dq.pop_front();      // remove from front
dq.front();          // peek front
dq.back();           // peek back

// Priority queue (max-heap)
priority_queue<int> maxPQ;
priority_queue<int, vector<int>, greater<int>> minPQ;  // min-heap
maxPQ.push(10);
maxPQ.top();         // peek max element
maxPQ.pop();         // remove max element
```

---

## Pattern Categories

| Pattern | Core Idea | Typical Problems |
|---|---|---|
| **BFS (Level Order)** | Queue processes nodes level by level | Binary Tree Level Order, Rotting Oranges |
| **Sliding Window (Deque)** | Monotonic deque maintains window max/min in O(1) | Sliding Window Maximum, Shortest Subarray |
| **Top-K / Scheduling** | Priority queue for ordered processing | Top K Frequent, Task Scheduler |
| **Stream Processing** | Queue buffers elements for ordered consumption | Moving Average, Hit Counter |
| **Multi-source BFS** | Enqueue all sources first, expand simultaneously | 01-Matrix, Walls and Gates |
| **Design / Simulation** | Queue models real-world FIFO behavior | Implement Stack using Queues, Circular Queue |

---

## BFS — Core Template

The most fundamental queue pattern. Processes nodes in order of distance from the source.

```cpp
// Generic BFS on a graph / grid
int bfs(vector<vector<int>>& graph, int start, int target) {
    queue<int> q;
    unordered_set<int> visited;
    q.push(start);
    visited.insert(start);
    int level = 0;
    while (!q.empty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {     // process entire level
            int node = q.front(); q.pop();
            if (node == target) return level;
            for (int neighbor : graph[node]) {
                if (!visited.count(neighbor)) {
                    visited.insert(neighbor);
                    q.push(neighbor);
                }
            }
        }
        level++;
    }
    return -1;  // not reachable
}
```

### BFS on a Grid

```cpp
int bfsGrid(vector<vector<int>>& grid, pair<int,int> start, pair<int,int> target) {
    int m = grid.size(), n = grid[0].size();
    int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
    queue<pair<int,int>> q;
    vector<vector<bool>> visited(m, vector<bool>(n, false));
    q.push(start);
    visited[start.first][start.second] = true;
    int steps = 0;
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            auto [r, c] = q.front(); q.pop();
            if (r == target.first && c == target.second) return steps;
            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < m && nc >= 0 && nc < n
                    && !visited[nr][nc] && grid[nr][nc] == 0) {
                    visited[nr][nc] = true;
                    q.push({nr, nc});
                }
            }
        }
        steps++;
    }
    return -1;
}
```

---

## Monotonic Deque — Core Template

Maintains a window of elements in decreasing (or increasing) order. Gives O(1) max/min for any sliding window.

```cpp
// Sliding window maximum using monotonic deque
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;    // stores indices, front = index of current max
    vector<int> res;
    for (int i = 0; i < (int)nums.size(); i++) {
        // remove indices outside the window
        while (!dq.empty() && dq.front() <= i - k)
            dq.pop_front();
        // remove smaller elements from back (they'll never be max)
        while (!dq.empty() && nums[dq.back()] <= nums[i])
            dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1)
            res.push_back(nums[dq.front()]);
    }
    return res;
}
```

### Monotonic Deque Variants

| Variant | Deque Order (front→back) | Pop Back When | Gives |
|---|---|---|---|
| Sliding Window Max | Decreasing values | `nums[back] <= nums[i]` | O(1) window max |
| Sliding Window Min | Increasing values | `nums[back] >= nums[i]` | O(1) window min |

```
Window [1, 3, -1, -3, 5], k=3:

  i=0: dq=[0]           (val: 1)
  i=1: dq=[1]           (val: 3, popped 1)
  i=2: dq=[1, 2]        (val: 3, -1)  → max = nums[1] = 3
  i=3: dq=[1, 3]        (val: 3, -3)  → max = nums[1] = 3
  i=4: dq=[4]           (val: 5)      → max = nums[4] = 5
```

---

## When to Use a Queue

| Signal in Problem | Pattern |
|---|---|
| "Shortest path in unweighted graph/grid" | BFS with queue |
| "Level-order traversal" | BFS with level-size loop |
| "Rotting / spreading / infection" | Multi-source BFS |
| "Sliding window maximum / minimum" | Monotonic deque |
| "Top K elements / K most frequent" | Priority queue (heap) |
| "Task scheduling with cooldown" | Priority queue + queue simulation |
| "Moving average / recent counter" | Queue as sliding buffer |
| "Implement X using queues" | Design problem — queue simulation |

---

## Problem Catalog

### 102. Binary Tree Level Order Traversal — Medium

**Problem:** Return the level-order traversal of a binary tree as nested vectors.

**Key Idea:** BFS. Process one full level per outer loop iteration by capturing `q.size()` at the start.

**Complexity:** Time O(n) · Space O(n)

```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level;
        for (int i = 0; i < sz; i++) {
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

---

### 107. Binary Tree Level Order Traversal II — Medium

**Problem:** Return bottom-up level order traversal.

**Key Idea:** Same BFS as above, then reverse the result.

**Complexity:** Time O(n) · Space O(n)

```cpp
vector<vector<int>> levelOrderBottom(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level;
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    reverse(res.begin(), res.end());
    return res;
}
```

---

### 103. Binary Tree Zigzag Level Order — Medium

**Problem:** Return zigzag level-order traversal (left-to-right, then right-to-left, alternating).

**Key Idea:** Standard BFS. On odd levels, reverse the level vector before adding to result.

**Complexity:** Time O(n) · Space O(n)

```cpp
vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    bool leftToRight = true;
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level(sz);
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            int idx = leftToRight ? i : (sz - 1 - i);
            level[idx] = node->val;
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
        leftToRight = !leftToRight;
    }
    return res;
}
```

---

### 199. Binary Tree Right Side View — Medium

**Problem:** Return values visible from the right side of the tree.

**Key Idea:** BFS level by level. The last node processed in each level is the rightmost.

**Complexity:** Time O(n) · Space O(n)

```cpp
vector<int> rightSideView(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            if (i == sz - 1) res.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
    return res;
}
```

---

### 994. Rotting Oranges — Medium

**Problem:** Each minute, fresh oranges adjacent to rotten ones become rotten. Return minutes until no fresh orange remains, or -1.

**Key Idea:** Multi-source BFS. Enqueue all initially rotten oranges. Expand one level per minute.

**Complexity:** Time O(m × n) · Space O(m × n)

```cpp
int orangesRotting(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    queue<pair<int,int>> q;
    int fresh = 0;
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 2) q.push({i, j});
            else if (grid[i][j] == 1) fresh++;
        }
    if (fresh == 0) return 0;
    int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
    int minutes = 0;
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            auto [r, c] = q.front(); q.pop();
            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < m && nc >= 0 && nc < n
                    && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;
                    fresh--;
                    q.push({nr, nc});
                }
            }
        }
        minutes++;
    }
    return fresh == 0 ? minutes - 1 : -1;
}
```

---

### 286. Walls and Gates — Medium

**Problem:** Fill each empty room with the distance to its nearest gate. Gates are 0, walls are -1.

**Key Idea:** Multi-source BFS from all gates simultaneously. Each expansion increments distance by 1.

**Complexity:** Time O(m × n) · Space O(m × n)

```cpp
void wallsAndGates(vector<vector<int>>& rooms) {
    int m = rooms.size(), n = rooms[0].size();
    queue<pair<int,int>> q;
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (rooms[i][j] == 0) q.push({i, j});
    int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
    while (!q.empty()) {
        auto [r, c] = q.front(); q.pop();
        for (auto& d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < m && nc >= 0 && nc < n
                && rooms[nr][nc] == INT_MAX) {
                rooms[nr][nc] = rooms[r][c] + 1;
                q.push({nr, nc});
            }
        }
    }
}
```

---

### 542. 01 Matrix — Medium

**Problem:** Given a binary matrix, find the distance of each cell to the nearest 0.

**Key Idea:** Multi-source BFS from all 0-cells. Identical approach to Walls and Gates.

**Complexity:** Time O(m × n) · Space O(m × n)

```cpp
vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
    int m = mat.size(), n = mat[0].size();
    vector<vector<int>> dist(m, vector<int>(n, INT_MAX));
    queue<pair<int,int>> q;
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (mat[i][j] == 0) { dist[i][j] = 0; q.push({i, j}); }
    int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
    while (!q.empty()) {
        auto [r, c] = q.front(); q.pop();
        for (auto& d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < m && nc >= 0 && nc < n
                && dist[nr][nc] > dist[r][c] + 1) {
                dist[nr][nc] = dist[r][c] + 1;
                q.push({nr, nc});
            }
        }
    }
    return dist;
}
```

---

### 239. Sliding Window Maximum — Hard

**Problem:** Return the max value in each sliding window of size k.

**Key Idea:** Monotonic decreasing deque. Front holds index of current window max. Pop stale indices from front, smaller values from back.

**Complexity:** Time O(n) · Space O(k)

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;
    vector<int> res;
    for (int i = 0; i < (int)nums.size(); i++) {
        while (!dq.empty() && dq.front() <= i - k)
            dq.pop_front();
        while (!dq.empty() && nums[dq.back()] <= nums[i])
            dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1)
            res.push_back(nums[dq.front()]);
    }
    return res;
}
```

---

### 862. Shortest Subarray with Sum at Least K — Hard

**Problem:** Find the length of the shortest subarray with sum >= k (may contain negatives).

**Key Idea:** Prefix sums + monotonic increasing deque. For each prefix sum, pop from front while subarray sum >= k (candidate answer), pop from back to maintain increasing order.

**Complexity:** Time O(n) · Space O(n)

```cpp
int shortestSubarray(vector<int>& nums, int k) {
    int n = nums.size();
    vector<long long> prefix(n + 1, 0);
    for (int i = 0; i < n; i++)
        prefix[i + 1] = prefix[i] + nums[i];
    deque<int> dq;
    int res = INT_MAX;
    for (int i = 0; i <= n; i++) {
        while (!dq.empty() && prefix[i] - prefix[dq.front()] >= k) {
            res = min(res, i - dq.front());
            dq.pop_front();
        }
        while (!dq.empty() && prefix[dq.back()] >= prefix[i])
            dq.pop_back();
        dq.push_back(i);
    }
    return res == INT_MAX ? -1 : res;
}
```

---

### 347. Top K Frequent Elements — Medium

**Problem:** Return the k most frequent elements.

**Key Idea:** Count frequencies, then use a min-heap of size k. If heap exceeds k, pop the least frequent.

**Complexity:** Time O(n log k) · Space O(n)

```cpp
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int, int> freq;
    for (int n : nums) freq[n]++;
    // min-heap by frequency
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    for (auto& [num, cnt] : freq) {
        pq.push({cnt, num});
        if ((int)pq.size() > k) pq.pop();
    }
    vector<int> res;
    while (!pq.empty()) {
        res.push_back(pq.top().second);
        pq.pop();
    }
    return res;
}
```

---

### 215. Kth Largest Element in an Array — Medium

**Problem:** Find the kth largest element (not kth distinct).

**Key Idea:** Min-heap of size k. After processing all elements, the top is the kth largest.

**Complexity:** Time O(n log k) · Space O(k)

```cpp
int findKthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> pq;
    for (int n : nums) {
        pq.push(n);
        if ((int)pq.size() > k) pq.pop();
    }
    return pq.top();
}
```

---

### 621. Task Scheduler — Medium

**Problem:** Schedule tasks with a cooldown of `n` between same tasks. Return minimum intervals.

**Key Idea:** Max-heap for task counts. Each round: pop up to `n+1` tasks, decrement, re-enqueue non-zero. If fewer than `n+1` tasks available, idle slots fill the gap.

**Complexity:** Time O(total × log 26) → O(total) · Space O(1)

```cpp
int leastInterval(vector<char>& tasks, int n) {
    vector<int> freq(26, 0);
    for (char t : tasks) freq[t - 'A']++;
    priority_queue<int> pq;
    for (int f : freq)
        if (f > 0) pq.push(f);
    int time = 0;
    while (!pq.empty()) {
        vector<int> temp;
        for (int i = 0; i <= n; i++) {
            if (!pq.empty()) {
                temp.push_back(pq.top() - 1);
                pq.pop();
            }
        }
        for (int t : temp)
            if (t > 0) pq.push(t);
        time += pq.empty() ? temp.size() : (n + 1);
    }
    return time;
}
```

---

### 232. Implement Queue using Stacks — Easy

**Problem:** Implement a FIFO queue using only two stacks.

**Key Idea:** Push stack and pop stack. On `peek`/`pop`, if pop stack is empty, transfer all from push stack (reverses order → FIFO).

**Complexity:** Time O(1) amortized · Space O(n)

```cpp
class MyQueue {
    stack<int> pushSt, popSt;
    void transfer() {
        if (popSt.empty())
            while (!pushSt.empty()) {
                popSt.push(pushSt.top());
                pushSt.pop();
            }
    }
public:
    void push(int x) { pushSt.push(x); }
    int pop() { transfer(); int v = popSt.top(); popSt.pop(); return v; }
    int peek() { transfer(); return popSt.top(); }
    bool empty() { return pushSt.empty() && popSt.empty(); }
};
```

---

### 225. Implement Stack using Queues — Easy

**Problem:** Implement a LIFO stack using only queues.

**Key Idea:** On push, enqueue the element then rotate all previous elements behind it (re-enqueue `size-1` elements).

**Complexity:** Time O(n) push, O(1) others · Space O(n)

```cpp
class MyStack {
    queue<int> q;
public:
    void push(int x) {
        q.push(x);
        for (int i = 0; i < (int)q.size() - 1; i++) {
            q.push(q.front());
            q.pop();
        }
    }
    int pop() { int v = q.front(); q.pop(); return v; }
    int top() { return q.front(); }
    bool empty() { return q.empty(); }
};
```

---

### 622. Design Circular Queue — Medium

**Problem:** Design a circular queue with fixed capacity.

**Key Idea:** Use a fixed-size array with `front` and `rear` pointers, modular arithmetic for wrap-around.

**Complexity:** Time O(1) all ops · Space O(k)

```cpp
class MyCircularQueue {
    vector<int> data;
    int front, rear, size, cap;
public:
    MyCircularQueue(int k) : data(k), front(0), rear(-1), size(0), cap(k) {}
    bool enQueue(int value) {
        if (isFull()) return false;
        rear = (rear + 1) % cap;
        data[rear] = value;
        size++;
        return true;
    }
    bool deQueue() {
        if (isEmpty()) return false;
        front = (front + 1) % cap;
        size--;
        return true;
    }
    int Front() { return isEmpty() ? -1 : data[front]; }
    int Rear() { return isEmpty() ? -1 : data[rear]; }
    bool isEmpty() { return size == 0; }
    bool isFull() { return size == cap; }
};
```

---

### 346. Moving Average from Data Stream — Easy

**Problem:** Calculate the moving average of the last `size` elements in a stream.

**Key Idea:** Queue holds the window elements. When window exceeds size, dequeue oldest and subtract from running sum.

**Complexity:** Time O(1) per call · Space O(size)

```cpp
class MovingAverage {
    queue<int> q;
    int maxSize;
    double sum;
public:
    MovingAverage(int size) : maxSize(size), sum(0) {}
    double next(int val) {
        q.push(val);
        sum += val;
        if ((int)q.size() > maxSize) {
            sum -= q.front();
            q.pop();
        }
        return sum / q.size();
    }
};
```

---

### 933. Number of Recent Calls — Easy

**Problem:** Count requests in the last 3000 milliseconds.

**Key Idea:** Queue stores timestamps. On each ping, dequeue timestamps older than `t - 3000`.

**Complexity:** Time O(1) amortized · Space O(W) where W = window size

```cpp
class RecentCounter {
    queue<int> q;
public:
    int ping(int t) {
        q.push(t);
        while (q.front() < t - 3000)
            q.pop();
        return q.size();
    }
};
```

---

### 1091. Shortest Path in Binary Matrix — Medium

**Problem:** Find the shortest clear path from top-left to bottom-right in a binary grid (8-directional movement).

**Key Idea:** Standard BFS on a grid with 8 directions.

**Complexity:** Time O(n²) · Space O(n²)

```cpp
int shortestPathBinaryMatrix(vector<vector<int>>& grid) {
    int n = grid.size();
    if (grid[0][0] != 0 || grid[n-1][n-1] != 0) return -1;
    int dirs[8][2] = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};
    queue<pair<int,int>> q;
    q.push({0, 0});
    grid[0][0] = 1;  // mark visited (also stores distance)
    while (!q.empty()) {
        auto [r, c] = q.front(); q.pop();
        if (r == n - 1 && c == n - 1) return grid[r][c];
        for (auto& d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 0) {
                grid[nr][nc] = grid[r][c] + 1;
                q.push({nr, nc});
            }
        }
    }
    return -1;
}
```

---

## Complexity Summary

| Problem | # | Time | Space | Technique |
|---|:---:|:---:|:---:|---|
| Level Order Traversal | 102 | O(n) | O(n) | BFS with level-size loop |
| Level Order Bottom-Up | 107 | O(n) | O(n) | BFS + reverse |
| Zigzag Level Order | 103 | O(n) | O(n) | BFS + alternating index |
| Right Side View | 199 | O(n) | O(n) | BFS, last in each level |
| Rotting Oranges | 994 | O(m×n) | O(m×n) | Multi-source BFS |
| Walls and Gates | 286 | O(m×n) | O(m×n) | Multi-source BFS |
| 01 Matrix | 542 | O(m×n) | O(m×n) | Multi-source BFS |
| Sliding Window Max | 239 | O(n) | O(k) | Monotonic deque |
| Shortest Subarray ≥ K | 862 | O(n) | O(n) | Prefix sum + monotonic deque |
| Top K Frequent | 347 | O(n log k) | O(n) | Min-heap of size k |
| Kth Largest Element | 215 | O(n log k) | O(k) | Min-heap of size k |
| Task Scheduler | 621 | O(total) | O(1) | Max-heap + round simulation |
| Queue using Stacks | 232 | O(1)* | O(n) | Push/pop dual stacks |
| Stack using Queues | 225 | O(n) push | O(n) | Rotate on push |
| Circular Queue | 622 | O(1) | O(k) | Array + modular pointers |
| Moving Average | 346 | O(1) | O(size) | Queue + running sum |
| Recent Calls | 933 | O(1)* | O(W) | Queue + evict stale |
| Shortest Path Binary | 1091 | O(n²) | O(n²) | BFS 8-directional |

\* amortized

---

## BFS Decision Guide

| Scenario | Approach |
|---|---|
| Shortest path, unweighted graph/grid | Standard BFS |
| Distance from multiple sources simultaneously | Multi-source BFS (enqueue all sources first) |
| Shortest path, weighted (0 or 1 costs) | 0-1 BFS with deque (`push_front` for 0-cost, `push_back` for 1-cost) |
| Shortest path, weighted (positive) | Dijkstra with `priority_queue` |
| Level-by-level processing | BFS with `int sz = q.size()` inner loop |
| Topological order | Kahn's BFS with in-degree array |

### 0-1 BFS Template

```cpp
// Shortest path where edges have weight 0 or 1
vector<int> bfs01(vector<vector<pair<int,int>>>& adj, int src, int n) {
    vector<int> dist(n, INT_MAX);
    deque<int> dq;
    dist[src] = 0;
    dq.push_front(src);
    while (!dq.empty()) {
        int u = dq.front(); dq.pop_front();
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (w == 0) dq.push_front(v);
                else dq.push_back(v);
            }
        }
    }
    return dist;
}
```

### Kahn's Topological Sort

```cpp
vector<int> topologicalSort(int n, vector<vector<int>>& adj) {
    vector<int> indegree(n, 0);
    for (int u = 0; u < n; u++)
        for (int v : adj[u]) indegree[v]++;
    queue<int> q;
    for (int i = 0; i < n; i++)
        if (indegree[i] == 0) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u])
            if (--indegree[v] == 0) q.push(v);
    }
    return order;  // if order.size() < n, cycle exists
}
```

---

## Priority Queue Patterns

### Custom Comparators

```cpp
// Min-heap (smallest on top)
priority_queue<int, vector<int>, greater<int>> minPQ;

// Custom struct with comparator
struct Task { int time, cost; };
auto cmp = [](const Task& a, const Task& b) { return a.time > b.time; };
priority_queue<Task, vector<Task>, decltype(cmp)> pq(cmp);

// Pair — default sorts by first, then second (max-heap)
priority_queue<pair<int,int>> pq;  // max by first element

// Min by first element
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
```

### K-Way Merge Pattern

```cpp
// Merge k sorted lists
ListNode* mergeKLists(vector<ListNode*>& lists) {
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
    for (auto* l : lists)
        if (l) pq.push(l);
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (!pq.empty()) {
        ListNode* node = pq.top(); pq.pop();
        tail->next = node;
        tail = tail->next;
        if (node->next) pq.push(node->next);
    }
    return dummy.next;
}
```

---

## Common Tricks & Edge Cases

### Queue/Deque Tricks

| Trick | Description |
|---|---|
| **Level-size capture** | `int sz = q.size()` before inner loop — processes exactly one BFS level |
| **Multi-source init** | Enqueue all sources before the BFS loop — computes min distance from nearest source |
| **Deque for 0-1 BFS** | `push_front` for 0-cost edges, `push_back` for 1-cost — avoids Dijkstra overhead |
| **Min-heap of size k** | Maintains top-k largest elements; top is the kth largest |
| **Lazy deletion in PQ** | Push duplicates, skip stale entries on pop (when entry doesn't match current state) |
| **Modular pointers** | `(index + 1) % capacity` for circular buffer implementations |

### Edge Cases

| Edge Case | Note |
|---|---|
| **Empty graph / grid** | Return 0 or empty — guard dimensions before BFS |
| **Source equals target** | Return 0 distance immediately |
| **Disconnected components** | BFS won't reach all nodes — check visited count |
| **Single element window** | Sliding window with k=1 — result is the array itself |
| **All same elements** | Deque won't pop anything — front stays the same |
| **Negative weights** | Standard BFS assumes unit cost — use 0-1 BFS or Dijkstra |
| **Priority queue is max-heap** | C++ default `priority_queue` is max-heap — use `greater<>` for min-heap |
| **Grid visited marking** | Mark visited when enqueuing, not when dequeuing — prevents duplicate entries |
