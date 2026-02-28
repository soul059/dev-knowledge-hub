# Unit 8: Counting Sort — Complexity Analysis

[← Previous: Stability Deep Dive](03-stability-deep-dive.md) | [Next: Limitations & When to Use →](05-limitations-and-when-to-use.md)

---

## Overview

Counting Sort runs in **O(n + k)** time and uses **O(n + k)** space, where n is the number of elements and k is the range of input values. This chapter breaks down exactly why.

---

## Time Complexity Breakdown

```
  COUNTING-SORT(A, n, k):
  
  Step 1: Find min and max              → O(n)
  Step 2: Initialize count array        → O(k)
  Step 3: Count occurrences             → O(n)
  Step 4: Prefix sum                    → O(k)
  Step 5: Build output array            → O(n)
  Step 6: Copy back                     → O(n)
  ──────────────────────────────────────────────
  Total:                                 O(n + k)
  
  ┌──────────────────────────────────────────────┐
  │  If k ≤ n:   O(n + k) = O(n)   → LINEAR!   │
  │  If k = O(n): O(n + n) = O(n)  → LINEAR!    │
  │  If k >> n:  O(k)     → WASTEFUL             │
  └──────────────────────────────────────────────┘
```

---

## When Is Counting Sort O(n)?

```
  Counting Sort is linear when k = O(n):
  
  ┌────────────────────────────────────────────────────────────┐
  │  Example 1: Sort exam scores (0-100) for 1000 students   │
  │  n = 1000, k = 100                                        │
  │  k < n → O(n + k) = O(n) ✓                               │
  │                                                            │
  │  Example 2: Sort ages (0-150) for 1 million people        │
  │  n = 1,000,000, k = 150                                   │
  │  k << n → O(n) ✓                                          │
  │                                                            │
  │  Example 3: Sort values in range [0, n²]                  │
  │  n = 1000, k = 1,000,000                                  │
  │  k >> n → O(k) = O(n²) ✗ Worse than comparison sorts!    │
  │                                                            │
  │  Example 4: Sort 32-bit integers (k = 4 billion)          │
  │  k = 2³² → impractical for direct counting sort          │
  │  → Use Radix Sort instead!                                │
  └────────────────────────────────────────────────────────────┘
```

---

## Space Complexity

```
  ┌─────────────────────────────────────────────────┐
  │  Count array:  O(k)   (size = range of values) │
  │  Output array: O(n)   (same size as input)      │
  │  ──────────────────────────────────              │
  │  Total:        O(n + k)                         │
  └─────────────────────────────────────────────────┘
  
  Counting Sort is NOT in-place.
  It requires significant extra memory.
  
  Compare with other sorts:
  ┌──────────────┬──────────┐
  │  Sort        │  Space   │
  ├──────────────┼──────────┤
  │  Heap Sort   │  O(1)    │
  │  Quick Sort  │  O(log n)│
  │  Merge Sort  │  O(n)    │
  │  Counting    │  O(n+k)  │
  └──────────────┴──────────┘
```

---

## Comparison with O(n log n) Sorts

```
  When is Counting Sort FASTER than Quick Sort?
  
  Quick Sort: O(n log n) comparisons, each O(1)
  Counting:   O(n + k) operations
  
  Counting wins when: n + k < n log n
                  ↔   k < n(log n - 1)
                  ↔   k < n log n  (approximately)
  
  For n = 1,000,000:
  - n log₂ n ≈ 20,000,000
  - If k < 20,000,000, Counting Sort wins
  - Each element can range up to 20 million ✓
  
  For n = 100:
  - n log₂ n ≈ 664
  - If k < 664, Counting Sort wins
  
  ┌──────────────────────────────────────────────────────┐
  │  Rule of thumb: counting sort is great when:        │
  │  k ≤ n, or at most k ≤ n × constant                │
  └──────────────────────────────────────────────────────┘
```

---

## Best, Average, and Worst Cases

```
  ┌─────────────┬─────────────┬──────────────────────────────┐
  │  Case       │ Complexity  │ Notes                        │
  ├─────────────┼─────────────┼──────────────────────────────┤
  │  Best       │ O(n + k)    │ Same — not adaptive          │
  │  Average    │ O(n + k)    │ Same — deterministic         │
  │  Worst      │ O(n + k)    │ Same — no degradation        │
  └─────────────┴─────────────┴──────────────────────────────┘
  
  Counting Sort is NOT adaptive:
  - Already-sorted input takes same time as reversed input
  - The algorithm doesn't check existing order
  - It always counts, sums, and places
```

---

## Number of Comparisons

```
  ┌──────────────────────────────────────────────────────┐
  │  Counting Sort makes ZERO element comparisons!      │
  │                                                      │
  │  It only uses:                                       │
  │  - Array indexing (A[i])                            │
  │  - Arithmetic (increment, add)                      │
  │  - Assignment                                        │
  │                                                      │
  │  This is why it can beat the Ω(n log n) lower bound │
  │  — that bound only applies to comparison-based sorts │
  └──────────────────────────────────────────────────────┘
```

---

## Memory Access Pattern & Cache Behavior

```
  Counting Sort has POOR cache behavior:
  
  Step 3 (counting): 
    Read A sequentially ✓  (good locality)
    Write C randomly ✗     (jumps around count array)
  
  Step 5 (placement):
    Read A sequentially ✓
    Read C randomly ✗
    Write B randomly ✗     (output positions jump around)
  
  This is why Counting Sort may be slower than expected
  on modern CPUs despite O(n + k) time complexity.
  
  Quick Sort with O(n log n) may beat Counting Sort
  due to superior cache performance when k is moderate.
```

---

## Summary Table

| Metric | Value |
|--------|-------|
| Time (all cases) | O(n + k) |
| Space | O(n + k) |
| Comparisons | 0 |
| Stable | Yes (with right-to-left) |
| In-place | No |
| Adaptive | No |
| Practical when | k = O(n) |
| Impractical when | k >> n |

---

## Quick Revision Questions

1. **What does k represent in Counting Sort's O(n + k)?**
2. **When does Counting Sort degrade worse than Quick Sort?**
3. **How many element-to-element comparisons does Counting Sort make?**
4. **Why is Counting Sort NOT in-place?**
5. **In what scenario is Counting Sort truly O(n)?**

---

[← Previous: Stability Deep Dive](03-stability-deep-dive.md) | [Next: Limitations & When to Use →](05-limitations-and-when-to-use.md)
