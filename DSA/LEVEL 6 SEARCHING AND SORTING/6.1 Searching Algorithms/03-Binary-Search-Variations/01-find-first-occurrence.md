# Chapter 1: Find First Occurrence

[← Previous: Unit 2 — Prerequisites & Gotchas](../02-Binary-Search-Basics/06-prerequisites-and-gotchas.md) | [Next: Find Last Occurrence →](02-find-last-occurrence.md)

---

## Overview

Standard binary search finds **any** occurrence of the target. But what if the array has **duplicates** and you need the **first (leftmost)** occurrence? This requires a modified binary search.

---

## Problem Statement

```
   Given a sorted array with duplicates, find the INDEX of the
   first occurrence of a target value. Return -1 if not found.

   Input:  arr = [1, 2, 3, 3, 3, 3, 5, 8], target = 3
   Output: 2 (first '3' is at index 2)
```

---

## Why Standard Binary Search Fails

```
   arr = [1, 2, 3, 3, 3, 3, 5, 8]    target = 3
   
   Standard binary search:
   lo=0, hi=7, mid=3 → arr[3]=3 == 3 → return 3
   
   But the FIRST occurrence is at index 2, not 3!
   
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 3 │ 3 │ 3 │ 5 │ 8 │
   └───┴───┴───┴───┴───┴───┴───┴───┘
            ↑   ↑
          FIRST  Standard BS stops here
```

---

## The Key Insight

When we find `arr[mid] == target`, **don't stop!** Instead, record it as a candidate and **keep searching LEFT** for an earlier occurrence.

```
   Standard:    arr[mid] == target → RETURN mid
   Modified:    arr[mid] == target → record mid, search LEFT (hi = mid - 1)
```

```
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 3 │ 3 │ 3 │ 5 │ 8 │
   └───┴───┴───┴───┴───┴───┴───┴───┘
                  ↑
   Found 3 here! But is there a 3 FURTHER LEFT?
   → Set result = 3, then hi = mid - 1 to check left side
```

---

## Algorithm (Pseudocode)

```
FUNCTION findFirstOccurrence(arr, n, target):
    lo = 0
    hi = n - 1
    result = -1                    // Store the answer

    WHILE lo <= hi:
        mid = lo + (hi - lo) / 2
        
        IF arr[mid] == target:
            result = mid           // Record this position
            hi = mid - 1           // Keep searching LEFT
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
| 1 | 0 | 7 | 3 | 3 | == target: result=3, hi=2 | 3 |
| 2 | 0 | 2 | 1 | 2 | < target: lo=2 | 3 |
| 3 | 2 | 2 | 2 | 3 | == target: result=2, hi=1 | **2** |
| 4 | 2 | 1 | — | — | lo > hi: STOP | **2** |

```
   Step 1:
   ┌───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 3 │ 3 │ 3 │ 5 │ 8 │   mid=3, arr[3]=3==3
   └───┴───┴───┴───┴───┴───┴───┴───┘   result=3, hi=2
    lo          mid              hi      (search left for earlier 3)

   Step 2:
   ┌───┬───┬───┐
   │ 1 │ 2 │ 3 │                        mid=1, arr[1]=2<3
   └───┴───┴───┘                        lo=2
    lo  mid  hi

   Step 3:
         ┌───┐
         │ 3 │                           mid=2, arr[2]=3==3
         └───┘                           result=2, hi=1
        lo=hi

   Step 4: lo=2 > hi=1 → STOP
   Answer: result = 2 ✓ (First occurrence of 3)
```

---

## Implementation

### Java

```java
public static int findFirst(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    int result = -1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (arr[mid] == target) {
            result = mid;      // Record and keep searching left
            hi = mid - 1;
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
def find_first(arr, target):
    lo, hi = 0, len(arr) - 1
    result = -1

    while lo <= hi:
        mid = lo + (hi - lo) // 2

        if arr[mid] == target:
            result = mid
            hi = mid - 1      # Keep searching left
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1

    return result
```

---

## Lower Bound (Alternative Approach)

The **lower bound** returns the index of the first element that is **≥ target**. This is equivalent to finding the first occurrence.

```
   arr = [1, 2, 3, 3, 3, 3, 5, 8]

   lower_bound(3) = index 2  (first element ≥ 3)
   
   Then check: if arr[2] == 3 → first occurrence is at index 2
               if arr[2] ≠ 3 → target not found
```

### Lower Bound Implementation

```python
def lower_bound(arr, target):
    lo, hi = 0, len(arr)  # Note: hi = len(arr), not len(arr) - 1
    
    while lo < hi:  # Note: < not <=
        mid = lo + (hi - lo) // 2
        if arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid  # Note: mid, not mid - 1
    
    return lo  # lo is the insertion point
```

---

## Real-World Applications

| Application | Description |
|-------------|-------------|
| **Dictionary lookup** | Find first word starting with prefix |
| **Database queries** | Find first record with given key |
| **Log analysis** | Find first log entry at given timestamp |
| **Range queries** | Start of range for "count elements = x" |
| **Competitive programming** | Foundation for many binary search problems |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Modification** | On match: record + search LEFT |
| **Key Change** | `arr[mid] == target → result = mid; hi = mid - 1` |
| **Time** | O(log n) |
| **Space** | O(1) |
| **Returns** | Index of first occurrence, or -1 |
| **Alternative** | lower_bound approach |

---

## Quick Revision Questions

1. **Why does standard binary search fail to find the first occurrence in a sorted array with duplicates?**
2. **What is the key modification to make binary search find the first occurrence?**
3. **Trace `findFirst([2, 4, 4, 4, 6, 8], 4)` step by step.**
4. **What does `lower_bound` return and how is it related to finding the first occurrence?**
5. **What is the time complexity of finding the first occurrence? Is it different from standard binary search?**
6. **What does `findFirst` return if the target is not in the array?**

---

[← Previous: Unit 2 — Prerequisites & Gotchas](../02-Binary-Search-Basics/06-prerequisites-and-gotchas.md) | [Next: Find Last Occurrence →](02-find-last-occurrence.md)
