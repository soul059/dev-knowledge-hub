# Chapter 2: Find Last Occurrence

[← Previous: Find First Occurrence](01-find-first-occurrence.md) | [Next: Count Occurrences →](03-count-occurrences.md)

---

## Overview

The mirror of "find first occurrence" — find the **last (rightmost)** occurrence of a target in a sorted array with duplicates.

---

## Problem Statement

```
   Input:  arr = [1, 2, 3, 3, 3, 3, 5, 8], target = 3
   Output: 5 (last '3' is at index 5)
   
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 3 │ 3 │ 3 │ 5 │ 8 │
   └───┴───┴───┴───┴───┴───┴───┴───┘
    0   1   2   3   4   5   6   7
                          ↑
                       LAST occurrence (index 5)
```

---

## The Key Insight

When we find `arr[mid] == target`, record it and **keep searching RIGHT** for a later occurrence.

```
   First Occurrence:   arr[mid] == target → hi = mid - 1  (go LEFT)
   Last Occurrence:    arr[mid] == target → lo = mid + 1  (go RIGHT)
```

---

## Algorithm (Pseudocode)

```
FUNCTION findLastOccurrence(arr, n, target):
    lo = 0
    hi = n - 1
    result = -1

    WHILE lo <= hi:
        mid = lo + (hi - lo) / 2
        
        IF arr[mid] == target:
            result = mid           // Record this position
            lo = mid + 1           // Keep searching RIGHT
        ELSE IF arr[mid] < target:
            lo = mid + 1
        ELSE:
            hi = mid - 1

    RETURN result
```

---

## Step-by-Step Trace

### Array: `[1, 2, 3, 3, 3, 3, 5, 8]`, Target: `3`

| Step | lo | hi | mid | arr[mid] | Action | result |
|------|----|----|-----|----------|--------|--------|
| 1 | 0 | 7 | 3 | 3 | == target: result=3, lo=4 | 3 |
| 2 | 4 | 7 | 5 | 3 | == target: result=5, lo=6 | **5** |
| 3 | 6 | 7 | 6 | 5 | > target: hi=5 | 5 |
| 4 | 6 | 5 | — | — | lo > hi: STOP | **5** |

```
   Step 1:
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 3 │ 3 │ 3 │ 5 │ 8 │   mid=3, found 3!
   └───┴───┴───┴───┴───┴───┴───┴───┘   result=3, search RIGHT (lo=4)
    lo          mid              hi

   Step 2:
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 3 │ 3 │ 3 │ 5 │ 8 │   mid=5, found 3!
   └───┴───┴───┴───┴───┴───┴───┴───┘   result=5, search RIGHT (lo=6)
                      lo  mid      hi

   Step 3:
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 3 │ 3 │ 3 │ 5 │ 8 │   mid=6, arr[6]=5 > 3
   └───┴───┴───┴───┴───┴───┴───┴───┘   hi=5
                              lo mid hi

   Step 4: lo=6 > hi=5 → STOP. Answer: 5 ✓
```

---

## Implementation

### Java

```java
public static int findLast(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    int result = -1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (arr[mid] == target) {
            result = mid;
            lo = mid + 1;      // Keep searching right
        } else if (arr[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return result;
}
```

### Python

```python
def find_last(arr, target):
    lo, hi = 0, len(arr) - 1
    result = -1

    while lo <= hi:
        mid = lo + (hi - lo) // 2

        if arr[mid] == target:
            result = mid
            lo = mid + 1       # Keep searching right
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1

    return result
```

---

## Upper Bound (Alternative)

**Upper bound** returns the index of the first element **strictly greater** than target. The last occurrence = upper_bound - 1.

```
   arr = [1, 2, 3, 3, 3, 3, 5, 8]

   upper_bound(3) = index 6  (first element > 3 is 5 at index 6)
   last occurrence = 6 - 1 = 5  ✓
```

```python
def upper_bound(arr, target):
    lo, hi = 0, len(arr)
    
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] <= target:   # Note: <= not <
            lo = mid + 1
        else:
            hi = mid
    
    return lo  # First index > target
```

---

## Side-by-Side Comparison: First vs Last

```
   ┌─────────────────────┬──────────────────────┐
   │   FIND FIRST        │   FIND LAST          │
   ├─────────────────────┼──────────────────────┤
   │ On match:           │ On match:            │
   │   result = mid      │   result = mid       │
   │   hi = mid - 1      │   lo = mid + 1       │
   │   (search LEFT)     │   (search RIGHT)     │
   ├─────────────────────┼──────────────────────┤
   │ Returns leftmost    │ Returns rightmost    │
   │ occurrence          │ occurrence           │
   └─────────────────────┴──────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Modification** | On match: record + search RIGHT |
| **Key Change** | `arr[mid] == target → result = mid; lo = mid + 1` |
| **Time** | O(log n) |
| **Space** | O(1) |
| **Returns** | Index of last occurrence, or -1 |
| **Alternative** | upper_bound - 1 |

---

## Quick Revision Questions

1. **What is the only difference in code between finding the first and last occurrence?**
2. **Trace `findLast([1, 3, 3, 3, 5], 3)` step by step.**
3. **How does `upper_bound` help find the last occurrence?**
4. **If the target appears only once, do `findFirst` and `findLast` return the same index?**
5. **What does `findLast` return when the target is not present?**
6. **Why do we set `lo = mid + 1` instead of just `lo = mid` when we find a match?**

---

[← Previous: Find First Occurrence](01-find-first-occurrence.md) | [Next: Count Occurrences →](03-count-occurrences.md)
