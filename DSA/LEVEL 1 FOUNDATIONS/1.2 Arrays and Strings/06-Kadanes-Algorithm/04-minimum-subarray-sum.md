# Chapter 4: Minimum Subarray Sum

[← Previous: Tracking the Subarray](03-tracking-the-subarray.md) | [Back to README](../README.md) | [Next: Maximum Circular Subarray →](05-maximum-circular-subarray.md)

---

## Overview

The **Minimum Subarray Sum** problem is the mirror image of the Maximum Subarray Problem. Instead of finding the largest sum, we find the **smallest (most negative) sum**. The solution is a simple inversion of Kadane's Algorithm.

---

## Problem Statement

```
  Input:  [3, -4, 2, -3, -1, 7, -5]
  Output: -6

  Explanation: Subarray [-3, -1, 7, -5]? No...
  Let's check: [-4, 2, -3, -1] has sum = -6
               [-3, -1] has sum = -4
               [-4] has sum = -4
               
  Actually: subarray [-4, 2, -3, -1] = -6  ← minimum
```

---

## Two Ways to Solve

### Method 1: Modify Kadane's Directly

Replace MAX with MIN throughout.

```
FUNCTION minSubarraySum(arr, n):
    minEndingHere = arr[0]
    minSoFar = arr[0]

    FOR i = 1 TO n-1:
        minEndingHere = MIN(arr[i], minEndingHere + arr[i])
        minSoFar = MIN(minSoFar, minEndingHere)

    RETURN minSoFar
```

### Method 2: Negate and Use Standard Kadane

```
FUNCTION minSubarraySum(arr, n):
    // Negate all elements
    negated = new array of size n
    FOR i = 0 TO n-1:
        negated[i] = -arr[i]

    // Find max subarray of negated array
    maxOfNegated = kadane(negated, n)

    // Negate back to get minimum
    RETURN -maxOfNegated
```

```
  ┌──────────────────────────────────────────────────┐
  │  min(arr) = -max(-arr)                           │
  │                                                  │
  │  Finding the most negative sum in arr is the     │
  │  same as finding the most positive sum in -arr.  │
  └──────────────────────────────────────────────────┘
```

---

## Trace: Method 1

```
  arr = [3, -4, 2, -3, -1, 7, -5]

  ┌─────┬────────┬────────────────────────────────────┬───────────┬─────────┐
  │  i  │ arr[i] │ Decision                           │ endHere   │ soFar   │
  ├─────┼────────┼────────────────────────────────────┼───────────┼─────────┤
  │  0  │    3   │ Initialize                         │    3      │    3    │
  │  1  │   -4   │ min(-4, 3+(-4)=-1) = -4 → NEW     │   -4      │   -4    │
  │  2  │    2   │ min(2, -4+2=-2) = -2 → EXTEND     │   -2      │   -4    │
  │  3  │   -3   │ min(-3, -2+(-3)=-5) = -5 → EXTEND │   -5      │   -5    │
  │  4  │   -1   │ min(-1, -5+(-1)=-6) = -6 → EXTEND │   -6      │   -6    │
  │  5  │    7   │ min(7, -6+7=1) = 1 → EXTEND       │    1      │   -6    │
  │  6  │   -5   │ min(-5, 1+(-5)=-4) = -5 → NEW     │   -5      │   -6    │
  └─────┴────────┴────────────────────────────────────┴───────────┴─────────┘

  Answer: -6  (subarray [-4, 2, -3, -1] at indices 1-4)  ✓
```

---

## The Decision (Inverted)

```
  Standard Kadane (MAX): 
  "Is the accumulated sum helpful? If negative, restart."
  
  Inverted Kadane (MIN):
  "Is the accumulated sum harmful (positive)? If positive 
   enough, restart because current element alone is worse."

  ┌──────────────────────────────────────────────────┐
  │  MIN version restarts when the running sum has   │
  │  become POSITIVE — a positive prefix only makes  │
  │  future sums LARGER (not what we want for MIN).  │
  └──────────────────────────────────────────────────┘
```

---

## All-Positive Array

```
  arr = [3, 5, 1, 8, 2]
  
  Every element is positive, so the minimum subarray
  is the single smallest element.

  Trace:
  i=0: endHere=3, soFar=3
  i=1: min(5, 3+5=8)=5 → NEW, endHere=5, soFar=3
  i=2: min(1, 5+1=6)=1 → NEW, endHere=1, soFar=1
  i=3: min(8, 1+8=9)=8 → NEW, endHere=8, soFar=1
  i=4: min(2, 8+2=10)=2 → NEW, endHere=2, soFar=1

  Answer: 1  ✓ (single element)
```

---

## Application: Maximum Circular Subarray

The minimum subarray sum is a key component in solving the Maximum Circular Subarray problem (next chapter). The idea:

```
  maxCircular = totalSum - minSubarraySum

  If the maximum subarray wraps around the array,
  then the elements NOT in it form the minimum subarray.
```

---

## Comparison: Max vs Min Subarray

| Aspect | Maximum Subarray | Minimum Subarray |
|--------|-----------------|------------------|
| Operator | MAX | MIN |
| Restart when | Running sum < 0 | Running sum > 0 |
| All positive | Return total sum | Return smallest element |
| All negative | Return largest element | Return total sum |
| Conversion | Direct | Negate → max → negate back |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Find contiguous subarray with minimum sum |
| Algorithm | Kadane's with MIN instead of MAX |
| Recurrence | dp[i] = min(arr[i], dp[i-1]+arr[i]) |
| Alternative | min = -max(-arr) |
| Time | O(n) |
| Space | O(1) |
| Use case | Circular subarray problem (next chapter) |

---

## Quick Revision Questions

1. **How do you convert Kadane's maximum algorithm** to find the minimum subarray sum?

2. **Why does the negate-and-max approach work?** Prove with an example.

3. **When does the minimum subarray algorithm restart?** How is this different from the max version?

4. **What is the minimum subarray sum** of an all-positive array?

5. **Trace through arr = [2, -1, 3, -5, 4, -2, 1]** using the min Kadane's algorithm.

6. **Why is the minimum subarray sum useful** for the circular subarray problem?

---

[← Previous: Tracking the Subarray](03-tracking-the-subarray.md) | [Back to README](../README.md) | [Next: Maximum Circular Subarray →](05-maximum-circular-subarray.md)
