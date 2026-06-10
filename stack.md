# Stack Pattern — C++ Cheat Sheet

Pattern for problems involving matching pairs, monotonic sequences, nearest greater/smaller elements, and expression evaluation.

---

## Core Concepts

**Stack Properties:** LIFO (Last In, First Out) · O(1) push/pop/top · `std::stack<T>` in C++

**When a stack shines:** Whenever you need to "remember" previous elements and process them in reverse order, or maintain a running context that can be unwound.

```cpp
#include <stack>

stack<int> st;
st.push(42);        // push element
st.top();           // peek top element
st.pop();           // remove top element
st.empty();         // check if empty
st.size();          // number of elements
```

---

## Pattern Categories

| Pattern | Core Idea | Typical Problems |
|---|---|---|
| **Monotonic Stack** | Maintain increasing/decreasing order to find next greater/smaller | Next Greater Element, Daily Temperatures, Largest Rectangle |
| **Matching Pairs** | Push openers, pop on matching closers | Valid Parentheses, Decode String |
| **Expression Eval** | Operand stack + operator stack (or postfix) | Basic Calculator, Evaluate RPN |
| **Stack as History** | Track state that can be undone | Min Stack, Browser History |
| **Nested Structure** | Stack frames for recursive-like nesting | Decode String, Flatten Nested List |

---

## Monotonic Stack — Core Template

The most common stack pattern in interviews. Used to find the **next greater/smaller element** for every position.

### Next Greater Element (Right)

```cpp
// For each element, find the next element to the right that is greater.
// Result[i] = index of next greater element, or -1 if none.
vector<int> nextGreaterRight(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, -1);
    stack<int> st;                    // stores indices
    for (int i = 0; i < n; i++) {
        while (!st.empty() && nums[st.top()] < nums[i]) {
            res[st.top()] = i;       // nums[i] is the answer for st.top()
            st.pop();
        }
        st.push(i);
    }
    return res;
}
```

### Monotonic Stack Variants

| Variant | Stack Order | Comparison | Use Case |
|---|---|---|---|
| Next Greater (right) | Decreasing from bottom | `nums[st.top()] < nums[i]` | Daily Temperatures, Stock Span |
| Next Smaller (right) | Increasing from bottom | `nums[st.top()] > nums[i]` | Largest Rectangle in Histogram |
| Next Greater (left) | Scan left to right, check stack | Same logic, iterate reversed | Previous Greater Element |
| Next Smaller (left) | Scan left to right | Same logic | Trapping Rain Water variant |

```
Decreasing monotonic stack (bottom to top):
  push 5 → [5]
  push 3 → [5, 3]
  push 4 → pop 3 (4 > 3, so 4 is next greater for 3) → [5, 4]
  push 2 → [5, 4, 2]
  push 6 → pop 2, pop 4, pop 5 → [6]
```

---

## When to Use a Stack

| Signal in Problem | Pattern |
|---|---|
| "Next greater / smaller element" | Monotonic stack |
| "Valid parentheses / brackets" | Matching pairs |
| "Evaluate expression / calculator" | Two-stack or postfix evaluation |
| "Decode / flatten nested structure" | Stack as recursion frames |
| "Largest rectangle / maximal area" | Monotonic stack (histogram) |
| "Trapping rain water" | Monotonic stack or two-pointer |
| "Remove k digits / smallest number" | Monotonic stack (greedy removal) |
| "Asteroid collision" | Simulation with stack |

---

## Problem Catalog

### 20. Valid Parentheses — Easy

**Problem:** Given a string of `()[]{}`, determine if the input is valid.

**Key Idea:** Push opening brackets. On a closing bracket, check if top matches. Stack must be empty at end.

**Complexity:** Time O(n) · Space O(n)

```cpp
bool isValid(string s) {
    stack<char> st;
    unordered_map<char, char> match = {{')', '('}, {']', '['}, {'}', '{'}};
    for (char c : s) {
        if (match.count(c)) {
            if (st.empty() || st.top() != match[c])
                return false;
            st.pop();
        } else {
            st.push(c);
        }
    }
    return st.empty();
}
```

---

### 155. Min Stack — Medium

**Problem:** Design a stack that supports push, pop, top, and retrieving the minimum in O(1).

**Key Idea:** Maintain a parallel stack (or pairs) that tracks the running minimum at each level.

**Complexity:** Time O(1) all ops · Space O(n)

```cpp
class MinStack {
    stack<pair<int, int>> st;  // {value, current_min}
public:
    void push(int val) {
        int curMin = st.empty() ? val : min(val, st.top().second);
        st.push({val, curMin});
    }
    void pop() { st.pop(); }
    int top() { return st.top().first; }
    int getMin() { return st.top().second; }
};
```

---

### 739. Daily Temperatures — Medium

**Problem:** Given daily temperatures, for each day find how many days until a warmer temperature.

**Key Idea:** Monotonic decreasing stack of indices. When a warmer day arrives, pop and compute distances.

**Complexity:** Time O(n) · Space O(n)

```cpp
vector<int> dailyTemperatures(vector<int>& temperatures) {
    int n = temperatures.size();
    vector<int> res(n, 0);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && temperatures[st.top()] < temperatures[i]) {
            res[st.top()] = i - st.top();
            st.pop();
        }
        st.push(i);
    }
    return res;
}
```

---

### 496. Next Greater Element I — Easy

**Problem:** Given `nums1` (subset of `nums2`), for each element in `nums1`, find its next greater element in `nums2`.

**Key Idea:** Build a next-greater map for all of `nums2` using a monotonic stack, then look up each element of `nums1`.

**Complexity:** Time O(m + n) · Space O(n)

```cpp
vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
    unordered_map<int, int> nextGreater;
    stack<int> st;
    for (int num : nums2) {
        while (!st.empty() && st.top() < num) {
            nextGreater[st.top()] = num;
            st.pop();
        }
        st.push(num);
    }
    vector<int> res;
    for (int num : nums1)
        res.push_back(nextGreater.count(num) ? nextGreater[num] : -1);
    return res;
}
```

---

### 503. Next Greater Element II — Medium

**Problem:** Same as above but array is **circular**.

**Key Idea:** Iterate through the array twice (modular indexing) to simulate the circular nature.

**Complexity:** Time O(n) · Space O(n)

```cpp
vector<int> nextGreaterElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, -1);
    stack<int> st;
    for (int i = 0; i < 2 * n; i++) {
        while (!st.empty() && nums[st.top()] < nums[i % n]) {
            res[st.top()] = nums[i % n];
            st.pop();
        }
        if (i < n) st.push(i);
    }
    return res;
}
```

---

### 84. Largest Rectangle in Histogram — Hard

**Problem:** Find the largest rectangular area in a histogram.

**Key Idea:** Monotonic increasing stack of indices. When a shorter bar arrives, pop and calculate area using the popped bar as the height and the current boundaries as width.

**Complexity:** Time O(n) · Space O(n)

```cpp
int largestRectangleArea(vector<int>& heights) {
    stack<int> st;
    int maxArea = 0;
    int n = heights.size();
    for (int i = 0; i <= n; i++) {
        int curHeight = (i == n) ? 0 : heights[i];
        while (!st.empty() && heights[st.top()] > curHeight) {
            int h = heights[st.top()]; st.pop();
            int w = st.empty() ? i : (i - st.top() - 1);
            maxArea = max(maxArea, h * w);
        }
        st.push(i);
    }
    return maxArea;
}
```

---

### 85. Maximal Rectangle — Hard

**Problem:** Given a binary matrix, find the largest rectangle containing only 1's.

**Key Idea:** Build histogram heights row by row, then apply Largest Rectangle in Histogram on each row.

**Complexity:** Time O(rows × cols) · Space O(cols)

```cpp
int maximalRectangle(vector<vector<char>>& matrix) {
    if (matrix.empty()) return 0;
    int cols = matrix[0].size();
    vector<int> heights(cols, 0);
    int maxArea = 0;
    for (auto& row : matrix) {
        for (int j = 0; j < cols; j++)
            heights[j] = (row[j] == '1') ? heights[j] + 1 : 0;
        maxArea = max(maxArea, largestRectangleArea(heights));
    }
    return maxArea;
}
```

---

### 42. Trapping Rain Water — Hard

**Problem:** Given elevation bars, compute how much water can be trapped.

**Key Idea (stack approach):** Monotonic decreasing stack. When a taller bar arrives, pop and compute water trapped in the "valley" between the current bar and the new stack top.

**Complexity:** Time O(n) · Space O(n)

```cpp
int trap(vector<int>& height) {
    stack<int> st;
    int water = 0;
    for (int i = 0; i < (int)height.size(); i++) {
        while (!st.empty() && height[st.top()] < height[i]) {
            int bottom = height[st.top()]; st.pop();
            if (st.empty()) break;
            int w = i - st.top() - 1;
            int h = min(height[i], height[st.top()]) - bottom;
            water += w * h;
        }
        st.push(i);
    }
    return water;
}
```

---

### 150. Evaluate Reverse Polish Notation — Medium

**Problem:** Evaluate an arithmetic expression in Reverse Polish Notation.

**Key Idea:** Push numbers onto stack. On an operator, pop two operands, compute, push result.

**Complexity:** Time O(n) · Space O(n)

```cpp
int evalRPN(vector<string>& tokens) {
    stack<long long> st;
    for (auto& t : tokens) {
        if (t == "+" || t == "-" || t == "*" || t == "/") {
            long long b = st.top(); st.pop();
            long long a = st.top(); st.pop();
            if (t == "+") st.push(a + b);
            else if (t == "-") st.push(a - b);
            else if (t == "*") st.push(a * b);
            else st.push(a / b);
        } else {
            st.push(stoll(t));
        }
    }
    return st.top();
}
```

---

### 224. Basic Calculator — Hard

**Problem:** Implement a basic calculator to evaluate `+`, `-`, `(`, `)` with integers.

**Key Idea:** Track running `result` and `sign`. On `(`, push current result and sign onto stack. On `)`, pop and combine.

**Complexity:** Time O(n) · Space O(n)

```cpp
int calculate(string s) {
    stack<int> st;
    int result = 0, num = 0, sign = 1;
    for (char c : s) {
        if (isdigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '+' || c == '-') {
            result += sign * num;
            num = 0;
            sign = (c == '+') ? 1 : -1;
        } else if (c == '(') {
            st.push(result);
            st.push(sign);
            result = 0;
            sign = 1;
        } else if (c == ')') {
            result += sign * num;
            num = 0;
            result *= st.top(); st.pop();  // sign before '('
            result += st.top(); st.pop();  // result before '('
        }
    }
    return result + sign * num;
}
```

---

### 394. Decode String — Medium

**Problem:** Decode `k[encoded_string]`, e.g. `3[a2[c]]` → `accaccacc`.

**Key Idea:** Use two stacks: one for repeat counts, one for accumulated strings. On `[`, push current state. On `]`, pop and repeat.

**Complexity:** Time O(output length) · Space O(output length)

```cpp
string decodeString(string s) {
    stack<int> countStack;
    stack<string> strStack;
    string cur;
    int k = 0;
    for (char c : s) {
        if (isdigit(c)) {
            k = k * 10 + (c - '0');
        } else if (c == '[') {
            countStack.push(k);
            strStack.push(cur);
            cur = "";
            k = 0;
        } else if (c == ']') {
            int repeat = countStack.top(); countStack.pop();
            string prev = strStack.top(); strStack.pop();
            string expanded;
            for (int i = 0; i < repeat; i++) expanded += cur;
            cur = prev + expanded;
        } else {
            cur += c;
        }
    }
    return cur;
}
```

---

### 735. Asteroid Collision — Medium

**Problem:** Asteroids move in a row. Positive = right, negative = left. Find surviving asteroids after collisions.

**Key Idea:** Stack simulates collisions. Push right-moving. On left-moving, pop smaller right-moving asteroids; if equal, both destroy.

**Complexity:** Time O(n) · Space O(n)

```cpp
vector<int> asteroidCollision(vector<int>& asteroids) {
    stack<int> st;
    for (int a : asteroids) {
        bool alive = true;
        while (alive && a < 0 && !st.empty() && st.top() > 0) {
            if (st.top() < -a) {
                st.pop();            // top asteroid destroyed
            } else if (st.top() == -a) {
                st.pop();            // both destroyed
                alive = false;
            } else {
                alive = false;       // incoming destroyed
            }
        }
        if (alive) st.push(a);
    }
    vector<int> res(st.size());
    for (int i = st.size() - 1; i >= 0; i--) {
        res[i] = st.top(); st.pop();
    }
    return res;
}
```

---

### 402. Remove K Digits — Medium

**Problem:** Remove k digits from a number string to make the smallest possible number.

**Key Idea:** Monotonic increasing stack. Remove larger digits from top when a smaller digit arrives (greedy). At most k removals.

**Complexity:** Time O(n) · Space O(n)

```cpp
string removeKdigits(string num, int k) {
    string st;  // use string as stack for easy result
    for (char c : num) {
        while (k > 0 && !st.empty() && st.back() > c) {
            st.pop_back();
            k--;
        }
        st.push_back(c);
    }
    while (k > 0) { st.pop_back(); k--; }
    // remove leading zeros
    int start = 0;
    while (start < (int)st.size() && st[start] == '0') start++;
    string res = st.substr(start);
    return res.empty() ? "0" : res;
}
```

---

### 32. Longest Valid Parentheses — Hard

**Problem:** Find the length of the longest valid (well-formed) parentheses substring.

**Key Idea:** Push indices onto stack. Start with `-1` as base. On `)`, pop and compute length from the new top. If stack is empty, push current index as new base.

**Complexity:** Time O(n) · Space O(n)

```cpp
int longestValidParentheses(string s) {
    stack<int> st;
    st.push(-1);  // base index
    int maxLen = 0;
    for (int i = 0; i < (int)s.size(); i++) {
        if (s[i] == '(') {
            st.push(i);
        } else {
            st.pop();
            if (st.empty()) {
                st.push(i);        // new base
            } else {
                maxLen = max(maxLen, i - st.top());
            }
        }
    }
    return maxLen;
}
```

---

### 901. Online Stock Span — Medium

**Problem:** For each day's stock price, return how many consecutive days (including today) the price was less than or equal to today's price.

**Key Idea:** Monotonic decreasing stack storing `{price, span}`. Pop smaller prices and accumulate their spans.

**Complexity:** Time O(1) amortized per call · Space O(n)

```cpp
class StockSpanner {
    stack<pair<int, int>> st;  // {price, span}
public:
    int next(int price) {
        int span = 1;
        while (!st.empty() && st.top().first <= price) {
            span += st.top().second;
            st.pop();
        }
        st.push({price, span});
        return span;
    }
};
```

---

## Complexity Summary

| Problem | # | Time | Space | Technique |
|---|:---:|:---:|:---:|---|
| Valid Parentheses | 20 | O(n) | O(n) | Matching pairs |
| Min Stack | 155 | O(1) | O(n) | Parallel min tracking |
| Daily Temperatures | 739 | O(n) | O(n) | Monotonic decreasing stack |
| Next Greater Element I | 496 | O(m+n) | O(n) | Monotonic stack + hashmap |
| Next Greater Element II | 503 | O(n) | O(n) | Circular monotonic stack |
| Largest Rectangle | 84 | O(n) | O(n) | Monotonic increasing stack |
| Maximal Rectangle | 85 | O(r×c) | O(c) | Histogram per row |
| Trapping Rain Water | 42 | O(n) | O(n) | Monotonic stack (valley fill) |
| Evaluate RPN | 150 | O(n) | O(n) | Operand stack |
| Basic Calculator | 224 | O(n) | O(n) | Result/sign stack |
| Decode String | 394 | O(output) | O(output) | Count + string dual stack |
| Asteroid Collision | 735 | O(n) | O(n) | Simulation stack |
| Remove K Digits | 402 | O(n) | O(n) | Monotonic increasing (greedy) |
| Longest Valid Parens | 32 | O(n) | O(n) | Index stack with base |
| Stock Span | 901 | O(1)* | O(n) | Monotonic decreasing + span |

\* amortized

---

## Monotonic Stack Decision Guide

| Question | Decreasing Stack (bottom→top) | Increasing Stack (bottom→top) |
|---|---|---|
| What does it find? | Next **greater** element | Next **smaller** element |
| Pop condition | `st.top() < current` | `st.top() > current` |
| Remaining in stack | Elements with no greater to the right | Elements with no smaller to the right |
| Classic problems | Daily Temperatures, Stock Span | Largest Rectangle, Remove K Digits |

---

## Common Tricks & Edge Cases

### Stack Tricks

| Trick | Description |
|---|---|
| **Index stack** | Store indices instead of values — lets you compute distances and widths |
| **Sentinel values** | Push `0` or `-1` at boundaries to avoid empty-stack checks (e.g., Largest Rectangle) |
| **String as stack** | Use `string` with `push_back`/`pop_back` when building a result character by character |
| **Dual stacks** | Separate stacks for different data (count + string, operand + operator) |
| **Circular array** | Iterate `2*n` with `i % n` to handle wrap-around |
| **Amortized O(1)** | Each element pushed and popped at most once → total O(n) for n operations |

### Edge Cases

| Edge Case | Note |
|---|---|
| **Empty string / array** | Return empty or 0 — guard before accessing elements |
| **All same elements** | Monotonic stack won't pop anything — results stay at default |
| **Single element** | Often a trivial answer — no pairs to match, no neighbors |
| **Nested brackets** | `((()))` — stack depth equals nesting level |
| **Unmatched brackets** | `(()` — stack not empty at end means invalid |
| **Negative numbers** | In calculator problems, handle unary minus (e.g., `(-1+2)`) |
| **Integer overflow** | Use `long long` in expression evaluation |

---

## Alternative: Stack with `std::vector`

When you need to access elements beyond the top (e.g., `st[st.size()-2]`), use a vector:

```cpp
vector<int> st;
st.push_back(x);              // push
st.back();                     // top
st.pop_back();                 // pop
st.empty();                    // empty check
st[st.size() - 2];            // access second-from-top
```
