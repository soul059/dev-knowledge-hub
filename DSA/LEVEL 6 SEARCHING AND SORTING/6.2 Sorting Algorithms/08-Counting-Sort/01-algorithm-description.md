# Unit 8: Counting Sort — Algorithm Description

[← Previous: Unit 7 - Priority Queue](../07-Heap-Sort/06-priority-queue.md) | [Next: Implementation →](02-implementation.md)

---

## Overview

Counting Sort breaks through the **O(n log n)** barrier! It sorts in **O(n + k)** time where k is the range of input values. The catch? It's a **non-comparison** sort — it never compares two elements directly. Instead, it **counts occurrences** of each value.

---

## The Key Idea

```
  Instead of comparing elements to determine order,
  simply COUNT how many of each value exist.
  
  Array: [4, 2, 2, 8, 3, 3, 1]
  
  Step 1: Count occurrences
  ┌──────┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
  │ Value│ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │
  ├──────┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
  │ Count│ 0 │ 1 │ 2 │ 2 │ 1 │ 0 │ 0 │ 0 │ 1 │
  └──────┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
  
  Step 2: Write values in order based on counts
  
  Value 0: count=0 → skip
  Value 1: count=1 → write [1]
  Value 2: count=2 → write [1, 2, 2]
  Value 3: count=2 → write [1, 2, 2, 3, 3]
  Value 4: count=1 → write [1, 2, 2, 3, 3, 4]
  Value 5-7: count=0 → skip
  Value 8: count=1 → write [1, 2, 2, 3, 3, 4, 8]
  
  Sorted: [1, 2, 2, 3, 3, 4, 8]  ✓
  
  NO COMPARISONS MADE!
```

---

## Why Can It Beat O(n log n)?

```
  ┌──────────────────────────────────────────────────────────┐
  │  The O(n log n) lower bound applies ONLY to             │
  │  COMPARISON-BASED sorts!                                 │
  │                                                          │
  │  Comparison sorts only learn about elements through      │
  │  pairwise comparisons (a < b? a > b?).                  │
  │                                                          │
  │  Counting Sort uses element VALUES directly as           │
  │  array indices — this gives MORE information per         │
  │  operation than a single comparison.                     │
  │                                                          │
  │  By exploiting the structure of the input (integers      │
  │  in a known range), it bypasses the lower bound.         │
  └──────────────────────────────────────────────────────────┘
```

---

## Algorithm Steps (Detailed)

```
  ╔═══════════════════════════════════════════════════════════╗
  ║  COUNTING SORT ALGORITHM                                 ║
  ║                                                          ║
  ║  Input:  Array A[0..n-1], values in range [0..k]        ║
  ║  Output: Sorted array                                    ║
  ║                                                          ║
  ║  1. Create count array C[0..k], initialized to 0        ║
  ║  2. Count: For each A[i], increment C[A[i]]             ║
  ║  3. Prefix sum: C[i] = C[i] + C[i-1]                   ║
  ║     (C[i] now = # elements ≤ i = final position)        ║
  ║  4. Build output: Place each element at its position    ║
  ║  5. Copy output back to A (if needed)                   ║
  ╚═══════════════════════════════════════════════════════════╝
```

---

## Step-by-Step Trace (Stable Version)

```
  Array A:  [4, 2, 2, 8, 3, 3, 1]     n=7, k=8
  
  ═══ STEP 1: Initialize count array ═══
  
  C: [0, 0, 0, 0, 0, 0, 0, 0, 0]    (size k+1 = 9)
      0  1  2  3  4  5  6  7  8
  
  ═══ STEP 2: Count occurrences ═══
  
  Process A[0]=4: C[4]++     C: [0, 0, 0, 0, 1, 0, 0, 0, 0]
  Process A[1]=2: C[2]++     C: [0, 0, 1, 0, 1, 0, 0, 0, 0]
  Process A[2]=2: C[2]++     C: [0, 0, 2, 0, 1, 0, 0, 0, 0]
  Process A[3]=8: C[8]++     C: [0, 0, 2, 0, 1, 0, 0, 0, 1]
  Process A[4]=3: C[3]++     C: [0, 0, 2, 1, 1, 0, 0, 0, 1]
  Process A[5]=3: C[3]++     C: [0, 0, 2, 2, 1, 0, 0, 0, 1]
  Process A[6]=1: C[1]++     C: [0, 1, 2, 2, 1, 0, 0, 0, 1]
  
  ═══ STEP 3: Prefix sum (cumulative count) ═══
  
  C[i] += C[i-1]:
  
  C[0] = 0                  C: [0, -, -, -, -, -, -, -, -]
  C[1] = 0 + 1 = 1          C: [0, 1, -, -, -, -, -, -, -]
  C[2] = 1 + 2 = 3          C: [0, 1, 3, -, -, -, -, -, -]
  C[3] = 3 + 2 = 5          C: [0, 1, 3, 5, -, -, -, -, -]
  C[4] = 5 + 1 = 6          C: [0, 1, 3, 5, 6, -, -, -, -]
  C[5] = 6 + 0 = 6          C: [0, 1, 3, 5, 6, 6, -, -, -]
  C[6] = 6 + 0 = 6          C: [0, 1, 3, 5, 6, 6, 6, -, -]
  C[7] = 6 + 0 = 6          C: [0, 1, 3, 5, 6, 6, 6, 6, -]
  C[8] = 6 + 1 = 7          C: [0, 1, 3, 5, 6, 6, 6, 6, 7]
  
  Meaning: C[i] = number of elements ≤ i
  C[3] = 5 means 5 elements are ≤ 3
  So element 3 belongs at positions 4 and 5 (0-indexed)
  
  ═══ STEP 4: Build output (traverse A RIGHT-TO-LEFT for stability) ═══
  
  Output: [_, _, _, _, _, _, _]
  
  i=6: A[6]=1  → C[1]=1 → pos=0  → Output[0]=1   → C[1]=0
  i=5: A[5]=3  → C[3]=5 → pos=4  → Output[4]=3   → C[3]=4
  i=4: A[4]=3  → C[3]=4 → pos=3  → Output[3]=3   → C[3]=3
  i=3: A[3]=8  → C[8]=7 → pos=6  → Output[6]=8   → C[8]=6
  i=2: A[2]=2  → C[2]=3 → pos=2  → Output[2]=2   → C[2]=2
  i=1: A[1]=2  → C[2]=2 → pos=1  → Output[1]=2   → C[2]=1
  i=0: A[0]=4  → C[4]=6 → pos=5  → Output[5]=4   → C[4]=5
  
  Output: [1, 2, 2, 3, 3, 4, 8]  ✓  SORTED & STABLE!
```

---

## Why Traverse Right-to-Left? (Stability)

```
  For stability, equal elements must maintain their
  original relative order.
  
  Array: [2a, 2b]  (a appears before b)
  
  After prefix sum: C[2] = 2
  
  RIGHT-TO-LEFT (stable ✓):
    Process 2b first → position 1 → C[2]=1
    Process 2a next  → position 0 → C[2]=0
    Result: [2a, 2b]  ← original order preserved ✓
  
  LEFT-TO-RIGHT (unstable ✗):
    Process 2a first → position 1 → C[2]=1
    Process 2b next  → position 0 → C[2]=0  
    Result: [2b, 2a]  ← order reversed ✗
  
  NOTE: If using C[A[i]]-1 as position (0-indexed), 
  right-to-left preserves stability.
```

---

## Pseudocode

```
  COUNTING-SORT(A, n, k):
      // A = input array, n = size, k = max value
      
      // Step 1: Initialize
      C = new array of size (k+1), filled with 0
      B = new array of size n       // output
      
      // Step 2: Count
      for i = 0 to n-1:
          C[A[i]] = C[A[i]] + 1
      
      // Step 3: Prefix sum
      for i = 1 to k:
          C[i] = C[i] + C[i-1]
      
      // Step 4: Build output (right-to-left for stability)
      for i = n-1 downto 0:
          B[C[A[i]] - 1] = A[i]
          C[A[i]] = C[A[i]] - 1
      
      // Step 5: Copy back
      copy B to A
```

---

## Properties at a Glance

| Property | Value |
|----------|-------|
| Type | Non-comparison sort |
| Time Complexity | O(n + k) |
| Space Complexity | O(n + k) |
| Stable | Yes (with right-to-left placement) |
| In-place | No (needs output array + count array) |
| Adaptive | No |
| Works on | Non-negative integers (or mappable data) |

---

## Quick Revision Questions

1. **How does Counting Sort avoid comparisons?**
2. **What does the prefix sum array represent?**
3. **Why is right-to-left traversal important for stability?**
4. **When is k (range) too large for Counting Sort to be practical?**
5. **Can Counting Sort handle negative numbers? How?**

---

[← Previous: Unit 7 - Priority Queue](../07-Heap-Sort/06-priority-queue.md) | [Next: Implementation →](02-implementation.md)
