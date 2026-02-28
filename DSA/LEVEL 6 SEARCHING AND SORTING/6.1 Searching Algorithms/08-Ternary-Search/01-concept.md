# Chapter 1: Ternary Search — Concept

[← Previous Unit: Matrix Practice](../07-Search-In-Matrix/05-practice-problems.md) | [Next: Unimodal Functions →](02-unimodal-functions.md)

---

## Overview

**Ternary search** divides the search space into **three parts** instead of two. It's primarily useful for finding the **maximum or minimum of unimodal functions** — a task binary search cannot handle directly.

---

## How It Works

```
   Binary Search:  divide into 2 parts → eliminate 1/2
   Ternary Search: divide into 3 parts → eliminate 1/3
   
   ┌─────────┬─────────┬─────────┐
   │  Part 1 │  Part 2 │  Part 3 │
   └─────────┴─────────┴─────────┘
   lo      mid1      mid2        hi
   
   mid1 = lo + (hi - lo) / 3
   mid2 = hi - (hi - lo) / 3
```

---

## Algorithm for Sorted Array

```
   TERNARY-SEARCH(arr, target):
       lo = 0, hi = n - 1
       while lo <= hi:
           mid1 = lo + (hi - lo) / 3
           mid2 = hi - (hi - lo) / 3
           
           if arr[mid1] == target: return mid1
           if arr[mid2] == target: return mid2
           
           if target < arr[mid1]:
               hi = mid1 - 1        // target in Part 1
           elif target > arr[mid2]:
               lo = mid2 + 1        // target in Part 3
           else:
               lo = mid1 + 1        // target in Part 2
               hi = mid2 - 1
       return -1
```

---

## Visual

```
   arr = [1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23]
   target = 17
   
   Step 1: lo=0 hi=11
           mid1 = 0 + 11/3 = 3 → arr[3] = 7
           mid2 = 11 - 11/3 = 8 → arr[8] = 17 → FOUND ✓
   
   Only 1 iteration! (got lucky)
```

---

## Ternary vs Binary Search on Sorted Arrays

```
   Binary Search:
   - Comparisons per step: 1-2
   - Reduction factor: 1/2
   - Total comparisons: ~2 log₂ n = 2 log₂ n
   
   Ternary Search:
   - Comparisons per step: 2-4
   - Reduction factor: 1/3
   - Total comparisons: ~4 log₃ n = 4 × log n / log 3
   
   Ratio: (4 / log 3) / (2 / log 2) ≈ 4/1.585 / 2/1 ≈ 2.52/2 ≈ 1.26
   
   Ternary search is ~26% SLOWER than binary on sorted arrays!
```

### Why Binary Search Wins on Sorted Arrays

```
   ┌──────────────────────────────────────────────────────────┐
   │                                                          │
   │  Binary: 1 comparison → eliminate 50% of space          │
   │  Ternary: 2 comparisons → eliminate 33% of space        │
   │                                                          │
   │  Cost per eliminated element:                            │
   │  Binary: 1 comparison / 50% = 2 comparisons per halving │
   │  Ternary: 2 comparisons / 67% = 3 comparisons per 2/3   │
   │                                                          │
   │  Binary search has better "comparison efficiency"        │
   │                                                          │
   └──────────────────────────────────────────────────────────┘
```

---

## When Ternary Search IS Useful

```
   Ternary search shines for UNIMODAL FUNCTIONS:
   
   Functions that increase then decrease (find maximum)
   or decrease then increase (find minimum)
   
           ●
         ●   ●
       ●       ●          ← unimodal (one peak)
     ●           ●
   ●               ●
   
   Binary search can't directly find the peak of a continuous function
   because there's no "target" to compare against.
   
   Ternary search can: compare f(mid1) vs f(mid2)
```

---

## Implementation: Sorted Array (Java)

```java
public int ternarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    
    while (lo <= hi) {
        int mid1 = lo + (hi - lo) / 3;
        int mid2 = hi - (hi - lo) / 3;
        
        if (arr[mid1] == target) return mid1;
        if (arr[mid2] == target) return mid2;
        
        if (target < arr[mid1]) {
            hi = mid1 - 1;
        } else if (target > arr[mid2]) {
            lo = mid2 + 1;
        } else {
            lo = mid1 + 1;
            hi = mid2 - 1;
        }
    }
    return -1;
}
```

---

## Complexity Comparison

| Algorithm | Comparisons | Time | Best For |
|-----------|-------------|------|----------|
| Binary Search | ~log₂ n | O(log n) | Sorted arrays |
| Ternary Search | ~2 log₃ n | O(log n) | Unimodal functions |
| Linear Search | ~n | O(n) | Unsorted data |

Both are O(log n), but binary has a smaller constant factor for sorted arrays.

---

## Quick Revision Questions

1. **How does ternary search divide the search space?**
2. **Why is ternary search slower than binary for sorted arrays?**
3. **What are the formulas for mid1 and mid2?**
4. **When is ternary search preferred over binary search?**
5. **What is a unimodal function?**

---

[← Previous Unit: Matrix Practice](../07-Search-In-Matrix/05-practice-problems.md) | [Next: Unimodal Functions →](02-unimodal-functions.md)
