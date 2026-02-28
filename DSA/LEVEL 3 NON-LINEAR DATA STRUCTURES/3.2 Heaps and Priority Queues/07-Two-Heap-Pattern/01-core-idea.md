# 7.1 The Core Idea: Two Heap Pattern

[← Previous: Unit 6 — K-Pairs & Framework](../06-K-Element-Problems/06-k-pairs-and-framework.md) | [Next: Median from Data Stream →](02-median-from-data-stream.md)

---

## Overview

The **Two Heap Pattern** is one of the most elegant heap patterns. By maintaining a **max-heap** for the smaller half and a **min-heap** for the larger half of a dataset, we can efficiently track the **median** and solve a family of related optimization problems.

---

## Splitting Data into Two Halves

```
All elements, sorted conceptually:

[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
|←── lower half ──→|←── upper half ──→|
   MAX-HEAP              MIN-HEAP
   
   Max-heap root → 5   6 ← Min-heap root
   (largest of          (smallest of
    small half)          large half)
```

---

## Why This Works for Median

- **Max-heap** stores the **smaller half** → its root is the **largest** of the small elements
- **Min-heap** stores the **larger half** → its root is the **smallest** of the large elements
- The **median** is at the boundary: it's either the max-heap root, the min-heap root, or their average

---

## The Two Rules

```
Rule 1 - ORDERING:   max_heap.root ≤ min_heap.root
         (Every element in lower half ≤ every element in upper half)

Rule 2 - BALANCE:    |max_heap.size - min_heap.size| ≤ 1
         (Sizes differ by at most 1)
```

---

## When to Use Two Heaps

| Signal | Example Problem |
|--------|----------------|
| "Find the median" | Streaming median, sliding window median |
| "Balance two groups" | IPO/maximize capital |
| "Partition into two optimized halves" | Minimize difference between two groups |
| "Track boundary between small and large" | Running percentile |

---

## Quick Check

1. **Why does the lower half use a MAX-heap and not a min-heap?**
   <details><summary>Answer</summary>
   We need quick access to the LARGEST element in the lower half (to find the median boundary). A max-heap gives us that in O(1). A min-heap would give us the smallest of the lower half, which is useless for finding the median.
   </details>

---

[← Previous: Unit 6 — K-Pairs & Framework](../06-K-Element-Problems/06-k-pairs-and-framework.md) | [Next: Median from Data Stream →](02-median-from-data-stream.md)
