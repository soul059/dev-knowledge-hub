# Chapter 2: Permutations With Duplicates

## Overview
When the input contains **duplicate elements**, naive permutation generation produces duplicate permutations. The solution uses a **frequency map** or **sort + skip** technique to generate only unique permutations.

---

## The Problem

```
Input: [1, 1, 2]

NAIVE (all 3! = 6, includes duplicates):
  [1,1,2] [1,2,1] [1,1,2] [1,2,1] [2,1,1] [2,1,1]
   ↑                ↑               ↑
   duplicate of      duplicate of     duplicate of
   3rd result        4th result       6th result

CORRECT (only 3 unique):
  [1,1,2]  [1,2,1]  [2,1,1]
  Count: 3!/2! = 3  (divide by duplicates of 1)
```

---

## Formula: Unique Permutations

```
For n elements with duplicates:

Unique permutations = n! / (c₁! × c₂! × ... × cₖ!)

Where cᵢ = count of the i-th distinct element

Example: [1, 1, 2]
  n = 3, c₁ = 2 (two 1s), c₂ = 1 (one 2)
  Unique = 3! / (2! × 1!) = 6/2 = 3

Example: [1, 1, 2, 2]
  n = 4, c₁ = 2, c₂ = 2
  Unique = 4! / (2! × 2!) = 24/4 = 6
```

---

## Approach 1: Sort + Skip (Used-Array)

```
FUNCTION permuteUnique(nums):
    SORT(nums)                         ← Essential!
    result = []
    used = [false] × len(nums)
    backtrack(nums, used, [], result)
    RETURN result

FUNCTION backtrack(nums, used, current, result):
    IF len(current) == len(nums):
        result.add(copy(current))
        RETURN
    
    FOR i = 0 TO len(nums) - 1:
        IF used[i]: CONTINUE
        
        // Skip duplicate: same value as previous, and previous not used
        IF i > 0 AND nums[i] == nums[i-1] AND NOT used[i-1]:
            CONTINUE                   ← SKIP DUPLICATE!
        
        used[i] = true
        current.add(nums[i])
        backtrack(nums, used, current, result)
        current.removeLast()
        used[i] = false
```

---

## Why `NOT used[i-1]`?

```
This is the trickiest part. Consider [1₁, 1₂, 2]:

RULE: Among identical elements, always use them in ORDER.
      Use 1₁ before 1₂.

If used[i-1] is FALSE and nums[i]==nums[i-1]:
  → We're trying to use 1₂ without using 1₁ first
  → This would create a duplicate (same as if we used 1₁)
  → SKIP!

If used[i-1] is TRUE:
  → 1₁ is already in the permutation
  → Now we CAN use 1₂ (they're both in order)
  → PROCEED ✓

Example with [1₁, 1₂, 2]:
  1₁ first, then 1₂ → [1₁,1₂,2] ✓ allowed
  1₂ first (1₁ unused) → SKIP! (would duplicate above)
```

---

## Approach 2: Frequency Map (No Sorting Needed)

```
FUNCTION permuteUnique(nums):
    freq = buildFrequencyMap(nums)     ← {value: count}
    result = []
    backtrack(freq, len(nums), [], result)
    RETURN result

FUNCTION backtrack(freq, remaining, current, result):
    IF remaining == 0:
        result.add(copy(current))
        RETURN
    
    FOR each (value, count) in freq:
        IF count == 0: CONTINUE
        
        current.add(value)
        freq[value] -= 1               ← MAKE (decrement count)
        backtrack(freq, remaining - 1, current, result)
        current.removeLast()
        freq[value] += 1               ← UNDO (restore count)
```

---

## Complete Trace: [1, 1, 2] using Frequency Map

```
freq = {1: 2, 2: 1}

backtrack(remaining=3, [])
├── value=1 (count=2→1): backtrack(rem=2, [1])
│   ├── value=1 (count=1→0): backtrack(rem=1, [1,1])
│   │   ├── value=1: count=0 → SKIP
│   │   └── value=2 (count=1→0): backtrack(rem=0, [1,1,2])
│   │       → RECORD [1,1,2] ✓
│   └── value=2 (count=1→0): backtrack(rem=1, [1,2])
│       ├── value=1 (count=1→0): backtrack(rem=0, [1,2,1])
│       │   → RECORD [1,2,1] ✓
│       └── value=2: count=0 → SKIP
└── value=2 (count=1→0): backtrack(rem=2, [2])
    ├── value=1 (count=2→1): backtrack(rem=1, [2,1])
    │   ├── value=1 (count=1→0): backtrack(rem=0, [2,1,1])
    │   │   → RECORD [2,1,1] ✓
    │   └── value=2: count=0 → SKIP
    └── value=2: count=0 → SKIP

Result: [1,1,2], [1,2,1], [2,1,1]
Count: 3 = 3!/2! ✓
```

---

## Comparing Approaches

```
┌──────────────────┬─────────────────┬─────────────────┐
│ Aspect           │ Sort + Skip     │ Frequency Map   │
├──────────────────┼─────────────────┼─────────────────┤
│ Pre-processing   │ Sort O(n log n) │ Build map O(n)  │
│ Extra space      │ O(n) used array │ O(k) map (k≤n)  │
│ Simplicity       │ Trickier logic  │ More intuitive  │
│ Skip logic       │ i>0 && ...      │ count==0        │
│ Input modified?  │ Yes (sorted)    │ No              │
│ Complexity       │ Same            │ Same            │
└──────────────────┴─────────────────┴─────────────────┘
```

---

## Complexity Analysis

```
┌────────────────────────────────────────────────────────┐
│ Let P = number of unique permutations                  │
│ P = n! / (c₁! × c₂! × ... × cₖ!)                     │
│                                                        │
│ Time: O(n × P)                                         │
│   P permutations, each takes O(n) to record            │
│                                                        │
│ Space: O(n) for recursion + current permutation        │
│                                                        │
│ Note: P can be MUCH less than n!                       │
│   [1,1,1,1]: P = 4!/4! = 1                            │
│   [1,1,2,2]: P = 4!/2!2! = 6                          │
│   [1,2,3,4]: P = 4!/1!1!1!1! = 24 = 4!                │
└────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Input | Total perms (n!) | Unique perms | Reduction |
|-------|-----------------|--------------|-----------|
| [1,2,3] | 6 | 6 | None |
| [1,1,2] | 6 | 3 | 50% |
| [1,1,2,2] | 24 | 6 | 75% |
| [1,1,1,2] | 24 | 4 | 83% |
| [1,1,1,1] | 24 | 1 | 96% |

---

## Quick Revision Questions

1. **How many unique permutations** of [A, A, B, B, C]?
2. **Why does `NOT used[i-1]`** prevent duplicates?
3. **How does the frequency map** approach avoid duplicates naturally?
4. **Which approach** would you use for [1,1,1,...,1,2]?
5. **Trace the sort+skip approach** for [1, 2, 1].
6. **What is the skip condition** in each approach?

---

[← Previous: Generate Permutations](01-generate-permutations.md) | [Next: Next Permutation →](03-next-permutation.md)

[← Back to README](../README.md)
