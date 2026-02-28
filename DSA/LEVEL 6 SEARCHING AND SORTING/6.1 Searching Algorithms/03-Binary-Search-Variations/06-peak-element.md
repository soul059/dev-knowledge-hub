# Chapter 6: Peak Element

[← Previous: Floor and Ceiling](05-floor-and-ceiling.md) | [Next: Unit 4 — Standard Template →](../04-Binary-Search-Template/01-standard-template.md)

---

## Overview

A **peak element** is one that is **greater than or equal to its neighbors**. Binary search can find a peak in O(log n) even in an **unsorted** array — a surprising and powerful application.

---

## Problem Statement

```
   Find ANY peak element in an array.
   A peak element: arr[i] ≥ arr[i-1] AND arr[i] ≥ arr[i+1]
   Edge elements: only need to be ≥ one neighbor.

   Input:  [1, 3, 20, 4, 1, 0]
   Output: index 2 (arr[2]=20 is a peak: 20 > 3 and 20 > 4)

   ┌───┬───┬────┬───┬───┬───┐
   │ 1 │ 3 │ 20 │ 4 │ 1 │ 0 │
   └───┴───┴────┴───┴───┴───┘
              ↑
            PEAK (20)
```

---

## Key Insight: Why Binary Search Works

Even though the array isn't sorted, binary search works because:

> **If `arr[mid] < arr[mid + 1]`, a peak MUST exist on the right side.**

This is because the right side has a higher value. If values keep increasing, the last element is a peak. If they decrease at some point, that creates a peak.

```
   If arr[mid] < arr[mid+1]:
   
        ?   ?   ?  mid mid+1  ?   ?   ?
   ─────────────────╱──╱──────────────────
                    ╱  ╱  ← Going up means peak is RIGHT
                   ╱  ╱     (values can't increase forever)
                  ╱  ╱
   
   Similarly, if arr[mid] < arr[mid-1]:
   Peak is on the LEFT side.
```

---

## Algorithm (Pseudocode)

```
FUNCTION findPeak(arr, n):
    lo = 0
    hi = n - 1

    WHILE lo < hi:                      // Note: < not <=
        mid = lo + (hi - lo) / 2

        IF arr[mid] < arr[mid + 1]:
            lo = mid + 1                // Peak is on right
        ELSE:
            hi = mid                    // Peak is at mid or left

    RETURN lo                            // lo == hi == peak index
```

---

## Step-by-Step Trace

### Array: `[1, 3, 20, 4, 1, 0]`

| Step | lo | hi | mid | arr[mid] | arr[mid+1] | Action |
|------|----|----|-----|----------|----------|--------|
| 1 | 0 | 5 | 2 | 20 | 4 | 20 > 4 → hi=2 |
| 2 | 0 | 2 | 1 | 3 | 20 | 3 < 20 → lo=2 |
| 3 | 2 | 2 | — | — | — | lo == hi → STOP |

```
   Step 1:
   ┌───┬───┬────┬───┬───┬───┐
   │ 1 │ 3 │ 20 │ 4 │ 1 │ 0 │     mid=2, arr[2]=20 > arr[3]=4
   └───┴───┴────┴───┴───┴───┘     → peak is at mid or left, hi=2
    lo       mid              hi

   Step 2:
   ┌───┬───┬────┐
   │ 1 │ 3 │ 20 │                  mid=1, arr[1]=3 < arr[2]=20
   └───┴───┴────┘                  → peak is on right, lo=2
    lo  mid  hi

   Step 3: lo=2 == hi=2 → PEAK at index 2, arr[2]=20 ✓
```

### Array with Multiple Peaks: `[5, 10, 2, 8, 3]`

```
          10
         ╱  ╲       8
        ╱    ╲     ╱ ╲
       5      2   ╱   3
                 ╱
   (Peak at 1 or 3 — binary search finds ONE)
```

| Step | lo | hi | mid | arr[mid] | arr[mid+1] | Action |
|------|----|----|-----|----------|----------|--------|
| 1 | 0 | 4 | 2 | 2 | 8 | 2 < 8 → lo=3 |
| 2 | 3 | 4 | 3 | 8 | 3 | 8 > 3 → hi=3 |
| 3 | 3 | 3 | — | — | — | STOP → index 3 |

Peak = arr[3] = 8 ✓ (8 > 2 and 8 > 3)

---

## Why `lo < hi` (not `lo <= hi`)?

```
   We use lo < hi because:
   1. The loop converges lo and hi to the same index
   2. When lo == hi, we've found the peak
   3. Using lo <= hi would need different logic to avoid accessing mid+1 out of bounds
   
   Loop terminates when: lo == hi → both point to the peak
```

---

## Implementation

### Java

```java
public static int findPeakElement(int[] arr) {
    int lo = 0, hi = arr.length - 1;

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;

        if (arr[mid] < arr[mid + 1]) {
            lo = mid + 1;     // Peak on right side
        } else {
            hi = mid;         // Peak at mid or left
        }
    }
    return lo;  // lo == hi == peak index
}
```

### Python

```python
def find_peak_element(arr):
    lo, hi = 0, len(arr) - 1

    while lo < hi:
        mid = lo + (hi - lo) // 2

        if arr[mid] < arr[mid + 1]:
            lo = mid + 1
        else:
            hi = mid

    return lo
```

---

## Visual Intuition: Mountain Climbing

```
   Think of the array as a terrain:

              Peak
              ╱╲
            ╱    ╲
          ╱        ╲         ╱╲
        ╱            ╲     ╱    ╲
      ╱                ╲ ╱        ╲
   ──╱──────────────────╲──────────╲──
   
   At any point (mid):
   - If slope goes UP (arr[mid] < arr[mid+1]) → climb RIGHT
   - If slope goes DOWN (arr[mid] > arr[mid+1]) → peak is HERE or LEFT
   
   You're guaranteed to reach A peak (not necessarily the highest).
```

---

## Edge Cases

| Case | Array | Peak |
|------|-------|------|
| Ascending | [1, 2, 3, 4, 5] | index 4 (last element) |
| Descending | [5, 4, 3, 2, 1] | index 0 (first element) |
| Single element | [42] | index 0 |
| Two elements | [1, 2] | index 1 |
| All equal | [5, 5, 5, 5] | any index |

---

## Important Notes

1. **Finds ANY peak, not necessarily the global maximum**
2. Binary search works because a peak is **guaranteed to exist** (pigeonhole principle on slopes)
3. This is one of the rare cases where binary search works on **unsorted** arrays
4. The condition uses **strict comparison** to handle edge cases

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Definition** | Element ≥ both neighbors |
| **Loop Condition** | `while (lo < hi)` |
| **Left Direction** | `arr[mid] ≥ arr[mid+1] → hi = mid` |
| **Right Direction** | `arr[mid] < arr[mid+1] → lo = mid + 1` |
| **Returns** | `lo` (or `hi`, they're equal) |
| **Time** | O(log n) |
| **Space** | O(1) |
| **Array needs sorting?** | **No!** |

---

## Quick Revision Questions

1. **Why can binary search find a peak element even in an unsorted array?**
2. **If `arr[mid] < arr[mid+1]`, why must a peak exist on the right side?**
3. **Why is the loop condition `lo < hi` instead of `lo <= hi`?**
4. **Does this algorithm find the global maximum? Why or why not?**
5. **Trace the algorithm on `[1, 2, 1, 3, 5, 6, 4]`.**
6. **What happens if all elements in the array are equal?**

---

[← Previous: Floor and Ceiling](05-floor-and-ceiling.md) | [Next: Unit 4 — Standard Template →](../04-Binary-Search-Template/01-standard-template.md)
