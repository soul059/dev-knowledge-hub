# Unit 5: Merge Sort — Algorithm Description

[← Previous Unit: Binary Insertion Sort](../04-Insertion-Sort/06-binary-insertion-sort.md) | [Next: Implementation →](02-implementation.md)

---

## Overview

Merge Sort is the first **O(n log n)** sorting algorithm we study. It uses the **Divide and Conquer** paradigm — splitting the problem in half repeatedly, solving each half, then combining the results. It guarantees O(n log n) in ALL cases — best, average, and worst.

---

## The Divide and Conquer Paradigm

```
  ┌──────────────────────────────────────────────────────┐
  │           DIVIDE AND CONQUER                         │
  │                                                      │
  │  1. DIVIDE:   Split the problem into subproblems    │
  │  2. CONQUER:  Solve subproblems recursively         │
  │  3. COMBINE:  Merge solutions into final answer     │
  │                                                      │
  │  For Merge Sort:                                     │
  │  1. DIVIDE:   Split array into two halves           │
  │  2. CONQUER:  Recursively sort each half            │
  │  3. COMBINE:  Merge two sorted halves               │
  └──────────────────────────────────────────────────────┘
```

---

## How Merge Sort Works

### Step 1: Divide Until Trivially Sorted

```
  Original: [38, 27, 43, 3, 9, 82, 10]
  
  Level 0:  [38, 27, 43, 3, 9, 82, 10]
                      │
            ┌─────────┴──────────┐
            ▼                    ▼
  Level 1:  [38, 27, 43, 3]   [9, 82, 10]
             │                   │
        ┌────┴────┐         ┌───┴────┐
        ▼         ▼         ▼        ▼
  Level 2: [38,27] [43,3]  [9,82]  [10]
             │       │       │       │
           ┌─┴─┐  ┌─┴─┐  ┌─┴─┐     │
           ▼   ▼  ▼   ▼  ▼   ▼     ▼
  Level 3: [38][27][43][3][9][82]  [10]
           
  Base case: single elements are already sorted!
```

### Step 2: Merge Back Up

```
  Level 3: [38][27] [43][3]  [9][82]  [10]
              │        │        │        │
              ▼        ▼        ▼        │
  Level 2: [27,38]  [3,43]  [9,82]    [10]
              │        │        │        │
              └───┬────┘        └───┬────┘
                  ▼                 ▼
  Level 1:  [3, 27, 38, 43]   [9, 10, 82]
                  │                 │
                  └────────┬────────┘
                           ▼
  Level 0:  [3, 9, 10, 27, 38, 43, 82]  ✓ SORTED!
```

---

## The Merge Operation (Core of the Algorithm)

```
  Merging [3, 27, 38, 43] and [9, 10, 82]:
  
  Left:  [3, 27, 38, 43]     Right: [9, 10, 82]
          ↑ i                         ↑ j
  Result: []
  
  Step 1: Compare 3 vs 9 → take 3
  Left:  [3, 27, 38, 43]     Right: [9, 10, 82]
             ↑ i                      ↑ j
  Result: [3]
  
  Step 2: Compare 27 vs 9 → take 9
  Left:  [3, 27, 38, 43]     Right: [9, 10, 82]
             ↑ i                         ↑ j
  Result: [3, 9]
  
  Step 3: Compare 27 vs 10 → take 10
  Left:  [3, 27, 38, 43]     Right: [9, 10, 82]
             ↑ i                             ↑ j
  Result: [3, 9, 10]
  
  Step 4: Compare 27 vs 82 → take 27
  Left:  [3, 27, 38, 43]     Right: [9, 10, 82]
                 ↑ i                         ↑ j
  Result: [3, 9, 10, 27]
  
  Step 5: Compare 38 vs 82 → take 38
  Left:  [3, 27, 38, 43]     Right: [9, 10, 82]
                     ↑ i                     ↑ j
  Result: [3, 9, 10, 27, 38]
  
  Step 6: Compare 43 vs 82 → take 43
  Left:  [3, 27, 38, 43]     Right: [9, 10, 82]
                         ↑ i                 ↑ j
  Result: [3, 9, 10, 27, 38, 43]
  
  Step 7: Left exhausted, copy remaining from Right → 82
  Result: [3, 9, 10, 27, 38, 43, 82]  ✓
```

---

## Properties of Merge Sort

```
  ┌────────────────────────────────────────────────────┐
  │  Property           │  Value                       │
  ├─────────────────────┼──────────────────────────────┤
  │  Time (Best)        │  O(n log n)                  │
  │  Time (Average)     │  O(n log n)                  │
  │  Time (Worst)       │  O(n log n)                  │
  │  Space              │  O(n) auxiliary               │
  │  Stable             │  YES ✓                       │
  │  In-place           │  NO (needs O(n) extra space) │
  │  Adaptive           │  NO (same time always)       │
  │  Comparison-based   │  YES                         │
  │  Paradigm           │  Divide and Conquer          │
  │  Online             │  NO                          │
  └─────────────────────┴──────────────────────────────┘
```

---

## Why Is Merge Sort Stable?

```
  When merging two sorted halves, if elements are EQUAL:
  
  Left:  [..., 5a, ...]     Right: [..., 5b, ...]
  
  We take from LEFT first → 5a comes before 5b in result
  
  This preserves the original relative order of equal elements!
  
  Code: if (left[i] <= right[j])  ← the ≤ ensures stability
            take from left
        else
            take from right
```

---

## Recursion Tree

```
  T(n) = 2·T(n/2) + O(n)
  
  Level 0:           T(n)                    work = n
                    /     \
  Level 1:     T(n/2)   T(n/2)              work = n
                /  \      /  \
  Level 2:  T(n/4) T(n/4) T(n/4) T(n/4)    work = n
               ...        ...
  Level k:  ...  n/(2^k) sized problems ... work = n
               ...        ...
  Level log n: T(1) T(1) ... T(1) (n nodes)  work = n
  
  Total levels = log₂(n)
  Work per level = O(n)
  Total work = O(n) × log₂(n) = O(n log n)
```

---

## Merge Sort vs. Previous Algorithms

```
  Input: [5, 4, 3, 2, 1]  (reverse sorted — WORST case for simple sorts)
  
  Bubble Sort:   10 comparisons, 10 swaps     → O(n²)
  Selection Sort: 10 comparisons, 2 swaps     → O(n²)
  Insertion Sort: 10 comparisons, 10 shifts   → O(n²)
  Merge Sort:     ~8 comparisons, 0 swaps     → O(n log n)
  
  For n = 1,000,000:
  O(n²) ≈ 10¹²  operations  → ~17 minutes
  O(n log n) ≈ 2×10⁷ operations → ~0.02 seconds
```

---

## Pseudocode

```
  MERGE-SORT(A, left, right)
  ──────────────────────────
  if left < right
      mid ← ⌊(left + right) / 2⌋
      
      MERGE-SORT(A, left, mid)        // sort left half
      MERGE-SORT(A, mid + 1, right)   // sort right half
      
      MERGE(A, left, mid, right)      // merge sorted halves
  
  
  MERGE(A, left, mid, right)
  ─────────────────────────
  Create temp arrays L[0..n1-1] and R[0..n2-1]
  Copy A[left..mid] into L
  Copy A[mid+1..right] into R
  
  i ← 0, j ← 0, k ← left
  
  while i < n1 AND j < n2
      if L[i] ≤ R[j]           // ≤ for stability
          A[k] ← L[i]
          i ← i + 1
      else
          A[k] ← R[j]
          j ← j + 1
      k ← k + 1
  
  Copy remaining elements of L (if any) into A
  Copy remaining elements of R (if any) into A
```

---

## Summary Table

| Aspect | Description |
|--------|-------------|
| Strategy | Divide and Conquer |
| Divide | Split array into two halves at midpoint |
| Conquer | Recursively sort each half |
| Combine | Merge two sorted halves |
| Time | O(n log n) always |
| Space | O(n) extra |
| Stable | Yes |
| Key advantage | Guaranteed O(n log n) — no worst case degradation |
| Key disadvantage | O(n) extra space |

---

## Quick Revision Questions

1. **What are the three steps of the Divide and Conquer paradigm?**
2. **What is the base case in Merge Sort's recursion?**
3. **How many levels are in the recursion tree for array of size n?**
4. **How much work is done at each level of the recursion tree?**
5. **Why is Merge Sort stable? What condition in the merge ensures this?**
6. **What is the recurrence relation for Merge Sort?**

---

[← Previous Unit: Binary Insertion Sort](../04-Insertion-Sort/06-binary-insertion-sort.md) | [Next: Implementation →](02-implementation.md)
