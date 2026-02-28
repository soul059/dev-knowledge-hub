# Chapter 1: Generate All Permutations

## Overview
A permutation is an **arrangement of all elements in a specific order**. For $n$ elements, there are exactly $n!$ permutations. This chapter covers the two main approaches: **swap-based** and **used-array** methods.

---

## What Is a Permutation?

```
Set: {1, 2, 3}
Permutations (all orderings):
  [1, 2, 3]    [1, 3, 2]
  [2, 1, 3]    [2, 3, 1]
  [3, 1, 2]    [3, 2, 1]

Count: 3! = 6

Unlike subsets (which choose WHICH elements),
permutations use ALL elements but vary the ORDER.
```

---

## Approach 1: Swap-Based (In-Place)

```
Idea: Fix one element at position i by swapping,
then recursively permute the remaining positions.

FUNCTION permute(arr, start):
    IF start == len(arr):
        PRINT arr               ← Record this permutation
        RETURN
    
    FOR i = start TO len(arr) - 1:
        swap(arr, start, i)     ← CHOOSE: put arr[i] at position start
        permute(arr, start + 1) ← EXPLORE: permute rest
        swap(arr, start, i)     ← UNDO: restore original order
```

### State Space Tree for [1, 2, 3]
```
                        [1, 2, 3]
             swap(0,0) /  swap(0,1) |  swap(0,2) \
           [1,2,3]      [2,1,3]       [3,2,1]
           /     \       /     \       /     \
      [1,2,3] [1,3,2] [2,1,3] [2,3,1] [3,2,1] [3,1,2]
      swap(1,1) swap(1,2) swap(1,1) swap(1,2) swap(1,1) swap(1,2)
        ↓       ↓         ↓       ↓       ↓       ↓
      [1,2,3][1,3,2]  [2,1,3][2,3,1] [3,2,1][3,1,2]

6 permutations = 3! ✓
```

---

## Approach 2: Used-Array (Build from Scratch)

```
Idea: Build the permutation one element at a time.
Use a boolean array to track which elements are used.

FUNCTION permute(nums):
    result = []
    used = [false] × len(nums)
    backtrack(nums, used, [], result)
    RETURN result

FUNCTION backtrack(nums, used, current, result):
    IF len(current) == len(nums):
        result.add(copy(current))
        RETURN
    
    FOR i = 0 TO len(nums) - 1:
        IF used[i]: CONTINUE        ← Skip used elements
        
        used[i] = true              ← MAKE choice
        current.add(nums[i])
        backtrack(nums, used, current, result)
        current.removeLast()         ← UNDO
        used[i] = false
```

---

## Complete Trace: Swap-Based for [1, 2, 3]

```
permute([1,2,3], start=0)
├── swap(0,0): [1,2,3]
│   permute([1,2,3], start=1)
│   ├── swap(1,1): [1,2,3]
│   │   permute([1,2,3], start=2)
│   │   ├── swap(2,2): [1,2,3]
│   │   │   start==3 → RECORD [1,2,3] ✓
│   │   │   swap(2,2): [1,2,3]  ← undo
│   │   swap(1,1): [1,2,3]  ← undo
│   ├── swap(1,2): [1,3,2]
│   │   permute([1,3,2], start=2)
│   │   ├── start==3 → RECORD [1,3,2] ✓
│   │   swap(1,2): [1,2,3]  ← undo
│   swap(0,0): [1,2,3]  ← undo
├── swap(0,1): [2,1,3]
│   permute([2,1,3], start=1)
│   ├── swap(1,1) → RECORD [2,1,3] ✓
│   ├── swap(1,2) → RECORD [2,3,1] ✓
│   swap(0,1): [1,2,3]  ← undo
├── swap(0,2): [3,2,1]
│   permute([3,2,1], start=1)
│   ├── swap(1,1) → RECORD [3,2,1] ✓
│   ├── swap(1,2) → RECORD [3,1,2] ✓
│   swap(0,2): [1,2,3]  ← undo

Result: [1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,2,1], [3,1,2]
```

---

## Comparing the Two Approaches

```
┌──────────────────┬──────────────────┬──────────────────┐
│ Aspect           │ Swap-Based       │ Used-Array       │
├──────────────────┼──────────────────┼──────────────────┤
│ Extra space      │ O(1) (in-place)  │ O(n) (used array)│
│ Preserves order? │ No (swaps)       │ Yes              │
│ Easy to extend?  │ Harder           │ Easier           │
│ Dup handling     │ Tricky           │ Straightforward  │
│ Output order     │ Different order  │ Lexicographic    │
│ Array modified?  │ Yes (temporarily)│ No               │
└──────────────────┴──────────────────┴──────────────────┘

Recommendation:
  Use SWAP for in-place, performance-critical
  Use USED-ARRAY for clarity and when handling duplicates
```

---

## Complexity Analysis

```
┌────────────────────────────────────────────────────┐
│ Permutations: n!                                   │
│                                                    │
│ Time: O(n × n!)                                    │
│   • n! permutations generated                      │
│   • Each permutation: O(n) to record               │
│                                                    │
│ Space: O(n) recursion depth                        │
│   • Swap-based: no extra arrays                    │
│   • Used-array: O(n) for used + current            │
│                                                    │
│ n! grows VERY fast:                                │
│   5! = 120                                         │
│   10! = 3,628,800                                  │
│   15! = 1,307,674,368,000                          │
│   20! ≈ 2.4 × 10^18 (impractical!)                │
└────────────────────────────────────────────────────┘
```

---

## Summary Table

| n | n! | Time (approx) | Feasible? |
|---|-----|---------------|-----------|
| 5 | 120 | Instant | Yes |
| 8 | 40,320 | Fast | Yes |
| 10 | 3.6M | ~1 sec | Borderline |
| 12 | 479M | ~minutes | Slow |
| 15 | 1.3T | Hours | Impractical |

---

## Quick Revision Questions

1. **How many permutations** does a set of n elements have?
2. **Why does swap-based** use O(1) extra space?
3. **In the used-array approach**, what prevents reusing an element?
4. **Trace the swap-based** algorithm for [A, B, C].
5. **What happens** if you forget the second swap (undo)?
6. **For n=10**, is generating all permutations feasible?

---

[← Previous Unit: Letter Combinations](../07-Subset-and-Combinations/06-letter-combinations-of-phone.md) | [Next: Permutations With Duplicates →](02-permutations-with-duplicates.md)

[← Back to README](../README.md)
