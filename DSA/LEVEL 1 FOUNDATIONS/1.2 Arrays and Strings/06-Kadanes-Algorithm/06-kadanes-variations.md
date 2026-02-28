# Chapter 6: Kadane's Variations and Extensions

[← Previous: Maximum Circular Subarray](05-maximum-circular-subarray.md) | [Back to README](../README.md) | [Next: What Is a String →](../07-String-Fundamentals/01-what-is-a-string.md)

---

## Overview

Kadane's core idea — the **"extend or start fresh"** decision at each element — extends far beyond the basic maximum subarray problem. This chapter explores powerful variations and related problems.

---

## 1. Maximum Product Subarray (LeetCode 152)

**Problem:** Find the contiguous subarray with the largest product.

**Challenge:** A negative × negative = positive! So we can't just track max.

```
  arr = [2, 3, -2, 4]

  Unlike sum, a very negative product can become
  very positive if multiplied by another negative!

  Track BOTH maxProduct and minProduct at each step.
```

### Algorithm

```
FUNCTION maxProductSubarray(arr, n):
    maxProduct = arr[0]
    minProduct = arr[0]
    result = arr[0]

    FOR i = 1 TO n-1:
        IF arr[i] < 0:
            SWAP(maxProduct, minProduct)
        // When arr[i] is negative, the max becomes min and vice versa
        
        maxProduct = MAX(arr[i], maxProduct * arr[i])
        minProduct = MIN(arr[i], minProduct * arr[i])
        
        result = MAX(result, maxProduct)

    RETURN result
```

### Trace

```
  arr = [2, 3, -2, 4]

  i=0: maxP=2, minP=2, result=2

  i=1: arr[i]=3 > 0, no swap
       maxP = max(3, 2*3) = 6
       minP = min(3, 2*3) = 3
       result = max(2, 6) = 6

  i=2: arr[i]=-2 < 0, SWAP → maxP=3, minP=6
       maxP = max(-2, 3*(-2)) = max(-2,-6) = -2
       minP = min(-2, 6*(-2)) = min(-2,-12) = -12
       result = max(6, -2) = 6

  i=3: arr[i]=4 > 0, no swap
       maxP = max(4, -2*4) = max(4,-8) = 4
       minP = min(4, -12*4) = min(4,-48) = -48
       result = max(6, 4) = 6

  Answer: 6 (subarray [2, 3])
```

### Why Track Min?

```
  arr = [2, -5, -2, 4]

  After [2, -5]: minP = -10
  At -2:  -10 * -2 = 20  ← min becomes max!

  Without tracking min, we'd miss this!
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(1) |

---

## 2. Maximum Sum with No Two Adjacent (House Robber — LeetCode 198)

**Problem:** Find the maximum sum of elements such that no two selected elements are adjacent.

```
  arr = [2, 7, 9, 3, 1]

  Can't take adjacent pairs:
  Take 2, 9, 1 → 12
  Take 7, 3   → 10
  Take 2, 9   → 11
  Take 7, 1   → 8

  Best: 2 + 9 + 1 = 12  ✓
```

### The Connection to Kadane's

Both use the "decision at each element" pattern:
- **Kadane:** extend current subarray or start new?
- **House Robber:** include this element (skip previous) or skip this?

```
FUNCTION maxSumNoAdjacent(arr, n):
    IF n == 0: RETURN 0
    IF n == 1: RETURN arr[0]

    prev2 = 0              // best sum excluding i-1 and before
    prev1 = arr[0]         // best sum including up to i-1

    FOR i = 1 TO n-1:
        current = MAX(prev1, prev2 + arr[i])
        //          skip i     take i
        prev2 = prev1
        prev1 = current

    RETURN prev1
```

### Trace

```
  arr = [2, 7, 9, 3, 1]

  prev2=0, prev1=2

  i=1: current = max(2, 0+7) = 7
       prev2=2, prev1=7

  i=2: current = max(7, 2+9) = 11
       prev2=7, prev1=11

  i=3: current = max(11, 7+3) = 11
       prev2=11, prev1=11

  i=4: current = max(11, 11+1) = 12
       prev2=11, prev1=12

  Answer: 12  ✓ (took 2, 9, 1)
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(1) |

---

## 3. Maximum Subarray Sum of Length ≥ K

**Problem:** Find the maximum subarray sum with at least k elements.

**Idea:** Combine prefix sum with Kadane's.

```
  For each index i (i ≥ k):
    maxSum = max of:
      1) sum of arr[i-k+1..i] (exactly k elements)  
      2) sum of arr[i-k+1..i] + maxEndHere at position i-k

  Use prefix sums for the k-element sum, and Kadane's 
  for the optional extension before the window.
```

### Algorithm

```
FUNCTION maxSumAtLeastK(arr, n, k):
    // Compute prefix sums
    prefix[0..n] where prefix[0]=0, prefix[i]=prefix[i-1]+arr[i-1]

    // Kadane-like pass for max ending here
    maxEndHere = array of size n
    maxEndHere[0] = arr[0]
    FOR i = 1 TO n-1:
        maxEndHere[i] = MAX(arr[i], maxEndHere[i-1] + arr[i])

    result = prefix[k]    // first k elements

    FOR i = k TO n-1:
        // Sum of last k elements ending at i
        windowSum = prefix[i+1] - prefix[i+1-k]
        
        // Option 1: just the k-element window
        result = MAX(result, windowSum)
        
        // Option 2: k elements + best extension before window
        IF i-k >= 0:
            result = MAX(result, windowSum + maxEndHere[i-k])

    RETURN result
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(n) for prefix and maxEndHere arrays |

---

## 4. 2D Kadane's: Maximum Rectangle Sum

**Problem:** Given an m × n matrix of integers, find the rectangle (submatrix) with the maximum sum.

```
  Matrix:
  ┌────┬────┬────┬────┐
  │  1 │  2 │ -1 │ -4 │
  ├────┼────┼────┼────┤
  │ -8 │ -3 │  4 │  2 │
  ├────┼────┼────┼────┤
  │  3 │  8 │ 10 │  1 │
  ├────┼────┼────┼────┤
  │ -4 │ -1 │  1 │  7 │
  └────┴────┴────┴────┘

  Maximum rectangle:  columns 1-3, rows 1-2
  ┌────┬────┬────┐
  │ -3 │  4 │  2 │  = 3
  ├────┼────┼────┤
  │  8 │ 10 │  1 │  = 19
  └────┴────┴────┘
  Total = 22
```

### Key Insight — Reduce to 1D

```
  Fix LEFT and RIGHT column boundaries.
  Compress each row between these columns into a single sum.
  Apply 1D Kadane's on the compressed column!

  Left=1, Right=3:
  Row 0: 2 + (-1) + (-4) = -3
  Row 1: -3 + 4 + 2     =  3
  Row 2: 8 + 10 + 1      = 19
  Row 3: -1 + 1 + 7      =  7

  Compressed: [-3, 3, 19, 7]
  Kadane's max: 3 + 19 + 7 = 29?  Let's check...
  Actually: -3 + 3 + 19 + 7 = 26, or 3+19+7=29.
  Kadane: max(3, 19, 7, 3+19, 19+7, 3+19+7, -3+3+19+7)
  = max(29, 26) = 29
```

### Algorithm

```
FUNCTION maxRectangleSum(matrix, m, n):
    result = -INFINITY

    FOR left = 0 TO n-1:
        temp[0..m-1] = {0, 0, ..., 0}

        FOR right = left TO n-1:
            // Add current column to temp
            FOR i = 0 TO m-1:
                temp[i] = temp[i] + matrix[i][right]
            
            // Apply 1D Kadane's on temp[]
            kadane_result = kadaneMax(temp, m)
            result = MAX(result, kadane_result)

    RETURN result
```

### Complexity

| Metric | Value |
|--------|-------|
| Time | O(n² × m) — n² column pairs, O(m) Kadane per pair |
| Space | O(m) — temp array |

For a square matrix (m = n): **O(n³)**

---

## 5. Best Time to Buy and Sell Stock (LeetCode 121)

**Problem:** Given stock prices, find the maximum profit from one transaction (buy then sell).

```
  prices = [7, 1, 5, 3, 6, 4]

  Buy at 1, sell at 6 → profit = 5
```

### Kadane's Connection

```
  Convert prices to daily changes:
  prices:  [7,  1,  5,  3,  6,  4]
  changes: [  -6,  4, -2,  3, -2]

  Maximum subarray of changes = maximum profit!
  Kadane on [-6, 4, -2, 3, -2]:
  -6, 4, 2, 3, 1 → max = 5  ✓
  (Corresponds to buy day 1, sell day 4: 6-1=5)
```

### Direct Approach (Equivalent to Kadane's)

```
FUNCTION maxProfit(prices, n):
    minPrice = prices[0]
    maxProfit = 0

    FOR i = 1 TO n-1:
        maxProfit = MAX(maxProfit, prices[i] - minPrice)
        minPrice = MIN(minPrice, prices[i])

    RETURN maxProfit
```

### Trace

```
  prices = [7, 1, 5, 3, 6, 4]

  i=1: maxProfit = max(0, 1-7) = 0, minPrice = min(7,1) = 1
  i=2: maxProfit = max(0, 5-1) = 4, minPrice = 1
  i=3: maxProfit = max(4, 3-1) = 4, minPrice = 1
  i=4: maxProfit = max(4, 6-1) = 5, minPrice = 1
  i=5: maxProfit = max(5, 4-1) = 5, minPrice = 1

  Answer: 5  ✓
```

---

## 6. Maximum Alternating Subarray Sum

**Problem:** Find the maximum sum of a subarray where signs alternate: +a₁ - a₂ + a₃ - a₄ + ...

```
FUNCTION maxAlternatingSum(arr, n):
    // odd: max sum ending here with + sign on last element
    // even: max sum ending here with - sign on last element
    odd = arr[0]
    even = -INFINITY
    result = arr[0]

    FOR i = 1 TO n-1:
        newOdd = MAX(arr[i], even + arr[i])
        newEven = odd - arr[i]
        odd = newOdd
        even = newEven
        result = MAX(result, odd)

    RETURN result
```

---

## Pattern Recognition: The Kadane Family

```
  ┌──────────────────────────────────────────────────────┐
  │              THE KADANE DECISION PATTERN             │
  │                                                      │
  │   At each element, make a LOCAL decision:            │
  │   "Extend the current solution or start fresh?"      │
  │                                                      │
  │  ┌─────────────────────┬──────────────────────────┐  │
  │  │ Problem              │ Decision                 │  │
  │  ├─────────────────────┼──────────────────────────┤  │
  │  │ Max Sum Subarray     │ extend sum or restart    │  │
  │  │ Min Sum Subarray     │ extend sum or restart    │  │
  │  │ Max Product          │ track max AND min        │  │
  │  │ Circular Max         │ max(normal, total-min)   │  │
  │  │ House Robber         │ take (skip prev) or skip │  │
  │  │ Buy/Sell Stock       │ track minSoFar           │  │
  │  │ 2D Max Rectangle    │ compress + 1D Kadane     │  │
  │  │ Max Sum ≥ k length  │ prefix + Kadane          │  │
  │  └─────────────────────┴──────────────────────────┘  │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Variation | Key Modification | Time | Space |
|-----------|-----------------|------|-------|
| Max sum subarray | Standard Kadane | O(n) | O(1) |
| Min sum subarray | Inverted Kadane | O(n) | O(1) |
| Circular max sum | max + min Kadane | O(n) | O(1) |
| Max product | Track max AND min | O(n) | O(1) |
| House Robber | Include/exclude DP | O(n) | O(1) |
| Buy/Sell Stock | Track min price | O(n) | O(1) |
| Max sum ≥ k | Prefix + Kadane | O(n) | O(n) |
| 2D max rectangle | Column compress + Kadane | O(n²m) | O(m) |

---

## Quick Revision Questions

1. **Why does max product subarray need to track both max and min?** Give an example.

2. **How does the stock buy/sell problem relate to Kadane's?** What transformation connects them?

3. **Explain the column compression trick** in 2D Kadane's algorithm.

4. **How is the House Robber problem similar to Kadane's?** What is the "extend or start fresh" decision?

5. **Trace max product on [-2, 3, -4].** What is the answer and why?

6. **What is the common pattern** across all Kadane variations? State it in one sentence.

---

[← Previous: Maximum Circular Subarray](05-maximum-circular-subarray.md) | [Back to README](../README.md) | [Next: What Is a String →](../07-String-Fundamentals/01-what-is-a-string.md)
