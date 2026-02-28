# Chapter 2: Binary Search - Recursive

## Overview
Binary search is the classic **divide and conquer** algorithm that works on sorted arrays. The recursive implementation is elegant and naturally expresses the "halving" logic. It runs in O(log n) time.

---

## Prerequisites
- Array must be **sorted**
- Elements must be **comparable**

---

## Recursive Solution

### Pseudocode
```
function binarySearch(arr, target, low, high):
    if low > high: return -1             // Base: not found
    
    mid = low + (high - low) / 2        // Avoid overflow
    
    if arr[mid] == target: return mid     // Base: found!
    
    if target < arr[mid]:
        return binarySearch(arr, target, low, mid - 1)    // Search left half
    else:
        return binarySearch(arr, target, mid + 1, high)   // Search right half
```

### Trace: binarySearch([2, 5, 8, 12, 16, 23, 38, 56, 72, 91], 23, 0, 9)

```
Search for 23 in sorted array:

Step 1: low=0, high=9, mid=4
  [2, 5, 8, 12, |16|, 23, 38, 56, 72, 91]
                  ↑mid
  23 > 16 → search RIGHT half [23, 38, 56, 72, 91]

Step 2: low=5, high=9, mid=7
  [23, 38, |56|, 72, 91]
             ↑mid
  23 < 56 → search LEFT half [23, 38]

Step 3: low=5, high=6, mid=5
  [|23|, 38]
    ↑mid
  23 == 23 → FOUND at index 5! ✓

Visual elimination:
┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ 2  │ 5  │ 8  │ 12 │ 16 │ 23 │ 38 │ 56 │ 72 │ 91 │  Step 1
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
                       ↑                              mid=4, go right
                           ┌────┬────┬────┬────┬────┐
                           │ 23 │ 38 │ 56 │ 72 │ 91 │  Step 2
                           └────┴────┴────┴────┴────┘
                                       ↑              mid=7, go left
                           ┌────┬────┐
                           │ 23 │ 38 │                Step 3
                           └────┴────┘
                             ↑                         mid=5, FOUND!
```

---

## Call Stack

```
┌──────────────────────────────────┐
│ binarySearch(arr, 23, 5, 6)     │ ← returns 5 (FOUND)
│ mid=5, arr[5]=23==23            │
├──────────────────────────────────┤
│ binarySearch(arr, 23, 5, 9)     │
│ mid=7, arr[7]=56 > 23, go left  │
├──────────────────────────────────┤
│ binarySearch(arr, 23, 0, 9)     │ ← initial call
│ mid=4, arr[4]=16 < 23, go right │
└──────────────────────────────────┘

Max depth = 3 (for 10 elements)
          = O(log₂ 10) ≈ 3.32
```

---

## Not Found Case

```
binarySearch([2, 5, 8, 12, 16], 10, 0, 4)

Step 1: low=0, high=4, mid=2, arr[2]=8
  10 > 8 → go right (low=3, high=4)

Step 2: low=3, high=4, mid=3, arr[3]=12  
  10 < 12 → go left (low=3, high=2)

Step 3: low=3, high=2 → low > high → return -1  (NOT FOUND)

[2,  5,  8,  12,  16]
              ↑mid=2, 10>8 go right
                  [12,  16]
                   ↑mid=3, 10<12 go left
                  [∅]  low > high → NOT FOUND
```

---

## Why `mid = low + (high - low) / 2`?

```
Using (low + high) / 2 can cause INTEGER OVERFLOW:
  If low = 2,000,000,000 and high = 2,000,000,000
  low + high = 4,000,000,000 → OVERFLOW! (> 2^31 - 1)

Using low + (high - low) / 2:
  high - low = 0  (never overflows)
  low + 0 = 2,000,000,000  ✓

[!] This is a classic bug in production code!
    (Found in Java's Arrays.binarySearch in 2006)
```

---

## Complexity Analysis

```
┌──────────────────────────────────────────────────┐
│         COMPLEXITY ANALYSIS                       │
│                                                  │
│  Time: O(log n)                                  │
│    Each call halves the search space              │
│    n → n/2 → n/4 → ... → 1                      │
│    Steps = log₂(n)                               │
│                                                  │
│  Space: O(log n) — recursive stack depth         │
│  (Iterative version: O(1) space)                 │
│                                                  │
│  Recurrence: T(n) = T(n/2) + O(1)               │
│              T(n) = O(log n)                     │
│                                                  │
│  For n = 1,000,000: only ~20 comparisons!        │
│  For n = 1,000,000,000: only ~30 comparisons!    │
└──────────────────────────────────────────────────┘

  Linear Search vs Binary Search:
  ┌───────────┬──────────────┬──────────────┐
  │     n     │ Linear O(n)  │ Binary O(lgn)│
  ├───────────┼──────────────┼──────────────┤
  │    100    │    100       │     7        │
  │   1,000   │   1,000      │    10        │
  │  100,000  │  100,000     │    17        │
  │ 1,000,000 │ 1,000,000    │    20        │
  └───────────┴──────────────┴──────────────┘
```

---

## Variants

| Variant | Modification |
|---------|-------------|
| Find first occurrence | When found, continue searching left |
| Find last occurrence | When found, continue searching right |
| Lower bound | First element ≥ target |
| Upper bound | First element > target |
| Search in rotated array | Check which half is sorted |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Prerequisite | Array must be sorted |
| Base cases | low > high (not found) or arr[mid] == target |
| Recursive case | Search left or right half |
| Time | O(log n) |
| Space | O(log n) recursive, O(1) iterative |
| Recurrence | T(n) = T(n/2) + O(1) |
| Key insight | Halve search space each step |

---

## Quick Revision Questions

1. **Why must the array be sorted** for binary search?
2. **Why use `low + (high - low) / 2`** instead of `(low + high) / 2`?
3. **What is the maximum number of comparisons** for binary search on 1 million elements?
4. **Trace binary search** for target=15 in [3, 7, 11, 15, 19, 23].
5. **What is the space complexity** difference between recursive and iterative binary search?
6. **How would you modify** binary search to find the first occurrence of a duplicate?

---

[← Previous: Linear Search Recursive](01-linear-search-recursive.md) | [Next: Check Sorted Array →](03-check-sorted-array.md)

[← Back to README](../README.md)
