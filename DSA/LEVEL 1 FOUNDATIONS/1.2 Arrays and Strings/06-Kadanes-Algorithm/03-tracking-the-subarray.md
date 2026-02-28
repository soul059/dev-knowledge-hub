# Chapter 3: Tracking the Subarray

[← Previous: Kadane's Algorithm Explained](02-kadanes-algorithm.md) | [Back to README](../README.md) | [Next: Minimum Subarray Sum →](04-minimum-subarray-sum.md)

---

## Overview

Kadane's Algorithm returns the **maximum sum**, but many problems also ask: "Which subarray achieves this sum?" This chapter shows how to track the start and end indices of the maximum subarray alongside Kadane's.

---

## Algorithm: Kadane with Indices

```
FUNCTION kadaneWithIndices(arr, n):
    maxEndingHere = arr[0]
    maxSoFar = arr[0]

    start = 0          // start of best subarray
    end = 0            // end of best subarray
    tempStart = 0      // potential start of current subarray

    FOR i = 1 TO n-1:
        IF arr[i] > maxEndingHere + arr[i]:
            // Start new subarray
            maxEndingHere = arr[i]
            tempStart = i
        ELSE:
            // Extend current subarray
            maxEndingHere = maxEndingHere + arr[i]

        IF maxEndingHere > maxSoFar:
            maxSoFar = maxEndingHere
            start = tempStart
            end = i

    RETURN (maxSoFar, start, end)
```

---

## Understanding the Variables

```
  ┌────────────────────────────────────────────────────────┐
  │  maxEndingHere : best sum ending at current position   │
  │  maxSoFar      : best sum found anywhere so far        │
  │                                                        │
  │  tempStart : where the current candidate subarray      │
  │              started (resets when we start new)         │
  │  start     : where the best subarray starts            │
  │  end       : where the best subarray ends              │
  │                                                        │
  │  tempStart gets "promoted" to start when the           │
  │  current subarray becomes the new global best.         │
  └────────────────────────────────────────────────────────┘
```

---

## Detailed Trace

```
  arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

  Init: endHere=-2, soFar=-2, start=0, end=0, tempStart=0

  ┌─────┬────────┬────────────┬────────┬───────────┬───────┬─────┬──────────┐
  │  i  │ arr[i] │ Decision   │endHere │ soFar     │tStart │start│ end      │
  ├─────┼────────┼────────────┼────────┼───────────┼───────┼─────┼──────────┤
  │  1  │    1   │ 1>-2+1=-1  │    1   │ 1>-2→1   │   1   │  1  │  1       │
  │     │        │ START NEW  │        │ NEW BEST  │       │     │          │
  ├─────┼────────┼────────────┼────────┼───────────┼───────┼─────┼──────────┤
  │  2  │   -3   │ -3<1-3=-2  │   -2   │ -2<1     │   1   │  1  │  1       │
  │     │        │ EXTEND     │        │ no change │       │     │          │
  ├─────┼────────┼────────────┼────────┼───────────┼───────┼─────┼──────────┤
  │  3  │    4   │ 4>-2+4=2   │    4   │ 4>1→4    │   3   │  3  │  3       │
  │     │        │ START NEW  │        │ NEW BEST  │       │     │          │
  ├─────┼────────┼────────────┼────────┼───────────┼───────┼─────┼──────────┤
  │  4  │   -1   │ -1<4-1=3   │    3   │ 3<4      │   3   │  3  │  3       │
  │     │        │ EXTEND     │        │ no change │       │     │          │
  ├─────┼────────┼────────────┼────────┼───────────┼───────┼─────┼──────────┤
  │  5  │    2   │ 2<3+2=5    │    5   │ 5>4→5    │   3   │  3  │  5       │
  │     │        │ EXTEND     │        │ NEW BEST  │       │     │          │
  ├─────┼────────┼────────────┼────────┼───────────┼───────┼─────┼──────────┤
  │  6  │    1   │ 1<5+1=6    │    6   │ 6>5→6    │   3   │  3  │  6       │
  │     │        │ EXTEND     │        │ NEW BEST  │       │     │          │
  ├─────┼────────┼────────────┼────────┼───────────┼───────┼─────┼──────────┤
  │  7  │   -5   │ -5<6-5=1   │    1   │ 1<6      │   3   │  3  │  6       │
  │     │        │ EXTEND     │        │ no change │       │     │          │
  ├─────┼────────┼────────────┼────────┼───────────┼───────┼─────┼──────────┤
  │  8  │    4   │ 4<1+4=5    │    5   │ 5<6      │   3   │  3  │  6       │
  │     │        │ EXTEND     │        │ no change │       │     │          │
  └─────┴────────┴────────────┴────────┴───────────┴───────┴─────┴──────────┘

  Result: maxSoFar=6, start=3, end=6
  Subarray: arr[3..6] = [4, -1, 2, 1]  ✓
```

---

## Visual of Start/End Tracking

```
  arr:       -2   1  -3   4  -1   2   1  -5   4
  index:      0   1   2   3   4   5   6   7   8
  
  tempStart:  0   1   1   3   3   3   3   3   3
                  ↑       ↑
                  new     new (restarted at 3)
                  
  start:      0   1   1   3   3   3   3   3   3
  end:        0   1   1   3   3   5   6   6   6
                              ↑       ↑
                              |       end updated as sum grows
                              start settled at 3

  Best subarray: index 3 to 6 = [4, -1, 2, 1]
```

---

## Returning the Subarray

```
FUNCTION getMaxSubarray(arr, n):
    (maxSum, start, end) = kadaneWithIndices(arr, n)
    subarray = arr[start .. end]
    RETURN subarray

  // For arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]:
  // Returns [4, -1, 2, 1]
```

---

## Edge Case: Multiple Subarrays with Same Max Sum

```
  arr = [1, 2, -3, 1, 2]

  Subarray [1, 2] (indices 0-1) → sum = 3
  Subarray [1, 2] (indices 3-4) → sum = 3

  Kadane's will return the FIRST one found (indices 0-1).
  To find ALL, you'd need additional processing.
```

---

## Edge Case: All Negative

```
  arr = [-5, -2, -8, -1, -6]

  Trace:
  i=0: endHere=-5, soFar=-5, tempStart=0, start=0, end=0
  i=1: -2>-5+(-2)=-7 → START NEW, endHere=-2, soFar=-2
       tempStart=1, start=1, end=1
  i=2: -8<-2+(-8)=-10 → neither helps... max(-8,-10)=-8
       START NEW, endHere=-8, tempStart=2
       soFar stays -2
  i=3: -1>-8+(-1)=-9 → START NEW, endHere=-1, soFar=-1
       tempStart=3, start=3, end=3
  i=4: -6<-1+(-6)=-7 → max(-6,-7)=-6
       START NEW, endHere=-6, tempStart=4
       soFar stays -1

  Result: maxSoFar=-1, start=3, end=3
  Subarray: [-1]  ✓ (single least-negative element)
```

---

## Printing All Maximum Subarrays

```
FUNCTION allMaxSubarrays(arr, n):
    // First pass: find the max sum
    maxSum = kadane(arr, n)

    // Second pass: find all subarrays with that sum
    results = []

    FOR i = 0 TO n-1:
        currentSum = 0
        FOR j = i TO n-1:
            currentSum = currentSum + arr[j]
            IF currentSum == maxSum:
                APPEND (i, j) TO results

    RETURN results
```

Note: Second pass is O(n²) — finding ALL is inherently harder.

---

## Practical Tip: Print Without Extra Variables

If you just want the subarray (not the indices), you can reconstruct it after knowing the sum:

```
FUNCTION printMaxSubarray(arr, n, maxSum):
    currentSum = 0
    FOR i = 0 TO n-1:
        currentSum = currentSum + arr[i]
        IF currentSum == maxSum:
            // Found end, now backtrack
            tempSum = 0
            j = i
            WHILE tempSum ≠ maxSum:
                tempSum = tempSum + arr[j]
                j = j - 1
            PRINT arr[j+1 .. i]
            RETURN
        IF currentSum < 0:
            currentSum = 0
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Extra variables needed | tempStart, start, end |
| tempStart resets when | maxEndingHere = arr[i] (new subarray) |
| start/end update when | maxEndingHere > maxSoFar (new global best) |
| Time | O(n) — same as basic Kadane |
| Space | O(1) — just 3 more integers |
| All-negative arrays | Returns single element (least negative) |
| Multiple solutions | Returns first found |

---

## Quick Revision Questions

1. **What is `tempStart`** and when does it reset?

2. **When are `start` and `end` updated?** What triggers the update?

3. **Trace through arr = [5, -4, 3, -2, 1]** tracking all variables. What subarray is returned?

4. **How does the algorithm handle ties** (multiple subarrays with the same max sum)?

5. **Why do we need `tempStart` as a separate variable** from `start`?

6. **Can you find ALL subarrays with the maximum sum** in O(n) time? Why or why not?

---

[← Previous: Kadane's Algorithm Explained](02-kadanes-algorithm.md) | [Back to README](../README.md) | [Next: Minimum Subarray Sum →](04-minimum-subarray-sum.md)
