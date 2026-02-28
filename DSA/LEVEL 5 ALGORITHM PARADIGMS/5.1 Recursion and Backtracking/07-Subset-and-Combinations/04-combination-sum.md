# Chapter 4: Combination Sum

## Overview
Combination Sum finds all **unique combinations** of candidates that sum to a target. The twist: **each candidate can be used unlimited times**. This introduces a critical difference in how we recurse — using `i` (not `i+1`) as the start index.

---

## Problem Statement

```
Given: candidates = [2, 3, 6, 7], target = 7
Find all unique combinations where chosen numbers sum to 7.
Each number may be used UNLIMITED times.

Output: [[2,2,3], [7]]
Explanation:
  2+2+3 = 7 ✓
  7 = 7     ✓
```

---

## Key Insight: Unlimited Reuse

```
SUBSETS/COMBINATIONS:          COMBINATION SUM:
  Start next call at i+1         Start next call at i (same!)
  Each element used AT MOST once Each element used UNLIMITED times
  
  backtrack(i + 1, ...)          backtrack(i, ...)  ← KEY DIFFERENCE!
```

---

## Pseudocode

```
FUNCTION combinationSum(candidates, target):
    result = []
    backtrack(candidates, target, 0, [], result)
    RETURN result

FUNCTION backtrack(candidates, remaining, start, current, result):
    IF remaining == 0:              ← Found valid combination!
        result.add(copy(current))
        RETURN
    IF remaining < 0:               ← Overshot! Prune.
        RETURN
    
    FOR i = start TO len(candidates) - 1:
        current.add(candidates[i])
        backtrack(candidates, remaining - candidates[i],
                  i,                ← NOT i+1! Can reuse same element
                  current, result)
        current.removeLast()
```

---

## Complete Trace: candidates=[2,3,6,7], target=7

```
backtrack(remaining=7, start=0, current=[])
├── i=0 (pick 2): backtrack(rem=5, start=0, [2])
│   ├── i=0 (pick 2): backtrack(rem=3, start=0, [2,2])
│   │   ├── i=0 (pick 2): backtrack(rem=1, start=0, [2,2,2])
│   │   │   ├── i=0 (pick 2): backtrack(rem=-1) → PRUNE ✗
│   │   │   ├── i=1 (pick 3): backtrack(rem=-2) → PRUNE ✗
│   │   │   └── ... all pruned
│   │   ├── i=1 (pick 3): backtrack(rem=0, [2,2,3]) → RECORD ✓
│   │   ├── i=2 (pick 6): backtrack(rem=-3) → PRUNE ✗
│   │   └── i=3 (pick 7): backtrack(rem=-4) → PRUNE ✗
│   ├── i=1 (pick 3): backtrack(rem=2, start=1, [2,3])
│   │   ├── i=1 (pick 3): backtrack(rem=-1) → PRUNE ✗
│   │   └── ... all pruned
│   ├── i=2 (pick 6): backtrack(rem=-1) → PRUNE ✗
│   └── i=3 (pick 7): backtrack(rem=-2) → PRUNE ✗
├── i=1 (pick 3): backtrack(rem=4, start=1, [3])
│   ├── i=1 (pick 3): backtrack(rem=1, start=1, [3,3])
│   │   └── ... all pruned
│   ├── i=2 (pick 6): PRUNE ✗
│   └── i=3 (pick 7): PRUNE ✗
├── i=2 (pick 6): backtrack(rem=1, start=2, [6])
│   └── ... all pruned
└── i=3 (pick 7): backtrack(rem=0, [7]) → RECORD ✓

Result: [[2,2,3], [7]]  ✓
```

---

## Optimization: Sort + Early Break

```
OPTIMIZATION: Sort candidates, then BREAK when candidate > remaining

FUNCTION backtrack(candidates, remaining, start, current, result):
    IF remaining == 0:
        result.add(copy(current))
        RETURN
    
    FOR i = start TO len(candidates) - 1:
        IF candidates[i] > remaining:
            BREAK                   ← All subsequent are larger too!
        current.add(candidates[i])
        backtrack(candidates, remaining - candidates[i], i, current, result)
        current.removeLast()

This works because array is sorted:
  candidates = [2, 3, 6, 7], remaining = 4
  i=0: 2 ≤ 4 → try
  i=1: 3 ≤ 4 → try
  i=2: 6 > 4 → BREAK! (don't try 6 or 7)
```

---

## Avoiding Duplicate Combinations

```
WHY use start index?

Without start (allows duplicates):
  [2,3,2] and [3,2,2] and [2,2,3] would all appear
  These are the SAME combination!

With start:
  We always pick in non-decreasing order
  [2,2,3] → valid (order maintained)
  [2,3,2] → impossible (would need to go back to i=0)
  [3,2,2] → impossible (2 < 3, can't go back)

This ensures each unique combination appears EXACTLY ONCE.
```

---

## Complexity Analysis

```
Let T = target, n = number of candidates, m = min(candidates)

┌────────────────────────────────────────────────────────┐
│ Worst-case tree depth: T/m (use smallest coin T/m times│)
│ Branching factor: up to n at each level                │
│                                                        │
│ Upper bound: O(n^(T/m))                                │
│ Actual: much less due to pruning                       │
│                                                        │
│ Time: O(n^(T/m)) worst case                            │
│ Space: O(T/m) for recursion depth                      │
└────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Combinations of k | Combination Sum |
|--------|-------------------|----------------|
| Constraint | Exactly k elements | Sum equals target |
| Reuse | No (i+1) | Yes (i) |
| Stop condition | size == k | remaining == 0 |
| Prune condition | Not enough elements | remaining < 0 |
| Sort benefit | Upper bound pruning | Early break |

---

## Quick Revision Questions

1. **Why do we pass `i` instead of `i+1`** in the recursive call?
2. **How does the start index** prevent duplicate combinations?
3. **What is the pruning condition** that cuts branches?
4. **Why does sorting** enable the early break optimization?
5. **Trace the algorithm** for candidates=[2,3,5], target=8.
6. **What would change** if each candidate could only be used once?

---

[← Previous: Combinations of Size K](03-combinations-of-size-k.md) | [Next: Combination Sum II →](05-combination-sum-ii.md)

[← Back to README](../README.md)
