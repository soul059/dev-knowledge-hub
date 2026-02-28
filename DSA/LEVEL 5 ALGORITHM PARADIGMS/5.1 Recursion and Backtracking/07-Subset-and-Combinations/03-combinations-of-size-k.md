# Chapter 3: Combinations of Size K

## Overview
Generating all combinations of exactly $k$ elements from $n$ elements is a **constrained subset problem**. The count is $C(n,k) = \binom{n}{k}$, and the key optimization is **pruning branches that can't reach size k**.

---

## The Pattern

```
Like subsets, but with TWO constraints:
  1. Must include exactly k elements
  2. Order doesn't matter → use "start index" to avoid duplicates

State space tree is a SUBSET TREE with pruning:
  • Prune if: remaining elements can't fill k slots
  • Prune if: current subset already has k elements
```

---

## Pseudocode

```
FUNCTION combine(n, k):
    result = []
    backtrack(n, k, 1, [], result)
    RETURN result

FUNCTION backtrack(n, k, start, current, result):
    IF len(current) == k:           ← Found a valid combination
        result.add(copy(current))
        RETURN
    
    // Pruning: need (k - len(current)) more elements
    // Available: n - start + 1 elements
    // If available < needed → impossible → prune
    remaining_needed = k - len(current)
    remaining_available = n - start + 1
    IF remaining_available < remaining_needed:
        RETURN                      ← PRUNE!
    
    FOR i = start TO n:
        current.add(i)
        backtrack(n, k, i + 1, current, result)
        current.removeLast()
```

---

## Complete Trace: C(4, 2) — Choose 2 from {1,2,3,4}

```
backtrack(start=1, current=[])
├── i=1: add 1 → backtrack(start=2, [1])
│   ├── i=2: add 2 → backtrack(start=3, [1,2])
│   │   → len==k==2 → RECORD [1,2] ✓
│   ├── i=3: add 3 → backtrack(start=4, [1,3])
│   │   → len==k==2 → RECORD [1,3] ✓
│   └── i=4: add 4 → backtrack(start=5, [1,4])
│       → len==k==2 → RECORD [1,4] ✓
├── i=2: add 2 → backtrack(start=3, [2])
│   ├── i=3: add 3 → backtrack(start=4, [2,3])
│   │   → RECORD [2,3] ✓
│   └── i=4: add 4 → backtrack(start=5, [2,4])
│       → RECORD [2,4] ✓
├── i=3: add 3 → backtrack(start=4, [3])
│   └── i=4: add 4 → backtrack(start=5, [3,4])
│       → RECORD [3,4] ✓
└── i=4: add 4 → backtrack(start=5, [4])
    → need 1 more, available 0 → PRUNED ✗

Result: [1,2], [1,3], [1,4], [2,3], [2,4], [3,4]
Count: 6 = C(4,2) ✓
```

---

## The Pruning Optimization

```
Without pruning:
  Start index ensures no duplicates
  But we still explore branches that can't reach size k

With pruning (upper bound optimization):
  Instead of: FOR i = start TO n
  Use:        FOR i = start TO n - (k - len(current)) + 1

  This means: don't start a branch if there aren't
  enough remaining elements to fill the combination

Example: C(5, 3), current = [1]
  Need 2 more elements
  Without pruning: try i = 2, 3, 4, 5
  With pruning:    try i = 2, 3, 4 only
                   (i=5: only 1 element left, need 2 → skip)

  FOR i = start TO n - remaining_needed + 1
```

---

## Visualization of Pruning Effect

```
C(6, 4): Choose 4 from {1,2,3,4,5,6}

WITHOUT PRUNING:               WITH PRUNING:
Start with each i:             Loop bound: 6 - (4-0) + 1 = 3
i=1 ✓ (5 remaining)           i=1 ✓
i=2 ✓ (4 remaining)           i=2 ✓
i=3 ✓ (3 remaining)           i=3 ✓
i=4 ✗ (2 remaining < 3 need)  i=4 → NOT tried (pruned by bound)
i=5 ✗ (1 remaining < 3 need)  i=5 → NOT tried
i=6 ✗ (0 remaining < 3 need)  i=6 → NOT tried

Pruning eliminates 3 branches at ROOT level alone!
Compounds deeper → massive savings
```

---

## General Combinations from Array

```
Given an array (not just 1..n):

FUNCTION combineArray(arr, k):
    result = []
    backtrack(arr, k, 0, [], result)
    RETURN result

FUNCTION backtrack(arr, k, start, current, result):
    IF len(current) == k:
        result.add(copy(current))
        RETURN
    
    FOR i = start TO len(arr) - (k - len(current)):
        current.add(arr[i])
        backtrack(arr, k, i + 1, current, result)
        current.removeLast()
```

---

## Complexity Analysis

```
┌────────────────────────────────────────────────────────┐
│ Number of combinations: C(n, k) = n! / (k!(n-k)!)     │
│                                                        │
│ Time Complexity: O(k × C(n,k))                         │
│   • C(n,k) combinations generated                      │
│   • Each takes O(k) to copy                            │
│                                                        │
│ Space Complexity: O(k) for recursion + current          │
│                                                        │
│ With pruning, nodes visited ≈ O(C(n,k)) — optimal!     │
└────────────────────────────────────────────────────────┘

C(n,k) values for reference:
┌──────┬────┬────┬─────┬─────┬──────┐
│ n\k  │  1 │  2 │  3  │  4  │  5   │
├──────┼────┼────┼─────┼─────┼──────┤
│  5   │  5 │ 10 │  10 │   5 │   1  │
│ 10   │ 10 │ 45 │ 120 │ 210 │ 252  │
│ 20   │ 20 │190 │1140 │4845 │15504 │
└──────┴────┴────┴─────┴─────┴──────┘
```

---

## Summary Table

| Aspect | Subsets | Combinations of k |
|--------|--------|-------------------|
| Constraint | None | Exactly k elements |
| Count | 2^n | C(n, k) |
| Pruning | None possible | Upper bound on loop |
| Record when | At every leaf | When size == k |

---

## Quick Revision Questions

1. **What is the formula** for C(n, k)?
2. **Why use a start index** instead of trying all elements?
3. **What is the pruning condition** that limits the loop range?
4. **How many combinations** of 3 from {1..7}?
5. **Trace C(5, 3)** showing all combinations found.
6. **What is the relationship** between subsets and combinations?

---

[← Previous: Subsets With Duplicates](02-subsets-with-duplicates.md) | [Next: Combination Sum →](04-combination-sum.md)

[← Back to README](../README.md)
