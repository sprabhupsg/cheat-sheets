# Two Pointers Cheat Sheet in C++

---

## 1. Opposite-End (Converging) Pointers

**When to use:** Input is **sorted** (or can be sorted), and you need to find pairs or optimize a value by squeezing inward.

**Template:**

```cpp
pair<int,int> twoSumSorted(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;

    while (lo < hi) {
        int sum = nums[lo] + nums[hi];
        if (sum == target)       return {lo, hi};
        else if (sum < target)   lo++;
        else                     hi--;
    }
    return {-1, -1};
}
```

**Classic problems:**
- Two Sum II (sorted input)
- 3Sum / 3Sum Closest / 4Sum (outer loop + converging inner)
- Container With Most Water
- Trapping Rain Water (two-pointer approach)
- Boats to Save People
- Two Sum Less Than K
- Minimize Maximum Pair Sum in Array
- Bag of Tokens (sort + two pointers for score)
- Valid Triangle Number (fix largest side, converge on smaller two)
- Sort Colors / Dutch National Flag (three-way partition — lo/mid/hi)

**Key insight:** Sorting costs O(n log n) but enables O(n) scans. Worth it when the problem is pair/triplet based.

---

## 2. Same-Direction (Fast & Slow) Pointers — Cycle Detection

**When to use:** Detect **cycles** in linked lists, arrays, or number sequences.

**Template:**

```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Finding the cycle start (Floyd's Phase 2):**

```cpp
ListNode* detectCycleStart(ListNode* head) {
    ListNode* slow = head, *fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            slow = head;
            while (slow != fast) {
                slow = slow->next;
                fast = fast->next;
            }
            return slow; // cycle start
        }
    }
    return nullptr;
}
```

**Classic problems:**
- Linked List Cycle / Linked List Cycle II
- Find the Duplicate Number (treat array as implicit linked list)
- Happy Number (detect cycle in digit-square sequence)
- Circular Array Loop

**Key insight:** Slow moves 1 step, fast moves 2. If they meet, a cycle exists. To find the start, reset one pointer to head and advance both by 1.

---

## 3. Same-Direction — Read/Write (In-Place Transform)

**When to use:** Modify an array **in-place** — remove duplicates, filter, compact.

**Template:**

```cpp
int removeDuplicates(vector<int>& nums) {
    if (nums.empty()) return 0;
    int write = 1; // slow — next write position

    for (int read = 1; read < (int)nums.size(); read++) {
        if (nums[read] != nums[read - 1]) {
            nums[write++] = nums[read];
        }
    }
    return write;
}
```

**Generalized (keep at most k duplicates):**

```cpp
int removeDuplicatesK(vector<int>& nums, int k) {
    int write = 0;
    for (int read = 0; read < (int)nums.size(); read++) {
        if (write < k || nums[read] != nums[write - k]) {
            nums[write++] = nums[read];
        }
    }
    return write;
}
```

**Classic problems:**
- Remove Duplicates from Sorted Array / II (at most 2)
- Remove Element
- Move Zeroes
- Remove Linked List Elements
- Sort Array By Parity / Sort Array By Parity II
- Squeeze / compact operations in streaming contexts
- Apply Operations to an Array
- Separate 0s and 1s in-place

**Key insight:** `read` scans every element; `write` only advances when we keep one. Everything before `write` is the answer.

---

## 4. Same-Direction — Merge Two Sorted Sequences

**When to use:** Merge or compare two sorted arrays/lists element by element.

**Template:**

```cpp
vector<int> mergeSorted(vector<int>& a, vector<int>& b) {
    vector<int> result;
    int i = 0, j = 0;

    while (i < (int)a.size() && j < (int)b.size()) {
        if (a[i] <= b[j]) result.push_back(a[i++]);
        else               result.push_back(b[j++]);
    }
    while (i < (int)a.size()) result.push_back(a[i++]);
    while (j < (int)b.size()) result.push_back(b[j++]);
    return result;
}
```

**Classic problems:**
- Merge Two Sorted Lists (linked list version)
- Merge Sorted Array (in-place, fill from the end)
- Intersection of Two Arrays / II
- Interval List Intersections
- Merge k Sorted Lists (k-way merge with heap, but each pair uses two pointers)
- Squares of a Sorted Array (negative squares from left, positive from right — merge)
- Compare Version Numbers
- Shortest Word Distance

**Key insight:** Both pointers only move forward. Whoever is smaller advances. Remaining tail gets appended.

---

## 5. Palindrome Verification / Construction

**When to use:** Check or build palindromes by comparing from both ends.

**Template:**

```cpp
bool isPalindrome(const string& s) {
    int lo = 0, hi = (int)s.size() - 1;

    while (lo < hi) {
        if (s[lo] != s[hi]) return false;
        lo++; hi--;
    }
    return true;
}
```

**With skip (valid palindrome II — remove at most one char):**

```cpp
bool validPalindromeII(const string& s) {
    int lo = 0, hi = (int)s.size() - 1;

    while (lo < hi) {
        if (s[lo] != s[hi]) {
            return isPalin(s, lo + 1, hi) || isPalin(s, lo, hi - 1);
        }
        lo++; hi--;
    }
    return true;
}

bool isPalin(const string& s, int lo, int hi) {
    while (lo < hi) {
        if (s[lo++] != s[hi--]) return false;
    }
    return true;
}
```

**Classic problems:**
- Valid Palindrome (skip non-alphanumeric)
- Valid Palindrome II (remove at most one character)
- Palindrome Linked List (reverse second half, compare)
- Longest Palindromic Substring (expand-around-center — see Pattern 6)
- Minimum Number of Moves to Make Palindrome (greedy + two pointers)
- Break a Palindrome
- Make String a Subsequence Using Cyclic Increments

---

## 6. Expand-Around-Center

**When to use:** Find palindromic substrings/subsequences by **expanding outward** from each center.

**Template:**

```cpp
string longestPalindrome(string s) {
    int n = s.size(), start = 0, maxLen = 1;

    auto expand = [&](int lo, int hi) {
        while (lo >= 0 && hi < n && s[lo] == s[hi]) {
            if (hi - lo + 1 > maxLen) {
                start = lo;
                maxLen = hi - lo + 1;
            }
            lo--; hi++;
        }
    };

    for (int i = 0; i < n; i++) {
        expand(i, i);     // odd-length
        expand(i, i + 1); // even-length
    }
    return s.substr(start, maxLen);
}
```

**Classic problems:**
- Longest Palindromic Substring
- Palindromic Substrings (count all)
- Longest Palindromic Subsequence (DP, but expand-center gives intuition)
- Count different palindromic subsequences

**Key insight:** There are `2n − 1` centers (n single + n−1 gaps). Each expansion is O(n) worst case → O(n^2) total. For O(n), use Manacher's algorithm.

---

## 7. Linked List — Slow/Fast Structural Operations

**When to use:** Find **middle**, detect **length**, or reorder linked lists without extra space.

**Template (find middle):**

```cpp
ListNode* findMiddle(ListNode* head) {
    ListNode* slow = head, *fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow; // for even length, returns second middle
}
```

**Template (remove N-th from end — gap pointer):**

```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy(0, head);
    ListNode* fast = &dummy, *slow = &dummy;

    for (int i = 0; i <= n; i++) fast = fast->next;

    while (fast) {
        slow = slow->next;
        fast = fast->next;
    }
    ListNode* toDelete = slow->next;
    slow->next = toDelete->next;
    delete toDelete;
    return dummy.next;
}
```

**Classic problems:**
- Middle of the Linked List
- Remove Nth Node From End of List
- Reorder List (find middle → reverse second half → merge)
- Palindrome Linked List (find middle → reverse → compare)
- Split Linked List in Parts
- Delete the Middle Node of a Linked List
- Rotate List (find length with pointer, then relink)
- Intersection of Two Linked Lists (align + walk together)
- Odd Even Linked List (two-pointer weave)
- Swap Nodes in Pairs

**Key insight:** "N-th from end" = create an n-gap between two pointers, then walk until the leader hits null.

---

## 8. Partition / Three-Way Partition (Dutch National Flag)

**When to use:** Partition elements into **2 or 3 groups** in a single pass.

**Template (three-way partition):**

```cpp
void sortColors(vector<int>& nums) {
    int lo = 0, mid = 0, hi = (int)nums.size() - 1;

    while (mid <= hi) {
        if (nums[mid] == 0)      swap(nums[lo++], nums[mid++]);
        else if (nums[mid] == 1) mid++;
        else                     swap(nums[mid], nums[hi--]);
    }
}
```

**Classic problems:**
- Sort Colors (Dutch National Flag)
- Move Zeroes (two-way: non-zero vs zero)
- Sort Array By Parity
- Wiggle Sort / Wiggle Sort II
- Partition Labels (greedy + two-pointer tracking)
- Rearrange Array Elements by Sign
- Separate Digits (partition odd/even digits)
- Quick Select / Quick Sort partition step

**Key insight:** Three pointers — `lo` marks the boundary of group-0, `hi` marks group-2, `mid` scans. Only `mid` decides what moves where.

---

## 9. Substring / Subsequence Matching

**When to use:** Check if one string is a **subsequence** of another, or greedily match patterns.

**Template (is subsequence):**

```cpp
bool isSubsequence(string s, string t) {
    int i = 0, j = 0;

    while (i < (int)s.size() && j < (int)t.size()) {
        if (s[i] == t[j]) i++;
        j++;
    }
    return i == (int)s.size();
}
```

**Template (assign cookies — greedy match):**

```cpp
int assignCookies(vector<int>& children, vector<int>& cookies) {
    sort(children.begin(), children.end());
    sort(cookies.begin(), cookies.end());
    int i = 0, j = 0;

    while (i < (int)children.size() && j < (int)cookies.size()) {
        if (cookies[j] >= children[i]) i++; // child satisfied
        j++;
    }
    return i;
}
```

**Classic problems:**
- Is Subsequence
- Number of Matching Subsequences (multi-pointer with buckets)
- Longest Word in Dictionary through Subsequence
- Assign Cookies
- Minimum Number of Arrows to Burst Balloons (sort + greedy scan)
- Longest Uncommon Subsequence II
- Backspace String Compare (reverse scan with skip counters)
- Append Characters to String to Make Subsequence
- Make String a Subsequence Using Cyclic Increments

---

## 10. Interval / Event Sweep with Two Pointers

**When to use:** Process sorted **intervals or events** — merging, intersecting, or gap-finding.

**Template (interval intersection):**

```cpp
vector<vector<int>> intervalIntersection(
    vector<vector<int>>& A, vector<vector<int>>& B) {
    vector<vector<int>> result;
    int i = 0, j = 0;

    while (i < (int)A.size() && j < (int)B.size()) {
        int lo = max(A[i][0], B[j][0]);
        int hi = min(A[i][1], B[j][1]);
        if (lo <= hi) result.push_back({lo, hi});

        if (A[i][1] < B[j][1]) i++;
        else j++;
    }
    return result;
}
```

**Classic problems:**
- Interval List Intersections
- Merge Intervals (sort + scan)
- Insert Interval
- Meeting Rooms / Meeting Rooms II (sort start/end arrays, sweep)
- Non-overlapping Intervals (greedy, advance the one that ends first)
- Remove Covered Intervals
- Minimum Number of Arrows to Burst Balloons
- Employee Free Time

---

## Quick Decision Guide

| Clue in the problem | Pattern |
|---|---|
| sorted array + find pair with target sum | **1 — Opposite-end** |
| detect cycle in list / sequence | **2 — Fast & slow** |
| remove / compact in-place | **3 — Read/write** |
| merge or intersect sorted data | **4 — Merge pointers** |
| check/build palindrome | **5 — Palindrome verify** |
| find all palindromic substrings | **6 — Expand around center** |
| find middle / k-th from end in list | **7 — Slow/fast structural** |
| partition into 2–3 groups in one pass | **8 — Dutch National Flag** |
| subsequence check / greedy matching | **9 — Subsequence match** |
| sorted intervals — merge/intersect/gap | **10 — Interval sweep** |

---

## Complexity Summary

| Pattern | Time | Space |
|---|---|---|
| Opposite-end | O(n) per scan, O(n log n) with sort | O(1) |
| Fast & slow (cycle) | O(n) | O(1) |
| Read/write (in-place) | O(n) | O(1) |
| Merge two sorted | O(n + m) | O(n + m) or O(1) in-place |
| Palindrome check | O(n) | O(1) |
| Expand around center | O(n^2) | O(1) |
| Linked list structural | O(n) | O(1) |
| Three-way partition | O(n) | O(1) |
| Subsequence match | O(n + m) | O(1) |
| Interval sweep | O(n + m) or O(n log n) with sort | O(1) extra |

---

## Common Pitfalls

1. **Forgetting to sort first** — opposite-end pointers require sorted input. If the problem gives unsorted data and asks for indices, you may need a hashmap instead.
2. **Off-by-one with `lo < hi` vs `lo <= hi`** — use `<` for pair finding (avoid pairing an element with itself); use `<=` for partitioning / palindromes.
3. **Skipping duplicates** — in 3Sum/4Sum, skip duplicates on both the outer loop and the inner pointers to avoid duplicate triplets.
4. **Modifying the list while iterating** — in linked list problems, always use a dummy head node to simplify edge cases (delete head, single node, etc.).
5. **Fast pointer null checks** — always check `fast && fast->next` before advancing two steps to avoid segfaults.
6. **Merge from the wrong end** — when merging in-place (Merge Sorted Array), fill from the **back** to avoid overwriting.
7. **Subsequence vs substring confusion** — subsequence allows gaps, substring doesn't. Two-pointer works for subsequence; sliding window for substring.
8. **Three-way partition: don't increment mid after swapping with hi** — the swapped-in element hasn't been inspected yet.