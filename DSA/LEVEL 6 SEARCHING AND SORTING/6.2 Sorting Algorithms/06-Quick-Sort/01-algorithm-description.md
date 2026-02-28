# Unit 6: Quick Sort — Algorithm Description

[← Previous Unit: Iterative Merge Sort](../05-Merge-Sort/06-iterative-merge-sort.md) | [Next: Partition Schemes →](02-partition-schemes.md)

---

## Overview

Quick Sort is the most widely used sorting algorithm in practice. Like Merge Sort, it uses **Divide and Conquer**, but with a twist: instead of dividing at the midpoint, it divides based on a **pivot element**. Elements smaller than the pivot go left, elements larger go right. This is called **partitioning**.

---

## The Big Idea

```
  Merge Sort:  Easy split, hard combine (merge is the work)
  Quick Sort:  Hard split (partition is the work), easy combine (nothing!)
  
  ┌──────────────────────────────────────────────────────┐
  │  MERGE SORT                    QUICK SORT            │
  │                                                      │
  │  Split at midpoint (trivial)   Partition around pivot │
  │  Recursively sort halves       Recursively sort halves│
  │  Merge sorted halves (work!)   Done! (no combine)   │
  └──────────────────────────────────────────────────────┘
```

---

## How Quick Sort Works

```
  Step 1: CHOOSE a pivot element
  Step 2: PARTITION — rearrange so all elements ≤ pivot are left,
          all elements > pivot are right
  Step 3: RECURSE on left and right partitions
  
  [7, 2, 1, 6, 8, 5, 3, 4]
                         ↑ pivot = 4
  
  After partition:
  [2, 1, 3] [4] [7, 6, 8, 5]
   ≤ pivot   ↑    > pivot
           pivot in
         final position!
  
  Now recursively sort [2, 1, 3] and [7, 6, 8, 5]
```

---

## Full Example Trace

```
  Array: [7, 2, 1, 6, 8, 5, 3, 4]    pivot = 4 (last element)
  
  PARTITION:
  ┌───────────────────────────────────────────────────────┐
  │  Scan from left to right, separating ≤4 and >4       │
  │                                                       │
  │  [7, 2, 1, 6, 8, 5, 3, 4]   7>4 → skip             │
  │  [7, 2, 1, 6, 8, 5, 3, 4]   2≤4 → swap with 7      │
  │  [2, 7, 1, 6, 8, 5, 3, 4]   1≤4 → swap with 7      │
  │  [2, 1, 7, 6, 8, 5, 3, 4]   6>4 → skip             │
  │  [2, 1, 7, 6, 8, 5, 3, 4]   8>4 → skip             │
  │  [2, 1, 7, 6, 8, 5, 3, 4]   5>4 → skip             │
  │  [2, 1, 7, 6, 8, 5, 3, 4]   3≤4 → swap with 7      │
  │  [2, 1, 3, 6, 8, 5, 7, 4]   Place pivot after ≤ section│
  │  [2, 1, 3, 4, 8, 5, 7, 6]                           │
  │            ↑                                          │
  │         pivot is now in correct final position!       │
  └───────────────────────────────────────────────────────┘
  
  RECURSE:
  
  quickSort([2, 1, 3])           quickSort([8, 5, 7, 6])
     pivot=3                        pivot=6
     [2, 1] [3] []                  [5] [6] [8, 7]
     
     quickSort([2, 1])              quickSort([8, 7])
        pivot=1                        pivot=7
        [] [1] [2]                     [7] [8]
  
  Result: [1, 2, 3, 4, 5, 6, 7, 8]  ✓ SORTED!
```

---

## Why the Pivot Ends Up in Final Position

```
  ┌──────────────────────────────────────────────────────────┐
  │  After partitioning around pivot p:                     │
  │                                                          │
  │  [all elements ≤ p] [p] [all elements > p]             │
  │                      ↑                                   │
  │  Everything left is ≤ p ← correct                      │
  │  Everything right is > p ← correct                     │
  │  p is exactly where it belongs in sorted array!        │
  │                                                          │
  │  The pivot NEVER moves again in any recursive call!     │
  └──────────────────────────────────────────────────────────┘
```

---

## Properties of Quick Sort

```
  ┌────────────────────────────────────────────────────┐
  │  Property           │  Value                       │
  ├─────────────────────┼──────────────────────────────┤
  │  Time (Best)        │  O(n log n)                  │
  │  Time (Average)     │  O(n log n)                  │
  │  Time (Worst)       │  O(n²)                       │
  │  Space (Best/Avg)   │  O(log n) stack              │
  │  Space (Worst)      │  O(n) stack                  │
  │  Stable             │  NO ✗                        │
  │  In-place           │  YES ✓ (O(1) aux space)     │
  │  Adaptive           │  NO                          │
  │  Comparison-based   │  YES                         │
  │  Paradigm           │  Divide and Conquer          │
  │  Cache performance  │  EXCELLENT                   │
  └─────────────────────┴──────────────────────────────┘
```

---

## Why Quick Sort Is Preferred in Practice

```
  Despite O(n²) worst case, Quick Sort is preferred over Merge Sort:
  
  1. IN-PLACE: O(1) auxiliary space (Merge Sort needs O(n))
  2. CACHE-FRIENDLY: Sequential memory access pattern
  3. SMALL CONSTANTS: Inner loop is very tight and fast
  4. AVERAGE O(n log n): Worst case rarely occurs with randomization
  
  Practical speed comparison (same hardware, random data):
  
  n = 1,000,000:
  Quick Sort:  ████████████         1.0x (fastest)
  Merge Sort:  ████████████████     1.3x
  Heap Sort:   ████████████████████ 1.7x
  
  Quick Sort's constant factor is ~2-3x smaller than Merge Sort!
```

---

## Quick Sort vs Merge Sort

```
  ┌──────────────────┬──────────────────┬──────────────────┐
  │  Aspect          │  Quick Sort      │  Merge Sort      │
  ├──────────────────┼──────────────────┼──────────────────┤
  │  Worst case      │  O(n²)           │  O(n log n)      │
  │  Average case    │  O(n log n)      │  O(n log n)      │
  │  Space           │  O(log n)        │  O(n)            │
  │  In-place        │  Yes             │  No              │
  │  Stable          │  No              │  Yes             │
  │  Cache friendly  │  Excellent       │  Good            │
  │  Practical speed │  Fastest         │  Slightly slower │
  │  For linked lists│  Poor            │  Excellent       │
  │  For arrays      │  Excellent       │  Good            │
  │  Used in         │  C++ std::sort   │  Java obj sort   │
  │                  │  (as introsort)  │  Python (TimSort)│
  └──────────────────┴──────────────────┴──────────────────┘
```

---

## Pseudocode

```
  QUICKSORT(A, low, high)
  ───────────────────────
  if low < high
      pivotIndex ← PARTITION(A, low, high)
      QUICKSORT(A, low, pivotIndex - 1)     // Left of pivot
      QUICKSORT(A, pivotIndex + 1, high)     // Right of pivot
  
  
  PARTITION(A, low, high)     // Lomuto partition
  ─────────────────────────
  pivot ← A[high]             // Choose last element as pivot
  i ← low - 1                 // Index of smaller element boundary
  
  for j ← low to high - 1
      if A[j] ≤ pivot
          i ← i + 1
          swap A[i] and A[j]
  
  swap A[i+1] and A[high]     // Place pivot in correct position
  return i + 1                 // Return pivot's final index
```

---

## Summary Table

| Aspect | Description |
|--------|-------------|
| Strategy | Divide and Conquer via partitioning |
| Divide | Partition array around a pivot element |
| Conquer | Recursively sort elements left and right of pivot |
| Combine | Nothing — partitioning puts everything in place |
| Key operation | Partition (rearrange around pivot) |
| Pivot placement | Pivot ends up in its final sorted position |
| In-place | Yes, O(1) auxiliary space |
| Main weakness | O(n²) worst case (mitigated by randomization) |

---

## Quick Revision Questions

1. **How does Quick Sort's divide step differ from Merge Sort's?**
2. **Why does the pivot end up in its correct final position?**
3. **What is the worst-case time complexity and when does it occur?**
4. **Why is Quick Sort faster than Merge Sort in practice despite worse worst case?**
5. **Is Quick Sort stable? Why or why not?**
6. **What are the space requirements of Quick Sort?**

---

[← Previous Unit: Iterative Merge Sort](../05-Merge-Sort/06-iterative-merge-sort.md) | [Next: Partition Schemes →](02-partition-schemes.md)
