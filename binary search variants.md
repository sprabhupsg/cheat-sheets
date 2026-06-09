Absolutely — here’s a side-by-side cheat sheet for the common **binary search variants** in C++.

## Side-by-side view

| Variant | When to use | Core idea | Update rule on match | Time |
|---|---|---|---|---|
| Normal binary search | Find any occurrence in a sorted array | Compare `mid` with target | Return `mid` immediately | \(O(\log n)\) |
| First occurrence | Sorted array with duplicates, want leftmost index | Keep searching left after a match | `ans = mid; high = mid - 1` | \(O(\log n)\) |
| Last occurrence | Sorted array with duplicates, want rightmost index | Keep searching right after a match | `ans = mid; low = mid + 1` | \(O(\log n)\) |
| Rotated sorted array search | Array was sorted, then rotated | One half is always sorted | Decide which half may contain target | \(O(\log n)\) |

## Code patterns

### 1) Normal binary search
```cpp
int binarySearch(const vector<int>& arr, int target) {
    int low = 0, high = (int)arr.size() - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2;

        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```

### 2) First occurrence
```cpp
int firstOccurrence(const vector<int>& arr, int target) {
    int low = 0, high = (int)arr.size() - 1;
    int ans = -1;

    while (low <= high) {
        int mid = low + (high - low) / 2;

        if (arr[mid] == target) {
            ans = mid;
            high = mid - 1;
        } else if (arr[mid] < target) {
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return ans;
}
```

### 3) Last occurrence
```cpp
int lastOccurrence(const vector<int>& arr, int target) {
    int low = 0, high = (int)arr.size() - 1;
    int ans = -1;

    while (low <= high) {
        int mid = low + (high - low) / 2;

        if (arr[mid] == target) {
            ans = mid;
            low = mid + 1;
        } else if (arr[mid] < target) {
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return ans;
}
```

### 4) Rotated sorted array search
```cpp
int searchRotated(const vector<int>& nums, int target) {
    int left = 0, right = (int)nums.size() - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) return mid;

        if (nums[left] <= nums[mid]) {
            if (nums[left] <= target && target < nums[mid])
                right = mid - 1;
            else
                left = mid + 1;
        } else {
            if (nums[mid] < target && target <= nums[right])
                left = mid + 1;
            else
                right = mid - 1;
        }
    }
    return -1;
}
```

## How to choose

- Use **normal binary search** when you only need to know whether the target exists.
- Use **first occurrence** when duplicates exist and you need the leftmost index.
- Use **last occurrence** when duplicates exist and you need the rightmost index.
- Use **rotated search** when the array is sorted but shifted around a pivot.

## One-line memory trick

- Match and stop: normal search.
- Match and go left: first occurrence.
- Match and go right: last occurrence.
- Find sorted half first: rotated array.

Here are more binary search variant patterns to extend your cheat sheet.

---

## Extended side-by-side view

| Variant | When to use | Core idea | Update rule on match | Time |
|---|---|---|---|---|
| Lower bound (\(\ge\) target) | First element not less than target | Same as first occurrence but for \(\ge\) | `ans = mid; high = mid - 1` | \(O(\log n)\) |
| Upper bound (\(>\) target) | First element strictly greater than target | Keep going left on `>` match | `ans = mid; high = mid - 1` | \(O(\log n)\) |
| Count of element | How many times target appears | Combine first & last occurrence | — | \(O(\log n)\) |
| Peak element | Find a local max in a bitonic/unsorted array | Compare `mid` with `mid+1` | Move toward the rising side | \(O(\log n)\) |
| Min in rotated sorted array | Find the pivot/minimum after rotation | Sorted half can't hold the min | Narrow toward the unsorted half | \(O(\log n)\) |
| Search in 2D matrix | Row-major sorted matrix treated as 1D | Map 1D index → `[row][col]` | Same as normal search | \(O(\log(m \cdot n))\) |
| Binary search on answer | Minimize/maximize value satisfying a predicate | Monotonic predicate over answer space | `ans = mid; high = mid - 1` (or `low = mid + 1`) | \(O(\log(\text{range}) \cdot f)\) |
| Square root (integer) | Largest \(x\) where \(x^2 \le n\) | Search on answer variant | `ans = mid; low = mid + 1` | \(O(\log n)\) |
| Allocate minimum pages / Split array | Minimize the maximum sum when splitting into k parts | Binary search on answer + greedy check | `ans = mid; high = mid - 1` | \(O(n \log(\text{sum}))\) |

---

## Code patterns (continued)

### 5) Lower bound — first element \(\ge\) target

```cpp
int lowerBound(const vector<int>& arr, int target) {
    int low = 0, high = (int)arr.size() - 1;
    int ans = (int)arr.size();          // default: past-the-end

    while (low <= high) {
        int mid = low + (high - low) / 2;

        if (arr[mid] >= target) {
            ans = mid;
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return ans;
}
```

### 6) Upper bound — first element \(>\) target

```cpp
int upperBound(const vector<int>& arr, int target) {
    int low = 0, high = (int)arr.size() - 1;
    int ans = (int)arr.size();

    while (low <= high) {
        int mid = low + (high - low) / 2;

        if (arr[mid] > target) {
            ans = mid;
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return ans;
}
```

### 7) Count of an element (uses first & last occurrence)

```cpp
int countOccurrences(const vector<int>& arr, int target) {
    int first = firstOccurrence(arr, target);
    if (first == -1) return 0;
    int last = lastOccurrence(arr, target);
    return last - first + 1;
}
```

### 8) Find peak element

```cpp
int findPeakElement(const vector<int>& nums) {
    int low = 0, high = (int)nums.size() - 1;

    while (low < high) {                   // note: low < high, NOT <=
        int mid = low + (high - low) / 2;

        if (nums[mid] < nums[mid + 1])
            low = mid + 1;                 // peak is to the right
        else
            high = mid;                    // peak is at mid or to the left
    }
    return low;                            // low == high == peak index
}
```

### 9) Minimum in rotated sorted array (no duplicates)

```cpp
int findMin(const vector<int>& nums) {
    int low = 0, high = (int)nums.size() - 1;

    while (low < high) {
        int mid = low + (high - low) / 2;

        if (nums[mid] > nums[high])
            low = mid + 1;                 // min is in the right half
        else
            high = mid;                    // mid itself could be min
    }
    return nums[low];
}
```

### 10) Search in a row-wise & column-wise sorted 2D matrix

Treats an `m x n` matrix (where `row[0]` of next row > `row[n-1]` of current) as a flat sorted array.

```cpp
bool searchMatrix(const vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
    int low = 0, high = m * n - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2;
        int val = matrix[mid / n][mid % n];

        if (val == target) return true;
        else if (val < target) low = mid + 1;
        else high = mid - 1;
    }
    return false;
}
```

### 11) Binary search on answer — generic template

```cpp
// pred(x) must be monotonic: FFFF...TTTT (or TTTT...FFFF)
// This finds the FIRST x in [lo, hi] where pred(x) is true.
int binarySearchOnAnswer(int lo, int hi, function<bool(int)> pred) {
    int ans = -1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (pred(mid)) {
            ans = mid;
            hi = mid - 1;          // try to go smaller (minimize)
        } else {
            lo = mid + 1;
        }
    }
    return ans;
}
```

### 12) Integer square root

```cpp
long long intSqrt(long long n) {
    long long lo = 0, hi = n, ans = 0;

    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;

        if (mid <= n / mid) {      // avoids overflow vs mid*mid <= n
            ans = mid;
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}
```

### 13) Allocate minimum pages / Split array into k subarrays minimizing max sum

```cpp
bool canSplit(const vector<int>& arr, int k, int maxSum) {
    int parts = 1, curSum = 0;
    for (int x : arr) {
        if (curSum + x > maxSum) {
            ++parts;
            curSum = x;
            if (parts > k) return false;
        } else {
            curSum += x;
        }
    }
    return true;
}

int minMaxSplit(const vector<int>& arr, int k) {
    int lo = *max_element(arr.begin(), arr.end());
    int hi = accumulate(arr.begin(), arr.end(), 0);
    int ans = hi;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (canSplit(arr, k, mid)) {
            ans = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return ans;
}
```

---

## Extended "How to choose"

| Situation | Pattern |
|---|---|
| Just check existence | Normal binary search |
| Leftmost match | First occurrence |
| Rightmost match | Last occurrence |
| Insertion point / `std::lower_bound` | Lower bound |
| First element after target / `std::upper_bound` | Upper bound |
| Frequency of target | Count = last − first + 1 |
| Array is rotated | Rotated search or find-min |
| Local maximum needed | Peak element |
| Flat sorted matrix | 2D matrix search |
| "Minimize the maximum" or "maximize the minimum" | Binary search on answer + greedy check |
| Compute integer sqrt / nth root | Binary search on answer |

---

## One-line memory tricks (extended)

- **Lower bound:** first occurrence, but condition is `>=`.
- **Upper bound:** first occurrence, but condition is `>`.
- **Peak element:** chase the uphill neighbor; use `low < high` (not `<=`).
- **Rotated min:** compare `mid` with `high`; chase the unsorted half.
- **Search on answer:** forget the array — binary search the *result space* with a predicate.

Want me to save all of this into a single `.md` or `.cpp` reference file in your workspace?