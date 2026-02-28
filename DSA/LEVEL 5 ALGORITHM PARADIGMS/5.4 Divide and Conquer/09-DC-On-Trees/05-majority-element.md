# Chapter 5: Majority Element — D&C Approach

## Overview

The Majority Element problem asks: given an array of n elements, find the element that appears **more than n/2 times** (if it exists). The D&C approach exploits a key insight: if a majority element exists in the whole array, it must be the majority of at least one half.

---

## Problem Statement

```
INPUT:  Array of n elements
        nums = [3, 2, 3, 3, 1, 3, 3]

OUTPUT: The majority element (appears > n/2 times)
        Answer: 3  (appears 5 times, n/2 = 3.5)

GUARANTEE: Majority element always exists (problem assumption)
```

---

## Approach Comparison

```
┌────────────────────────┬──────────┬──────────┬──────────────────┐
│ Approach               │ Time     │ Space    │ Notes            │
├────────────────────────┼──────────┼──────────┼──────────────────┤
│ HashMap counting       │ O(n)     │ O(n)     │ Simple, extra mem│
│ Sorting + middle       │ O(n logn)│ O(1)*    │ Sort, pick n/2   │
│ Boyer-Moore Voting     │ O(n)     │ O(1)     │ Optimal          │
│ D&C                    │ O(n logn)│ O(logn)  │ Elegant D&C      │
│ Randomized             │ O(n)†    │ O(1)     │ Expected time    │
│ Bit manipulation       │ O(n·32)  │ O(1)     │ Check each bit   │
└────────────────────────┴──────────┴──────────┴──────────────────┘
* with in-place sort   † expected, not worst case
```

---

## D&C Key Insight

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  THEOREM: If x is the majority of the entire array,     │
│  then x must be the majority of at least one half.       │
│                                                          │
│  PROOF (by contradiction):                               │
│    Suppose x is NOT majority of left half                │
│    AND x is NOT majority of right half.                  │
│                                                          │
│    In left half (size n/2):  count(x) ≤ n/4             │
│    In right half (size n/2): count(x) ≤ n/4             │
│    Total count(x) ≤ n/4 + n/4 = n/2                     │
│                                                          │
│    But x is majority → count(x) > n/2                   │
│    CONTRADICTION! □                                      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## The Algorithm

```
MAJORITY_ELEMENT(arr, lo, hi):
    // Base case: single element
    if lo == hi:
        return arr[lo]
    
    mid ← (lo + hi) / 2
    
    // CONQUER: find majority of each half
    left_maj  ← MAJORITY_ELEMENT(arr, lo, mid)
    right_maj ← MAJORITY_ELEMENT(arr, mid+1, hi)
    
    // COMBINE: if they agree, that's the answer
    if left_maj == right_maj:
        return left_maj
    
    // Otherwise, count each candidate in the full range
    left_count  ← count(left_maj  in arr[lo..hi])
    right_count ← count(right_maj in arr[lo..hi])
    
    if left_count > (hi - lo + 1) / 2:
        return left_maj
    else:
        return right_maj
```

---

## Detailed Trace

```
arr = [2, 2, 1, 1, 1, 2, 2]
idx =  0  1  2  3  4  5  6

MAJORITY(0, 6): mid=3
├── MAJORITY(0, 3): mid=1
│   ├── MAJORITY(0, 1): mid=0
│   │   ├── MAJORITY(0, 0): return 2
│   │   ├── MAJORITY(1, 1): return 2
│   │   └── left_maj == right_maj == 2 → return 2
│   │
│   ├── MAJORITY(2, 3): mid=2
│   │   ├── MAJORITY(2, 2): return 1
│   │   ├── MAJORITY(3, 3): return 1
│   │   └── left_maj == right_maj == 1 → return 1
│   │
│   └── left_maj=2, right_maj=1 — disagree!
│       count(2 in [0..3]) = 2  (indices 0,1)
│       count(1 in [0..3]) = 2  (indices 2,3)
│       2 > 4/2 = 2? NO.  2 > 2? NO.
│       1 > 2? NO.
│       return 2 (or 1, neither is majority of this half)
│       Note: In this sub-problem, there's no majority.
│       The algorithm still picks one — it'll be verified later.
│       Let's say left_count >= right_count → return 2
│
├── MAJORITY(4, 6): mid=5
│   ├── MAJORITY(4, 5): mid=4
│   │   ├── MAJORITY(4, 4): return 1
│   │   ├── MAJORITY(5, 5): return 2
│   │   └── disagree! count(1 in [4..5])=1, count(2)=1
│   │       return 1 or 2 (tie)
│   │
│   ├── MAJORITY(6, 6): return 2
│   └── Combine candidates...
│       (Depends on tie-breaking, but eventually returns 2)
│
└── Combine: candidates from left=2 and right=2
    count(2 in [0..6]) = 4
    4 > 7/2 = 3.5? YES → return 2 ✓

Answer: 2 (appears 4 times out of 7)
```

---

## Complexity Analysis

```
RECURRENCE:
    T(n) = 2T(n/2) + O(n)
                      ↑
            counting step in combine is O(n)

Master Theorem: a=2, b=2, f(n)=O(n)
    Case 2: T(n) = O(n log n)

BEST CASE:
    If both halves agree (same majority), no counting needed.
    T(n) = 2T(n/2) + O(1) = O(n)

WORST CASE:
    Halves always disagree → counting every time.
    T(n) = 2T(n/2) + O(n) = O(n log n)

SPACE: O(log n) recursion stack
```

---

## Comparison: Boyer-Moore Voting Algorithm

```
BOYER_MOORE(nums):
    candidate ← nums[0]
    count ← 1
    
    for i ← 1 to n-1:
        if count == 0:
            candidate ← nums[i]
            count ← 1
        else if nums[i] == candidate:
            count ← count + 1
        else:
            count ← count - 1
    
    return candidate

Trace: [2, 2, 1, 1, 1, 2, 2]
  i=0: cand=2, count=1
  i=1: cand=2, count=2
  i=2: cand=2, count=1  (1≠2, decrement)
  i=3: cand=2, count=0  (1≠2, decrement)
  i=4: cand=1, count=1  (reset to 1)
  i=5: cand=1, count=0  (2≠1, decrement)
  i=6: cand=2, count=1  (reset to 2)
  
  Answer: 2 ✓

Why O(n), O(1)? Single pass, two variables.
```

---

## When D&C Shines

```
The D&C approach has value when:

1. LEARNING: Illustrates how D&C handles voting/counting
2. VERIFICATION: If no guarantee majority exists,
   D&C naturally handles the "no majority" case
3. PARALLEL COMPUTING: Each half is independent
   → Merge results from parallel processors
4. DISTRIBUTED SYSTEMS: Each machine finds local majority
   → Combine candidates (MapReduce pattern)

      Machine 1     Machine 2     Machine 3
      [2,2,1,3]     [2,2,2,1]     [2,1,2,2]
      cand: 2        cand: 2        cand: 2
           \            |            /
            └────── cand: 2 ──────┘
                   verify globally
```

---

## Related Problems

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  1. Majority Element II: Find elements appearing     │
│     more than n/3 times. (At most 2 such elements)   │
│     → Boyer-Moore with 2 candidates                  │
│                                                      │
│  2. Heavy Hitters: Find elements appearing more      │
│     than n/k times.                                  │
│     → Misra-Gries algorithm                          │
│                                                      │
│  3. Majority in sorted array:                        │
│     → Binary search for first/last occurrence O(logn)│
│                                                      │
│  4. Majority in stream:                              │
│     → Boyer-Moore (single pass, O(1) space)          │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Python Implementation

```python
def majorityElement_DC(nums):
    def majority(lo, hi):
        # Base case
        if lo == hi:
            return nums[lo]
        
        mid = (lo + hi) // 2
        left = majority(lo, mid)
        right = majority(mid + 1, hi)
        
        # If both halves agree
        if left == right:
            return left
        
        # Count each candidate in full range
        left_count = sum(1 for i in range(lo, hi + 1) if nums[i] == left)
        right_count = sum(1 for i in range(lo, hi + 1) if nums[i] == right)
        
        return left if left_count > right_count else right
    
    return majority(0, len(nums) - 1)


# Boyer-Moore for comparison
def majorityElement_BM(nums):
    candidate, count = nums[0], 1
    for x in nums[1:]:
        if count == 0:
            candidate, count = x, 1
        elif x == candidate:
            count += 1
        else:
            count -= 1
    return candidate
```

---

## Summary Table

| Aspect | D&C | Boyer-Moore |
|--------|-----|-------------|
| Time | O(n log n) | O(n) |
| Space | O(log n) | O(1) |
| Parallelizable | Yes | No (sequential) |
| Distributed | Yes (MapReduce) | Limited |
| Educational | High | Medium |
| Practical | Good | Best |

---

## Quick Revision Questions

1. **Why must the majority element be majority of at least one half?**
2. **What happens in the combine step when halves disagree?**
3. **What gives the D&C approach O(n log n) complexity?**
4. **How does Boyer-Moore work in O(n), O(1)?**
5. **When would you prefer D&C over Boyer-Moore?**

---

[← Previous: Maximum Subarray](04-maximum-subarray.md) | [Back to README](../README.md)
