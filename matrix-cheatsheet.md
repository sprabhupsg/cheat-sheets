# C++ Matrix Coding Cheat Sheet

A quick reference for solving matrix / 2D-grid problems in competitive programming and interviews.

---

## 1. Declaration & Initialization

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int R = 3, C = 4;

    // C-style fixed array (compile-time size)
    int a[3][4] = {{0}};                 // all zeros

    // vector of vectors (preferred, dynamic)
    vector<vector<int>> g(R, vector<int>(C, 0));     // R x C filled with 0
    vector<vector<int>> grid = {                      // literal init
        {1, 2, 3},
        {4, 5, 6}
    };

    // Sizes
    int rows = grid.size();
    int cols = grid.empty() ? 0 : grid[0].size();
}
```

---

## 2. Reading Input

```cpp
int R, C;
cin >> R >> C;
vector<vector<int>> g(R, vector<int>(C));
for (int i = 0; i < R; i++)
    for (int j = 0; j < C; j++)
        cin >> g[i][j];

// Char grid (mazes, islands)
vector<string> grid(R);
for (auto &row : grid) cin >> row;     // grid[i][j] is a char
```

---

## 3. Iteration Patterns

```cpp
// Row-major
for (int i = 0; i < R; i++)
    for (int j = 0; j < C; j++)
        process(g[i][j]);

// Column-major
for (int j = 0; j < C; j++)
    for (int i = 0; i < R; i++)
        process(g[i][j]);

// Range-based (read-only or by reference)
for (auto &row : g)
    for (auto &x : row)
        x *= 2;
```

---

## 4. Neighbor Traversal (the core trick)

```cpp
// 4-directional (up, down, left, right)
int dr4[] = {-1, 1, 0, 0};
int dc4[] = {0, 0, -1, 1};

// 8-directional (includes diagonals)
int dr8[] = {-1, -1, -1, 0, 0, 1, 1, 1};
int dc8[] = {-1,  0,  1,-1, 1,-1, 0, 1};

// Bounds check helper
auto inBounds = [&](int r, int c) {
    return r >= 0 && r < R && c >= 0 && c < C;
};

for (int d = 0; d < 4; d++) {
    int nr = r + dr4[d], nc = c + dc4[d];
    if (inBounds(nr, nc)) { /* visit g[nr][nc] */ }
}
```

---

## 5. BFS on a Grid (shortest path in unweighted grid)

```cpp
int bfs(vector<vector<int>>& g, int sr, int sc) {
    int R = g.size(), C = g[0].size();
    vector<vector<bool>> vis(R, vector<bool>(C, false));
    queue<pair<int,int>> q;
    q.push({sr, sc});
    vis[sr][sc] = true;
    int dist = 0;

    int dr[] = {-1,1,0,0}, dc[] = {0,0,-1,1};
    while (!q.empty()) {
        int sz = q.size();
        while (sz--) {
            auto [r, c] = q.front(); q.pop();
            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr>=0 && nr<R && nc>=0 && nc<C
                    && !vis[nr][nc] && g[nr][nc] == 0) {
                    vis[nr][nc] = true;
                    q.push({nr, nc});
                }
            }
        }
        dist++;            // increment per level
    }
    return dist;
}
```

---

## 6. DFS on a Grid (flood fill / connected components / islands)

```cpp
void dfs(vector<vector<char>>& g, int r, int c) {
    int R = g.size(), C = g[0].size();
    if (r<0 || r>=R || c<0 || c>=C || g[r][c] != '1') return;
    g[r][c] = '0';                       // mark visited in place
    dfs(g, r+1, c);
    dfs(g, r-1, c);
    dfs(g, r, c+1);
    dfs(g, r, c-1);
}

int numIslands(vector<vector<char>>& g) {
    int count = 0;
    for (int i = 0; i < (int)g.size(); i++)
        for (int j = 0; j < (int)g[0].size(); j++)
            if (g[i][j] == '1') { dfs(g, i, j); count++; }
    return count;
}
```

> Tip: For large grids prefer iterative DFS (explicit stack) or BFS to avoid stack overflow.

---

## 7. Common Transformations

```cpp
// Transpose (square, in place)
for (int i = 0; i < n; i++)
    for (int j = i+1; j < n; j++)
        swap(g[i][j], g[j][i]);

// Rotate 90 clockwise = transpose + reverse each row
for (auto &row : g) reverse(row.begin(), row.end());

// Rotate 90 counter-clockwise = transpose + reverse each column
for (int j = 0; j < n; j++)
    for (int i = 0, k = n-1; i < k; i++, k--)
        swap(g[i][j], g[k][j]);

// Reflect horizontally (flip rows)
reverse(g.begin(), g.end());

// Transpose for non-square (R x C -> C x R)
vector<vector<int>> t(C, vector<int>(R));
for (int i = 0; i < R; i++)
    for (int j = 0; j < C; j++)
        t[j][i] = g[i][j];
```

---

## 8. Spiral Traversal

```cpp
vector<int> spiral(vector<vector<int>>& g) {
    vector<int> res;
    int top = 0, bottom = g.size()-1;
    int left = 0, right = g[0].size()-1;
    while (top <= bottom && left <= right) {
        for (int j = left; j <= right; j++) res.push_back(g[top][j]);
        top++;
        for (int i = top; i <= bottom; i++) res.push_back(g[i][right]);
        right--;
        if (top <= bottom) {
            for (int j = right; j >= left; j--) res.push_back(g[bottom][j]);
            bottom--;
        }
        if (left <= right) {
            for (int i = bottom; i >= top; i--) res.push_back(g[i][left]);
            left++;
        }
    }
    return res;
}
```

---

## 9. Prefix Sum (2D) — fast submatrix sum queries

```cpp
// Build: pre[i][j] = sum of rectangle (0,0)..(i-1,j-1)
vector<vector<long long>> pre(R+1, vector<long long>(C+1, 0));
for (int i = 1; i <= R; i++)
    for (int j = 1; j <= C; j++)
        pre[i][j] = g[i-1][j-1] + pre[i-1][j] + pre[i][j-1] - pre[i-1][j-1];

// Query sum of submatrix (r1,c1)..(r2,c2) inclusive, 0-indexed
auto query = [&](int r1, int c1, int r2, int c2) {
    return pre[r2+1][c2+1] - pre[r1][c2+1] - pre[r2+1][c1] + pre[r1][c1];
};
```

---

## 10. Matrix Multiplication

```cpp
vector<vector<long long>> multiply(
        const vector<vector<long long>>& A,
        const vector<vector<long long>>& B) {
    int n = A.size(), m = B[0].size(), k = B.size();
    vector<vector<long long>> C(n, vector<long long>(m, 0));
    for (int i = 0; i < n; i++)
        for (int p = 0; p < k; p++)
            for (int j = 0; j < m; j++)
                C[i][j] += A[i][p] * B[p][j];      // i-p-j order = cache friendly
    return C;
}
```

---

## 11. Dynamic Programming on Grids

```cpp
// Min path sum from top-left to bottom-right (move right/down only)
int minPathSum(vector<vector<int>>& g) {
    int R = g.size(), C = g[0].size();
    vector<vector<int>> dp(R, vector<int>(C, 0));
    dp[0][0] = g[0][0];
    for (int j = 1; j < C; j++) dp[0][j] = dp[0][j-1] + g[0][j];
    for (int i = 1; i < R; i++) dp[i][0] = dp[i-1][0] + g[i][0];
    for (int i = 1; i < R; i++)
        for (int j = 1; j < C; j++)
            dp[i][j] = g[i][j] + min(dp[i-1][j], dp[i][j-1]);
    return dp[R-1][C-1];
}

// Unique paths count
// dp[i][j] = dp[i-1][j] + dp[i][j-1], dp[0][*]=dp[*][0]=1
```

---

## 12. Useful STL Snippets

```cpp
// Resize / reset a 2D vector
g.assign(R, vector<int>(C, 0));

// Max / sum across whole matrix
int mx = INT_MIN; long long total = 0;
for (auto &row : g)
    for (int x : row) { mx = max(mx, x); total += x; }

// Print matrix
for (auto &row : g) {
    for (int x : row) cout << x << ' ';
    cout << '\n';
}

// Hash a cell for a set/map: key = r * C + c
set<int> seen;
seen.insert(r * C + c);
```

---

## 13. Complexity Quick Reference

| Operation                  | Time        | Space   |
|----------------------------|-------------|---------|
| Full traversal             | O(R·C)      | O(1)    |
| BFS / DFS (grid)           | O(R·C)      | O(R·C)  |
| 2D prefix sum build        | O(R·C)      | O(R·C)  |
| 2D prefix sum query        | O(1)        | —       |
| Matrix multiply (naive)    | O(n·m·k)    | O(n·m)  |
| Transpose / rotate         | O(R·C)      | O(1)*   |

\* In-place possible for square matrices.

---

## 14. Common Pitfalls

- **Row vs column order**: `g[row][col]`, not `g[col][row]`.
- **Bounds checks**: always validate `0 <= r < R && 0 <= c < C` before access.
- **`g[0].size()` on empty grid**: check `g.empty()` first.
- **Signed/unsigned**: `.size()` is `size_t`; cast to `int` when comparing with negative loop vars.
- **Recursion depth**: deep DFS on big grids can overflow the stack — use BFS/iterative.
- **Visited tracking**: mark cells *when enqueued* in BFS (not when dequeued) to avoid duplicates.
- **Overflow**: use `long long` for sums/products of large matrices.
```