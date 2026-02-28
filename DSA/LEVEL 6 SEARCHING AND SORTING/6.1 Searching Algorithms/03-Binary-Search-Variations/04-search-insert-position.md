# Chapter 4: Search Insert Position

[← Previous: Count Occurrences](03-count-occurrences.md) | [Next: Floor and Ceiling →](05-floor-and-ceiling.md)

---

## Overview

Given a sorted array and a target, find the index where the target **is or should be inserted** to keep the array sorted. This is also known as **lower bound**.

---

## Problem Statement

```
   Input:  arr = [1, 3, 5, 6], target = 5    →  Output: 2  (found at index 2)
   Input:  arr = [1, 3, 5, 6], target = 2    →  Output: 1  (insert at index 1)
   Input:  arr = [1, 3, 5, 6], target = 7    →  Output: 4  (insert at end)
   Input:  arr = [1, 3, 5, 6], target = 0    →  Output: 0  (insert at beginning)
```

```
   arr = [1, 3, 5, 6],  target = 2

   ┌───┬───┬───┬───┐
   │ 1 │ 3 │ 5 │ 6 │     Where does 2 go?
   └───┴───┴───┴───┘
    0   1   2   3
        ↑
   Insert here (index 1) → [1, 2, 3, 5, 6]
```

---

## The Insight

When binary search ends (target not found), `lo` points to exactly where the target **should be inserted**.

```
   arr = [1, 3, 5, 6], target = 4

   Step 1: lo=0, hi=3, mid=1, arr[1]=3 < 4 → lo=2
   Step 2: lo=2, hi=3, mid=2, arr[2]=5 > 4 → hi=1
   Step 3: lo=2 > hi=1 → STOP

   lo = 2 → Insert position!
   
   ┌───┬───┬───┬───┐
   │ 1 │ 3 │ 5 │ 6 │
   └───┴───┴───┴───┘
            ↑
           lo=2 → 4 goes HERE, between 3 and 5 ✓
```

---

## Algorithm (Pseudocode)

```
FUNCTION searchInsertPosition(arr, n, target):
    lo = 0
    hi = n - 1

    WHILE lo <= hi:
        mid = lo + (hi - lo) / 2

        IF arr[mid] == target:
            RETURN mid             // Found at this index
        ELSE IF arr[mid] < target:
            lo = mid + 1
        ELSE:
            hi = mid - 1

    RETURN lo                       // Insert position
```

> **Key:** The only change from standard binary search is the last line: `return lo` instead of `return -1`.

---

## Step-by-Step Traces

### Case 1: Target Found — `arr = [1, 3, 5, 6]`, target = `5`

| Step | lo | hi | mid | arr[mid] | Action |
|------|----|----|----|----------|--------|
| 1 | 0 | 3 | 1 | 3 | 3 < 5 → lo=2 |
| 2 | 2 | 3 | 2 | 5 | 5 == 5 → **return 2** |

### Case 2: Insert in middle — target = `4`

| Step | lo | hi | mid | arr[mid] | Action |
|------|----|----|----|----------|--------|
| 1 | 0 | 3 | 1 | 3 | 3 < 4 → lo=2 |
| 2 | 2 | 3 | 2 | 5 | 5 > 4 → hi=1 |
| 3 | 2 | 1 | — | — | lo > hi → **return lo=2** |

### Case 3: Insert at end — target = `7`

| Step | lo | hi | mid | arr[mid] | Action |
|------|----|----|----|----------|--------|
| 1 | 0 | 3 | 1 | 3 | 3 < 7 → lo=2 |
| 2 | 2 | 3 | 2 | 5 | 5 < 7 → lo=3 |
| 3 | 3 | 3 | 3 | 6 | 6 < 7 → lo=4 |
| 4 | 4 | 3 | — | — | lo > hi → **return lo=4** |

### Case 4: Insert at beginning — target = `0`

| Step | lo | hi | mid | arr[mid] | Action |
|------|----|----|----|----------|--------|
| 1 | 0 | 3 | 1 | 3 | 3 > 0 → hi=0 |
| 2 | 0 | 0 | 0 | 1 | 1 > 0 → hi=-1 |
| 3 | 0 | -1 | — | — | lo > hi → **return lo=0** |

---

## Why `lo` is the Insert Position

```
   After the loop, the state is always:

   ┌─────────────────┬┬──────────────────┐
   │ All < target    ││  All ≥ target    │
   └─────────────────┴┴──────────────────┘
                     hi lo
                     
   hi points to last element < target
   lo points to first element ≥ target
   
   → lo is exactly where target should be inserted!
```

---

## Implementation

### Java

```java
public static int searchInsert(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return lo;
}
```

### Python

```python
def search_insert(arr, target):
    lo, hi = 0, len(arr) - 1

    while lo <= hi:
        mid = lo + (hi - lo) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1

    return lo

# Python built-in: bisect.bisect_left(arr, target)
```

---

## Connection to Python's `bisect` Module

```python
import bisect

arr = [1, 3, 5, 6]

bisect.bisect_left(arr, 4)   # Returns 2 (insert position for 4)
bisect.bisect_left(arr, 5)   # Returns 2 (index of first 5)
bisect.bisect_right(arr, 5)  # Returns 3 (index AFTER last 5)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Returns** | Index if found, otherwise insert position |
| **Key change from standard BS** | `return lo` instead of `return -1` |
| **Time** | O(log n) |
| **Space** | O(1) |
| **Equivalent to** | `lower_bound` / `bisect_left` |

---

## Quick Revision Questions

1. **What is the only difference between standard binary search and search insert position?**
2. **Why does `lo` give the correct insert position after the loop?**
3. **What does search insert return for an empty array with any target?**
4. **How is search insert position related to `bisect_left` in Python?**
5. **Trace search insert on `[2, 4, 6, 8]` with target = 5.**
6. **If all array elements are less than the target, what index is returned?**

---

[← Previous: Count Occurrences](03-count-occurrences.md) | [Next: Floor and Ceiling →](05-floor-and-ceiling.md)
