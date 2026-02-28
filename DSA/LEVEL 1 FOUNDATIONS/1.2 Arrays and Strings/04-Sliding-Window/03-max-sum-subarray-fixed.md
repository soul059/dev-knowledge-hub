# Chapter 3: Max Sum Subarray of Size K (Fixed Window)

[← Previous: Variable Size Window](02-variable-size-window.md) | [Back to README](../README.md) | [Next: Longest Substring Without Repeating Characters →](04-longest-substring-no-repeat.md)

---

## Overview

**Problem:** Given an array of integers and a number k, find the maximum sum of any contiguous subarray of size k.

This is the most fundamental fixed-size sliding window problem and serves as the gateway to understanding the technique.

---

## Problem Statement

```
  Input:  arr = [2, 1, 5, 1, 3, 2],  k = 3
  Output: 9

  Explanation: Subarray [5, 1, 3] has the maximum sum = 9

  All windows of size 3:
  [2, 1, 5] → sum = 8
  [1, 5, 1] → sum = 7
  [5, 1, 3] → sum = 9  ← maximum
  [1, 3, 2] → sum = 6
```

---

## Approach 1: Brute Force

```
FUNCTION maxSumBrute(arr, n, k):
    maxSum = -INFINITY

    FOR i = 0 TO n-k:
        currentSum = 0
        FOR j = i TO i+k-1:
            currentSum = currentSum + arr[j]
        maxSum = MAX(maxSum, currentSum)

    RETURN maxSum
```

### Time: O(n × k) — recomputes sum for each window

```
  arr = [2, 1, 5, 1, 3, 2]   k=3

  Window 0: 2+1+5 = 8       (3 additions)
  Window 1: 1+5+1 = 7       (3 additions)  ← recalculating 5
  Window 2: 5+1+3 = 9       (3 additions)  ← recalculating 5,1
  Window 3: 1+3+2 = 6       (3 additions)

  Total: 4 windows × 3 additions = 12 operations
```

---

## Approach 2: Sliding Window (Optimal)

```
FUNCTION maxSumWindow(arr, n, k):
    // Step 1: Compute first window
    windowSum = 0
    FOR i = 0 TO k-1:
        windowSum = windowSum + arr[i]

    maxSum = windowSum

    // Step 2: Slide
    FOR i = k TO n-1:
        windowSum = windowSum + arr[i] - arr[i - k]
        maxSum = MAX(maxSum, windowSum)

    RETURN maxSum
```

### Time: O(n) — each element visited exactly once

---

## Detailed Trace

```
  arr = [2, 1, 5, 1, 3, 2]   k = 3

  Step 1: Build first window
  ─────────────────────────
  ┌───┬───┬───┬───┬───┬───┐
  │ 2 │ 1 │ 5 │ 1 │ 3 │ 2 │
  └───┴───┴───┴───┴───┴───┘
  [───────────]
  windowSum = 2 + 1 + 5 = 8
  maxSum = 8

  Step 2: Slide window
  ────────────────────

  i = 3:
  ┌───┬───┬───┬───┬───┬───┐
  │ 2 │ 1 │ 5 │ 1 │ 3 │ 2 │
  └───┴───┴───┴───┴───┴───┘
   out     [───────────]
   ─2              +1
  windowSum = 8 + arr[3] - arr[0] = 8 + 1 - 2 = 7
  maxSum = max(8, 7) = 8

  i = 4:
  ┌───┬───┬───┬───┬───┬───┐
  │ 2 │ 1 │ 5 │ 1 │ 3 │ 2 │
  └───┴───┴───┴───┴───┴───┘
       out     [───────────]
       ─1              +3
  windowSum = 7 + arr[4] - arr[1] = 7 + 3 - 1 = 9
  maxSum = max(8, 9) = 9   ← new max!

  i = 5:
  ┌───┬───┬───┬───┬───┬───┐
  │ 2 │ 1 │ 5 │ 1 │ 3 │ 2 │
  └───┴───┴───┴───┴───┴───┘
           out     [───────────]
           ─5              +2
  windowSum = 9 + arr[5] - arr[2] = 9 + 2 - 5 = 6
  maxSum = max(9, 6) = 9

  Final answer: 9  ✓  (subarray [5, 1, 3])
```

---

## Why This Works

```
  Window at position i:
  sum = arr[i-k+1] + arr[i-k+2] + ... + arr[i]

  Window at position i+1:
  sum = arr[i-k+2] + arr[i-k+3] + ... + arr[i+1]

  Difference:
  new_sum = old_sum - arr[i-k+1] + arr[i+1]
                      ^^^^^^^^^^^   ^^^^^^^^
                      leaving        entering

  ┌─────────────────────────────────────────┐
  │  Overlapping elements are NOT           │
  │  recomputed — only the edges change!    │
  └─────────────────────────────────────────┘
```

---

## Edge Cases

```
  1. k > n:     No valid window → return -1 or handle error
  2. k == n:    Only one window → return sum of entire array
  3. k == 1:    Return max element
  4. All negative: Works correctly — returns least negative sum
  5. Empty array: Return 0 or handle error
```

---

## Variations

### Variation 1: Return the Starting Index

```
FUNCTION maxSumIndex(arr, n, k):
    windowSum = 0
    FOR i = 0 TO k-1:
        windowSum = windowSum + arr[i]

    maxSum = windowSum
    startIndex = 0

    FOR i = k TO n-1:
        windowSum = windowSum + arr[i] - arr[i - k]
        IF windowSum > maxSum:
            maxSum = windowSum
            startIndex = i - k + 1

    RETURN startIndex
```

### Variation 2: Return the Subarray

```
  Using startIndex from above:
  result = arr[startIndex .. startIndex + k - 1]
```

### Variation 3: Minimum Sum Subarray of Size k

Change `MAX` to `MIN` — same algorithm.

### Variation 4: Average of Each Window

```
FUNCTION windowAverages(arr, n, k):
    result = []
    windowSum = 0
    FOR i = 0 TO k-1:
        windowSum = windowSum + arr[i]

    APPEND windowSum / k TO result

    FOR i = k TO n-1:
        windowSum = windowSum + arr[i] - arr[i - k]
        APPEND windowSum / k TO result

    RETURN result
```

---

## Complexity Analysis

| Metric | Brute Force | Sliding Window |
|--------|-------------|----------------|
| Time | O(n × k) | O(n) |
| Space | O(1) | O(1) |
| Additions | (n-k+1) × k | n |
| Comparisons | n-k | n-k |

```
      Time comparison (n = 10⁶):
      
      k = 100:   Brute = 10⁸    Window = 10⁶    100× faster
      k = 1000:  Brute = 10⁹    Window = 10⁶    1000× faster
      k = 10⁵:  Brute = 10¹¹   Window = 10⁶    100000× faster
```

---

## Common Mistakes

```
  ❌ Off-by-one in initial window:
     Wrong:  FOR i = 0 TO k       ← processes k+1 elements
     Right:  FOR i = 0 TO k-1     ← processes exactly k elements

  ❌ Wrong index for element leaving:
     Wrong:  arr[i]               ← this is the entering element!
     Right:  arr[i - k]           ← element k positions behind

  ❌ Forgetting to handle k > n:
     Always check if k ≤ n before proceeding

  ❌ Integer overflow for large values:
     Use long/long long for windowSum if needed
```

---

## Real-World Applications

| Application | Description |
|------------|-------------|
| Stock analysis | Best k-day average price |
| Network monitoring | Average throughput over k seconds |
| Sensor data | Smoothing with k-sample moving average |
| Gaming | Highest k-round score streak |
| Weather | k-day temperature average |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem type | Fixed-size window |
| Window state | Running sum |
| Update rule | sum += arr[i] - arr[i-k] |
| Time complexity | O(n) |
| Space complexity | O(1) |
| Key insight | Reuse previous computation |
| Number of windows | n - k + 1 |

---

## Quick Revision Questions

1. **What is the formula** to update the window sum when sliding from position i to i+1?

2. **What is the time complexity** of brute force vs sliding window for this problem?

3. **How would you modify the algorithm** to find the minimum sum subarray of size k?

4. **What subarray gives the answer** for arr = [4, 2, 1, 7, 8, 1, 2, 8, 1, 0] with k = 3? Trace through.

5. **Why is `arr[i-k]` the correct element to subtract** when the new element is `arr[i]`?

6. **How would you handle the case** where multiple subarrays have the same maximum sum?

---

[← Previous: Variable Size Window](02-variable-size-window.md) | [Back to README](../README.md) | [Next: Longest Substring Without Repeating Characters →](04-longest-substring-no-repeat.md)
