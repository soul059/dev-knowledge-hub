# Chapter 5: Combination Sum II

## Overview
Combination Sum II is a variant where **each candidate can be used at most once** and the input may contain **duplicates**. This combines the "no reuse" rule (use `i+1`) with the "skip duplicates" technique from Subsets II.

---

## Problem Statement

```
Given: candidates = [10, 1, 2, 7, 6, 1, 5], target = 8
Find all unique combinations that sum to 8.
Each number may be used AT MOST ONCE.

Output: [[1,1,6], [1,2,5], [1,7], [2,6]]
```

---

## Key Differences from Combination Sum I

```
┌─────────────────────┬──────────────────┬──────────────────┐
│ Aspect              │ Combination Sum I│ Combination Sum II│
├─────────────────────┼──────────────────┼──────────────────┤
│ Reuse elements?     │ YES (unlimited)  │ NO (at most once)│
│ Duplicates in input?│ NO               │ YES              │
│ Recursive start     │ i (same)         │ i + 1 (next)     │
│ Duplicate handling  │ Not needed       │ Sort + skip      │
└─────────────────────┴──────────────────┴──────────────────┘
```

---

## Pseudocode

```
FUNCTION combinationSum2(candidates, target):
    SORT(candidates)            ← Essential for duplicate handling
    result = []
    backtrack(candidates, target, 0, [], result)
    RETURN result

FUNCTION backtrack(candidates, remaining, start, current, result):
    IF remaining == 0:
        result.add(copy(current))
        RETURN
    
    FOR i = start TO len(candidates) - 1:
        // PRUNE: too large
        IF candidates[i] > remaining:
            BREAK
        
        // SKIP DUPLICATES at same level
        IF i > start AND candidates[i] == candidates[i - 1]:
            CONTINUE
        
        current.add(candidates[i])
        backtrack(candidates, remaining - candidates[i],
                  i + 1,            ← i+1: no reuse!
                  current, result)
        current.removeLast()
```

---

## The Duplicate Skipping Logic

```
Why: i > start AND candidates[i] == candidates[i-1] → SKIP?

Sorted: [1, 1, 2, 5, 6, 7, 10]

At start=0, choosing among [1, 1, 2, 5, 6, 7, 10]:
  i=0: pick first 1  ✓ (first occurrence)
  i=1: second 1, BUT i>start AND candidates[1]==candidates[0]
       → SKIP! (would create duplicate combinations)
  i=2: pick 2  ✓
  ...

WHY this works:
  If we pick the first 1, the subtree includes all combos
  using "at least one 1" — including [1,1,6].
  
  If we ALSO pick the second 1 here, we'd explore
  the SAME combos again (duplicate subtrees).
  
  The first 1's subtree ALREADY covers using both 1s together!
```

---

## Complete Trace: candidates=[1,1,2,5,6,7,10], target=8

```
Sorted: [1, 1, 2, 5, 6, 7, 10]

backtrack(rem=8, start=0, [])
├── i=0 (pick 1): backtrack(rem=7, start=1, [1])
│   ├── i=1 (pick 1): backtrack(rem=6, start=2, [1,1])
│   │   ├── i=2 (pick 2): backtrack(rem=4, start=3, [1,1,2])
│   │   │   ├── i=3 (pick 5): rem=-1 → BREAK ✗
│   │   │   └── (pruned)
│   │   ├── i=3 (pick 5): backtrack(rem=1, start=4, [1,1,5])
│   │   │   └── all > 1 → BREAK ✗
│   │   ├── i=4 (pick 6): backtrack(rem=0, [1,1,6]) → RECORD ✓
│   │   ├── i=5 (pick 7): rem=-1 → BREAK ✗
│   │   └── (done)
│   ├── i=2 (pick 2): backtrack(rem=5, start=3, [1,2])
│   │   ├── i=3 (pick 5): backtrack(rem=0, [1,2,5]) → RECORD ✓
│   │   └── i=4 (pick 6): BREAK ✗
│   ├── i=4 (pick 6): backtrack(rem=1) → pruned ✗
│   ├── i=5 (pick 7): backtrack(rem=0, [1,7]) → RECORD ✓
│   └── i=6 (pick 10): BREAK ✗
├── i=1: SKIP (duplicate of i=0)
├── i=2 (pick 2): backtrack(rem=6, start=3, [2])
│   ├── i=3 (pick 5): backtrack(rem=1) → pruned ✗
│   ├── i=4 (pick 6): backtrack(rem=0, [2,6]) → RECORD ✓
│   └── i=5 (pick 7): BREAK ✗
├── i=3 (pick 5): backtrack(rem=3) → all remaining too large ✗
├── i=4 (pick 6): backtrack(rem=2) → pruned ✗
├── i=5 (pick 7): backtrack(rem=1) → pruned ✗
└── i=6 (pick 10): BREAK ✗

Result: [[1,1,6], [1,2,5], [1,7], [2,6]]  ✓
```

---

## Comparison of All Combination Problems

```
┌──────────────────────┬────────┬───────────┬──────────┬───────────┐
│ Problem              │ Reuse? │ Dup Input?│ Start    │ Skip Dup? │
├──────────────────────┼────────┼───────────┼──────────┼───────────┤
│ Subsets              │ No     │ No        │ i+1      │ No        │
│ Subsets II           │ No     │ Yes       │ i+1      │ Yes       │
│ Combinations C(n,k)  │ No     │ No        │ i+1      │ No        │
│ Combination Sum I    │ Yes    │ No        │ i (same) │ No        │
│ Combination Sum II   │ No     │ Yes       │ i+1      │ Yes       │
└──────────────────────┴────────┴───────────┴──────────┴───────────┘
```

---

## Complexity Analysis

```
Worst case: O(2^n) — similar to subsets
With pruning (sort + break): typically much better

Time: O(2^n) worst case
Space: O(n) for recursion stack + current combination
```

---

## Summary Table

| Technique | Purpose | Implementation |
|-----------|---------|---------------|
| Sort | Enable duplicate detection + early break | Sort array first |
| i > start check | Only skip at same decision level | `i > start` |
| Same value check | Detect duplicate element | `nums[i] == nums[i-1]` |
| i+1 recursion | Prevent element reuse | `start = i + 1` |
| Break on too large | Prune impossible branches | `if nums[i] > remaining: break` |

---

## Quick Revision Questions

1. **What two techniques** does Combination Sum II combine?
2. **Why use i+1** in the recursive call but i in Combination Sum I?
3. **What does `i > start`** ensure in the skip condition?
4. **If input is [1,1,1,2]** and target is 3, what are the valid combinations?
5. **Why must we sort** before applying the skip logic?
6. **Compare Combination Sum I and II** — when would you use each?

---

[← Previous: Combination Sum](04-combination-sum.md) | [Next: Letter Combinations of Phone →](06-letter-combinations-of-phone.md)

[← Back to README](../README.md)
