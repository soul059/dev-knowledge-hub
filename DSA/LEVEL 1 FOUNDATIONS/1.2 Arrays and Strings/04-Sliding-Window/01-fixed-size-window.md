# Chapter 1: Fixed Size Window

[← Previous Unit: Container with Most Water](../03-Two-Pointer-Technique/06-container-with-most-water.md) | [Back to README](../README.md) | [Next: Variable Size Window →](02-variable-size-window.md)

---

## Overview

The sliding window technique processes subarrays (or substrings) by maintaining a "window" that slides across the data. A fixed-size window has constant width k and slides one position at a time, reusing previous computation to avoid redundant work.

---

## Core Idea

Instead of recomputing the answer for each subarray from scratch, **add the new element and remove the old one**.

```
  Array: [1, 3, 5, 2, 8, 7, 4]    Window size k = 3

  BRUTE FORCE: Recalculate sum for each window
  Window 1: sum(1,3,5) = 9       ← computed from scratch
  Window 2: sum(3,5,2) = 10      ← computed from scratch
  Window 3: sum(5,2,8) = 15      ← computed from scratch

  SLIDING WINDOW: Reuse previous sum
  Window 1: sum = 1+3+5 = 9
  Window 2: sum = 9 - 1 + 2 = 10    ← subtract old, add new
  Window 3: sum = 10 - 3 + 8 = 15   ← subtract old, add new

  ┌───┬───┬───┬───┬───┬───┬───┐
  │ 1 │ 3 │ 5 │ 2 │ 8 │ 7 │ 4 │
  └───┴───┴───┴───┴───┴───┴───┘
  [─────────]                       Window 1: sum=9
      [─────────]                   Window 2: 9-1+2=10
          [─────────]               Window 3: 10-3+8=15
              [─────────]           Window 4: 15-5+7=17
                  [─────────]       Window 5: 17-2+4=19
```

---

## Template: Fixed-Size Sliding Window

```
FUNCTION fixedWindow(arr, n, k):
    // Step 1: Compute initial window (first k elements)
    windowValue = 0
    FOR i = 0 TO k-1:
        windowValue = windowValue + arr[i]

    bestValue = windowValue

    // Step 2: Slide the window
    FOR i = k TO n-1:
        windowValue = windowValue + arr[i] - arr[i - k]
        //                         ^^^^^^^^   ^^^^^^^^^^
        //                         add new     remove old
        bestValue = MAX(bestValue, windowValue)

    RETURN bestValue
```

---

## Visualization of Window Sliding

```
  k = 3, array = [a, b, c, d, e, f]

  ┌───┬───┬───┬───┬───┬───┐
  │ a │ b │ c │ d │ e │ f │
  └───┴───┴───┴───┴───┴───┘

  Window i=0:  [a, b, c]
               ├─────────┤
               add a,b,c           sum = a+b+c

  Window i=1:     [b, c, d]
                  ├─────────┤
               remove a, add d     sum = sum - a + d

  Window i=2:        [c, d, e]
                     ├─────────┤
               remove b, add e     sum = sum - b + e

  Window i=3:           [d, e, f]
                        ├─────────┤
               remove c, add f     sum = sum - c + f

  Each slide: O(1) — just one addition and one subtraction!
```

---

## Complexity Comparison

| Approach | Per Window | Total |
|----------|-----------|-------|
| Brute force | O(k) | O(n × k) |
| Sliding window | O(1) | O(n) |

```
  For n = 10⁶, k = 10³:
  
  Brute force: 10⁶ × 10³ = 10⁹ operations → TLE!
  Sliding window: 10⁶ operations → fast!
```

---

## Problem: Maximum Average Subarray of Size k

```
FUNCTION maxAverage(arr, n, k):
    windowSum = 0
    FOR i = 0 TO k-1:
        windowSum = windowSum + arr[i]

    maxSum = windowSum

    FOR i = k TO n-1:
        windowSum = windowSum + arr[i] - arr[i - k]
        maxSum = MAX(maxSum, windowSum)

    RETURN maxSum / k
```

### Trace: arr = [1, 12, -5, -6, 50, 3], k = 4

```
  Initial: sum(1,12,-5,-6) = 2,    maxSum = 2
  Slide:   2 - 1 + 50 = 51,        maxSum = 51
  Slide:   51 - 12 + 3 = 42,       maxSum = 51

  Max average = 51 / 4 = 12.75  ✓
```

---

## Problem: Count Occurrences of Anagrams

Given a string and a pattern, count how many windows of the string are anagrams of the pattern.

```
  text = "cbaebabacd", pattern = "abc"

  Window "cba" → sorted = "abc" → anagram ✓
  Window "bae" → no
  Window "aeb" → no
  Window "eba" → no
  Window "bab" → no
  Window "aba" → no
  Window "bac" → sorted = "abc" → anagram ✓
  Window "acd" → no

  Count = 2
```

Use a frequency array as the window state instead of a sum.

---

## Problem: Max Sum of k Consecutive Elements

Already shown above — the quintessential fixed window problem.

---

## When to Use Fixed-Size Window

```
  ┌──────────────────────────────────────────────┐
  │  USE FIXED-SIZE WINDOW WHEN:                  │
  │                                               │
  │  ✓ Subarray/substring size is GIVEN (k)       │
  │  ✓ Need to find max/min/count over all        │
  │    windows of size k                          │
  │  ✓ Window property can be updated in O(1)     │
  │    when element enters/leaves                 │
  │                                               │
  │  EXAMPLES:                                    │
  │  • Max sum of k consecutive elements          │
  │  • Average of every k consecutive elements    │
  │  • Max/min in every window (needs deque)      │
  │  • Anagram counting                           │
  │  • String permutation check                   │
  └──────────────────────────────────────────────┘
```

---

## Number of Windows

For an array of size n with window size k:

$$\text{Number of windows} = n - k + 1$$

```
  n=7, k=3: 7-3+1 = 5 windows
  
  [a b c d e f g]
  [───]               window 1
    [───]             window 2
      [───]           window 3
        [───]         window 4
          [───]       window 5
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Window size | Fixed at k |
| Slide step | 1 position per iteration |
| Update cost | O(1) per slide |
| Total time | O(n) |
| Space | O(1) for sum; O(k) for frequency |
| Number of windows | n - k + 1 |
| Key operation | Add new element, remove old element |

---

## Quick Revision Questions

1. **What is the key optimization** of sliding window over brute force for fixed-size subarray problems?

2. **How many windows of size k exist in an array of size n?** What if k > n?

3. **What information can you maintain** in a fixed window? (Sum, count, frequency, ...?)

4. **Write the update formula** when the window slides one position to the right.

5. **Can fixed-size window find the maximum element** in each window efficiently? (Hint: needs more than just tracking sum.)

6. **What's the time complexity of brute force vs sliding window** for finding the max sum of k consecutive elements in an array of size 10⁶?

---

[← Previous Unit: Container with Most Water](../03-Two-Pointer-Technique/06-container-with-most-water.md) | [Back to README](../README.md) | [Next: Variable Size Window →](02-variable-size-window.md)
