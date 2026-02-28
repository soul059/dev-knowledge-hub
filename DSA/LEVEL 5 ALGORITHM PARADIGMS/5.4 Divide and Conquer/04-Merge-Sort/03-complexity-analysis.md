# Chapter 3: Merge Sort Complexity Analysis

## Overview

Merge Sort achieves **O(n log n)** in all cases — best, average, and worst. This chapter rigorously derives the time and space complexity using multiple proof techniques.

---

## The Recurrence Relation

```
T(n) = 2T(n/2) + O(n)

Where:
  2T(n/2) = cost of recursively sorting two halves
  O(n)    = cost of merging two sorted halves

Base case: T(1) = O(1)
```

---

## Proof 1: Recursion Tree Method

```
Level 0:            cn                        cost = cn
                   /    \
Level 1:       cn/2      cn/2                 cost = cn
               / \        / \
Level 2:    cn/4  cn/4  cn/4  cn/4            cost = cn
              ...            ...
Level k:  c  c  c  c ... c  c  c  c          cost = cn
          (n nodes of cost c each)

Number of levels: log₂(n)
Cost per level: cn (constant!)

Total = cn × log₂(n) = O(n log n)
```

### Why Each Level Costs cn

```
Level k has 2^k subproblems, each of size n/2^k.

Merge cost at level k = 2^k × c(n/2^k) = cn

This is constant across ALL levels!

Level 0: 1 × cn = cn
Level 1: 2 × cn/2 = cn
Level 2: 4 × cn/4 = cn
Level 3: 8 × cn/8 = cn
...every level = cn
```

---

## Proof 2: Master Theorem

```
T(n) = 2T(n/2) + cn

a = 2, b = 2, f(n) = cn

Critical exponent: log_b(a) = log₂(2) = 1

Compare f(n) = cn = n^1 with n^(log_b(a)) = n^1

f(n) = Θ(n^1) → Case 2 of Master Theorem

T(n) = Θ(n^1 × log n) = Θ(n log n) ✓
```

---

## Proof 3: Substitution Method (Induction)

```
Claim: T(n) ≤ cn log₂(n) for some constant c > 0

Base: T(1) = c₁ ≤ c × 1 × log₂(1) = 0
  → Need T(2) = 2T(1) + c₂ ≤ c × 2 × 1 = 2c  ✓ (for large enough c)

Inductive step: Assume T(k) ≤ ck log₂(k) for all k < n

T(n) = 2T(n/2) + cn
     ≤ 2 × c(n/2)log₂(n/2) + cn
     = cn(log₂(n) - 1) + cn
     = cn·log₂(n) - cn + cn
     = cn·log₂(n)

T(n) ≤ cn log₂(n) = O(n log n) ✓
```

---

## Space Complexity Analysis

```
┌────────────────────────────────────────────────────────────┐
│  AUXILIARY SPACE: O(n)                                     │
│                                                            │
│  The merge step needs a temporary array of size n.         │
│  Even though merge is called many times, the arrays are    │
│  reused (or can be allocated once).                        │
│                                                            │
│  CALL STACK SPACE: O(log n)                               │
│                                                            │
│  Recursion depth = log₂(n) levels                         │
│  Each level uses O(1) stack frame space                    │
│                                                            │
│  TOTAL SPACE: O(n) + O(log n) = O(n)                     │
└────────────────────────────────────────────────────────────┘
```

### Stack Space Visualization

```
mergeSort(0,7)
  mergeSort(0,3)
    mergeSort(0,1)
      mergeSort(0,0) ← base      ← max depth = log₂(8) = 3
      mergeSort(1,1) ← base
    mergeSort(2,3)
      ...
  mergeSort(4,7)
    ...

At any point, at most log₂(n) frames are on the stack.
```

---

## Comparison Count Analysis

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  BEST CASE comparisons:                                  │
│    Each merge: n/2 comparisons (one half entirely < other)│
│    Total: (n/2) × log₂(n)                               │
│    ≈ (n log n) / 2                                       │
│                                                          │
│  WORST CASE comparisons:                                 │
│    Each merge: n - 1 comparisons (perfect interleaving)  │
│    Total: (n-1) × log₂(n)                               │
│    ≈ n log n                                             │
│                                                          │
│  AVERAGE CASE:                                           │
│    ≈ n log₂(n) - 1.4427n + O(log n)                     │
│                                                          │
│  All three are Θ(n log n)                                │
└──────────────────────────────────────────────────────────┘
```

---

## Why Best = Worst = Average?

```
Unlike Quick Sort, Merge Sort always:
  1. Splits exactly in half    → perfectly balanced tree
  2. Processes ALL elements    → no skipping
  3. Always merges             → merge cost is always O(n)

The input order affects the NUMBER of comparisons in merge,
but NOT the asymptotic complexity:

  Best case merge:   n/2 comparisons → O(n)
  Worst case merge:  n-1 comparisons → O(n)
  
Both are O(n), so the total is O(n log n) regardless.
```

---

## Comparison with Other Sorts

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Tim Sort | O(n) | O(n log n) | O(n log n) | O(n) | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |

---

## Is n log n Optimal?

```
Theorem: Any comparison-based sorting algorithm requires
         Ω(n log n) comparisons in the worst case.

Proof sketch:
  - n! possible permutations of n elements
  - Each comparison eliminates half the possibilities
  - Need log₂(n!) comparisons to distinguish all permutations
  - By Stirling's approximation: log₂(n!) ≈ n log₂(n)
  
  → Lower bound = Ω(n log n)

Merge Sort achieves O(n log n) → IT IS OPTIMAL! ✓
```

### Decision Tree Argument (n = 3)

```
For [a, b, c]:    3! = 6 permutations

                   a < b?
                 /       \
              yes         no
             /               \
          a < c?            a < c?
         /     \           /     \
       yes      no       yes      no
       /         \       /         \
    b < c?      (a,c,b) (b,a,c)   b < c?
    /    \                         /    \
  yes     no                     yes     no
(a,b,c)  (a,c,b)              (b,c,a)  (c,b,a)

Height of tree ≥ ⌈log₂(6)⌉ = 3
→ At least 3 comparisons needed for 3 elements
```

---

## Move Operations

```
Number of data moves in merge sort:

Each merge of n elements: 2n moves
  - n moves to copy to temp array
  - n moves to copy back

Total moves = 2n × log₂(n) = O(n log n)

Note: This is MORE moves than Quick Sort (which uses swaps).
This contributes to merge sort's larger constant factor.
```

---

## Summary Table

| Metric | Value |
|--------|-------|
| Recurrence | T(n) = 2T(n/2) + O(n) |
| Time complexity | Θ(n log n) all cases |
| Space | O(n) auxiliary + O(log n) stack |
| Best case comparisons | ~(n log n)/2 |
| Worst case comparisons | ~n log n |
| Data moves | 2n log n |
| Optimal? | Yes — matches Ω(n log n) lower bound |
| Master Theorem case | Case 2 (balanced) |

---

## Quick Revision Questions

1. **Write the recurrence for Merge Sort and identify the Master Theorem case.**
2. **Why does each level of the recursion tree cost exactly cn?**
3. **Why is Merge Sort's time complexity the same in best, average, and worst cases?**
4. **What is the space complexity breakdown (auxiliary + stack)?**
5. **Prove that comparison-based sorting requires Ω(n log n) comparisons.**
6. **How many data moves does Merge Sort make in total?**

---

[← Previous: The Merge Process](02-merge-process.md) | [Next: Counting Inversions →](04-counting-inversions.md)
