# Backtracking in C++ — Coding Interview Cheat Sheet

Backtracking = **DFS over a decision tree**. At each step you *choose* an option, *explore* deeper, then *un-choose* (undo) to try the next option. It's brute force with pruning.

---

## 1. The Universal Template

Every backtracking problem fits this shape:

```cpp
void backtrack(State& state, /* extra args */) {
    if (isSolution(state)) {          // base case
        recordSolution(state);
        return;                       // (sometimes keep going)
    }
    for (Choice c : choices(state)) { // iterate candidates
        if (!isValid(c, state)) continue;   // PRUNE early
        makeChoice(state, c);         // choose
        backtrack(state, /* ... */);  // explore
        undoChoice(state, c);         // un-choose (BACKTRACK)
    }
}
```

**Three questions to answer for any problem:**
1. **What is a "choice"?** (which number, which candidate, include/exclude...)
2. **When is a state a complete solution?** (base case / depth reached)
3. **How do I prune?** (validity checks, `used[]`, sorting + skipping dups)

---

## 2. Core Patterns

### 2a. Subsets (include / exclude each element)

```cpp
void subsets(int i, vector<int>& nums, vector<int>& cur, vector<vector<int>>& res) {
    if (i == nums.size()) { res.push_back(cur); return; }
    // exclude nums[i]
    subsets(i + 1, nums, cur, res);
    // include nums[i]
    cur.push_back(nums[i]);
    subsets(i + 1, nums, cur, res);
    cur.pop_back();                    // undo
}
```

### 2b. Combinations (choose k from n, no reuse, order doesn't matter)

Use a `start` index so you never look backwards → avoids duplicates.

```cpp
void combine(int start, int n, int k, vector<int>& cur, vector<vector<int>>& res) {
    if (cur.size() == k) { res.push_back(cur); return; }
    for (int i = start; i <= n; ++i) {
        cur.push_back(i);
        combine(i + 1, n, k, cur, res);   // i+1: no reuse
        cur.pop_back();
    }
}
```

### 2c. Permutations (order matters, use every element once)

```cpp
void permute(vector<int>& nums, vector<bool>& used,
             vector<int>& cur, vector<vector<int>>& res) {
    if (cur.size() == nums.size()) { res.push_back(cur); return; }
    for (int i = 0; i < nums.size(); ++i) {
        if (used[i]) continue;
        used[i] = true; cur.push_back(nums[i]);
        permute(nums, used, cur, res);
        cur.pop_back(); used[i] = false;   // undo both
    }
}
```

---

## 3. Handling Duplicates

Sort first, then **skip a candidate if it equals the previous one at the same tree level**.

```cpp
sort(nums.begin(), nums.end());
// inside the for loop:
for (int i = start; i < nums.size(); ++i) {
    if (i > start && nums[i] == nums[i - 1]) continue;  // skip dup at this level
    ...
}
```

For **permutations with duplicates**, combine `used[]` with the dedup check:

```cpp
if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;
```

---

## 4. Reuse Allowed (Combination Sum style)

Pass `i` (not `i + 1`) to allow reusing the same element.

```cpp
void combSum(int start, int target, vector<int>& c,
             vector<int>& cur, vector<vector<int>>& res) {
    if (target == 0) { res.push_back(cur); return; }
    if (target < 0)  return;                 // prune
    for (int i = start; i < c.size(); ++i) {
        cur.push_back(c[i]);
        combSum(i, target - c[i], c, cur, res);  // i = reuse allowed
        cur.pop_back();
    }
}
```

---

## 5. Grid / Board Backtracking (Sudoku, N-Queens, Word Search)

Return `bool` when you only need **one** valid solution (early exit on `true`).

### N-Queens skeleton

```cpp
bool place(int row, int n, vector<int>& cols,
           vector<bool>& col, vector<bool>& diag1, vector<bool>& diag2) {
    if (row == n) return true;               // all queens placed
    for (int c = 0; c < n; ++c) {
        int d1 = row + c, d2 = row - c + n;
        if (col[c] || diag1[d1] || diag2[d2]) continue;   // prune
        col[c] = diag1[d1] = diag2[d2] = true; cols[row] = c;
        if (place(row + 1, n, cols, col, diag1, diag2)) return true;
        col[c] = diag1[d1] = diag2[d2] = false;           // undo
    }
    return false;
}
```

### Sudoku solver skeleton (very relevant to this repo!)

```cpp
bool solve(vector<vector<char>>& board) {
    for (int r = 0; r < 9; ++r)
        for (int c = 0; c < 9; ++c)
            if (board[r][c] == '.') {
                for (char d = '1'; d <= '9'; ++d) {
                    if (!isValid(board, r, c, d)) continue;
                    board[r][c] = d;              // choose
                    if (solve(board)) return true;// explore
                    board[r][c] = '.';            // undo
                }
                return false;   // no digit worked → backtrack
            }
    return true;                // no empty cell left → solved
}
```

### Word Search (DFS with visited restore)

```cpp
bool dfs(vector<vector<char>>& b, string& w, int k, int r, int c) {
    if (k == w.size()) return true;
    if (r < 0 || c < 0 || r >= b.size() || c >= b[0].size() || b[r][c] != w[k])
        return false;
    char tmp = b[r][c]; b[r][c] = '#';            // mark visited
    bool found = dfs(b,w,k+1,r+1,c) || dfs(b,w,k+1,r-1,c)
              || dfs(b,w,k+1,r,c+1) || dfs(b,w,k+1,r,c-1);
    b[r][c] = tmp;                                // undo
    return found;
}
```

---

## 6. Pruning Techniques (what makes backtracking fast)

- **Constraint check before recursing** (`isValid`) — the biggest win.
- **Sort + early break**: if candidates are sorted and `target - c[i] < 0`, `break` (not `continue`).
- **`used[]` / bitmask** to track state in O(1): columns/diagonals in N-Queens, seen digits in Sudoku.
- **Bitmasks** replace `vector<bool>` for speed: `mask & (1 << i)`, set `mask |= (1 << i)`, clear `mask &= ~(1 << i)`.
- **Skip duplicate branches** (see section 3).
- **Memoization** when subproblems repeat (turns it into DP).

---

## 7. Complexity Cheat Table

| Problem type            | Rough time complexity |
|-------------------------|-----------------------|
| Subsets                 | O(2^n · n)            |
| Permutations            | O(n! · n)             |
| Combinations (k of n)   | O(C(n,k) · k)         |
| Combination Sum         | exponential, pruned   |
| N-Queens                | O(n!)                 |
| Sudoku                  | O(9^m), m = empties   |

Space is usually **O(depth)** for the recursion stack + O(n) for the `cur` path.

---

## 8. Common Bugs & Interview Tips

- **Forgot to undo** (`pop_back`, reset `used[i]`, restore board cell) → corrupted state.
- **Copy vs reference**: push a **copy** into results (`res.push_back(cur)`), but pass `cur` by **reference** during recursion.
- **`start` index**: use `i` for reuse, `i+1` for no reuse, `0` for permutations.
- **Base case placement**: record solution *then* `return`.
- **Return type**: `void` to collect *all* solutions; `bool` to stop at the *first* one.
- **Talk out loud**: state your choice/explore/undo structure — interviewers love hearing the template.
- **Draw the recursion tree** for a tiny input (n=2 or 3) to verify correctness.

---

## 9. Must-Practice LeetCode Problems

| # | Problem | Pattern |
|---|---------|---------|
| 78 | Subsets | include/exclude |
| 90 | Subsets II | dedup |
| 46 | Permutations | used[] |
| 47 | Permutations II | dedup + used[] |
| 77 | Combinations | start index |
| 39 | Combination Sum | reuse (pass i) |
| 40 | Combination Sum II | dedup + no reuse |
| 22 | Generate Parentheses | counter constraints |
| 79 | Word Search | grid DFS |
| 51 | N-Queens | bitmask pruning |
| 37 | Sudoku Solver | grid + bool return |
| 131 | Palindrome Partitioning | substring choices |
| 17 | Letter Combinations | map choices |

---

## 10. Full Solutions

Each solution is self-contained (define the method inside a `Solution` class as on LeetCode).

### 78. Subsets

```cpp
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> res;
        vector<int> cur;
        dfs(0, nums, cur, res);
        return res;
    }
    void dfs(int i, vector<int>& nums, vector<int>& cur, vector<vector<int>>& res) {
        if (i == nums.size()) { res.push_back(cur); return; }
        dfs(i + 1, nums, cur, res);          // exclude nums[i]
        cur.push_back(nums[i]);
        dfs(i + 1, nums, cur, res);          // include nums[i]
        cur.pop_back();
    }
};
```

### 90. Subsets II (with duplicates)

```cpp
class Solution {
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        vector<int> cur;
        dfs(0, nums, cur, res);
        return res;
    }
    void dfs(int start, vector<int>& nums, vector<int>& cur, vector<vector<int>>& res) {
        res.push_back(cur);
        for (int i = start; i < nums.size(); ++i) {
            if (i > start && nums[i] == nums[i - 1]) continue;   // skip dup at this level
            cur.push_back(nums[i]);
            dfs(i + 1, nums, cur, res);
            cur.pop_back();
        }
    }
};
```

### 46. Permutations

```cpp
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> res;
        vector<int> cur;
        vector<bool> used(nums.size(), false);
        dfs(nums, used, cur, res);
        return res;
    }
    void dfs(vector<int>& nums, vector<bool>& used, vector<int>& cur, vector<vector<int>>& res) {
        if (cur.size() == nums.size()) { res.push_back(cur); return; }
        for (int i = 0; i < nums.size(); ++i) {
            if (used[i]) continue;
            used[i] = true; cur.push_back(nums[i]);
            dfs(nums, used, cur, res);
            cur.pop_back(); used[i] = false;
        }
    }
};
```

### 47. Permutations II (with duplicates)

```cpp
class Solution {
public:
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        vector<int> cur;
        vector<bool> used(nums.size(), false);
        dfs(nums, used, cur, res);
        return res;
    }
    void dfs(vector<int>& nums, vector<bool>& used, vector<int>& cur, vector<vector<int>>& res) {
        if (cur.size() == nums.size()) { res.push_back(cur); return; }
        for (int i = 0; i < nums.size(); ++i) {
            if (used[i]) continue;
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;  // dedup
            used[i] = true; cur.push_back(nums[i]);
            dfs(nums, used, cur, res);
            cur.pop_back(); used[i] = false;
        }
    }
};
```

### 77. Combinations

```cpp
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> res;
        vector<int> cur;
        dfs(1, n, k, cur, res);
        return res;
    }
    void dfs(int start, int n, int k, vector<int>& cur, vector<vector<int>>& res) {
        if (cur.size() == k) { res.push_back(cur); return; }
        // prune: not enough numbers left to reach size k
        for (int i = start; i <= n - (k - cur.size()) + 1; ++i) {
            cur.push_back(i);
            dfs(i + 1, n, k, cur, res);
            cur.pop_back();
        }
    }
};
```

### 39. Combination Sum (reuse allowed)

```cpp
class Solution {
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<vector<int>> res;
        vector<int> cur;
        dfs(0, target, candidates, cur, res);
        return res;
    }
    void dfs(int start, int target, vector<int>& c, vector<int>& cur, vector<vector<int>>& res) {
        if (target == 0) { res.push_back(cur); return; }
        for (int i = start; i < c.size(); ++i) {
            if (c[i] > target) break;             // sorted → no larger will fit
            cur.push_back(c[i]);
            dfs(i, target - c[i], c, cur, res);   // i: reuse same element
            cur.pop_back();
        }
    }
};
```

### 40. Combination Sum II (no reuse, dedup)

```cpp
class Solution {
public:
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<vector<int>> res;
        vector<int> cur;
        dfs(0, target, candidates, cur, res);
        return res;
    }
    void dfs(int start, int target, vector<int>& c, vector<int>& cur, vector<vector<int>>& res) {
        if (target == 0) { res.push_back(cur); return; }
        for (int i = start; i < c.size(); ++i) {
            if (i > start && c[i] == c[i - 1]) continue;  // dedup at this level
            if (c[i] > target) break;
            cur.push_back(c[i]);
            dfs(i + 1, target - c[i], c, cur, res);       // i+1: no reuse
            cur.pop_back();
        }
    }
};
```

### 22. Generate Parentheses

```cpp
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> res;
        string cur;
        dfs(0, 0, n, cur, res);
        return res;
    }
    void dfs(int open, int close, int n, string& cur, vector<string>& res) {
        if (cur.size() == 2 * n) { res.push_back(cur); return; }
        if (open < n)  { cur.push_back('('); dfs(open + 1, close, n, cur, res); cur.pop_back(); }
        if (close < open) { cur.push_back(')'); dfs(open, close + 1, n, cur, res); cur.pop_back(); }
    }
};
```

### 79. Word Search

```cpp
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        for (int r = 0; r < board.size(); ++r)
            for (int c = 0; c < board[0].size(); ++c)
                if (dfs(board, word, 0, r, c)) return true;
        return false;
    }
    bool dfs(vector<vector<char>>& b, string& w, int k, int r, int c) {
        if (k == w.size()) return true;
        if (r < 0 || c < 0 || r >= b.size() || c >= b[0].size() || b[r][c] != w[k])
            return false;
        char tmp = b[r][c]; b[r][c] = '#';       // mark visited
        bool found = dfs(b, w, k + 1, r + 1, c) || dfs(b, w, k + 1, r - 1, c)
                  || dfs(b, w, k + 1, r, c + 1) || dfs(b, w, k + 1, r, c - 1);
        b[r][c] = tmp;                           // restore
        return found;
    }
};
```

### 51. N-Queens

```cpp
class Solution {
public:
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> res;
        vector<string> board(n, string(n, '.'));
        vector<bool> col(n, false), d1(2 * n, false), d2(2 * n, false);
        dfs(0, n, board, col, d1, d2, res);
        return res;
    }
    void dfs(int row, int n, vector<string>& board, vector<bool>& col,
             vector<bool>& d1, vector<bool>& d2, vector<vector<string>>& res) {
        if (row == n) { res.push_back(board); return; }
        for (int c = 0; c < n; ++c) {
            int i1 = row + c, i2 = row - c + n;
            if (col[c] || d1[i1] || d2[i2]) continue;
            col[c] = d1[i1] = d2[i2] = true; board[row][c] = 'Q';
            dfs(row + 1, n, board, col, d1, d2, res);
            col[c] = d1[i1] = d2[i2] = false; board[row][c] = '.';
        }
    }
};
```

### 37. Sudoku Solver

```cpp
class Solution {
public:
    void solveSudoku(vector<vector<char>>& board) { solve(board); }

    bool solve(vector<vector<char>>& board) {
        for (int r = 0; r < 9; ++r)
            for (int c = 0; c < 9; ++c)
                if (board[r][c] == '.') {
                    for (char d = '1'; d <= '9'; ++d) {
                        if (!valid(board, r, c, d)) continue;
                        board[r][c] = d;
                        if (solve(board)) return true;
                        board[r][c] = '.';           // undo
                    }
                    return false;                    // no digit fit → backtrack
                }
        return true;                                 // no empty cell → solved
    }
    bool valid(vector<vector<char>>& b, int r, int c, char d) {
        int br = (r / 3) * 3, bc = (c / 3) * 3;
        for (int i = 0; i < 9; ++i) {
            if (b[r][i] == d || b[i][c] == d) return false;
            if (b[br + i / 3][bc + i % 3] == d) return false;
        }
        return true;
    }
};
```

### 131. Palindrome Partitioning

```cpp
class Solution {
public:
    vector<vector<string>> partition(string s) {
        vector<vector<string>> res;
        vector<string> cur;
        dfs(0, s, cur, res);
        return res;
    }
    void dfs(int start, string& s, vector<string>& cur, vector<vector<string>>& res) {
        if (start == s.size()) { res.push_back(cur); return; }
        for (int end = start; end < s.size(); ++end) {
            if (!isPalindrome(s, start, end)) continue;
            cur.push_back(s.substr(start, end - start + 1));
            dfs(end + 1, s, cur, res);
            cur.pop_back();
        }
    }
    bool isPalindrome(string& s, int l, int r) {
        while (l < r) if (s[l++] != s[r--]) return false;
        return true;
    }
};
```

### 17. Letter Combinations of a Phone Number

```cpp
class Solution {
public:
    vector<string> letterCombinations(string digits) {
        if (digits.empty()) return {};
        vector<string> map = {"", "", "abc", "def", "ghi", "jkl",
                              "mno", "pqrs", "tuv", "wxyz"};
        vector<string> res;
        string cur;
        dfs(0, digits, map, cur, res);
        return res;
    }
    void dfs(int i, string& digits, vector<string>& map, string& cur, vector<string>& res) {
        if (i == digits.size()) { res.push_back(cur); return; }
        for (char ch : map[digits[i] - '0']) {
            cur.push_back(ch);
            dfs(i + 1, digits, map, cur, res);
            cur.pop_back();
        }
    }
};
```

---

**One-liner to remember:** *Choose → Explore → Un-choose.* Prune hard, undo everything.
