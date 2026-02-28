# Chapter 4: Order Statistics via D&C

## Overview

The **kth order statistic** is the kth smallest element in an unsorted array. While Quick Select (Unit 6) gives expected O(n), the **Median of Medians** algorithm guarantees **worst-case O(n)** through a clever D&C pivot selection strategy. This chapter covers the full theory: deterministic selection, weighted medians, and applications in computational geometry and database queries.

---

## Problem Recap

```
Given: Array A of n distinct elements, integer k (1 ≤ k ≤ n)
Find: The kth smallest element

Special cases:
    k = 1           → Minimum
    k = n           → Maximum
    k = ⌈n/2⌉      → Median

Approaches:
    Sort then index:        O(n log n)
    Quick Select (random):  O(n) expected, O(n²) worst
    Median of Medians:      O(n) WORST CASE ← this chapter
```

---

## Median of Medians Algorithm (BFPRT)

```
Named after Blum, Floyd, Pratt, Rivest, Tarjan (1973)

Core idea: Choose a pivot that GUARANTEES a good split.

┌───────────────────────────────────────────────────────┐
│                                                       │
│  STEP 1: Divide n elements into ⌈n/5⌉ groups of 5    │
│                                                       │
│  STEP 2: Find median of each group (O(1) per group)   │
│                                                       │
│  STEP 3: Recursively find median of these medians     │
│          → This is the pivot!                         │
│                                                       │
│  STEP 4: Partition around this pivot                   │
│                                                       │
│  STEP 5: Recurse on the appropriate side              │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

## Why Groups of 5?

```
Array (25 elements), divided into groups of 5:

Group 1: [8, 3, 1, 6, 4]   → sorted: [1, 3, 4, 6, 8]   median = 4
Group 2: [9, 7, 5, 2, 0]   → sorted: [0, 2, 5, 7, 9]   median = 5  
Group 3: [15,11,13,12,14]  → sorted: [11,12,13,14,15]  median = 13
Group 4: [19,16,18,17,20]  → sorted: [16,17,18,19,20]  median = 18
Group 5: [24,21,23,22,25]  → sorted: [21,22,23,24,25]  median = 23

Medians: [4, 5, 13, 18, 23]
Median of medians = 13 ← this is our pivot!

         Elements ≤ pivot (13)     Elements > pivot (13)
        at least 3 per group       at least 3 per group
        for half the groups        for half the groups

Guarantee: At least 3 × ⌈(⌈n/5⌉)/2⌉ - 6 ≈ 3n/10 elements
           on EACH side of pivot

→ The larger partition has at most ~7n/10 elements
→ Guaranteed good split!
```

---

## Pivot Quality Visualization

```
Groups sorted by their medians:

  G1    G2    G3    G4    G5
  ─     ─     ─     ─     ─
  8     9    15    19    25    ← max of group
  6     7    14    20    24
 [4]   [5]  [13]  [18]  [23]  ← medians (sorted)
  3     2    12    17    22            ▲
  1     0    11    16    21            │
                                  MoM = 13

Elements GUARANTEED ≤ 13:
    All elements ≤ median in groups with median ≤ 13
    = 3 elements each from G1, G2, G3 = 9 elements
    
Elements GUARANTEED > 13:
    All elements > median in groups with median > 13
    = 3 elements each from G4, G5 = 6 elements (at least 2 groups)

AT LEAST 30% of elements on each side → at most 70% on larger side
```

---

## Pseudocode

```
SELECT(A, k):
    if len(A) ≤ 5:
        sort A
        return A[k-1]
    
    // Step 1: Divide into groups of 5
    groups ← divide A into ⌈n/5⌉ groups of 5
    
    // Step 2: Find median of each group
    medians ← []
    for each group g in groups:
        sort g  (at most 5 elements → O(1))
        medians.append(g[2])  // middle element
    
    // Step 3: Recursively find median of medians
    pivot ← SELECT(medians, ⌈len(medians)/2⌉)
    
    // Step 4: Partition
    L ← elements < pivot
    E ← elements = pivot
    G ← elements > pivot
    
    // Step 5: Recurse on correct side
    if k ≤ len(L):
        return SELECT(L, k)
    elif k ≤ len(L) + len(E):
        return pivot
    else:
        return SELECT(G, k - len(L) - len(E))
```

---

## Complexity Proof

```
RECURRENCE:
    T(n) = T(n/5) + T(7n/10) + O(n)
           ─────   ────────   ────
           find      worst     partition
           MoM       case      step
                    recurse    

Proof that T(n) = O(n):
    Assume T(n) ≤ cn for some constant c.
    
    T(n) ≤ c(n/5) + c(7n/10) + an
         = cn/5 + 7cn/10 + an
         = cn(1/5 + 7/10) + an
         = cn(2/10 + 7/10) + an
         = cn(9/10) + an
         = n(9c/10 + a)
         ≤ cn    when c ≥ 10a

    Since 9/10 < 1, the recursive costs fit within O(n)!

┌─────────────────────────────────────────────────┐
│                                                 │
│  WHY NOT GROUPS OF 3?                           │
│                                                 │
│  T(n) = T(n/3) + T(2n/3) + O(n)                │
│  1/3 + 2/3 = 1 → T(n) = O(n log n) ✗          │
│                                                 │
│  WHY NOT GROUPS OF 7?                           │
│                                                 │
│  T(n) = T(n/7) + T(5n/7) + O(n)                │
│  1/7 + 5/7 = 6/7 < 1 → T(n) = O(n) ✓          │
│  Works! But groups of 5 is simpler and enough.  │
│                                                 │
│  Groups of 5 is the SMALLEST group size that    │
│  guarantees linear time.                        │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## Step-by-Step Trace

```
Array: [33, 12, 8, 45, 27, 19, 3, 41, 22, 15, 36, 6, 29, 50, 38]
Find: 6th smallest (k=6)

Step 1: Groups of 5
    G1: [33, 12, 8, 45, 27] → sorted [8,12,27,33,45]  → median = 27
    G2: [19, 3, 41, 22, 15] → sorted [3,15,19,22,41]  → median = 19
    G3: [36, 6, 29, 50, 38] → sorted [6,29,36,38,50]  → median = 36

Step 2: Medians = [27, 19, 36]
Step 3: MoM = median of [19, 27, 36] = 27  ← PIVOT

Step 4: Partition around 27
    L = [12, 8, 19, 3, 22, 15, 6] → 7 elements < 27
    E = [27]                       → 1 element = 27
    G = [33, 45, 41, 36, 29, 50, 38] → 7 elements > 27

Step 5: k=6, |L|=7
    k ≤ |L| → 6 ≤ 7 → recurse on L
    SELECT([12, 8, 19, 3, 22, 15, 6], 6)

    G1: [12, 8, 19, 3, 22] → median = 12
    G2: [15, 6]             → median = 6 (or 15)
    Medians = [12, 6 or 15] → MoM = 12  ← PIVOT

    Partition around 12:
        L = [8, 3, 6]    → 3 elements
        E = [12]          → 1 element
        G = [19, 22, 15]  → 3 elements
    
    k=6, |L|=3, |L|+|E|=4
    k > 4 → recurse on G with k' = 6-4 = 2
    SELECT([19, 22, 15], 2)

    |A| ≤ 5 → sort: [15, 19, 22]
    return A[1] = 19

ANSWER: 6th smallest element = 19

Verify: sorted array = [3, 6, 8, 12, 15, 19, 22, 27, 29, 33, 36, 38, 41, 45, 50]
                         1  2  3  4   5   6  ← position 6 = 19 ✓
```

---

## Weighted Median

```
PROBLEM:
    Given n elements with weights w₁, w₂, ..., wₙ (Σwᵢ = 1)
    Find element xₘ such that:
        Σ wᵢ ≤ 1/2  (for xᵢ < xₘ)
        Σ wᵢ ≤ 1/2  (for xᵢ > xₘ)

APPLICATION: Facility location — minimize weighted distance

D&C Solution:
    1. Find median using SELECT → O(n)
    2. Partition and sum weights on each side
    3. Recurse on the heavier side if needed
    
Total: O(n) per level × O(log n) levels? 
No — just O(n) total (like Quick Select analysis)

APPLICATIONS:
    - 1D facility location
    - k-medians clustering
    - Scheduling with weighted jobs
```

---

## Python Implementation

```python
def median_of_medians(arr, k):
    """Find kth smallest element in O(n) worst case."""
    if len(arr) <= 5:
        return sorted(arr)[k - 1]
    
    # Step 1: Divide into groups of 5
    groups = [arr[i:i+5] for i in range(0, len(arr), 5)]
    
    # Step 2: Find medians
    medians = [sorted(g)[len(g) // 2] for g in groups]
    
    # Step 3: Recursively find median of medians
    pivot = median_of_medians(medians, (len(medians) + 1) // 2)
    
    # Step 4: Partition
    low  = [x for x in arr if x < pivot]
    equal = [x for x in arr if x == pivot]
    high = [x for x in arr if x > pivot]
    
    # Step 5: Recurse
    if k <= len(low):
        return median_of_medians(low, k)
    elif k <= len(low) + len(equal):
        return pivot
    else:
        return median_of_medians(high, k - len(low) - len(equal))

# Test
arr = [33, 12, 8, 45, 27, 19, 3, 41, 22, 15, 36, 6, 29, 50, 38]
for k in range(1, len(arr) + 1):
    result = median_of_medians(arr[:], k)
    expected = sorted(arr)[k - 1]
    assert result == expected, f"k={k}: got {result}, expected {expected}"
    print(f"k={k}: {result}")
```

---

## Comparison: Selection Algorithms

```
┌──────────────────────┬──────────────┬──────────────┬───────────┐
│ Algorithm            │ Average      │ Worst Case   │ In-Place  │
├──────────────────────┼──────────────┼──────────────┼───────────┤
│ Sort + Index         │ O(n log n)   │ O(n log n)   │ Yes       │
│ Min-heap k times     │ O(n + k logn)│ O(n + k logn)│ No        │
│ Quick Select         │ O(n)         │ O(n²)        │ Yes       │
│ Quick Select + rand  │ O(n)         │ O(n²)*       │ Yes       │
│ Median of Medians    │ O(n)         │ O(n)         │ Yes       │
│ Introselect          │ O(n)         │ O(n)         │ Yes       │
└──────────────────────┴──────────────┴──────────────┴───────────┘

* Very unlikely worst case with randomization

Introselect: Start with Quick Select, switch to MoM if 
recursion depth exceeds threshold. Used in C++ std::nth_element.
```

---

## Applications in Computational Geometry

```
1. MEDIAN-BASED SPLITTING (kd-trees):
    Alternate splitting dimensions, use median for balanced splits.
    Linear selection → O(n log n) kd-tree construction.

2. HAM-SANDWICH THEOREM:
    Any n red and m blue points in 2D can be simultaneously 
    bisected by a single line. Finding it uses median computations.

3. LINEAR-TIME CLOSEST PAIR:
    Using median to split points guarantees balanced recursion.

4. PRUNING IN GEOMETRIC ALGORITHMS:
    Discard constant fraction of points per round using medians.
    → Problems like smallest enclosing circle benefit.
```

---

## Common Interview Problems

```
LEETCODE CONNECTIONS:
┌──────────────────────────────────────────────────────┐
│                                                      │
│  LC 215: Kth Largest Element in an Array             │
│    → Quick Select or Median of Medians               │
│                                                      │
│  LC 347: Top K Frequent Elements                     │
│    → Selection on frequency array                    │
│                                                      │
│  LC 973: K Closest Points to Origin                  │
│    → Quick Select on distances                       │
│                                                      │
│  LC 4: Median of Two Sorted Arrays                   │
│    → Binary search D&C approach, O(log(m+n))         │
│                                                      │
│  LC 480: Sliding Window Median                       │
│    → Two heaps / sorted set approach                 │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Find kth smallest in unsorted array |
| Key idea | Use median of medians for guaranteed good pivot |
| Why groups of 5 | Smallest size giving constant < 1 in recurrence |
| Recurrence | T(n) = T(n/5) + T(7n/10) + O(n) |
| Complexity | O(n) worst case |
| vs Quick Select | Same average, but O(n) vs O(n²) worst case |
| Practical use | Introselect (C++ std::nth_element) |
| Discoverers | Blum, Floyd, Pratt, Rivest, Tarjan (1973) |

---

## Quick Revision Questions

1. **Why does the Median of Medians guarantee a good pivot?**
2. **What fraction of elements are guaranteed on each side of the MoM pivot?**
3. **Why do groups of 3 fail but groups of 5 (and 7) work?**
4. **Write the recurrence for MoM and prove T(n) = O(n).**
5. **What is Introselect and how does it combine Quick Select and MoM?**
6. **How is linear selection useful in constructing kd-trees?**

---

[← Previous: Convex Hull](03-convex-hull.md) | [Next: Skyline Problem →](05-skyline-problem.md)
