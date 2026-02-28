# Chapter 4: Maximum Subarray — D&C Approach

## Overview

The Maximum Subarray problem asks for the contiguous subarray with the largest sum. While Kadane's algorithm solves it in O(n) using DP, the **D&C approach** provides an elegant O(n log n) solution that beautifully illustrates the paradigm — especially the combine step.

---

## Problem Statement

```
INPUT:  Array of integers (may include negatives)
        nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

OUTPUT: Maximum contiguous subarray sum
        Answer: 6  (subarray [4, -1, 2, 1])

        -2  1  -3 | 4  -1   2   1 | -5  4
                    └──────────────┘
                     sum = 4-1+2+1 = 6
```

---

## D&C Approach: Three Cases

```
Divide array at midpoint. The maximum subarray is in ONE of:

    ┌──────────────────────────────────────────────┐
    │     Left half     │     Right half            │
    │                   │                           │
    │  Case 1: entirely │  Case 2: entirely         │
    │  in left half     │  in right half            │
    │                   │                           │
    │              Case 3: CROSSES the midpoint     │
    │         ◄────────────────────────►            │
    └──────────────────────────────────────────────┘

    Answer = max(Case 1, Case 2, Case 3)

    Case 1: Solve recursively on left half
    Case 2: Solve recursively on right half
    Case 3: Find max crossing subarray (the key step!)
```

---

## Finding Max Crossing Subarray

```
A crossing subarray MUST include:
  - At least one element from the left half (ending at mid)
  - At least one element from the right half (starting at mid+1)

Strategy:
  1. From mid, extend LEFT as far as helps (max left sum)
  2. From mid+1, extend RIGHT as far as helps (max right sum)
  3. Crossing sum = left sum + right sum

  ───────────────────│───────────────────
    ◄── extend left  │  extend right ──►
         from mid    │  from mid+1

This takes O(n) — linear scan in each direction.
```

---

## Complete Algorithm

```
MAX_SUBARRAY(arr, lo, hi):
    // Base case
    if lo == hi:
        return arr[lo]
    
    mid ← (lo + hi) / 2
    
    // Case 1: entirely in left half
    left_max ← MAX_SUBARRAY(arr, lo, mid)
    
    // Case 2: entirely in right half
    right_max ← MAX_SUBARRAY(arr, mid+1, hi)
    
    // Case 3: crossing the midpoint
    cross_max ← MAX_CROSSING(arr, lo, mid, hi)
    
    return max(left_max, right_max, cross_max)

MAX_CROSSING(arr, lo, mid, hi):
    // Extend left from mid
    left_sum ← -∞
    sum ← 0
    for i ← mid down to lo:
        sum ← sum + arr[i]
        left_sum ← max(left_sum, sum)
    
    // Extend right from mid+1
    right_sum ← -∞
    sum ← 0
    for j ← mid+1 to hi:
        sum ← sum + arr[j]
        right_sum ← max(right_sum, sum)
    
    return left_sum + right_sum
```

---

## Detailed Trace

```
arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
idx =  0   1   2  3   4  5  6   7  8

MAX_SUBARRAY(0, 8):  mid = 4
├── MAX_SUBARRAY(0, 4):  mid = 2       ← Left half
│   ├── MAX_SUBARRAY(0, 2):  mid = 1
│   │   ├── MAX_SUBARRAY(0, 1):  mid = 0
│   │   │   ├── MS(0,0): return -2
│   │   │   ├── MS(1,1): return 1
│   │   │   └── CROSS(0,0,1): left=-2, right=1 → -1
│   │   │   return max(-2, 1, -1) = 1
│   │   ├── MS(2,2): return -3
│   │   └── CROSS(0,1,2): 
│   │       left: from idx 1 → sum=1, then idx 0 → sum=-1
│   │             left_sum = 1
│   │       right: from idx 2 → sum=-3
│   │             right_sum = -3
│   │       cross = 1 + (-3) = -2
│   │   return max(1, -3, -2) = 1
│   ├── MAX_SUBARRAY(3, 4):  mid = 3
│   │   ├── MS(3,3): return 4
│   │   ├── MS(4,4): return -1
│   │   └── CROSS(3,3,4): left=4, right=-1 → 3
│   │   return max(4, -1, 3) = 4
│   └── CROSS(0,2,4):
│       left: idx 2→-3, idx 1→-2, idx 0→-4 → left_sum = -2
│       right: idx 3→4, idx 4→3 → right_sum = 4
│       cross = -2 + 4 = 2
│   return max(1, 4, 2) = 4
│
├── MAX_SUBARRAY(5, 8):  mid = 6       ← Right half
│   ├── MAX_SUBARRAY(5, 6):  mid = 5
│   │   ├── MS(5,5): return 2
│   │   ├── MS(6,6): return 1
│   │   └── CROSS(5,5,6): left=2, right=1 → 3
│   │   return max(2, 1, 3) = 3
│   ├── MAX_SUBARRAY(7, 8):  mid = 7
│   │   ├── MS(7,7): return -5
│   │   ├── MS(8,8): return 4
│   │   └── CROSS(7,7,8): left=-5, right=4 → -1
│   │   return max(-5, 4, -1) = 4
│   └── CROSS(5,6,8):
│       left: idx 6→1, idx 5→3 → left_sum = 3
│       right: idx 7→-5, idx 8→-1 → right_sum = -1
│       cross = 3 + (-1) = 2
│   return max(3, 4, 2) = 4
│
└── CROSS(0,4,8):                       ← Crossing
    left: idx 4→-1, idx 3→3, idx 2→0, idx 1→1, idx 0→-1
          best: 3 (at idx 3-4)... actually:
          idx 4: sum=-1
          idx 3: sum=-1+4=3
          idx 2: sum=3-3=0
          idx 1: sum=0+1=1
          idx 0: sum=1-2=-1
          left_sum = 3
    right: idx 5→2, idx 6→3, idx 7→-2, idx 8→2
          idx 5: sum=2
          idx 6: sum=3
          idx 7: sum=-2
          idx 8: sum=2
          right_sum = 3
    cross = 3 + 3 = 6

return max(4, 4, 6) = 6 ✓
```

---

## Complexity Analysis

```
Recurrence:
    T(n) = 2T(n/2) + O(n)
                      ↑
              crossing check is O(n)

Master Theorem: a=2, b=2, f(n)=O(n)
    log_b a = 1, n^1 = n
    f(n) = Θ(n) → Case 2
    T(n) = Θ(n log n)

Space: O(log n) recursion stack
```

---

## Comparison with Kadane's Algorithm

```
KADANE'S ALGORITHM (O(n), DP approach):

    max_ending_here ← 0
    max_so_far ← -∞
    for each x in arr:
        max_ending_here ← max(x, max_ending_here + x)
        max_so_far ← max(max_so_far, max_ending_here)
    return max_so_far

┌─────────────────┬──────────┬────────────────────────┐
│ Algorithm       │ Time     │ When to Use            │
├─────────────────┼──────────┼────────────────────────┤
│ Brute Force     │ O(n³)    │ Never (too slow)       │
│ Prefix Sums     │ O(n²)    │ Small n                │
│ D&C             │ O(n log n)│ Illustrate D&C concept│
│ Kadane's        │ O(n)     │ Always (in practice)   │
└─────────────────┴──────────┴────────────────────────┘

D&C is NOT the fastest, but it:
  - Beautifully demonstrates the D&C paradigm
  - The combine step is instructive
  - Extends to related problems (2D max subarray)
```

---

## Why D&C Is Still Valuable Here

```
1. EDUCATIONAL: Perfect example of all three D&C steps
   - Non-trivial combine step
   - Three-case analysis

2. PARALLELISM: D&C version parallelizes well
   - Left and right halves independent
   - Can use parallel processors

3. EXTENSIONS: D&C adapts to harder variants
   - 2D maximum subarray (max sum rectangle)
   - Maximum subarray with constraints
   
4. INTERVIEW: Understanding the D&C approach
   shows depth of algorithm knowledge
```

---

## Python Implementation

```python
def maxSubArray_DC(nums):
    def helper(lo, hi):
        if lo == hi:
            return nums[lo]
        
        mid = (lo + hi) // 2
        
        # Max crossing subarray
        left_sum = float('-inf')
        s = 0
        for i in range(mid, lo - 1, -1):
            s += nums[i]
            left_sum = max(left_sum, s)
        
        right_sum = float('-inf')
        s = 0
        for i in range(mid + 1, hi + 1):
            s += nums[i]
            right_sum = max(right_sum, s)
        
        cross = left_sum + right_sum
        left_max = helper(lo, mid)
        right_max = helper(mid + 1, hi)
        
        return max(left_max, right_max, cross)
    
    return helper(0, len(nums) - 1)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Find contiguous subarray with max sum |
| D&C idea | Left max, right max, or crossing max |
| Combine step | Extend left + right from midpoint |
| Time | O(n log n) |
| Space | O(log n) recursion |
| Better alternative | Kadane's O(n) |
| D&C value | Educational + parallelizable |

---

## Quick Revision Questions

1. **What are the three cases in the D&C approach?**
2. **How do we find the max crossing subarray?**
3. **What is the recurrence and its solution?**
4. **Why is Kadane's algorithm faster?**
5. **When might D&C be preferred over Kadane's?**

---

[← Previous: Merge K Sorted Lists](03-merge-k-sorted-lists.md) | [Next: Majority Element →](05-majority-element.md)
