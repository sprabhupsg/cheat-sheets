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

