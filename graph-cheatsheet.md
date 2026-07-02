# Graphs in C++ — Coding Interview Cheat Sheet

Graph problems look scary but reduce to a small set of patterns: **traversal (BFS/DFS)**, **shortest path**, **topological sort**, **union-find**, and **MST**. The interview skill is spotting which one applies from the problem wording.

---

## 0. Representations

### Adjacency list (default choice — sparse graphs)

```cpp
int n;                                   // number of nodes (0..n-1)
vector<vector<int>> adj(n);              // unweighted
adj[u].push_back(v);                     // edge u -> v (add both for undirected)

vector<vector<pair<int,int>>> wadj(n);   // weighted: (neighbor, weight)
wadj[u].push_back({v, w});
```

### Adjacency matrix (dense / small n, O(1) edge lookup)

```cpp
vector<vector<int>> mat(n, vector<int>(n, 0));
mat[u][v] = 1;                           // or weight
```

### Grid as an implicit graph (very common)

```cpp
int dr[4] = {-1, 1, 0, 0};
int dc[4] = {0, 0, -1, 1};               // 4-directional moves
// 8-dir: add diagonals {-1,-1,-1,1,1,-1,1,1}
```

---

## 1. Traversal

### BFS (shortest path in *unweighted* graphs, level order)

```cpp
vector<int> bfs(int start, vector<vector<int>>& adj, int n) {
    vector<int> dist(n, -1);
    queue<int> q;
    dist[start] = 0; q.push(start);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u])
            if (dist[v] == -1) {           // unvisited
                dist[v] = dist[u] + 1;
                q.push(v);
            }
    }
    return dist;
}
```

### DFS (recursive)

```cpp
void dfs(int u, vector<vector<int>>& adj, vector<bool>& visited) {
    visited[u] = true;
    for (int v : adj[u])
        if (!visited[v]) dfs(v, adj, visited);
}
```

### DFS (iterative, avoids stack overflow on deep graphs)

```cpp
void dfs(int start, vector<vector<int>>& adj, vector<bool>& visited) {
    stack<int> st; st.push(start);
    while (!st.empty()) {
        int u = st.top(); st.pop();
        if (visited[u]) continue;
        visited[u] = true;
        for (int v : adj[u]) if (!visited[v]) st.push(v);
    }
}
```

---

## 2. Connected Components / Flood Fill

Count components or fill regions (grids, friend circles).

```cpp
int countComponents(int n, vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    int count = 0;
    for (int i = 0; i < n; ++i)
        if (!visited[i]) { dfs(i, adj, visited); ++count; }
    return count;
}
```

Grid flood fill (islands): loop every cell, DFS/BFS on unvisited land, count.

---

## 3. Cycle Detection

### Undirected — DFS with parent

```cpp
bool hasCycleU(int u, int parent, vector<vector<int>>& adj, vector<bool>& vis) {
    vis[u] = true;
    for (int v : adj[u]) {
        if (!vis[v]) { if (hasCycleU(v, u, adj, vis)) return true; }
        else if (v != parent) return true;      // visited & not parent → cycle
    }
    return false;
}
```

### Directed — DFS with 3 colors (white/gray/black)

```cpp
// state: 0=unvisited, 1=in-progress(on stack), 2=done
bool hasCycleD(int u, vector<vector<int>>& adj, vector<int>& state) {
    state[u] = 1;
    for (int v : adj[u]) {
        if (state[v] == 1) return true;         // back edge → cycle
        if (state[v] == 0 && hasCycleD(v, adj, state)) return true;
    }
    state[u] = 2;
    return false;
}
```

---

## 4. Topological Sort (DAGs — ordering with dependencies)

### Kahn's algorithm (BFS on in-degrees)

```cpp
vector<int> topoSort(int n, vector<vector<int>>& adj) {
    vector<int> indeg(n, 0);
    for (int u = 0; u < n; ++u) for (int v : adj[u]) indeg[v]++;
    queue<int> q;
    for (int i = 0; i < n; ++i) if (indeg[i] == 0) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
    }
    return order.size() == n ? order : vector<int>{};   // empty → cycle exists
}
```
Use for: course schedule, build order, alien dictionary.

---

## 5. Shortest Path

| Algorithm      | Use when                                   | Time            |
|----------------|--------------------------------------------|-----------------|
| BFS            | unweighted (all edges = 1)                 | O(V + E)        |
| 0-1 BFS (deque)| weights are only 0 or 1                     | O(V + E)        |
| Dijkstra       | non-negative weights                       | O(E log V)      |
| Bellman-Ford   | negative weights / detect negative cycle   | O(V·E)          |
| Floyd-Warshall | all-pairs, small V (<= ~400)               | O(V^3)          |

### Dijkstra (min-heap)

```cpp
vector<int> dijkstra(int src, int n, vector<vector<pair<int,int>>>& adj) {
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;  // (dist, node)
    dist[src] = 0; pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;                // stale entry
        for (auto& [v, w] : adj[u])
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
    }
    return dist;
}
```

### Bellman-Ford (handles negatives, detects negative cycles)

```cpp
bool bellmanFord(int src, int n, vector<array<int,3>>& edges, vector<int>& dist) {
    dist.assign(n, INT_MAX); dist[src] = 0;
    for (int i = 0; i < n - 1; ++i)
        for (auto& [u, v, w] : edges)
            if (dist[u] != INT_MAX && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;
    for (auto& [u, v, w] : edges)                 // extra pass
        if (dist[u] != INT_MAX && dist[u] + w < dist[v]) return false;  // neg cycle
    return true;
}
```

### Floyd-Warshall (all pairs)

```cpp
// dist[i][j] init: 0 if i==j, w if edge, INF otherwise
for (int k = 0; k < n; ++k)
  for (int i = 0; i < n; ++i)
    for (int j = 0; j < n; ++j)
      if (dist[i][k] != INF && dist[k][j] != INF)
          dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
```

---

## 6. Union-Find (Disjoint Set Union) — connectivity, Kruskal, cycle detection

```cpp
struct DSU {
    vector<int> parent, rank_;
    DSU(int n) : parent(n), rank_(n, 0) { iota(parent.begin(), parent.end(), 0); }
    int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); } // path compression
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;              // already connected → cycle
        if (rank_[a] < rank_[b]) swap(a, b);
        parent[b] = a;
        if (rank_[a] == rank_[b]) rank_[a]++;
        return true;
    }
};
```
Near O(1) amortized per op. Use for: number of components, redundant connection, accounts merge, Kruskal's MST.

---

## 7. Minimum Spanning Tree

### Kruskal's (sort edges + DSU)

```cpp
int kruskal(int n, vector<array<int,3>>& edges) {   // edges: {w, u, v}
    sort(edges.begin(), edges.end());
    DSU dsu(n);
    int cost = 0, used = 0;
    for (auto& [w, u, v] : edges)
        if (dsu.unite(u, v)) { cost += w; if (++used == n - 1) break; }
    return cost;
}
```

### Prim's (grow from a node with a min-heap)

```cpp
int prim(int n, vector<vector<pair<int,int>>>& adj) {
    vector<bool> inMST(n, false);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    pq.push({0, 0}); int cost = 0;
    while (!pq.empty()) {
        auto [w, u] = pq.top(); pq.pop();
        if (inMST[u]) continue;
        inMST[u] = true; cost += w;
        for (auto& [v, wt] : adj[u]) if (!inMST[v]) pq.push({wt, v});
    }
    return cost;
}
```

---

## 8. Bipartite Check / Graph Coloring

```cpp
bool isBipartite(int n, vector<vector<int>>& adj) {
    vector<int> color(n, -1);
    for (int s = 0; s < n; ++s) {
        if (color[s] != -1) continue;
        queue<int> q; q.push(s); color[s] = 0;
        while (!q.empty()) {
            int u = q.front(); q.pop();
            for (int v : adj[u]) {
                if (color[v] == -1) { color[v] = color[u] ^ 1; q.push(v); }
                else if (color[v] == color[u]) return false;   // same color adjacent
            }
        }
    }
    return true;
}
```

---

## 9. Pattern → Algorithm Cheat Table

| Problem says...                                    | Reach for            |
|----------------------------------------------------|----------------------|
| "shortest path", edges weight 1 / grid steps       | BFS                  |
| "shortest path", non-negative weights              | Dijkstra             |
| "shortest path", negative weights / detect neg cyc | Bellman-Ford         |
| "shortest path between all pairs", small V         | Floyd-Warshall       |
| "order tasks with prerequisites" / "can finish"    | Topological sort     |
| "number of islands / regions / provinces"          | DFS/BFS flood fill or DSU |
| "are these connected" / "redundant edge"           | Union-Find           |
| "minimum cost to connect all"                      | MST (Kruskal/Prim)   |
| "two groups", "no odd cycle"                        | Bipartite / coloring |
| "detect a cycle"                                    | DFS (colors or parent)|
| "clone / copy a graph"                              | BFS/DFS + hashmap    |

---

## 10. Must-Practice LeetCode Problems

| # | Problem | Pattern |
|---|---------|---------|
| 200 | Number of Islands | grid DFS/BFS |
| 133 | Clone Graph | DFS/BFS + map |
| 207 | Course Schedule | topo sort / cycle |
| 210 | Course Schedule II | topo order |
| 994 | Rotting Oranges | multi-source BFS |
| 417 | Pacific Atlantic Water Flow | reverse DFS from borders |
| 542 | 01 Matrix | multi-source BFS |
| 743 | Network Delay Time | Dijkstra |
| 787 | Cheapest Flights K Stops | Bellman-Ford / BFS |
| 743 | (see above) | — |
| 323 | Number of Connected Components | DSU / DFS |
| 684 | Redundant Connection | Union-Find |
| 547 | Number of Provinces | DSU / DFS |
| 785 | Is Graph Bipartite | coloring |
| 1584 | Min Cost to Connect All Points | MST |
| 269 | Alien Dictionary | topo sort |
| 127 | Word Ladder | BFS |
| 130 | Surrounded Regions | border DFS |

---

## 11. Full Solutions

Each is LeetCode-ready (method inside a `Solution` class). Assume standard headers.

### 200. Number of Islands

```cpp
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        int m = grid.size(), n = grid[0].size(), count = 0;
        for (int r = 0; r < m; ++r)
            for (int c = 0; c < n; ++c)
                if (grid[r][c] == '1') { ++count; dfs(grid, r, c); }
        return count;
    }
    void dfs(vector<vector<char>>& g, int r, int c) {
        if (r < 0 || c < 0 || r >= g.size() || c >= g[0].size() || g[r][c] != '1') return;
        g[r][c] = '0';                         // sink
        dfs(g, r+1, c); dfs(g, r-1, c); dfs(g, r, c+1); dfs(g, r, c-1);
    }
};
```

### 133. Clone Graph

```cpp
// Node { int val; vector<Node*> neighbors; };
class Solution {
    unordered_map<Node*, Node*> seen;
public:
    Node* cloneGraph(Node* node) {
        if (!node) return nullptr;
        if (seen.count(node)) return seen[node];
        Node* copy = new Node(node->val);
        seen[node] = copy;
        for (Node* nb : node->neighbors)
            copy->neighbors.push_back(cloneGraph(nb));
        return copy;
    }
};
```

### 207. Course Schedule (can finish? = no cycle)

```cpp
class Solution {
public:
    bool canFinish(int n, vector<vector<int>>& prereqs) {
        vector<vector<int>> adj(n);
        vector<int> indeg(n, 0);
        for (auto& p : prereqs) { adj[p[1]].push_back(p[0]); indeg[p[0]]++; }
        queue<int> q;
        for (int i = 0; i < n; ++i) if (!indeg[i]) q.push(i);
        int done = 0;
        while (!q.empty()) {
            int u = q.front(); q.pop(); ++done;
            for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
        }
        return done == n;
    }
};
```

### 210. Course Schedule II (return order)

```cpp
class Solution {
public:
    vector<int> findOrder(int n, vector<vector<int>>& prereqs) {
        vector<vector<int>> adj(n);
        vector<int> indeg(n, 0);
        for (auto& p : prereqs) { adj[p[1]].push_back(p[0]); indeg[p[0]]++; }
        queue<int> q;
        for (int i = 0; i < n; ++i) if (!indeg[i]) q.push(i);
        vector<int> order;
        while (!q.empty()) {
            int u = q.front(); q.pop(); order.push_back(u);
            for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
        }
        return order.size() == n ? order : vector<int>{};
    }
};
```

### 994. Rotting Oranges (multi-source BFS)

```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size(), fresh = 0, minutes = 0;
        queue<pair<int,int>> q;
        for (int r = 0; r < m; ++r)
            for (int c = 0; c < n; ++c) {
                if (grid[r][c] == 2) q.push({r, c});
                else if (grid[r][c] == 1) ++fresh;
            }
        int dr[] = {-1,1,0,0}, dc[] = {0,0,-1,1};
        while (!q.empty() && fresh) {
            int sz = q.size();
            while (sz--) {
                auto [r, c] = q.front(); q.pop();
                for (int d = 0; d < 4; ++d) {
                    int nr = r + dr[d], nc = c + dc[d];
                    if (nr<0||nc<0||nr>=m||nc>=n||grid[nr][nc]!=1) continue;
                    grid[nr][nc] = 2; --fresh; q.push({nr, nc});
                }
            }
            ++minutes;
        }
        return fresh ? -1 : minutes;
    }
};
```

### 417. Pacific Atlantic Water Flow

```cpp
class Solution {
public:
    vector<vector<int>> pacificAtlantic(vector<vector<int>>& h) {
        int m = h.size(), n = h[0].size();
        vector<vector<bool>> pac(m, vector<bool>(n, false)), atl = pac;
        for (int r = 0; r < m; ++r) { dfs(h, r, 0, pac); dfs(h, r, n-1, atl); }
        for (int c = 0; c < n; ++c) { dfs(h, 0, c, pac); dfs(h, m-1, c, atl); }
        vector<vector<int>> res;
        for (int r = 0; r < m; ++r)
            for (int c = 0; c < n; ++c)
                if (pac[r][c] && atl[r][c]) res.push_back({r, c});
        return res;
    }
    void dfs(vector<vector<int>>& h, int r, int c, vector<vector<bool>>& vis) {
        vis[r][c] = true;
        int dr[] = {-1,1,0,0}, dc[] = {0,0,-1,1};
        for (int d = 0; d < 4; ++d) {
            int nr = r+dr[d], nc = c+dc[d];
            if (nr<0||nc<0||nr>=h.size()||nc>=h[0].size()||vis[nr][nc]) continue;
            if (h[nr][nc] < h[r][c]) continue;    // water flows downhill → climb up
            dfs(h, nr, nc, vis);
        }
    }
};
```

### 542. 01 Matrix (multi-source BFS from all 0s)

```cpp
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int m = mat.size(), n = mat[0].size();
        vector<vector<int>> dist(m, vector<int>(n, -1));
        queue<pair<int,int>> q;
        for (int r = 0; r < m; ++r)
            for (int c = 0; c < n; ++c)
                if (mat[r][c] == 0) { dist[r][c] = 0; q.push({r, c}); }
        int dr[] = {-1,1,0,0}, dc[] = {0,0,-1,1};
        while (!q.empty()) {
            auto [r, c] = q.front(); q.pop();
            for (int d = 0; d < 4; ++d) {
                int nr = r+dr[d], nc = c+dc[d];
                if (nr<0||nc<0||nr>=m||nc>=n||dist[nr][nc]!=-1) continue;
                dist[nr][nc] = dist[r][c] + 1; q.push({nr, nc});
            }
        }
        return dist;
    }
};
```

### 743. Network Delay Time (Dijkstra)

```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<vector<pair<int,int>>> adj(n + 1);
        for (auto& t : times) adj[t[0]].push_back({t[1], t[2]});
        vector<int> dist(n + 1, INT_MAX);
        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
        dist[k] = 0; pq.push({0, k});
        while (!pq.empty()) {
            auto [d, u] = pq.top(); pq.pop();
            if (d > dist[u]) continue;
            for (auto& [v, w] : adj[u])
                if (d + w < dist[v]) { dist[v] = d + w; pq.push({dist[v], v}); }
        }
        int ans = 0;
        for (int i = 1; i <= n; ++i) {
            if (dist[i] == INT_MAX) return -1;
            ans = max(ans, dist[i]);
        }
        return ans;
    }
};
```

### 787. Cheapest Flights Within K Stops (Bellman-Ford variant)

```cpp
class Solution {
public:
    int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
        vector<int> dist(n, INT_MAX);
        dist[src] = 0;
        for (int i = 0; i <= k; ++i) {             // at most k+1 edges
            vector<int> tmp = dist;                // use previous round only
            for (auto& f : flights) {
                int u = f[0], v = f[1], w = f[2];
                if (dist[u] != INT_MAX && dist[u] + w < tmp[v])
                    tmp[v] = dist[u] + w;
            }
            dist = tmp;
        }
        return dist[dst] == INT_MAX ? -1 : dist[dst];
    }
};
```

### 323. Number of Connected Components (DSU)

```cpp
class Solution {
public:
    int countComponents(int n, vector<vector<int>>& edges) {
        vector<int> parent(n);
        iota(parent.begin(), parent.end(), 0);
        function<int(int)> find = [&](int x) {
            return parent[x] == x ? x : parent[x] = find(parent[x]);
        };
        int comps = n;
        for (auto& e : edges) {
            int a = find(e[0]), b = find(e[1]);
            if (a != b) { parent[a] = b; --comps; }
        }
        return comps;
    }
};
```

### 684. Redundant Connection (Union-Find)

```cpp
class Solution {
public:
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        int n = edges.size();
        vector<int> parent(n + 1);
        iota(parent.begin(), parent.end(), 0);
        function<int(int)> find = [&](int x) {
            return parent[x] == x ? x : parent[x] = find(parent[x]);
        };
        for (auto& e : edges) {
            int a = find(e[0]), b = find(e[1]);
            if (a == b) return e;              // edge closes a cycle
            parent[a] = b;
        }
        return {};
    }
};
```

### 547. Number of Provinces (DSU)

```cpp
class Solution {
public:
    int findCircleNum(vector<vector<int>>& isConnected) {
        int n = isConnected.size();
        vector<int> parent(n);
        iota(parent.begin(), parent.end(), 0);
        function<int(int)> find = [&](int x) {
            return parent[x] == x ? x : parent[x] = find(parent[x]);
        };
        int provinces = n;
        for (int i = 0; i < n; ++i)
            for (int j = i + 1; j < n; ++j)
                if (isConnected[i][j]) {
                    int a = find(i), b = find(j);
                    if (a != b) { parent[a] = b; --provinces; }
                }
        return provinces;
    }
};
```

### 785. Is Graph Bipartite

```cpp
class Solution {
public:
    bool isBipartite(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<int> color(n, -1);
        for (int s = 0; s < n; ++s) {
            if (color[s] != -1) continue;
            queue<int> q; q.push(s); color[s] = 0;
            while (!q.empty()) {
                int u = q.front(); q.pop();
                for (int v : graph[u]) {
                    if (color[v] == -1) { color[v] = color[u] ^ 1; q.push(v); }
                    else if (color[v] == color[u]) return false;
                }
            }
        }
        return true;
    }
};
```

### 1584. Min Cost to Connect All Points (MST / Prim)

```cpp
class Solution {
public:
    int minCostConnectPoints(vector<vector<int>>& points) {
        int n = points.size(), cost = 0, used = 0;
        vector<bool> inMST(n, false);
        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
        pq.push({0, 0});                       // (cost, point)
        while (used < n && !pq.empty()) {
            auto [w, u] = pq.top(); pq.pop();
            if (inMST[u]) continue;
            inMST[u] = true; cost += w; ++used;
            for (int v = 0; v < n; ++v)
                if (!inMST[v]) {
                    int d = abs(points[u][0]-points[v][0]) + abs(points[u][1]-points[v][1]);
                    pq.push({d, v});
                }
        }
        return cost;
    }
};
```

### 269. Alien Dictionary (topo sort of characters)

```cpp
class Solution {
public:
    string alienOrder(vector<string>& words) {
        unordered_map<char, unordered_set<char>> adj;
        unordered_map<char, int> indeg;
        for (auto& w : words) for (char c : w) indeg[c] = 0;
        for (int i = 0; i + 1 < words.size(); ++i) {
            string& a = words[i]; string& b = words[i+1];
            int len = min(a.size(), b.size()), j = 0;
            for (; j < len; ++j)
                if (a[j] != b[j]) {
                    if (!adj[a[j]].count(b[j])) { adj[a[j]].insert(b[j]); indeg[b[j]]++; }
                    break;
                }
            if (j == len && a.size() > b.size()) return "";   // invalid: prefix after longer
        }
        queue<char> q;
        for (auto& [c, d] : indeg) if (d == 0) q.push(c);
        string order;
        while (!q.empty()) {
            char c = q.front(); q.pop(); order += c;
            for (char nx : adj[c]) if (--indeg[nx] == 0) q.push(nx);
        }
        return order.size() == indeg.size() ? order : "";     // cycle → ""
    }
};
```

### 127. Word Ladder (BFS)

```cpp
class Solution {
public:
    int ladderLength(string begin, string end, vector<string>& wordList) {
        unordered_set<string> dict(wordList.begin(), wordList.end());
        if (!dict.count(end)) return 0;
        queue<string> q; q.push(begin);
        int steps = 1;
        while (!q.empty()) {
            int sz = q.size();
            while (sz--) {
                string w = q.front(); q.pop();
                if (w == end) return steps;
                for (int i = 0; i < w.size(); ++i) {
                    char orig = w[i];
                    for (char c = 'a'; c <= 'z'; ++c) {
                        w[i] = c;
                        if (dict.count(w)) { q.push(w); dict.erase(w); }
                    }
                    w[i] = orig;
                }
            }
            ++steps;
        }
        return 0;
    }
};
```

### 130. Surrounded Regions (border DFS)

```cpp
class Solution {
public:
    void solve(vector<vector<char>>& board) {
        int m = board.size(), n = board[0].size();
        for (int r = 0; r < m; ++r) { dfs(board, r, 0); dfs(board, r, n-1); }
        for (int c = 0; c < n; ++c) { dfs(board, 0, c); dfs(board, m-1, c); }
        for (int r = 0; r < m; ++r)
            for (int c = 0; c < n; ++c)
                board[r][c] = (board[r][c] == 'S') ? 'O' : 'X';   // restore / flip
    }
    void dfs(vector<vector<char>>& b, int r, int c) {
        if (r<0||c<0||r>=b.size()||c>=b[0].size()||b[r][c]!='O') return;
        b[r][c] = 'S';                         // mark border-connected 'O'
        dfs(b,r+1,c); dfs(b,r-1,c); dfs(b,r,c+1); dfs(b,r,c-1);
    }
};
```

---

## 12. Common Bugs & Interview Tips

- **Mark visited on enqueue, not dequeue** in BFS, or nodes get pushed multiple times.
- **Undirected edges** must be added both ways: `adj[u].push_back(v)` *and* `adj[v].push_back(u)`.
- **Dijkstra can't handle negatives** — use Bellman-Ford. Also skip stale heap entries (`if (d > dist[u]) continue;`).
- **Topo sort empty result** = there's a cycle. Great way to answer "is it possible?".
- **Recursion depth**: deep/large grids can overflow the stack — switch to iterative BFS/DFS.
- **Grid bounds first**: always check `r,c` in range *before* accessing `grid[r][c]`.
- **`function<int(int)>` for recursive lambdas** (needed for inline DSU `find`).
- **Multi-source BFS**: push *all* sources first (rotting oranges, 01-matrix) for simultaneous spread.
- **Level-by-level BFS**: capture `q.size()` before the inner loop when you need distance/levels.

---

**One-liner to remember:** *Unweighted shortest path → BFS. Weighted → Dijkstra. Dependencies → topo sort. Connectivity → Union-Find.*
