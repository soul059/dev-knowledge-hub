# Chapter 5: Complexity Analysis of Linear Search

[← Previous: When Linear Search is Best](04-when-linear-search-is-best.md) | [Next: Unit 2 — Binary Search Concept →](../02-Binary-Search-Basics/01-binary-search-concept.md)

---

## Overview

Understanding **time and space complexity** is essential for evaluating when to use linear search and predicting its performance on different input sizes. This chapter provides a rigorous analysis of all cases.

---

## Time Complexity Analysis

### Best Case: O(1)

The target is found at the **very first position** (index 0).

```
   Target = 10

   ┌────┬────┬────┬────┬────┐
   │ 10 │ 23 │  5 │ 42 │ 17 │
   └────┴────┴────┴────┴────┘
     ↑
   FOUND immediately! Only 1 comparison.
```

- **Comparisons:** 1
- **Time:** O(1) — constant

---

### Worst Case: O(n)

The target is at the **last position** or **not present at all**.

```
   Case A: Target at last position (target = 17)
   ┌────┬────┬────┬────┬────┐
   │ 10 │ 23 │  5 │ 42 │ 17 │
   └────┴────┴────┴────┴────┘
     ↑    ↑    ↑    ↑    ↑
     1    2    3    4    5 = n comparisons

   Case B: Target not found (target = 99)
   ┌────┬────┬────┬────┬────┐
   │ 10 │ 23 │  5 │ 42 │ 17 │
   └────┴────┴────┴────┴────┘
     ↑    ↑    ↑    ↑    ↑
     1    2    3    4    5 = n comparisons → return -1
```

- **Comparisons:** n
- **Time:** O(n) — linear

---

### Average Case: O(n)

Assuming the target is **equally likely** to be at any position (or not present):

```
   Probability of target at index i:  P(i) = 1/n   (for i = 0 to n-1)

   Expected comparisons (if target IS present):
   E = Σ (i+1) × (1/n)  for i = 0 to n-1
   E = (1/n) × Σ (i+1)
   E = (1/n) × (1 + 2 + 3 + ... + n)
   E = (1/n) × n(n+1)/2
   E = (n + 1) / 2
```

```
   Visualization of average case:

   Position:  0    1    2    3    4    5    6    7
             ┌────┬────┬────┬────┬────┬────┬────┬────┐
             │    │    │    │    │    │    │    │    │
             └────┴────┴────┴────┴────┴────┴────┴────┘
                              ↑
                        Average finds here
                        at position n/2
                        
   For n = 8: Average comparisons = (8+1)/2 = 4.5
   For n = 100: Average comparisons = 50.5
   For n = 1000: Average comparisons = 500.5
```

So the average case is **(n+1)/2 ≈ n/2**, which is still **O(n)**.

---

### If Target May Not Be Present

If there's a probability `p` that the target exists:

```
   E = p × (n+1)/2  +  (1-p) × n

   If p = 0.5 (50% chance target exists):
   E = 0.5 × (n+1)/2 + 0.5 × n
   E = (n+1)/4 + n/2
   E = (3n + 1)/4
   E ≈ 3n/4

   Still O(n)!
```

---

## Space Complexity: O(1)

Linear search uses only a **constant amount of extra space**:

```
   Variables used:
   ┌──────────────────────────────┐
   │  i (loop counter)     → O(1)│
   │  target (search key)  → O(1)│
   │  n (array size)       → O(1)│
   │  result (return val)  → O(1)│
   └──────────────────────────────┘
   
   Total extra space: O(1)
   
   Note: The input array is NOT counted as extra space
   (it's given as input, not created by the algorithm)
```

### Exception: Recursive Linear Search

```
   Call Stack for recursive version (n = 5):
   
   ┌─────────────────────────┐
   │ search(arr, 4, target)  │  ← Frame 5
   ├─────────────────────────┤
   │ search(arr, 3, target)  │  ← Frame 4
   ├─────────────────────────┤
   │ search(arr, 2, target)  │  ← Frame 3
   ├─────────────────────────┤
   │ search(arr, 1, target)  │  ← Frame 2
   ├─────────────────────────┤
   │ search(arr, 0, target)  │  ← Frame 1
   └─────────────────────────┘
   
   Space: O(n) due to recursive call stack
```

---

## Comparison Count Analysis

### Basic Linear Search
| Operation per iteration | Count |
|------------------------|-------|
| Boundary check: `i < n` | 1 |
| Equality check: `arr[i] == target` | 1 |
| Increment: `i++` | 1 |
| **Total per iteration** | **3 ops** |
| **Worst case total** | **3n ops** |

### Sentinel Search
| Operation per iteration | Count |
|------------------------|-------|
| Equality check: `arr[i] == target` | 1 |
| Increment: `i++` | 1 |
| **Total per iteration** | **2 ops** |
| **Worst case total** | **2n + 3 ops** |

(+3 for save, set sentinel, restore)

---

## Growth Rate Visualization

```
   Comparisons
   ▲
   │
   │                                          ╱ n (worst case)
   │                                        ╱
   │                                      ╱
   │                                    ╱
   │                                  ╱
   │                                ╱
   │                              ╱
   │                            ╱
   │                          ╱      ╱ n/2 (average)
   │                        ╱     ╱
   │                      ╱    ╱
   │                    ╱   ╱
   │                  ╱  ╱
   │                ╱╱
   │             ╱╱
   │          ╱╱
   │       ╱╱
   │    ╱╱  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  1 (best case)
   │ ╱╱
   └──────────────────────────────────────► n (input size)
```

---

## Practical Performance Impact

| Array Size (n) | Best Case | Average Case | Worst Case |
|----------------|-----------|-------------|------------|
| 10 | 1 | 5.5 | 10 |
| 100 | 1 | 50.5 | 100 |
| 1,000 | 1 | 500.5 | 1,000 |
| 10,000 | 1 | 5,000.5 | 10,000 |
| 1,000,000 | 1 | 500,000.5 | 1,000,000 |
| 10⁹ | 1 | 5 × 10⁸ | 10⁹ (~1 sec) |

> **Rule of thumb:** At ~10⁸ operations per second, linear search on 10⁹ elements takes about **10 seconds** in the worst case.

---

## Comparing with Binary Search Growth

```
   Operations
   ▲
   │                                           ╱  O(n) Linear
   │                                         ╱
   │                                       ╱
   │                                     ╱
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
   │             ╱      ╌╌╌╌╌╌╌╌╌╌╌╌╌    O(log n) Binary
   │           ╱  ╌╌╌╌╌╌
   │         ╱╌╌╌╌
   │       ╱╌
   │     ╱╌
   │   ╱╌
   │  ╱
   └──────────────────────────────────────────────► n
```

| n | Linear O(n) | Binary O(log n) | Speedup |
|---|-------------|----------------|---------|
| 16 | 16 | 4 | 4× |
| 256 | 256 | 8 | 32× |
| 1,024 | 1,024 | 10 | 102× |
| 1,048,576 | 1,048,576 | 20 | 52,428× |
| 10⁹ | 10⁹ | 30 | 33,333,333× |

---

## Amortized Analysis

If you perform **multiple searches** on the same unsorted array:

```
   Approach 1: k Linear Searches
   Total = k × O(n) = O(kn)

   Approach 2: Sort Once + k Binary Searches
   Total = O(n log n) + k × O(log n) = O(n log n + k log n)

   Approach 2 wins when:
   kn > n log n + k log n
   k(n - log n) > n log n
   k > (n log n) / (n - log n)
   k > log n  (approximately)
```

---

## Summary Table

| Aspect | Value | Notes |
|--------|-------|-------|
| **Best Time** | O(1) | Target at index 0 |
| **Average Time** | O(n) | (n+1)/2 comparisons |
| **Worst Time** | O(n) | Target at end or absent |
| **Space (Iterative)** | O(1) | Only loop variable |
| **Space (Recursive)** | O(n) | Call stack depth |
| **In-place** | Yes | No extra data structures |
| **Comparison count** | n (worst) | Basic: 2n checks, Sentinel: n checks |

---

## Quick Revision Questions

1. **Derive the average number of comparisons for linear search when the target is guaranteed to be present.**
2. **Why is the average case still O(n) even though the expected comparisons are n/2?**
3. **What is the space complexity of recursive linear search and why?**
4. **At what number of repeated queries does it become more efficient to sort first and then use binary search?**
5. **How many comparisons does sentinel search save compared to basic linear search in the worst case?**
6. **If a computer performs 10⁸ operations per second, approximately how long does linear search take on an array of 10⁷ elements in the worst case?**

---

[← Previous: When Linear Search is Best](04-when-linear-search-is-best.md) | [Next: Unit 2 — Binary Search Concept →](../02-Binary-Search-Basics/01-binary-search-concept.md)
