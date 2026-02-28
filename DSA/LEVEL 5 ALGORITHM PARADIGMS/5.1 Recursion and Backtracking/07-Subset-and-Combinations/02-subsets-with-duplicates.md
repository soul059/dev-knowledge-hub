# Chapter 2: Subsets With Duplicates

## Overview
When the input contains **duplicate elements**, naive subset generation produces **duplicate subsets**. The key technique is to **sort first, then skip duplicates** at the same decision level.

---

## The Problem

```
Input: [1, 2, 2]

NAIVE (wrong): would generate:
  [], [1], [2], [1,2], [2], [1,2], [2,2], [1,2,2]
                  ↑             ↑
               DUPLICATE!    DUPLICATE!

CORRECT:
  [], [1], [2], [1,2], [2,2], [1,2,2]
  6 unique subsets
```

---

## The Strategy: Sort + Skip

```
┌──────────────────────────────────────────────────────┐
│  Step 1: SORT the input array                        │
│          [2, 1, 2] → [1, 2, 2]                       │
│                                                      │
│  Step 2: When EXCLUDING an element, skip ALL          │
│          consecutive duplicates of that element       │
│                                                      │
│  WHY? If we exclude nums[i], we should not then       │
│  consider including nums[i+1] if nums[i+1]==nums[i]  │
│  because that creates the same subset.               │
└──────────────────────────────────────────────────────┘
```

---

## Pseudocode (Approach 1: Include/Exclude with Skip)

```
FUNCTION subsetsWithDup(nums):
    SORT(nums)
    result = []
    backtrack(nums, 0, [], result)
    RETURN result

FUNCTION backtrack(nums, index, current, result):
    IF index == len(nums):
        result.add(copy(current))
        RETURN
    
    // CHOICE 1: Include nums[index]
    current.add(nums[index])
    backtrack(nums, index + 1, current, result)
    current.removeLast()
    
    // CHOICE 2: Exclude nums[index]
    // Skip ALL duplicates of nums[index]
    WHILE index + 1 < len(nums) AND nums[index + 1] == nums[index]:
        index += 1                     ← Skip duplicates!
    backtrack(nums, index + 1, current, result)
```

---

## Pseudocode (Approach 2: Loop-Based with Skip)

```
FUNCTION subsetsWithDup(nums):
    SORT(nums)
    result = []
    backtrack(nums, 0, [], result)
    RETURN result

FUNCTION backtrack(nums, start, current, result):
    result.add(copy(current))          ← Record at every node
    
    FOR i = start TO len(nums) - 1:
        // Skip duplicates at the SAME level
        IF i > start AND nums[i] == nums[i - 1]:
            CONTINUE                   ← Skip!
        
        current.add(nums[i])
        backtrack(nums, i + 1, current, result)
        current.removeLast()
```

---

## Complete Trace: [1, 2, 2]

```
Sorted: [1, 2, 2]  (indices: 0, 1, 2)

Using Approach 2 (loop-based):

backtrack(start=0, current=[])  → RECORD []
├── i=0: add 1 → backtrack(start=1, [1])  → RECORD [1]
│   ├── i=1: add 2 → backtrack(start=2, [1,2])  → RECORD [1,2]
│   │   ├── i=2: add 2 → backtrack(start=3, [1,2,2])  → RECORD [1,2,2]
│   │   └── (done)
│   ├── i=2: nums[2]==nums[1] AND i>start → SKIP!
│   └── (done)
├── i=1: add 2 → backtrack(start=2, [2])  → RECORD [2]
│   ├── i=2: add 2 → backtrack(start=3, [2,2])  → RECORD [2,2]
│   └── (done)
├── i=2: nums[2]==nums[1] AND i>start → SKIP!
└── (done)

Result: [], [1], [1,2], [1,2,2], [2], [2,2]
Count: 6 unique subsets ✓
```

---

## Why Does Skipping Work?

```
Consider nums = [1, 2, 2, 2] (three 2's)

Valid subsets containing 2:
  [2]       → include first 2 only
  [2,2]     → include first two 2s
  [2,2,2]   → include all three 2s
  
The KEY insight:
  If we INCLUDE the first 2, we get to decide about the second.
  If we EXCLUDE the first 2, the second 2 would create the
  SAME subset → so we must SKIP it too!

  "If you exclude an element, you must exclude ALL copies"
  "If you include, you include a PREFIX of the copies"

Allowed patterns for three 2s:
  Include Include Include → [2,2,2]  ✓
  Include Include Exclude → [2,2]    ✓
  Include Exclude (skip)  → [2]      ✓
  Exclude (skip all)      → []       ✓
  
  Exclude Include ?       → SKIP (duplicate of above)
```

---

## Complexity Analysis

```
With k distinct elements, each appearing c_1, c_2, ..., c_k times:

Total unique subsets = (c_1+1) × (c_2+1) × ... × (c_k+1)

Example: [1, 2, 2, 3, 3, 3]
  1 appears 1 time → (1+1) = 2 choices
  2 appears 2 times → (2+1) = 3 choices
  3 appears 3 times → (3+1) = 4 choices
  Total = 2 × 3 × 4 = 24 unique subsets

Time: O(n × number of unique subsets)
Space: O(n) for recursion + current subset
```

---

## Summary Table

| Aspect | Without Duplicates | With Duplicates |
|--------|-------------------|----------------|
| Pre-processing | None | Sort array |
| Subsets count | 2^n | Product of (count+1) |
| Extra logic | None | Skip condition |
| Skip condition | N/A | `i > start && nums[i] == nums[i-1]` |

---

## Quick Revision Questions

1. **Why must we sort** the array before generating subsets with duplicates?
2. **What is the skip condition** in the loop-based approach?
3. **For [1,1,1,2]**, how many unique subsets exist?
4. **Why does "exclude first, include second"** create duplicates?
5. **Trace the algorithm** for input [2, 1, 2] (unsorted).
6. **How would you handle** duplicates without sorting (using a set)?

---

[← Previous: Generate All Subsets](01-generate-all-subsets.md) | [Next: Combinations of Size K →](03-combinations-of-size-k.md)

[← Back to README](../README.md)
