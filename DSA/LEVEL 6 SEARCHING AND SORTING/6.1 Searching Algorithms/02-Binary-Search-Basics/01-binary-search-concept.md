# Chapter 1: Binary Search Concept

[← Previous: Unit 1 — Complexity Analysis](../01-Linear-Search/05-complexity-analysis.md) | [Next: Iterative Implementation →](02-iterative-implementation.md)

---

## Overview

**Binary Search** is a divide-and-conquer algorithm that finds a target value in a **sorted array** by repeatedly dividing the search interval in half. It is one of the most important algorithms in computer science.

> **Key Idea:** Compare the target with the **middle element**. If they're not equal, eliminate the half where the target cannot lie, and search the remaining half.

---

## The Phone Book Analogy

Imagine searching for "Kumar" in a phone book with 1000 pages:

```
   Linear Search:  Start at page 1 → page 2 → page 3 → ... → page 500?
                   Up to 1000 page flips!

   Binary Search:  Open to middle (page 500) → "M" section
                   "K" < "M" → search first half
                   Open to page 250 → "F" section  
                   "K" > "F" → search right half (250-500)
                   Open to page 375 → "K" section → narrow down...
                   Only ~10 page flips for 1000 pages!
```

---

## Core Concept: Divide and Conquer

```
   Sorted Array: [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
   Target: 23

   Step 1: Search entire array [0..9]
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
   │ 2 │ 5 │ 8 │12 │16 │23 │38 │56 │72 │91 │
   └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
    lo                  mid                hi
                     arr[4]=16
                     16 < 23 → go RIGHT

   Step 2: Search [5..9]
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
   │ 2 │ 5 │ 8 │12 │16 │23 │38 │56 │72 │91 │
   └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                        lo   mid        hi
                          arr[7]=56
                          56 > 23 → go LEFT

   Step 3: Search [5..6]
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
   │ 2 │ 5 │ 8 │12 │16 │23 │38 │56 │72 │91 │
   └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                       lo=mid hi
                       arr[5]=23
                       23 == 23 → FOUND at index 5! ✓
```

---

## How It Works — The Key Insight

Binary search works because the array is **sorted**. This gives us a powerful guarantee:

```
   If arr[mid] < target:
   ┌───────────────────┬────┬───────────────────┐
   │  ALL < target     │mid │  target might be   │
   │  (can eliminate!) │    │  somewhere here     │
   └───────────────────┴────┴───────────────────┘
   
   If arr[mid] > target:
   ┌───────────────────┬────┬───────────────────┐
   │  target might be  │mid │  ALL > target      │
   │  somewhere here   │    │  (can eliminate!)  │
   └───────────────────┴────┴───────────────────┘
```

Each comparison eliminates **half** the remaining elements!

---

## Algorithm (Pseudocode)

```
FUNCTION binarySearch(arr, n, target):
    lo = 0
    hi = n - 1
    
    WHILE lo <= hi:
        mid = lo + (hi - lo) / 2       // Avoids integer overflow
        
        IF arr[mid] == target:
            RETURN mid                   // Found!
        ELSE IF arr[mid] < target:
            lo = mid + 1                 // Search right half
        ELSE:
            hi = mid - 1                 // Search left half
    
    RETURN -1                             // Not found
```

---

## Complete Step-by-Step Trace

### Example: `arr = [1, 3, 5, 7, 9, 11, 13, 15]`, target = `11`

| Step | lo | hi | mid | arr[mid] | Comparison | Action |
|------|----|----|-----|----------|------------|--------|
| 1 | 0 | 7 | 3 | 7 | 7 < 11 | lo = 4 |
| 2 | 4 | 7 | 5 | 11 | 11 == 11 | **FOUND!** |

```
   Step 1:
   Index: 0   1   2   3   4   5   6   7
        [ 1 | 3 | 5 | 7 | 9 |11 |13 |15 ]
          lo          mid              hi
          arr[3] = 7 < 11 → lo = mid + 1 = 4

   Step 2:
   Index: 0   1   2   3   4   5   6   7
        [ 1 | 3 | 5 | 7 | 9 |11 |13 |15 ]
                          lo  mid      hi
          arr[5] = 11 == 11 → FOUND at index 5! ✓
```

Only **2 comparisons** instead of 6 (linear search)!

---

### Example: Target NOT found — `target = 6`

| Step | lo | hi | mid | arr[mid] | Comparison | Action |
|------|----|----|-----|----------|------------|--------|
| 1 | 0 | 7 | 3 | 7 | 7 > 6 | hi = 2 |
| 2 | 0 | 2 | 1 | 3 | 3 < 6 | lo = 2 |
| 3 | 2 | 2 | 2 | 5 | 5 < 6 | lo = 3 |
| 4 | 3 | 2 | — | — | lo > hi | **NOT FOUND** |

```
   Step 1: [1  3  5  7  9  11  13  15]     mid=3, 7>6 → hi=2
            lo         mid          hi

   Step 2: [1  3  5] 7  9  11  13  15      mid=1, 3<6 → lo=2
            lo mid hi

   Step 3: [1  3  5] 7  9  11  13  15      mid=2, 5<6 → lo=3
                  lo=mid=hi

   Step 4: lo=3 > hi=2 → STOP. Return -1
```

---

## Why `mid = lo + (hi - lo) / 2`?

The naive formula `mid = (lo + hi) / 2` can cause **integer overflow**:

```
   If lo = 2,000,000,000 and hi = 2,100,000,000:
   
   lo + hi = 4,100,000,000    ← OVERFLOW! (exceeds int max ≈ 2.1 billion)
   
   Safe version:
   lo + (hi - lo) / 2
   = 2,000,000,000 + (100,000,000) / 2
   = 2,000,000,000 + 50,000,000
   = 2,050,000,000               ← Safe! No overflow
```

---

## The Power of Halving

```
   Array of 1,000,000 elements:

   After comparison 1:  500,000 elements remain
   After comparison 2:  250,000 elements remain
   After comparison 3:  125,000 elements remain
   ...
   After comparison 10:    977 elements remain
   After comparison 15:     31 elements remain
   After comparison 20:      1 element remains
   
   Total: ~20 comparisons to search 1,000,000 elements!
   Linear search would need up to 1,000,000 comparisons!
```

---

## Visual Decision Tree

For array `[1, 3, 5, 7, 9]`:

```
                        arr[2]=5
                       ╱        ╲
                 target<5      target>5
                    ╱              ╲
              arr[0]=1          arr[3]=7      ← Not checking arr[4]=9
             ╱        ╲       ╱        ╲         yet because mid of
        target<1   target>1  target<7  target>7  [3,4] is 3
           ╱          ╲        ╱          ╲
        NOT         arr[1]=3  NOT       arr[4]=9
       FOUND       ╱    ╲   FOUND      ╱      ╲
               FOUND   NOT          FOUND    NOT
                (=3)   FOUND         (=9)   FOUND

   Maximum depth = 3 = ⌊log₂(5)⌋ + 1 comparisons
```

---

## Prerequisites for Binary Search

```
   ╔══════════════════════════════════════════════╗
   ║  BINARY SEARCH REQUIRES:                    ║
   ║                                              ║
   ║  1. SORTED DATA (ascending or descending)    ║
   ║  2. RANDOM ACCESS (arrays, not linked lists) ║
   ║  3. COMPARABLE ELEMENTS (can use <, >, ==)   ║
   ╚══════════════════════════════════════════════╝
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Type** | Divide and Conquer |
| **Prerequisite** | Array must be sorted |
| **Best Case** | O(1) — target at midpoint |
| **Average Case** | O(log n) |
| **Worst Case** | O(log n) |
| **Space (Iterative)** | O(1) |
| **Space (Recursive)** | O(log n) |
| **Key Operation** | Compare with middle, eliminate half |

---

## Quick Revision Questions

1. **What is the fundamental prerequisite for binary search to work?**
2. **In each step, what fraction of the search space is eliminated?**
3. **Why do we use `lo + (hi - lo) / 2` instead of `(lo + hi) / 2`?**
4. **How many comparisons does binary search need for an array of 1,000,000 elements?**
5. **Can binary search be applied to a linked list? Why or why not?**
6. **Draw the decision tree for binary search on the array `[2, 4, 6, 8, 10]`.**

---

[← Previous: Unit 1 — Complexity Analysis](../01-Linear-Search/05-complexity-analysis.md) | [Next: Iterative Implementation →](02-iterative-implementation.md)
