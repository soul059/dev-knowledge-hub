# Chapter 5: Maximum Circular Subarray

[← Previous: Minimum Subarray Sum](04-minimum-subarray-sum.md) | [Back to README](../README.md) | [Next: Kadane's Variations and Extensions →](06-kadanes-variations.md)

---

## Overview

**Problem (LeetCode 918):** Given a circular integer array, find the contiguous subarray with the maximum sum. In a circular array, the end connects back to the beginning, so subarrays can "wrap around."

---

## Problem Understanding

```
  Linear array:   [5, -3, 5]
  Max subarray:   [5, -3, 5] = 7  or  [5] = 5

  Circular array: [5, -3, 5] → 5, -3, 5, 5, -3, 5, ...
  
  The subarray can wrap:
  [5, |wrap| 5] = 10  ← takes last 5 and first 5!

  ┌───┬────┬───┐
  │ 5 │ -3 │ 5 │  Connect end → start
  └─┬─┴────┴─┬─┘
    └────←────┘
  
  Wrapping subarray: [5, 5] (index 2 then index 0) = 10
  This is BETTER than any non-wrapping subarray!
```

---

## Two Cases

```
  CASE 1: Maximum subarray does NOT wrap around
  ┌───┬───┬───┬───┬───┬───┬───┐
  │   │   │ █ │ █ │ █ │   │   │  ← standard Kadane's
  └───┴───┴───┴───┴───┴───┴───┘
          [───────────]

  CASE 2: Maximum subarray WRAPS around
  ┌───┬───┬───┬───┬───┬───┬───┐
  │ █ │ █ │   │   │   │ █ │ █ │  ← wrapping!
  └───┴───┴───┴───┴───┴───┴───┘
  [─────]               [─────]

  For Case 2: the elements NOT in the max subarray 
  form a MINIMUM non-wrapping subarray in the middle!

  ┌───┬───┬───┬───┬───┬───┬───┐
  │ █ │ █ │ m │ m │ m │ █ │ █ │
  └───┴───┴───┴───┴───┴───┴───┘
          [───────────]
          minimum subarray

  max_wrapping = totalSum - minSubarraySum
```

---

## The Key Formula

$$\text{maxCircular} = \max(\text{maxSubarray}, \text{totalSum} - \text{minSubarray})$$

```
  ┌──────────────────────────────────────────────────┐
  │  Case 1: Standard Kadane gives the answer        │
  │  Case 2: totalSum − minKadane gives the answer   │
  │                                                  │
  │  Take the MAX of both cases!                     │
  └──────────────────────────────────────────────────┘
```

---

## Algorithm

```
FUNCTION maxCircularSubarray(arr, n):
    // Case 1: Standard max subarray (no wrap)
    maxKadane = kadaneMax(arr, n)

    // Case 2: Wrapping subarray
    totalSum = SUM(arr)
    minKadane = kadaneMin(arr, n)
    maxWrap = totalSum - minKadane

    // Edge case: all elements are negative
    IF maxWrap == 0:
        RETURN maxKadane
    // (When all negative, minKadane = totalSum → maxWrap = 0)

    RETURN MAX(maxKadane, maxWrap)
```

---

## Why the Edge Case?

```
  arr = [-3, -5, -1]
  totalSum = -9
  minKadane = -9 (entire array)
  maxWrap = -9 - (-9) = 0

  But an empty subarray isn't valid!
  We need at least one element.
  So when maxWrap = 0, use maxKadane = -1.

  Equivalently: when ALL elements are negative,
  maxWrap makes no sense (it means "take nothing").
  Use standard Kadane's answer instead.
```

---

## Detailed Trace

```
  arr = [5, -3, 5]

  Standard Kadane (max):
  i=0: endHere=5, soFar=5
  i=1: max(-3, 5-3=2)=2, soFar=5
  i=2: max(5, 2+5=7)=7, soFar=7
  maxKadane = 7

  Kadane (min):
  i=0: endHere=5, soFar=5
  i=1: min(-3, 5-3=2)=-3, soFar=-3
  i=2: min(5, -3+5=2)=2, soFar=-3
  minKadane = -3

  totalSum = 5 + (-3) + 5 = 7
  maxWrap = 7 - (-3) = 10

  maxCircular = max(7, 10) = 10  ✓

  The wrapping subarray: total - min subarray
  = everything except [-3] = [5, 5] (wrapping) = 10
```

---

## Trace: No Wrap Is Better

```
  arr = [1, -2, 3, -2]

  Standard Kadane (max):
  i=0: endHere=1, soFar=1
  i=1: max(-2, 1-2=-1)=-1, soFar=1
  i=2: max(3, -1+3=2)=3, soFar=3
  i=3: max(-2, 3-2=1)=1, soFar=3
  maxKadane = 3

  Kadane (min):
  i=0: endHere=1, soFar=1
  i=1: min(-2, 1-2=-1)=-2, soFar=-2
  i=2: min(3, -2+3=1)=1, soFar=-2
  i=3: min(-2, 1-2=-1)=-2, soFar=-2
  
  Wait, let me redo: actually we can find min = -4?
  i=1: endHere = min(-2, 1+(-2))= min(-2,-1)=-2, soFar=-2
  i=3: endHere = min(-2, 1+(-2))= min(-2,-1)=-2, soFar=-2
  
  Hmm, minKadane=-2
  totalSum = 1-2+3-2 = 0
  maxWrap = 0-(-2) = 2

  maxCircular = max(3, 2) = 3  ✓ (non-wrapping is better)
```

---

## Trace: All Negative

```
  arr = [-3, -2, -5]

  maxKadane = -2 (least negative element)
  minKadane = -10 (entire array)
  totalSum = -10
  maxWrap = -10 - (-10) = 0

  maxWrap == 0 → return maxKadane = -2  ✓
```

---

## Visual Explanation

```
  CASE 1 (no wrap — standard):
  ┌───┬───┬───┬───┬───┬───┐
  │   │   │ ■ │ ■ │ ■ │   │   max = sum of ■
  └───┴───┴───┴───┴───┴───┘

  CASE 2 (wrap around):
  ┌───┬───┬───┬───┬───┬───┐
  │ ■ │ ■ │ ░ │ ░ │ ■ │ ■ │   ■ = in max subarray (wrapping)
  └─┬─┴───┴───┴───┴───┴─┬─┘   ░ = NOT in max subarray
    └────────────────────┘        = min subarray!

  sum(■) = totalSum - sum(░)
         = totalSum - minSubarraySum
```

---

## Single Pass Implementation

We can compute maxKadane, minKadane, and totalSum in one pass:

```
FUNCTION maxCircularSubarray(arr, n):
    maxEndHere = arr[0]
    maxSoFar = arr[0]
    minEndHere = arr[0]
    minSoFar = arr[0]
    totalSum = arr[0]

    FOR i = 1 TO n-1:
        totalSum = totalSum + arr[i]

        maxEndHere = MAX(arr[i], maxEndHere + arr[i])
        maxSoFar = MAX(maxSoFar, maxEndHere)

        minEndHere = MIN(arr[i], minEndHere + arr[i])
        minSoFar = MIN(minSoFar, minEndHere)

    IF maxSoFar < 0:
        RETURN maxSoFar     // all negative
    
    RETURN MAX(maxSoFar, totalSum - minSoFar)
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n) — single pass |
| Space | O(1) — five variables |

---

## Comparison: Linear vs Circular

| Aspect | Linear Max Subarray | Circular Max Subarray |
|--------|--------------------|-----------------------|
| Wrapping allowed | No | Yes |
| Algorithm | Kadane's | Kadane's max + min |
| Cases to consider | 1 | 2 |
| Edge case | All negative | All negative (maxWrap = 0) |
| Time | O(n) | O(n) |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Max subarray sum in circular array |
| Key insight | Answer is max(standard, total − min) |
| Two cases | Non-wrapping (Kadane) or wrapping (total − minKadane) |
| Edge case | All negative → maxWrap=0, use standard |
| Time | O(n) |
| Space | O(1) |

---

## Quick Revision Questions

1. **What are the two cases** for the maximum sum in a circular array?

2. **Why does `totalSum - minSubarray` give the wrapping answer?** Explain with a diagram.

3. **What happens when all elements are negative?** Why is maxWrap = 0 invalid?

4. **Can you implement the solution in a single pass?** What variables do you need?

5. **Trace through arr = [8, -1, 3, 4]** for the circular maximum subarray.

6. **What is the relationship** between the wrapping maximum and the non-wrapping minimum?

---

[← Previous: Minimum Subarray Sum](04-minimum-subarray-sum.md) | [Back to README](../README.md) | [Next: Kadane's Variations and Extensions →](06-kadanes-variations.md)
