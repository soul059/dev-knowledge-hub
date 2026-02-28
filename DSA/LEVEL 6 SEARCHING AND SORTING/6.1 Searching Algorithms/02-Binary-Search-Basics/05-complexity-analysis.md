# Chapter 5: Binary Search — Complexity Analysis

[← Previous: Loop Invariant](04-loop-invariant.md) | [Next: Prerequisites & Gotchas →](06-prerequisites-and-gotchas.md)

---

## Overview

Binary search achieves **O(log n)** time complexity by halving the search space each iteration. This chapter provides a rigorous analysis of why, and what it means in practice.

---

## Time Complexity Derivation

### The Halving Argument

```
   Start: n elements
   After 1 comparison:  n/2 elements remain
   After 2 comparisons: n/4 elements remain
   After 3 comparisons: n/8 elements remain
   ...
   After k comparisons: n/2^k elements remain

   Search ends when: n/2^k = 1
   Solving: 2^k = n
            k = log₂(n)

   Therefore: Binary search makes at most ⌊log₂(n)⌋ + 1 comparisons
```

---

### Formal Recurrence Relation

```
   T(n) = Time to search array of size n

   T(n) = T(n/2) + O(1)      ← One comparison + search half
   T(1) = O(1)                ← Base case

   Using Master Theorem (a=1, b=2, d=0):
   T(n) = O(n^(log_b(a)) × log n) = O(n^0 × log n) = O(log n)
   
   Or by expansion:
   T(n) = T(n/2) + c
        = T(n/4) + c + c
        = T(n/8) + c + c + c
        = T(n/2^k) + k×c
   
   When n/2^k = 1 → k = log₂(n):
   T(n) = T(1) + c × log₂(n) = O(log n)
```

---

## All Three Cases

### Best Case: O(1)

```
   Target is exactly at the midpoint!

   ┌───┬───┬───┬───┬───┬───┬───┐
   │ 2 │ 5 │ 8 │ 12│ 16│ 23│ 38│   Target = 12
   └───┴───┴───┴───┴───┴───┴───┘
                  ↑
                 mid
   arr[3] = 12 == 12 → FOUND in 1 comparison!
```

### Average Case: O(log n)

On average, binary search finds the target after approximately `log₂(n) - 1` comparisons.

```
   For a successful search on n elements:
   
   Elements found in 1 comparison:      1  (the middle)
   Elements found in 2 comparisons:     2  (midpoints of halves)
   Elements found in 3 comparisons:     4  (midpoints of quarters)
   ...
   Elements found in k comparisons:     2^(k-1)
   
   Average = (1×1 + 2×2 + 3×4 + ... + k×2^(k-1)) / n
   
   where k = ⌊log₂(n)⌋ + 1
   
   This simplifies to approximately: log₂(n) - 1 ≈ O(log n)
```

### Worst Case: O(log n)

```
   Target is at the very end or not present.
   
   Array: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16]
   Target: 1 (leftmost element — worst case for standard binary search)
   
   n = 16 → log₂(16) = 4 comparisons

   Step 1: mid=8,  arr[8]=9   > 1  → search [0..7]    (16→8)
   Step 2: mid=4,  arr[4]=5   > 1  → search [0..3]    (8→4)
   Step 3: mid=2,  arr[2]=3   > 1  → search [0..1]    (4→2)
   Step 4: mid=1,  arr[1]=2   > 1  → search [0..0]    (2→1)
   Step 5: mid=0,  arr[0]=1   == 1 → FOUND!            
   
   Total: 5 comparisons = ⌊log₂(16)⌋ + 1
```

---

## Space Complexity

### Iterative: O(1)

```
   Memory used:
   ┌─────────────┐
   │ lo    (int)  │  4 bytes
   │ hi    (int)  │  4 bytes
   │ mid   (int)  │  4 bytes
   └─────────────┘
   Total: 12 bytes = O(1) — CONSTANT regardless of n
```

### Recursive: O(log n)

```
   Each recursive call adds a stack frame:

   ┌──────────────────┐
   │ Frame log₂(n)    │  ← Deepest call
   ├──────────────────┤
   │ Frame log₂(n)-1  │
   ├──────────────────┤
   │       ...        │
   ├──────────────────┤
   │ Frame 2          │
   ├──────────────────┤
   │ Frame 1          │  ← First call
   └──────────────────┘

   Stack depth = O(log n)
   Each frame ≈ 20-40 bytes (lo, hi, mid, return address)
   Total: O(log n) space
```

---

## Practical Numbers

| Array Size (n) | Max Comparisons (log₂n + 1) | Linear Search (worst) |
|----------------|-----------------------------|-----------------------|
| 8 | 4 | 8 |
| 16 | 5 | 16 |
| 32 | 6 | 32 |
| 64 | 7 | 64 |
| 256 | 9 | 256 |
| 1,024 | 11 | 1,024 |
| 1,000,000 | 21 | 1,000,000 |
| 10⁹ | 31 | 10⁹ |
| 10¹⁸ | 61 | 10¹⁸ |

> Even for an array with **1 quintillion** elements, binary search needs only **61 comparisons!**

---

## Growth Rate Comparison

```
   Comparisons
   ▲
 1000│                                               ╱ O(n)
     │                                             ╱
     │                                           ╱
     │                                         ╱
     │                                       ╱
  500│                                     ╱
     │                                   ╱
     │                                 ╱
     │                               ╱
     │                             ╱
     │                           ╱
     │                         ╱
     │                       ╱
     │                     ╱
     │                   ╱
     │                 ╱
     │               ╱
     │             ╱
     │           ╱
   10│─ ─ ─ ─╱─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ O(log n)
     │       ╱
     │     ╱
     │   ╱╱
     │ ╱
     └──────────────────────────────────────────────► n
          100       200       500      1000
```

---

## Why O(log n) is Powerful

```
   ┌──────────────────────────────────────────────────┐
   │  Going from n to 2n:                             │
   │                                                    │
   │  Linear Search:  Doubles the work (n → 2n)       │
   │  Binary Search:  Just ONE more comparison!        │
   │                  (log n → log n + 1)              │
   │                                                    │
   │  Example:                                         │
   │  n = 1,000,000    → 20 comparisons                │
   │  n = 2,000,000    → 21 comparisons (just +1!)     │
   │  n = 4,000,000    → 22 comparisons (just +1!)     │
   │  n = 1,000,000,000 → 30 comparisons (just +10!)   │
   └──────────────────────────────────────────────────┘
```

---

## Comparison with Other Algorithms

| Algorithm | Time | For n = 10⁶ | For n = 10⁹ |
|-----------|------|-------------|-------------|
| Linear Search | O(n) | 10⁶ ops | 10⁹ ops (~1 sec) |
| Binary Search | O(log n) | 20 ops | 30 ops |
| Interpolation | O(log log n)* | ~4 ops | ~5 ops |
| Hash Table | O(1) amortized | 1 op | 1 op |

*Only for uniformly distributed data.

---

## Cost of Sorting First

```
   Total cost = Sort + Search

   One-time search:
   ┌──────────────┬──────────┐
   │ Sort: O(n log n) │ Search: O(log n) │ = O(n log n)  ← Worse than O(n)!
   └──────────────┴──────────┘

   Multiple searches (q queries):
   ┌──────────────┬────────────────┐
   │ Sort: O(n log n) │ q × O(log n)  │ = O(n log n + q log n)
   └──────────────┴────────────────┘

   Linear search for q queries: O(qn)

   Binary search wins when:
   n log n + q log n < qn
   q > (n log n) / (n - log n) ≈ log n queries
```

---

## Summary Table

| Aspect | Value |
|--------|-------|
| **Best Case Time** | O(1) |
| **Average Case Time** | O(log n) |
| **Worst Case Time** | O(log n) |
| **Space (Iterative)** | O(1) |
| **Space (Recursive)** | O(log n) |
| **Comparisons** | ⌊log₂(n)⌋ + 1 max |
| **Recurrence** | T(n) = T(n/2) + O(1) |
| **Doubling n adds** | Just 1 more comparison |

---

## Quick Revision Questions

1. **Derive the time complexity of binary search using the recurrence T(n) = T(n/2) + O(1).**
2. **How many comparisons does binary search need for an array of 10⁹ elements?**
3. **If you double the size of the array, how many additional comparisons does binary search need?**
4. **Why is the space complexity O(log n) for recursive binary search?**
5. **At what point (number of queries) does sorting + binary search become more efficient than repeated linear search?**
6. **Compare the number of operations for n = 1,000,000: linear search vs binary search.**

---

[← Previous: Loop Invariant](04-loop-invariant.md) | [Next: Prerequisites & Gotchas →](06-prerequisites-and-gotchas.md)
