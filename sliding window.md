# Sliding Window Cheat Sheet in C++ (Expanded)

---

## 1. Fixed-Size Window

**When to use:** Window size `k` is given upfront.

**Template:**

```cpp
int fixedWindow(vector<int>& nums, int k) {
    int n = nums.size(), windowSum = 0, best = INT_MIN;

    for (int i = 0; i < n; i++) {
        windowSum += nums[i];
        if (i >= k) windowSum -= nums[i - k];
        if (i >= k - 1) best = max(best, windowSum);
    }
    return best;
}
```

**Classic problems:**
- Max / Min sum subarray of size k
- Maximum of all subarrays of size k (use deque — see Pattern 5)
- Maximum average subarray of size k
- Number of sub-arrays of size k with average >= threshold
- Contains Duplicate II (`nums[i] == nums[j]` and `|i−j| <= k`)
- Grumpy Bookstore Owner (maximize satisfaction using k-minute trick)
- K Radius Subarray Averages
- Maximum Points You Can Obtain from Cards (prefix/suffix trick with window of size `n−k`)
- Diet Plan Performance (count calories in every k-day window)
- Defuse the Bomb (circular fixed-size window)

---

## 2. Variable-Size Window — Longest / Maximum

**When to use:** Find the **longest** subarray/substring satisfying a condition.

**Template:**

```cpp
int longestWindow(vector<int>& nums, int target) {
    int left = 0, best = 0, curr = 0;

    for (int right = 0; right < (int)nums.size(); right++) {
        curr += nums[right];

        while (curr > target) {
            curr -= nums[left++];
        }
        best = max(best, right - left + 1);
    }
    return best;
}
```

**Classic problems:**
- Longest Substring Without Repeating Characters
- Longest Substring with At Most K Distinct Characters
- Longest Repeating Character Replacement (at most k replacements)
- Max Consecutive Ones III (at most k flips)
- Longest Subarray of 1's After Deleting One Element
- Longest Nice Subarray (bitwise AND of any two elements is 0)
- Longest Semi-Repetitive Substring (at most one pair of adjacent equal chars)
- Longest Substring with At Least K Repeating Characters (divide & conquer + window hybrid)
- Maximum Erasure Value (longest subarray with unique elements, maximize sum)
- Frequency of the Most Frequent Element (with sorting + sliding window)
- Get Equal Substrings Within Budget (max length where cost <= maxCost)
- Longest Turbulent Subarray
- Replace the Substring for Balanced String

**Optimization — non-shrinking window trick:**

When you only care about the **length** of the best window and the window can only grow or stay the same, skip the inner while and use a single `if`:

```cpp
int longestNonShrink(string s, int k) {
    int left = 0, maxFreq = 0;
    unordered_map<char, int> freq;

    for (int right = 0; right < (int)s.size(); right++) {
        maxFreq = max(maxFreq, ++freq[s[right]]);

        if (right - left + 1 - maxFreq > k) {
            freq[s[left++]]--;
            // window size stays the same — never shrinks below best
        }
    }
    return (int)s.size() - left;
}
```

This replaces `while` with `if` — the window never shrinks below its best-so-far length. Works for **Longest Repeating Character Replacement** and similar.

---

## 3. Variable-Size Window — Shortest / Minimum

**When to use:** Find the **shortest** subarray/substring satisfying a condition.

**Template:**

```cpp
int shortestWindow(vector<int>& nums, int target) {
    int left = 0, best = INT_MAX, curr = 0;

    for (int right = 0; right < (int)nums.size(); right++) {
        curr += nums[right];

        while (curr >= target) {
            best = min(best, right - left + 1);
            curr -= nums[left++];
        }
    }
    return best == INT_MAX ? 0 : best;
}
```

**Classic problems:**
- Minimum Size Subarray Sum
- Minimum Window Substring
- Shortest Subarray with Sum at Least K (needs deque for negative numbers — see Pattern 7)
- Minimum Operations to Reduce X to Zero (invert: find longest subarray with sum = totalSum − x)
- Minimum Number of Flips to Make Binary String Alternating
- Minimum Window Subsequence (DP + two-pointer hybrid)
- Minimum Consecutive Cards to Pick Up
- Smallest Range Covering Elements from K Lists (multi-pointer + sorted set / heap)

---

## 4. Window with HashMap / Frequency Map

**When to use:** Tracking character/element counts inside the window.

**Template:**

```cpp
int windowWithMap(string s, int k) {
    unordered_map<char, int> freq;
    int left = 0, best = 0;

    for (int right = 0; right < (int)s.size(); right++) {
        freq[s[right]]++;

        while ((int)freq.size() > k) {
            if (--freq[s[left]] == 0) freq.erase(s[left]);
            left++;
        }
        best = max(best, right - left + 1);
    }
    return best;
}
```

**"Exactly K" via atMost decomposition:**

```cpp
// exactly(k) = atMost(k) - atMost(k - 1)
int atMost(vector<int>& nums, int k) {
    unordered_map<int, int> freq;
    int left = 0, count = 0;

    for (int right = 0; right < (int)nums.size(); right++) {
        if (++freq[nums[right]] == 1) k--;

        while (k < 0) {
            if (--freq[nums[left]] == 0) k++;
            left++;
        }
        count += right - left + 1; // all subarrays ending at right
    }
    return count;
}

int exactlyK(vector<int>& nums, int k) {
    return atMost(nums, k) - atMost(nums, k - 1);
}
```

**Classic problems:**
- Longest Substring with At Most K Distinct Characters
- Fruit Into Baskets (at most 2 distinct)
- Subarrays with K Different Integers (exactly k)
- Count Number of Nice Subarrays (exactly k odd numbers — treat odd as 1, even as 0)
- Binary Subarrays With Sum (exactly k)
- Number of Substrings Containing All Three Characters
- Count Subarrays Where Max Element Appears at Least K Times
- Maximum Sum of Distinct Subarrays With Length K (fixed window + set/map hybrid)
- Subarray Product Less Than K (all subarrays with product < k)
- Count Subarrays With Score Less Than K

---

## 5. Sliding Window + Monotonic Deque

**When to use:** Need the **max or min** in every window position in O(n).

**Template (max):**

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;
    vector<int> result;

    for (int i = 0; i < (int)nums.size(); i++) {
        while (!dq.empty() && dq.front() <= i - k)
            dq.pop_front();
        while (!dq.empty() && nums[dq.back()] <= nums[i])
            dq.pop_back();
        dq.push_back(i);

        if (i >= k - 1)
            result.push_back(nums[dq.front()]);
    }
    return result;
}
```

**Template (min) — flip the comparison:**

```cpp
// maintain increasing deque for min queries
while (!dq.empty() && nums[dq.back()] >= nums[i])
    dq.pop_back();
```

**Classic problems:**
- Sliding Window Maximum
- Sliding Window Minimum (same template, flip comparison)
- Longest Subarray With Absolute Diff <= Limit (two deques: one for max, one for min)
- Jump Game VI (DP + monotonic deque)
- Constrained Subsequence Sum (DP + monotonic deque)
- Shortest Subarray with Sum at Least K (monotonic deque on prefix sums — see Pattern 7)
- Maximum Number of Robots Within Budget (deque for max + prefix sum for running cost)
- Continuous Subarrays (two deques to maintain max−min <= 2)

---

## 6. Sliding Window + Match Count / Anagram Pattern

**When to use:** Check if a window contains a valid anagram/permutation of a target string.

**Template:**

```cpp
vector<int> findAnagrams(string s, string p) {
    vector<int> result;
    if (s.size() < p.size()) return result;

    array<int, 26> need{}, have{};
    for (char c : p) need[c - 'a']++;

    int matched = 0, required = 0;
    for (int x : need) if (x > 0) required++;

    for (int i = 0; i < (int)s.size(); i++) {
        int c = s[i] - 'a';
        have[c]++;
        if (have[c] == need[c]) matched++;

        if (i >= (int)p.size()) {
            int out = s[i - p.size()] - 'a';
            if (have[out] == need[out]) matched--;
            have[out]--;
        }
        if (matched == required)
            result.push_back(i - (int)p.size() + 1);
    }
    return result;
}
```

**Classic problems:**
- Find All Anagrams in a String
- Permutation in String
- Minimum Window Substring (variable window + match count)
- Check Inclusion (boolean version of permutation check)
- Substring with Concatenation of All Words (word-level sliding window with map)
- Minimum Number of Operations to Make Array Continuous (sort + window on unique values)

---

## 7. Sliding Window on Prefix Sums (Deque-Based)

**When to use:** Subarray sum problems with **negative numbers** where a plain two-pointer breaks monotonicity.

**Template:**

```cpp
int shortestSubarrayWithSumAtLeastK(vector<int>& nums, int k) {
    int n = nums.size(), best = INT_MAX;
    vector<long long> prefix(n + 1, 0);
    for (int i = 0; i < n; i++)
        prefix[i + 1] = prefix[i] + nums[i];

    deque<int> dq; // increasing deque of prefix-sum indices

    for (int i = 0; i <= n; i++) {
        while (!dq.empty() && prefix[i] - prefix[dq.front()] >= k) {
            best = min(best, i - dq.front());
            dq.pop_front();
        }
        while (!dq.empty() && prefix[i] <= prefix[dq.back()])
            dq.pop_back();
        dq.push_back(i);
    }
    return best == INT_MAX ? -1 : best;
}
```

**Classic problems:**
- Shortest Subarray with Sum at Least K (LC 862)
- Maximum Subarray Sum with Length at Least K (prefix + deque for min)
- Number of Subarrays with Bounded Maximum (two-pointer counting variant)

---

## 8. Two-Pointer / Shrink-from-Both-Ends Window

**When to use:** Window shrinks from **both** ends, or you need a centered / symmetric window.

**Template (container pattern):**

```cpp
int maxArea(vector<int>& height) {
    int left = 0, right = (int)height.size() - 1, best = 0;

    while (left < right) {
        int area = min(height[left], height[right]) * (right - left);
        best = max(best, area);
        if (height[left] < height[right]) left++;
        else right--;
    }
    return best;
}
```

**Classic problems:**
- Container With Most Water
- Trapping Rain Water (two-pointer approach)
- Two Sum II (sorted array)
- 3Sum / 4Sum (outer loop + two-pointer inner)
- Boats to Save People
- Bag of Tokens (sort + two pointers for score maximization)
- Minimize Maximum Pair Sum (greedy two-pointer)

---

## 9. Multi-Sequence / Multi-Pointer Sliding Window

**When to use:** Window spans multiple arrays or sorted sequences simultaneously.

**Template idea (smallest range covering elements from K lists):**

```cpp
// Priority queue of (value, listIndex, elementIndex)
// Track global max; the window is [pq.top(), globalMax]
// Expand by advancing the pointer of the list that produced the min
```

**Classic problems:**
- Smallest Range Covering Elements from K Lists
- Longest Word in Dictionary through Subsequence (sort + multi-pointer matching)
- Intersection of multiple sorted arrays (k-pointer sweep)

---

## Quick Decision Guide (Expanded)

| Clue in the problem | Pattern |
|---|---|
| "subarray/substring of size k" | **1 — Fixed window** |
| "longest/maximum … such that …" | **2 — Variable, expand-then-shrink** |
| "shortest/minimum … such that …" | **3 — Variable, shrink-while-valid** |
| "at most / at least k distinct" | **4 — HashMap window** |
| "exactly k" | **4 — atMost(k) − atMost(k−1)** |
| "count all valid subarrays" | **4 — add `right−left+1` each step** |
| "max/min of every window" | **5 — Monotonic deque** |
| "contains permutation / anagram" | **6 — Match count** |
| negative numbers + subarray sum | **7 — Prefix sum + deque** |
| sorted input + two ends | **8 — Shrink from both ends** |
| multiple sorted lists | **9 — Multi-pointer / heap** |

---

## Complexity Summary

| Pattern | Time | Space |
|---|---|---|
| Fixed window | O(n) | O(1) |
| Variable (longest/shortest) | O(n) | O(1) or O(k) |
| HashMap window | O(n) | O(k) |
| Monotonic deque | O(n) | O(k) |
| Match count | O(n) | O(alphabet) |
| Prefix sum + deque | O(n) | O(n) |
| Two-pointer (both ends) | O(n) | O(1) |
| Multi-pointer / heap | O(N log K) | O(K) |

---

## Common Pitfalls

1. **Off-by-one on window size** — `[left, right]` has size `right - left + 1`.
2. **Forgetting to erase zero-count keys** — stale keys break `freq.size()` checks.
3. **Shrinking too early vs too late** — for "longest" update **after** the while; for "shortest" update **inside** the while.
4. **Integer overflow** — use `long long` when summing large arrays or computing products.
5. **Negative numbers break monotonicity** — plain two-pointer won't work; switch to prefix-sum + deque (Pattern 7).
6. **Circular arrays** — either double the array (`nums + nums`) or use modular indexing.
7. **"Exactly K" traps** — don't try to maintain exactly k in the window; decompose into `atMost(k) - atMost(k-1)`.
8. **Non-shrinking window misuse** — only valid when you need the length of the best window, not its position or count.