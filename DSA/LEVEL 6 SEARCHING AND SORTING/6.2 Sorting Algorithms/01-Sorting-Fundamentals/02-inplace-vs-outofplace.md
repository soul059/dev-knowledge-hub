# Chapter 2: In-place vs Out-of-place Sorting

[← Previous: Why Sorting Matters](01-why-sorting-matters.md) | [Next: Stable vs Unstable →](03-stable-vs-unstable.md)

---

## Overview

One of the most important classifications of sorting algorithms is based on their **memory usage**. An algorithm is either **in-place** (uses minimal extra memory) or **out-of-place** (requires significant additional memory proportional to input size).

---

## In-place Sorting

An in-place sorting algorithm sorts the data within the **original array** using only a **constant amount of extra space** — O(1) auxiliary space.

### How It Works

```
  ORIGINAL ARRAY (sorted in-place)
  
  Before: ┌───┬───┬───┬───┬───┐
          │ 5 │ 3 │ 1 │ 4 │ 2 │    Extra space: O(1) — just a temp variable
          └───┴───┴───┴───┴───┘
                    │
              sorting happens
              inside the array
              via swaps
                    │
                    ▼
  After:  ┌───┬───┬───┬───┬───┐
          │ 1 │ 2 │ 3 │ 4 │ 5 │    Same array, reordered
          └───┴───┴───┴───┴───┘
  
  Memory layout:
  ┌─────────────────────────────┐
  │  Array (n elements)         │  ← Only this memory is used
  ├─────┐                       │
  │ tmp │ ← O(1) extra          │
  └─────┴───────────────────────┘
```

### Characteristics
- **Space complexity**: O(1) auxiliary
- Elements are rearranged **within the input array**
- Uses **swapping** as the primary operation
- Memory-efficient

### Examples of In-place Algorithms
- **Bubble Sort** — swaps adjacent elements
- **Selection Sort** — swaps minimum with current position
- **Insertion Sort** — shifts elements and inserts
- **Heap Sort** — uses array as implicit heap
- **Quick Sort** — partitions in-place (O(log n) stack space)

---

## Out-of-place Sorting

An out-of-place sorting algorithm requires **additional memory** proportional to the input size — typically O(n) auxiliary space.

### How It Works

```
  ORIGINAL ARRAY                    TEMPORARY ARRAY(S)
  ┌───┬───┬───┬───┬───┐            ┌───┬───┬───┬───┬───┐
  │ 5 │ 3 │ 1 │ 4 │ 2 │    ──►    │   │   │   │   │   │  ← Extra O(n) space
  └───┴───┴───┴───┴───┘            └───┴───┴───┴───┴───┘
         │                                   │
         │    Copy / merge elements           │
         │    between arrays                  │
         ▼                                   ▼
  ┌───┬───┬───┬───┬───┐            ┌───┬───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │ 5 │            │(used during sort) │
  └───┴───┴───┴───┴───┘            └───┴───┴───┴───┴───┘
  
  Memory layout:
  ┌─────────────────────────────┐
  │  Original Array (n)         │
  ├─────────────────────────────┤
  │  Temporary Array (n)        │  ← Additional O(n) memory
  └─────────────────────────────┘
```

### Characteristics
- **Space complexity**: O(n) or more
- Creates **copies** of data during sorting
- May be easier to implement correctly
- Less memory-efficient

### Examples of Out-of-place Algorithms
- **Merge Sort** — needs O(n) temporary array for merging
- **Counting Sort** — needs O(k) count array + O(n) output array
- **Radix Sort** — uses additional arrays for digit distribution
- **Bucket Sort** — creates multiple bucket arrays

---

## Side-by-Side Comparison

```
  ┌─────────────────────────────────────────────────────────────────────┐
  │                     IN-PLACE SORTING                                │
  │                                                                     │
  │   Array: [5, 3, 1, 4, 2]                                          │
  │                                                                     │
  │   Step 1: swap(5,3) → [3, 5, 1, 4, 2]     temp = 1 variable       │
  │   Step 2: swap(5,1) → [3, 1, 5, 4, 2]                             │
  │   ...                                                               │
  │   Result: [1, 2, 3, 4, 5]   Space: O(1)                           │
  ├─────────────────────────────────────────────────────────────────────┤
  │                    OUT-OF-PLACE SORTING                             │
  │                                                                     │
  │   Array: [5, 3, 1, 4, 2]                                          │
  │                                                                     │
  │   Split:  [5, 3] [1, 4, 2]                                        │
  │   Split:  [5][3] [1][4, 2]     temp arrays created at each level   │
  │   Merge:  [3,5]  [1][2,4]                                         │
  │   Merge:  [3,5]  [1, 2, 4]                                        │
  │   Merge:  [1, 2, 3, 4, 5]     Space: O(n)                         │
  └─────────────────────────────────────────────────────────────────────┘
```

---

## Detailed Comparison Table

| Property | In-place | Out-of-place |
|----------|----------|--------------|
| **Extra space** | O(1) | O(n) or more |
| **Memory efficiency** | High | Low |
| **Implementation** | Trickier (swap logic) | Often simpler |
| **Cache performance** | Better (locality) | May be worse |
| **Data movement** | Swaps within array | Copies between arrays |
| **Parallelization** | Harder | Easier (merge sort) |
| **Suitable for** | Memory-constrained | Large datasets with RAM |

---

## Edge Case: Quick Sort

Quick Sort is often called "in-place" but technically uses **O(log n) stack space** for recursion:

```
  Quick Sort Memory Usage:
  
  ┌─────────────────────────────┐
  │  Array (n elements)         │  ← Input array (modified in-place)
  ├─────────────────────────────┤
  │  Recursion Stack            │  ← O(log n) average
  │  ┌─────┐                   │     O(n) worst case
  │  │ f() │                   │
  │  ├─────┤                   │
  │  │ f() │                   │
  │  ├─────┤                   │
  │  │ f() │  ~log n frames    │
  │  └─────┘                   │
  └─────────────────────────────┘
  
  Verdict: "In-place" by convention, but O(log n) auxiliary space
```

---

## When to Choose Which?

```
  Decision Flow:
  
  Is memory limited?
       │
       ├── YES → Use IN-PLACE algorithms
       │         (Selection, Insertion, Heap Sort)
       │
       └── NO → Is stability needed?
                    │
                    ├── YES → Use Merge Sort (out-of-place, stable)
                    │
                    └── NO → Quick Sort (in-place-ish, fast)
```

---

## Problem-Solving Pattern

When a problem says "sort in-place" or "O(1) extra space":
1. Think **swap-based** approaches
2. Use **two-pointer** or **three-pointer** techniques
3. Consider **cyclic sort** for specific problems
4. **Overwrite** from one end of the array

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| In-place | O(1) extra space, sorts within original array |
| Out-of-place | O(n) extra space, uses temporary arrays |
| In-place examples | Bubble, Selection, Insertion, Heap Sort |
| Out-of-place examples | Merge Sort, Counting Sort, Radix Sort |
| Quick Sort | O(log n) stack space — considered in-place by convention |
| Trade-off | Memory savings vs implementation simplicity |

---

## Quick Revision Questions

1. **What defines an in-place sorting algorithm?**
2. **Why is Merge Sort considered out-of-place?**
3. **Is Quick Sort truly in-place? Explain.**
4. **Which is more cache-friendly: in-place or out-of-place? Why?**
5. **When would you prefer an out-of-place algorithm despite its memory cost?**
6. **What techniques enable in-place sorting? (Hint: swaps, pointers)**

---

[← Previous: Why Sorting Matters](01-why-sorting-matters.md) | [Next: Stable vs Unstable →](03-stable-vs-unstable.md)
