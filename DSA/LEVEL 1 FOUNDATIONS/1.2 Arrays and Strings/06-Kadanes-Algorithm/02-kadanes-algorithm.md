# Chapter 2: Kadane's Algorithm Explained

[← Previous: The Maximum Subarray Problem](01-maximum-subarray-problem.md) | [Back to README](../README.md) | [Next: Tracking the Subarray →](03-tracking-the-subarray.md)

---

## Overview

Kadane's Algorithm solves the Maximum Subarray Problem in **O(n) time and O(1) space**. It was published by Jay Kadane in 1984. The algorithm is a beautiful example of dynamic programming — at each position, it decides whether to extend the current subarray or start a new one.

---

## The Core Decision

```
  At each element arr[i], there are exactly TWO choices:

  1. EXTEND: Add arr[i] to the existing subarray
     → maxEndingHere + arr[i]

  2. START NEW: Begin a fresh subarray from arr[i]
     → arr[i]

  Pick whichever is LARGER:
  maxEndingHere = MAX(arr[i], maxEndingHere + arr[i])

  ┌──────────────────────────────────────────────────┐
  │  "Is it better to continue the current run,     │
  │   or is the accumulated sum so negative that     │
  │   starting fresh is better?"                     │
  └──────────────────────────────────────────────────┘
```

---

## The Algorithm

```
FUNCTION kadane(arr, n):
    maxEndingHere = arr[0]    // best sum ending at current position
    maxSoFar = arr[0]         // global best sum found so far

    FOR i = 1 TO n-1:
        // Decision: extend or start new
        maxEndingHere = MAX(arr[i], maxEndingHere + arr[i])

        // Update global best
        maxSoFar = MAX(maxSoFar, maxEndingHere)

    RETURN maxSoFar
```

---

## Detailed Trace

```
  arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

  ┌─────┬────────┬────────────────────────────────────┬───────────┬─────────┐
  │  i  │ arr[i] │ Decision                           │ endHere   │ soFar   │
  ├─────┼────────┼────────────────────────────────────┼───────────┼─────────┤
  │  0  │   -2   │ Initialize                         │   -2      │   -2    │
  │  1  │    1   │ max(1, -2+1=-1) = 1 → START NEW    │    1      │    1    │
  │  2  │   -3   │ max(-3, 1+(-3)=-2) = -2 → EXTEND   │   -2      │    1    │
  │  3  │    4   │ max(4, -2+4=2) = 4 → START NEW     │    4      │    4    │
  │  4  │   -1   │ max(-1, 4+(-1)=3) = 3 → EXTEND     │    3      │    4    │
  │  5  │    2   │ max(2, 3+2=5) = 5 → EXTEND         │    5      │    5    │
  │  6  │    1   │ max(1, 5+1=6) = 6 → EXTEND         │    6      │    6    │
  │  7  │   -5   │ max(-5, 6+(-5)=1) = 1 → EXTEND     │    1      │    6    │
  │  8  │    4   │ max(4, 1+4=5) = 5 → EXTEND         │    5      │    6    │
  └─────┴────────┴────────────────────────────────────┴───────────┴─────────┘

  Answer: 6  (subarray [4, -1, 2, 1])  ✓
```

### Visual of maxEndingHere Over Time

```
  arr:       -2   1  -3   4  -1   2   1  -5   4
  endHere:   -2   1  -2   4   3   5   6   1   5
                  ↑       ↑                    
                  new     new (restarted)       
                  start   start
  soFar:     -2   1   1   4   4   5   6   6   6
                                       ↑
                                    global max
```

---

## Why It Works: The DP Perspective

```
  Define: dp[i] = maximum subarray sum ENDING at index i

  Recurrence:
  dp[i] = MAX(arr[i], dp[i-1] + arr[i])
          ├───────┤  ├──────────────────┤
          start new  extend previous

  Base: dp[0] = arr[0]

  Answer: MAX(dp[0], dp[1], ..., dp[n-1])

  Full DP array:
  arr: [-2,  1, -3,  4, -1,  2,  1, -5,  4]
  dp:  [-2,  1, -2,  4,  3,  5,  6,  1,  5]
                       ↑
                       dp[3] = max(4, -2+4) = 4
                       (started new because past was worse)

  Global max = max of dp array = 6
```

But we don't need the entire dp array — just the previous value!

```
  dp[i] only depends on dp[i-1]
  → Only need ONE variable: maxEndingHere

  This is "space optimization of DP"
  O(n) space → O(1) space
```

---

## Alternative Formulation

Some implementations use a slightly different approach:

```
FUNCTION kadaneV2(arr, n):
    maxEndingHere = 0
    maxSoFar = -INFINITY

    FOR i = 0 TO n-1:
        maxEndingHere = maxEndingHere + arr[i]
        maxSoFar = MAX(maxSoFar, maxEndingHere)
        
        IF maxEndingHere < 0:
            maxEndingHere = 0    // reset — negative sum never helps

    RETURN maxSoFar
```

### This version resets to 0 instead of using MAX

```
  ┌────────────────────────────────────────────────┐
  │  "If the sum has gone negative, discard it —   │
  │   any future subarray is better off starting   │
  │   fresh than carrying negative baggage."       │
  └────────────────────────────────────────────────┘

  Important: maxSoFar is updated BEFORE the reset.
  This ensures all-negative arrays are handled.
```

### Trace with V2

```
  arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

  i=0: endHere=-2, soFar=-2, endHere<0→reset to 0
  i=1: endHere=1,  soFar=1
  i=2: endHere=-2, soFar=1,  endHere<0→reset to 0
  i=3: endHere=4,  soFar=4
  i=4: endHere=3,  soFar=4
  i=5: endHere=5,  soFar=5
  i=6: endHere=6,  soFar=6
  i=7: endHere=1,  soFar=6
  i=8: endHere=5,  soFar=6

  Answer: 6  ✓
```

---

## When Does Kadane Restart?

```
  maxEndingHere resets (starts new subarray) when:

  MAX(arr[i], maxEndingHere + arr[i]) = arr[i]

  This happens when:
  maxEndingHere + arr[i] < arr[i]
  maxEndingHere < 0

  ┌──────────────────────────────────────────┐
  │  A subarray is abandoned when its sum    │
  │  becomes negative — it would only hurt   │
  │  any future extension.                   │
  └──────────────────────────────────────────┘
```

---

## Proof of Correctness

```
  Claim: At each index i, maxEndingHere correctly stores
  the maximum subarray sum ending at i.

  Proof by induction:
  Base: maxEndingHere = arr[0] ✓ (only subarray ending at 0)

  Inductive step: Assume maxEndingHere = max subarray sum
  ending at i-1. At index i:

  Any subarray ending at i either:
  (a) Contains only arr[i] → sum = arr[i]
  (b) Extends a subarray ending at i-1 → sum = dp[i-1] + arr[i]

  maxEndingHere = MAX(arr[i], maxEndingHere + arr[i])
  covers both cases. ✓

  Since maxSoFar = MAX over all maxEndingHere values,
  it equals the maximum over all subarrays. ✓
```

---

## Handling All-Negative Arrays

```
  arr = [-5, -3, -8, -1, -4]

  Version 1 (MAX-based):
  i=0: endHere=-5, soFar=-5
  i=1: max(-3, -5+(-3)=-8)=-3 → START NEW, soFar=-3
  i=2: max(-8, -3+(-8)=-11)=-8 → START NEW, soFar=-3
  i=3: max(-1, -8+(-1)=-9)=-1 → START NEW, soFar=-1
  i=4: max(-4, -1+(-4)=-5)=-4 → START NEW, soFar=-1

  Answer: -1  ✓ (the single element -1)

  Version 2 (reset-to-0):
  i=0: endHere=-5, soFar=-5, reset to 0
  i=1: endHere=-3, soFar=-3, reset to 0
  i=2: endHere=-8, soFar=-3, reset to 0
  i=3: endHere=-1, soFar=-1, reset to 0
  i=4: endHere=-4, soFar=-1, reset to 0

  Answer: -1  ✓ (soFar updated before reset)
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n) — single pass |
| Space | O(1) — two variables |
| Comparisons | 2 per element (2 MAX operations) |
| Total comparisons | 2n |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Algorithm | Kadane's Algorithm |
| Type | Dynamic Programming (space-optimized) |
| Recurrence | dp[i] = max(arr[i], dp[i-1]+arr[i]) |
| Variables | maxEndingHere (current), maxSoFar (global) |
| Decision | Extend current subarray or start new |
| Reset condition | When accumulated sum is negative |
| Time | O(n) |
| Space | O(1) |

---

## Quick Revision Questions

1. **What are the two choices** at each element in Kadane's algorithm?

2. **When does the algorithm decide to start a new subarray?** Write the condition.

3. **How does Kadane's handle all-negative arrays?** Walk through an example.

4. **What is the DP recurrence** that Kadane's optimizes? Why can we use O(1) space?

5. **What are the two versions** of Kadane's algorithm? How do they differ?

6. **Prove informally** why a negative running sum should be discarded.

---

[← Previous: The Maximum Subarray Problem](01-maximum-subarray-problem.md) | [Back to README](../README.md) | [Next: Tracking the Subarray →](03-tracking-the-subarray.md)
