# Chapter 1: Generate All Subsets (Power Set)

## Overview
Generating all subsets of a set is one of the **foundational backtracking problems**. Every subset problem uses the **include/exclude** decision pattern, producing exactly $2^n$ subsets for a set of size $n$.

---

## The Include/Exclude Pattern

```
For each element, make a BINARY DECISION:
  → Include it in the current subset, OR
  → Exclude it from the current subset

This creates a binary state space tree of depth n.

       Decision for element 1
              / \
         Include  Exclude
            |        |
       Decision for element 2
              / \
         Include  Exclude
            ...
         Until all elements decided
```

---

## Pseudocode

```
FUNCTION subsets(nums):
    result = []
    backtrack(nums, 0, [], result)
    RETURN result

FUNCTION backtrack(nums, index, current, result):
    IF index == len(nums):          ← All elements decided
        result.add(copy(current))   ← Record this subset
        RETURN
    
    // CHOICE 1: Include nums[index]
    current.add(nums[index])
    backtrack(nums, index + 1, current, result)
    current.removeLast()            ← UNDO
    
    // CHOICE 2: Exclude nums[index]
    backtrack(nums, index + 1, current, result)
```

---

## Complete Trace: subsets({1, 2, 3})

```
backtrack(0, [])
├── Include 1 → backtrack(1, [1])
│   ├── Include 2 → backtrack(2, [1,2])
│   │   ├── Include 3 → backtrack(3, [1,2,3]) → RECORD [1,2,3]
│   │   └── Exclude 3 → backtrack(3, [1,2])   → RECORD [1,2]
│   └── Exclude 2 → backtrack(2, [1])
│       ├── Include 3 → backtrack(3, [1,3])    → RECORD [1,3]
│       └── Exclude 3 → backtrack(3, [1])      → RECORD [1]
└── Exclude 1 → backtrack(1, [])
    ├── Include 2 → backtrack(2, [2])
    │   ├── Include 3 → backtrack(3, [2,3])    → RECORD [2,3]
    │   └── Exclude 3 → backtrack(3, [2])      → RECORD [2]
    └── Exclude 2 → backtrack(2, [])
        ├── Include 3 → backtrack(3, [3])      → RECORD [3]
        └── Exclude 3 → backtrack(3, [])       → RECORD []

Result: [1,2,3], [1,2], [1,3], [1], [2,3], [2], [3], []
Count: 8 = 2³ ✓
```

---

## Alternative: Iterative Approach (Cascading)

```
Start with [[]]
For each element, ADD it to every existing subset:

Start:  [[]]
Add 1:  [[], [1]]
Add 2:  [[], [1], [2], [1,2]]
Add 3:  [[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]]

FUNCTION subsetsIterative(nums):
    result = [[]]
    FOR each num in nums:
        newSubsets = []
        FOR each existing in result:
            newSubsets.add(existing + [num])
        result.addAll(newSubsets)
    RETURN result
```

---

## Alternative: Bit Manipulation

```
Each subset corresponds to a binary number from 0 to 2^n - 1:

For {1, 2, 3}:
  Binary  →  Subset
  000     →  {}
  001     →  {3}
  010     →  {2}
  011     →  {2,3}
  100     →  {1}
  101     →  {1,3}
  110     →  {1,2}
  111     →  {1,2,3}

FUNCTION subsetsBitmask(nums):
    n = len(nums)
    result = []
    FOR mask = 0 TO 2^n - 1:
        subset = []
        FOR i = 0 TO n-1:
            IF mask AND (1 << i) != 0:
                subset.add(nums[i])
        result.add(subset)
    RETURN result
```

---

## Complexity Analysis

```
┌────────────────────────────────────────────┐
│ Time Complexity: O(n × 2^n)               │
│   • 2^n subsets generated                  │
│   • Each subset: O(n) to copy              │
│                                            │
│ Space Complexity: O(n)                     │
│   • Recursion depth: n                     │
│   • Current subset: at most n elements     │
│   • (Output space: O(n × 2^n) if stored)   │
└────────────────────────────────────────────┘
```

---

## Summary Table

| Approach | Time | Space | Pros | Cons |
|----------|------|-------|------|------|
| Backtracking | O(n·2^n) | O(n) stack | Intuitive, extensible | Recursive overhead |
| Cascading | O(n·2^n) | O(n·2^n) | Simple iterative | Stores all subsets |
| Bitmask | O(n·2^n) | O(n) | No recursion | Limited to n≤30ish |

---

## Quick Revision Questions

1. **How many subsets** does a set of size n have?
2. **What is the include/exclude** pattern?
3. **Why do we need** `copy(current)` when recording?
4. **How does bitmask 101** map to a subset of {A, B, C}?
5. **What is the recursion depth** for generating subsets of n elements?
6. **Trace the backtracking** for subsets of {A, B}.

---

[← Previous Unit: Backtracking vs Brute Force](../06-Backtracking-Fundamentals/06-backtracking-vs-brute-force.md) | [Next: Subsets With Duplicates →](02-subsets-with-duplicates.md)

[← Back to README](../README.md)
