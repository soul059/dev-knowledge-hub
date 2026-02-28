# Chapter 4: Median of Medians (Guaranteed O(n) Selection)

## Overview

The **Median of Medians** algorithm (also called BFPRT, after Blum, Floyd, Pratt, Rivest, Tarjan) guarantees **O(n) worst-case** time for selection. It achieves this by choosing a pivot that is guaranteed to be "good enough" — always eliminating at least 30% of elements.

---

## The Problem with Random Pivots

```
Random Quick Select: O(n) EXPECTED, but O(n²) WORST CASE.

Can we find a pivot that GUARANTEES at least a constant
fraction on each side?

Yes! The median of medians gives a pivot that is:
  → Greater than at least 30% of elements
  → Less than at least 30% of elements
  → So at most 70% remain after partition
```

---

## Algorithm: Step by Step

```
function select(arr, k):
    if n ≤ 5:
        sort arr, return arr[k]
    
    // Step 1: Divide into groups of 5
    groups = divide arr into ⌈n/5⌉ groups of 5

    // Step 2: Find median of each group (sort 5 elements)
    medians = []
    for each group:
        sort group (at most 5 elements → O(1))
        medians.append(group[2])       // middle element
    
    // Step 3: Recursively find median of medians
    pivot = select(medians, len(medians)/2)
    
    // Step 4: Partition around this pivot
    pivotIndex = partition(arr, pivot)
    
    // Step 5: Recurse on appropriate side
    if k == pivotIndex:
        return pivot
    else if k < pivotIndex:
        return select(arr[0..pivotIndex-1], k)
    else:
        return select(arr[pivotIndex+1..n-1], k - pivotIndex - 1)
```

---

## Visual: Groups of 5

```
Array: [25, 24, 33, 39, 3, 18, 19, 31, 23, 28, 
         7, 36, 21, 9, 4, 13, 41, 35, 16, 12]

Step 1: Divide into groups of 5
  G1: [25, 24, 33, 39, 3]
  G2: [18, 19, 31, 23, 28]
  G3: [7, 36, 21, 9, 4]
  G4: [13, 41, 35, 16, 12]

Step 2: Sort each group, find median
  G1: [3, 24, 25, 33, 39]   → median = 25
  G2: [18, 19, 23, 28, 31]  → median = 23
  G3: [4, 7, 9, 21, 36]     → median = 9
  G4: [12, 13, 16, 35, 41]  → median = 16

Step 3: Medians = [25, 23, 9, 16]
  Median of medians = select([25, 23, 9, 16], 2)
  Sort: [9, 16, 23, 25] → median = 16

Step 4: Use 16 as pivot for partitioning
```

---

## Why Does the Pivot Guarantee Work?

```
After finding the median of medians (call it M):

Step 1: Half (⌈n/10⌉) of the group medians are ≤ M
Step 2: For each such median, 3 out of 5 elements are ≤ M
        (the median and the two below it)

    ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
    │  ○  │ │  ○  │ │  ○  │ │  ○  │    ← groups
    │  ○  │ │  ○  │ │  ○  │ │  ○  │
    │  M₁ │ │  M₂ │ │  M₃ │ │  M₄ │    ← medians
    │  ●  │ │  ●  │ │  ○  │ │  ○  │
    │  ●  │ │  ●  │ │  ○  │ │  ○  │
    └─────┘ └─────┘ └─────┘ └─────┘
    
    If M₁ and M₂ ≤ median-of-medians:
    ● = guaranteed ≤ median-of-medians
    
    At least 3 × ⌈n/10⌉ elements are ≤ M
    At least 3 × ⌈n/10⌉ elements are ≥ M

So at least 3n/10 elements on each side.
At most 7n/10 elements on the larger side.

Guaranteed: max partition size ≤ 7n/10

This is a CONSTANT FRACTION reduction!
```

---

## Recurrence and Proof of O(n)

```
T(n) = T(n/5) + T(7n/10) + O(n)

Where:
  T(n/5):   finding median of the ⌈n/5⌉ medians
  T(7n/10): recursing on the larger partition (at most 7n/10)
  O(n):     partitioning + finding group medians

KEY: 1/5 + 7/10 = 2/10 + 7/10 = 9/10 < 1

Proof that T(n) = O(n):
  Assume T(n) ≤ cn for some constant c.
  
  T(n) ≤ c(n/5) + c(7n/10) + an
       = cn/5 + 7cn/10 + an
       = cn(1/5 + 7/10) + an
       = cn(9/10) + an
       = n(9c/10 + a)
       ≤ cn    when c ≥ 10a

  So T(n) ≤ cn = O(n). ✓

The algorithm is O(n) WORST CASE!
```

---

## Why Groups of 5?

```
GROUP SIZE  │  n/g + 7n/10  │  Sum < 1?  │ Result
────────────┼───────────────┼────────────┼──────────
    3       │  n/3 + 2n/3   │  3/3 = 1   │  ✗ NOT < 1
    5       │  n/5 + 7n/10  │  9/10      │  ✓ Works!
    7       │  n/7 + 5n/7   │  6/7 ≈ 0.86│  ✓ Works!

Why not 3?
  With groups of 3: T(n) = T(n/3) + T(2n/3) + O(n)
  1/3 + 2/3 = 1 → does NOT converge to O(n)
  → T(n) = O(n log n)

Why not 7?
  Groups of 7 work (6/7 < 1), but the constant is larger.
  Sorting groups of 7 is more expensive than groups of 5.

5 is the SMALLEST group size that works and is easy to sort.
```

---

## Practical Considerations

```
┌────────────────────────────────────────────────────────────┐
│  THEORY vs PRACTICE                                        │
│                                                            │
│  Median of Medians is O(n) worst case, but:                │
│    → Constant factor is ~20-40× larger than Quick Select   │
│    → Sorting groups + recursive median is expensive        │
│    → Random Quick Select is O(n) expected and MUCH faster  │
│                                                            │
│  In practice:                                              │
│    → Use Randomized Quick Select (simple, fast)            │
│    → Use Median of Medians only when GUARANTEE is needed   │
│    → Use IntroSelect: Quick Select + fallback to MoM       │
│                                                            │
│  C++ std::nth_element uses IntroSelect (since C++11):      │
│    First try random Quick Select                           │
│    If recursion goes too deep → switch to Median of Medians│
└────────────────────────────────────────────────────────────┘
```

---

## Complete Python Implementation

```python
def median_of_medians_select(arr, k):
    """Find k-th smallest element (0-indexed). Guaranteed O(n)."""
    if len(arr) <= 5:
        return sorted(arr)[k]
    
    # Step 1: Divide into groups of 5
    groups = [arr[i:i+5] for i in range(0, len(arr), 5)]
    
    # Step 2: Find median of each group
    medians = [sorted(g)[len(g)//2] for g in groups]
    
    # Step 3: Recursively find median of medians
    pivot = median_of_medians_select(medians, len(medians)//2)
    
    # Step 4: Three-way partition
    low  = [x for x in arr if x < pivot]
    eq   = [x for x in arr if x == pivot]
    high = [x for x in arr if x > pivot]
    
    # Step 5: Recurse on appropriate part
    if k < len(low):
        return median_of_medians_select(low, k)
    elif k < len(low) + len(eq):
        return pivot
    else:
        return median_of_medians_select(high, k - len(low) - len(eq))
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Guarantee | O(n) worst case |
| Group size | 5 (smallest that works) |
| Fraction eliminated | At least 3n/10 per step |
| Recurrence | T(n) = T(n/5) + T(7n/10) + O(n) |
| Key insight | 1/5 + 7/10 = 9/10 < 1 |
| Practical speed | ~20-40× slower than randomized |
| Used in | C++ std::nth_element (as fallback) |

---

## Quick Revision Questions

1. **What fraction of elements is guaranteed to be eliminated by the median-of-medians pivot?**
2. **Write the recurrence and explain why it solves to O(n).**
3. **Why do groups of 3 not work but groups of 5 do?**
4. **Why is median of medians rarely used directly in practice?**
5. **What is IntroSelect?**

---

[← Previous: Average Case Analysis](03-average-case-analysis.md) | [Next: Applications of Selection →](05-applications.md)
