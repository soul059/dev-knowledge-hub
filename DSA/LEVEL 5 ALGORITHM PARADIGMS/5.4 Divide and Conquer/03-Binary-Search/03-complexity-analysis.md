# Chapter 3: Binary Search — Complexity Analysis

## Overview

Binary Search achieves O(log n) by halving the search space with each comparison. This chapter provides rigorous analysis and comparisons.

---

## Time Complexity Analysis

### Recurrence Relation

```
T(n) = T(n/2) + O(1)

At each step:
  - O(1) to compute mid and compare
  - T(n/2) to recurse on one half

Base case: T(1) = O(1)
```

### Solving by Unrolling

```
T(n) = T(n/2) + c
     = T(n/4) + c + c
     = T(n/8) + c + c + c
     = T(n/2^k) + k·c

Stops when n/2^k = 1 → k = log₂(n)

T(n) = T(1) + log₂(n) · c = O(log n) ✓
```

### Solving by Master Theorem

```
T(n) = 1·T(n/2) + O(1)
a = 1, b = 2, f(n) = O(1) = O(n⁰)
c_crit = log₂(1) = 0
c = 0 = c_crit → Case 2
T(n) = Θ(n⁰ · log n) = Θ(log n) ✓
```

---

## Why O(log n) Is Incredibly Fast

```
n             | log₂(n)    | Linear scan | Speedup
──────────────|────────────|─────────────|────────
10            | 3.3        | 10          | 3x
100           | 6.6        | 100         | 15x
1,000         | 10         | 1,000       | 100x
1,000,000     | 20         | 1,000,000   | 50,000x
1,000,000,000 | 30         | 1,000,000,000| 33,000,000x

Even for a BILLION elements, only 30 comparisons!
```

```
Visual: Growth rate comparison

Operations
     │
 100 │ ·························· Linear O(n)
     │ ·
  80 │ ·
     │ ·
  60 │ ·
     │ ·
  40 │ ·
     │·
  20 │·         
     │·    ─────────────────────── O(log n)
   0 │·───────────────────────────
     └──────────────────────────── n
     0    20   40   60   80  100
```

---

## Space Complexity

### Iterative Implementation
```
Space: O(1)
Only uses: low, high, mid — three variables
No extra data structures needed
```

### Recursive Implementation
```
Space: O(log n)
Each recursive call adds a stack frame
Maximum recursion depth = log₂(n)

Call Stack (worst case for n=16):
┌──────────────────────────┐
│ binarySearch(0, 15)      │ ← Frame 1
│ binarySearch(8, 15)      │ ← Frame 2
│ binarySearch(8, 11)      │ ← Frame 3
│ binarySearch(8, 9)       │ ← Frame 4
│ binarySearch(9, 9)       │ ← Frame 5 (base case)
└──────────────────────────┘
Depth = log₂(16) = 4 frames
```

---

## Number of Comparisons

### Exact Analysis

```
For an array of n elements:

Successful search:
  Best case:  1 comparison (target at midpoint)
  Worst case: ⌊log₂(n)⌋ + 1 comparisons
  Average:    ≈ log₂(n) comparisons

Unsuccessful search:
  Always:     ⌊log₂(n)⌋ + 1 comparisons
  (must exhaust entire search space)
```

### Comparison Count by Array Size

| n | Max Comparisons | Search Space Reduction |
|---|----------------|----------------------|
| 1 | 1 | 1 → done |
| 2 | 2 | 2 → 1 → done |
| 4 | 3 | 4 → 2 → 1 |
| 8 | 4 | 8 → 4 → 2 → 1 |
| 16 | 5 | 16 → 8 → 4 → 2 → 1 |
| 1024 | 11 | Halves 10 times |
| 2^20 ≈ 10^6 | 21 | Halves 20 times |

---

## Lower Bound: Binary Search Is Optimal

```
Theorem: Any comparison-based search algorithm on a sorted
         array of n elements requires Ω(log n) comparisons
         in the worst case.

Proof:
  Each comparison has 3 outcomes: <, =, >
  With k comparisons, you can distinguish at most 3^k cases
  To find among n elements: 3^k ≥ n → k ≥ log₃(n)
  
  More precisely (binary model):
  Each comparison has 2 useful outcomes: left or right
  With k comparisons, 2^k ≥ n+1 (n elements + "not found")
  k ≥ log₂(n+1) = Ω(log n)

Binary Search achieves this bound → IT IS OPTIMAL!
```

---

## Comparison with Other Search Methods

| Method | Time | Space | Requires Sorted? | Notes |
|--------|------|-------|-------------------|-------|
| Linear Search | O(n) | O(1) | No | Works on any array |
| Binary Search | O(log n) | O(1) | Yes | Requires sorted array |
| Hash Table | O(1) average | O(n) | No | Extra space, hash collisions |
| BST Search | O(log n) average | O(n) | N/A (tree structure) | Can degrade to O(n) |
| Interpolation Search | O(log log n) avg | O(1) | Yes, uniform distribution | Degrades to O(n) |

---

## When Is Binary Search Worth the Sort?

```
If array searched k times:
  Linear search total: O(k · n)
  Sort + Binary search total: O(n log n + k · log n)

Break-even:
  k · n = n log n + k · log n
  k · n ≈ n log n  (for large n, k · log n is tiny)
  k ≈ log n

If k > log n searches → Binary Search wins!

For n = 1,000,000:  k > 20 searches → use Binary Search
For n = 1,000:      k > 10 searches → use Binary Search
```

---

## Summary Table

| Aspect | Complexity | Notes |
|--------|-----------|-------|
| Time (best) | O(1) | Target at midpoint |
| Time (average) | O(log n) | ≈ log₂(n) comparisons |
| Time (worst) | O(log n) | ⌊log₂(n)⌋ + 1 comparisons |
| Space (iterative) | O(1) | Three variables only |
| Space (recursive) | O(log n) | Call stack depth |
| Lower bound | Ω(log n) | Optimal for comparison-based search |
| Prerequisite | — | Array must be sorted |

---

## Quick Revision Questions

1. **Derive the time complexity of Binary Search using the unrolling method.**
2. **Why is the iterative version preferred over recursive in practice?**
3. **How many comparisons does Binary Search need for an array of 10⁹ elements?**
4. **Prove that Binary Search is optimal among comparison-based search algorithms.**
5. **When is it worth sorting an array just to use Binary Search?**
6. **What is the space complexity difference between iterative and recursive Binary Search?**

---

[← Previous: Implementation Variations](02-implementation-variations.md) | [Next: Binary Search on Answer →](04-binary-search-on-answer.md)
