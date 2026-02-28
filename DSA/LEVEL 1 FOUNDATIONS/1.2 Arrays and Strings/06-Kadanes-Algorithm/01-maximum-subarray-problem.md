# Chapter 1: The Maximum Subarray Problem

[← Previous Unit: Product of Array Except Self](../05-Prefix-Sum-and-Difference-Array/06-product-except-self.md) | [Back to README](../README.md) | [Next: Kadane's Algorithm Explained →](02-kadanes-algorithm.md)

---

## Overview

The **Maximum Subarray Problem** asks: given an array of integers (possibly negative), find the contiguous subarray with the largest sum. This is one of the most classic problems in computer science, and its optimal solution—Kadane's Algorithm—is an elegant example of dynamic programming.

---

## Problem Statement

```
  Input:  [-2, 1, -3, 4, -1, 2, 1, -5, 4]
  Output: 6

  Explanation: The subarray [4, -1, 2, 1] has the largest sum = 6

  ┌────┬───┬────┬───┬────┬───┬───┬────┬───┐
  │ -2 │ 1 │ -3 │ 4 │ -1 │ 2 │ 1 │ -5 │ 4 │
  └────┴───┴────┴───┴────┴───┴───┴────┴───┘
                 [────────────────]
                  4 + (-1) + 2 + 1 = 6  ← maximum
```

---

## Why Is This Problem Important?

```
  ┌──────────────────────────────────────────────────┐
  │  1. Classic interview question (top 10 most      │
  │     asked problems)                              │
  │  2. Foundation for dynamic programming           │
  │  3. Real-world: stock price analysis,            │
  │     signal processing, bio-informatics           │
  │  4. Multiple solution approaches teach           │
  │     algorithm design techniques                  │
  │  5. Extensions lead to harder problems           │
  └──────────────────────────────────────────────────┘
```

---

## Approach 1: Brute Force — O(n³)

Enumerate all subarrays and compute each sum.

```
FUNCTION maxSubarrayBrute(arr, n):
    maxSum = -INFINITY

    FOR i = 0 TO n-1:              // start index
        FOR j = i TO n-1:          // end index
            currentSum = 0
            FOR k = i TO j:        // compute sum
                currentSum = currentSum + arr[k]
            maxSum = MAX(maxSum, currentSum)

    RETURN maxSum
```

```
  Number of subarrays: n(n+1)/2
  Sum computation per subarray: O(n)
  Total: O(n³)
```

---

## Approach 2: Better Brute Force — O(n²)

Use running sum to eliminate the innermost loop.

```
FUNCTION maxSubarrayBetter(arr, n):
    maxSum = -INFINITY

    FOR i = 0 TO n-1:
        currentSum = 0
        FOR j = i TO n-1:
            currentSum = currentSum + arr[j]
            maxSum = MAX(maxSum, currentSum)

    RETURN maxSum
```

### Trace: arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

```
  i=0: sums: -2, -1, -4, 0, -1, 1, 2, -3, 1
       max in this row: 2

  i=1: sums: 1, -2, 2, 1, 3, 4, -1, 3
       max in this row: 4

  i=2: sums: -3, 1, 0, 2, 3, -2, 2
       max in this row: 3

  i=3: sums: 4, 3, 5, 6, 1, 5
       max in this row: 6  ← GLOBAL MAX

  i=4: sums: -1, 1, 2, -3, 1
       max in this row: 2

  ...

  Overall max: 6  ✓
```

---

## Approach 3: Divide and Conquer — O(n log n)

Split array in half, find max in left, right, and crossing the middle.

```
FUNCTION maxSubarrayDC(arr, low, high):
    IF low == high:
        RETURN arr[low]

    mid = (low + high) / 2

    leftMax = maxSubarrayDC(arr, low, mid)
    rightMax = maxSubarrayDC(arr, mid+1, high)
    crossMax = maxCrossingSubarray(arr, low, mid, high)

    RETURN MAX(leftMax, rightMax, crossMax)

FUNCTION maxCrossingSubarray(arr, low, mid, high):
    // Best sum crossing from mid to left
    leftSum = -INFINITY
    sum = 0
    FOR i = mid DOWN TO low:
        sum = sum + arr[i]
        leftSum = MAX(leftSum, sum)

    // Best sum crossing from mid+1 to right
    rightSum = -INFINITY
    sum = 0
    FOR i = mid+1 TO high:
        sum = sum + arr[i]
        rightSum = MAX(rightSum, sum)

    RETURN leftSum + rightSum
```

### Visualization

```
  [-2, 1, -3, 4, -1, 2, 1, -5, 4]
  ├──────────────┤├──────────────┤
      LEFT              RIGHT
  
  ┌──────┬──────┐  ┌──────┬──────┐
  │-2, 1 │-3, 4 │  │-1, 2 │1,-5,4│
  └──────┴──────┘  └──────┴──────┘
  
  Three candidates at each level:
  1. Best entirely in LEFT half
  2. Best entirely in RIGHT half
  3. Best CROSSING the midpoint

  T(n) = 2T(n/2) + O(n) → O(n log n)
```

---

## Approach 4: Kadane's Algorithm — O(n)

The optimal solution. We'll study it in depth in the next chapter.

```
FUNCTION maxSubarrayKadane(arr, n):
    maxSoFar = arr[0]
    maxEndingHere = arr[0]

    FOR i = 1 TO n-1:
        maxEndingHere = MAX(arr[i], maxEndingHere + arr[i])
        maxSoFar = MAX(maxSoFar, maxEndingHere)

    RETURN maxSoFar
```

Preview: O(n) time, O(1) space — single pass!

---

## Comparison of All Approaches

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute force (3 loops) | O(n³) | O(1) | Enumerate all + compute |
| Better brute force | O(n²) | O(1) | Running sum |
| Divide and conquer | O(n log n) | O(log n) | Recursive stack |
| Kadane's algorithm | O(n) | O(1) | Optimal |

```
  For n = 10⁶:
  O(n³)     = 10¹⁸    → years
  O(n²)     = 10¹²    → hours
  O(n log n) = 2×10⁷  → fast
  O(n)      = 10⁶     → instant
```

---

## Edge Cases

```
  All negative:  [-3, -5, -1, -8]
  Answer: -1 (the least negative element)
  Kadane handles this correctly.

  Single element: [5]
  Answer: 5

  All positive:  [1, 2, 3, 4]
  Answer: 10 (entire array)

  All zeros:  [0, 0, 0]
  Answer: 0

  Mix of large positives and negatives: [100, -1, 100]
  Answer: 199
```

---

## Real-World Applications

| Application | Description |
|------------|-------------|
| Stock trading | Maximum profit from contiguous trading days |
| Signal processing | Strongest signal segment |
| Genomics | Maximum scoring subsequence in DNA |
| Image processing | Brightest rectangular region (2D extension) |
| Finance | Best continuous period of returns |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Find contiguous subarray with max sum |
| Input | Array of integers (can be negative) |
| Output | Maximum sum (some variations ask for the subarray itself) |
| Brute force | O(n²) or O(n³) |
| Optimal | O(n) — Kadane's Algorithm |
| Key challenge | All-negative arrays, mixing positives and negatives |

---

## Quick Revision Questions

1. **How many contiguous subarrays** exist in an array of size n?

2. **Why does the O(n²) brute force improve over O(n³)?** What computation is saved?

3. **How does divide and conquer handle the crossing case?** Why is it needed?

4. **What should the answer be for an all-negative array?** Why is this a tricky edge case?

5. **Name three real-world applications** of the maximum subarray problem.

6. **What is the recurrence relation** for the divide-and-conquer approach, and what does it solve to?

---

[← Previous Unit: Product of Array Except Self](../05-Prefix-Sum-and-Difference-Array/06-product-except-self.md) | [Back to README](../README.md) | [Next: Kadane's Algorithm Explained →](02-kadanes-algorithm.md)
