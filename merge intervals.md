# Merge Intervals Pattern — C++ Cheat Sheet

Pattern for problems involving overlapping ranges, scheduling, and interval manipulation.

---

## Core Pattern

**Steps:** Sort by start → Scan linearly → Merge or create gap

**Typical complexity:** O(n log n) time, O(n) space

```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    vector<vector<int>> merged;
    for (auto& iv : intervals) {
        if (merged.empty() || merged.back()[1] < iv[0]) {
            merged.push_back(iv);                        // no overlap → new interval
        } else {
            merged.back()[1] = max(merged.back()[1], iv[1]); // extend
        }
    }
    return merged;
}
```

---

## Overlap Detection

Given sorted intervals `A=[a1,a2]` and `B=[b1,b2]` where `a1 <= b1`:

| Condition | Relationship | Action |
|---|---|---|
| `a2 < b1` | No overlap | Push B as new interval |
| `a2 >= b1 && a2 < b2` | Partial overlap | Extend end to `b2` |
| `a2 >= b2` | B inside A | Skip B (already covered) |

```
  A:  |---------|
  B:      |---------|     ← partial overlap
  B:    |---|             ← contained
  B:              |---|   ← no overlap
```

---

## When to Use This Pattern

| Signal in Problem | Example |
|---|---|
| "Merge overlapping intervals" | Consolidate meeting times |
| "Insert interval into sorted list" | Add a new booking to a calendar |
| "Find gaps / free time" | Employee free time across schedules |
| "Minimum platforms / rooms" | Meeting rooms, train platforms |
| "Interval intersection" | Common availability between two people |
| "Non-overlapping intervals" | Minimum removals to eliminate overlap |

---

## Problem Catalog

### 56. Merge Intervals — Medium

**Problem:** Given an array of intervals, merge all overlapping intervals.

**Key Idea:** Sort by start. Iterate and either extend the last merged interval or push a new one.

**Complexity:** Time O(n log n) · Space O(n)

```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    vector<vector<int>> res;
    for (auto& iv : intervals) {
        if (res.empty() || res.back()[1] < iv[0])
            res.push_back(iv);
        else
            res.back()[1] = max(res.back()[1], iv[1]);
    }
    return res;
}
```

---

### 57. Insert Interval — Medium

**Problem:** Insert a new interval into a sorted non-overlapping list, merging if necessary.

**Key Idea:** Three passes: copy intervals before overlap, merge overlapping ones, copy intervals after.

**Complexity:** Time O(n) · Space O(n)

```cpp
vector<vector<int>> insert(vector<vector<int>>& intervals,
                           vector<int>& newInterval) {
    vector<vector<int>> res;
    int i = 0, n = intervals.size();
    // intervals completely before newInterval
    while (i < n && intervals[i][1] < newInterval[0])
        res.push_back(intervals[i++]);
    // merge overlapping intervals
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = min(newInterval[0], intervals[i][0]);
        newInterval[1] = max(newInterval[1], intervals[i][1]);
        i++;
    }
    res.push_back(newInterval);
    // intervals completely after
    while (i < n)
        res.push_back(intervals[i++]);
    return res;
}
```

---

### 435. Non-overlapping Intervals — Medium

**Problem:** Return the minimum number of intervals to remove so the rest don't overlap.

**Key Idea:** Greedy: sort by end time. Keep the interval that ends earliest — it leaves maximum room for future intervals.

**Complexity:** Time O(n log n) · Space O(1)

```cpp
int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(),
         [](auto& a, auto& b) { return a[1] < b[1]; });
    int removals = 0, prevEnd = INT_MIN;
    for (auto& iv : intervals) {
        if (iv[0] >= prevEnd)
            prevEnd = iv[1];        // keep this interval
        else
            removals++;            // remove (overlaps)
    }
    return removals;
}
```

---

### 252. Meeting Rooms — Easy

**Problem:** Can a person attend all meetings? (No overlapping intervals)

**Key Idea:** Sort by start. If any interval starts before the previous one ends, return false.

**Complexity:** Time O(n log n) · Space O(1)

```cpp
bool canAttendMeetings(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    for (int i = 1; i < (int)intervals.size(); i++) {
        if (intervals[i][0] < intervals[i - 1][1])
            return false;
    }
    return true;
}
```

---

### 253. Meeting Rooms II — Medium

**Problem:** Find the minimum number of conference rooms required.

**Key Idea:** Use a min-heap tracking end times. If the earliest room is free (ends <= current start), reuse it; otherwise allocate a new room.

**Complexity:** Time O(n log n) · Space O(n)

```cpp
int minMeetingRooms(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    priority_queue<int, vector<int>, greater<int>> pq; // min-heap of end times
    for (auto& iv : intervals) {
        if (!pq.empty() && pq.top() <= iv[0])
            pq.pop();              // reuse earliest-free room
        pq.push(iv[1]);
    }
    return pq.size();
}
```

---

### 986. Interval List Intersections — Medium

**Problem:** Given two sorted interval lists, return their intersection.

**Key Idea:** Two pointers. The intersection is `[max(starts), min(ends)]` — valid only if `max(starts) <= min(ends)`. Advance the pointer with the smaller end.

**Complexity:** Time O(m + n) · Space O(m + n)

```cpp
vector<vector<int>> intervalIntersection(
        vector<vector<int>>& A, vector<vector<int>>& B) {
    vector<vector<int>> res;
    int i = 0, j = 0;
    while (i < (int)A.size() && j < (int)B.size()) {
        int lo = max(A[i][0], B[j][0]);
        int hi = min(A[i][1], B[j][1]);
        if (lo <= hi)
            res.push_back({lo, hi});
        // advance the interval that ends first
        if (A[i][1] < B[j][1]) i++;
        else j++;
    }
    return res;
}
```

---

### 759. Employee Free Time — Hard

**Problem:** Given each employee's sorted working intervals, find common free time across all.

**Key Idea:** Flatten all intervals, merge them, then the gaps between merged intervals are the free time.

**Complexity:** Time O(n log n) · Space O(n)

```cpp
vector<Interval> employeeFreeTime(vector<vector<Interval>>& schedule) {
    vector<Interval> all;
    for (auto& emp : schedule)
        for (auto& iv : emp)
            all.push_back(iv);
    sort(all.begin(), all.end(),
         [](auto& a, auto& b) { return a.start < b.start; });
    // merge
    vector<Interval> merged;
    for (auto& iv : all) {
        if (merged.empty() || merged.back().end < iv.start)
            merged.push_back(iv);
        else
            merged.back().end = max(merged.back().end, iv.end);
    }
    // gaps between merged intervals = free time
    vector<Interval> res;
    for (int i = 1; i < (int)merged.size(); i++)
        res.push_back({merged[i - 1].end, merged[i].start});
    return res;
}
```

---

### 1288. Remove Covered Intervals — Medium

**Problem:** Remove intervals that are covered by another interval. Return the count of remaining.

**Key Idea:** Sort by start ascending, then by end descending. An interval is covered if its end <= the running max end.

**Complexity:** Time O(n log n) · Space O(1)

```cpp
int removeCoveredIntervals(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(),
         [](auto& a, auto& b) {
             return a[0] != b[0] ? a[0] < b[0] : a[1] > b[1];
         });
    int remaining = 0, maxEnd = 0;
    for (auto& iv : intervals) {
        if (iv[1] > maxEnd) {
            remaining++;
            maxEnd = iv[1];
        }
    }
    return remaining;
}
```

---

## Complexity Summary

| Problem | # | Time | Space | Technique |
|---|:---:|:---:|:---:|---|
| Merge Intervals | 56 | O(n log n) | O(n) | Sort + greedy scan |
| Insert Interval | 57 | O(n) | O(n) | Three-phase linear scan |
| Non-overlapping Intervals | 435 | O(n log n) | O(1) | Sort by end + greedy |
| Meeting Rooms | 252 | O(n log n) | O(1) | Sort + adjacent check |
| Meeting Rooms II | 253 | O(n log n) | O(n) | Sort + min-heap |
| Interval Intersections | 986 | O(m+n) | O(m+n) | Two pointers |
| Employee Free Time | 759 | O(n log n) | O(n) | Flatten + merge + gaps |
| Remove Covered | 1288 | O(n log n) | O(1) | Custom sort + running max |

---

## Sorting Strategies

| Strategy | When to Use |
|---|---|
| **By start (default)** | Standard merge, insert, intersection problems |
| **By end** | Greedy removal / scheduling (keep earliest-finishing) |
| **By start asc, end desc** | Detecting covered intervals — longer intervals come first at each start point |
| **Separate start/end arrays** | Sweep-line variant for counting concurrent events |

---

## Sweep Line Variant

Alternative to heap-based approach for Meeting Rooms II. Useful for "max concurrent events" problems.

```cpp
int minMeetingRooms(vector<vector<int>>& intervals) {
    map<int, int> events;       // timeline of events
    for (auto& iv : intervals) {
        events[iv[0]]++;        // +1 at start
        events[iv[1]]--;        // -1 at end
    }
    int rooms = 0, maxRooms = 0;
    for (auto& [time, delta] : events) {
        rooms += delta;
        maxRooms = max(maxRooms, rooms);
    }
    return maxRooms;
}
```

---

## Edge Cases to Watch

| Edge Case | Note |
|---|---|
| **Single interval** | Result is itself — ensure no off-by-one in loops |
| **Touching boundaries** | `[1,3]` and `[3,5]` — clarify if these count as overlapping |
| **Identical intervals** | `[1,5]` and `[1,5]` — should collapse into one |
| **Empty input** | Return empty immediately — don't access index 0 |
| **Nested intervals** | `[1,10]` contains `[2,3]` — the merge should keep `[1,10]` |
