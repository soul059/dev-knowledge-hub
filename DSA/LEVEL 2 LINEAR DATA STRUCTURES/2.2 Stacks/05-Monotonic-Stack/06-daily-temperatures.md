# Chapter 6: Daily Temperatures

[← Previous: Next Smaller Element](05-next-smaller-element.md) | [Next: Stock Span Problem →](../06-Stock-Span-Problems/01-stock-span-problem.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)

---

## Overview

The **Daily Temperatures** problem (LeetCode #739) is a perfect real-world application of the monotonic stack. Given a list of daily temperatures, find how many days you have to wait until a **warmer** day. This is essentially "Next Greater Element" but returns **distances** instead of values.

---

## Problem Statement

```
Given: temperatures = [73, 74, 75, 71, 69, 72, 76, 73]

For each day, how many days until a warmer temperature?

  Day 0: 73 → warmer on Day 1 (74), wait 1 day
  Day 1: 74 → warmer on Day 2 (75), wait 1 day
  Day 2: 75 → warmer on Day 6 (76), wait 4 days
  Day 3: 71 → warmer on Day 5 (72), wait 2 days
  Day 4: 69 → warmer on Day 5 (72), wait 1 day
  Day 5: 72 → warmer on Day 6 (76), wait 1 day
  Day 6: 76 → no warmer day, wait 0 days
  Day 7: 73 → no warmer day, wait 0 days

Output: [1, 1, 4, 2, 1, 1, 0, 0]
```

---

## Connection to Next Greater Element

```
┌──────────────────────────────────────────────────────┐
│  Daily Temperatures = NGE with DISTANCE calculation  │
│                                                      │
│  NGE:   Pop idx, result[idx] = arr[i]    (VALUE)     │
│  Daily: Pop idx, result[idx] = i - idx   (DISTANCE)  │
│                                                      │
│  Same monotonically decreasing stack.                │
│  Same pop condition.                                 │
│  Different result computation.                       │
└──────────────────────────────────────────────────────┘
```

---

## Algorithm

```
FUNCTION dailyTemperatures(temps):
    n ← length(temps)
    result ← array of size n, fill with 0
    stack ← empty stack    // stores indices, decreasing by temp
    
    FOR i = 0 TO n-1:
        WHILE stack NOT empty AND temps[i] > temps[stack.top()]:
            idx ← stack.pop()
            result[idx] ← i - idx    // Distance!
        stack.push(i)
    
    RETURN result
```

---

## Detailed Trace

### Input: [73, 74, 75, 71, 69, 72, 76, 73]

```
┌──────┬──────┬───────────────────────────────┬────────────┬─────────────────────┐
│ Day  │ Temp │ Action                        │ Stack(idx) │ result              │
├──────┼──────┼───────────────────────────────┼────────────┼─────────────────────┤
│  0   │  73  │ Push 0                        │ [0]        │ [0,0,0,0,0,0,0,0]  │
│  1   │  74  │ 74>73 → Pop 0, res[0]=1-0=1  │ [1]        │ [1,0,0,0,0,0,0,0]  │
│      │      │ Push 1                        │            │                     │
│  2   │  75  │ 75>74 → Pop 1, res[1]=2-1=1  │ [2]        │ [1,1,0,0,0,0,0,0]  │
│      │      │ Push 2                        │            │                     │
│  3   │  71  │ 71<75 → Push 3               │ [2,3]      │ [1,1,0,0,0,0,0,0]  │
│  4   │  69  │ 69<71 → Push 4               │ [2,3,4]    │ [1,1,0,0,0,0,0,0]  │
│  5   │  72  │ 72>69 → Pop 4, res[4]=5-4=1  │ [2,3,5]    │ [1,1,0,0,1,0,0,0]  │
│      │      │ 72>71 → Pop 3, res[3]=5-3=2  │            │                     │
│      │      │ 72<75 → Push 5               │            │                     │
│  6   │  76  │ 76>72 → Pop 5, res[5]=6-5=1  │ [6]        │ [1,1,4,2,1,1,0,0]  │
│      │      │ 76>75 → Pop 2, res[2]=6-2=4  │            │                     │
│      │      │ Push 6                        │            │                     │
│  7   │  73  │ 73<76 → Push 7               │ [6,7]      │ [1,1,4,2,1,1,0,0]  │
└──────┴──────┴───────────────────────────────┴────────────┴─────────────────────┘

Remaining in stack: Days 6,7 → 0 (no warmer day)

Result: [1, 1, 4, 2, 1, 1, 0, 0] ✓
```

### Stack Visualization at Key Points

```
Day 4 (before processing):     Day 5 (after processing):
  ┌────┐                         ┌────┐
  │ 69 │ idx 4                   │ 72 │ idx 5
  │ 71 │ idx 3                   │ 75 │ idx 2
  │ 75 │ idx 2                   └────┘
  └────┘                         
  Decreasing ✓                   69 and 71 were popped by 72

Day 6 (after processing):
  ┌────┐
  │ 76 │ idx 6
  └────┘
  72 and 75 were popped by 76
```

---

## Brute Force Comparison

```
FUNCTION dailyTemps_BruteForce(temps):
    n ← length(temps)
    result ← array of size n, fill with 0
    
    FOR i = 0 TO n-1:
        FOR j = i+1 TO n-1:
            IF temps[j] > temps[i]:
                result[i] ← j - i
                BREAK
    
    RETURN result

Time: O(n²) — much slower for large inputs!
```

---

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| **Brute Force** | O(n²) | O(1) |
| **Monotonic Stack** | O(n) | O(n) |

```
Performance comparison:
  n = 30,000 (typical constraint):
    Brute: ~450,000,000 operations
    Stack: ~60,000 operations
    
  Speed improvement: ~7,500x faster!
```

---

## Variation: Average Waiting Time

Instead of waiting for a single warmer day, find average days until warmer across the week.

```
result = [1, 1, 4, 2, 1, 1, 0, 0]

Excluding days with no warmer day (0s):
  Average of [1, 1, 4, 2, 1, 1] = 10/6 ≈ 1.67 days

Including all:
  Average = 10/8 = 1.25 days
```

---

## Related Problems

```
┌──────────────────────────────────────────────────┐
│  Problems using the same pattern:                │
│                                                  │
│  1. Next Warmer Day → This problem               │
│  2. Days Until Rain → Same with different metric │
│  3. Stock Span → Previous Greater variant        │
│  4. Online Stock Span → Streaming version        │
│  5. Buildings with Ocean View → NSE from right   │
└──────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Days until warmer temperature |
| **Type** | Next Greater Element with distance |
| **Stack** | Monotonically decreasing (by temperature) |
| **Result** | `i - popped_index` (distance) |
| **Time** | O(n) |
| **Space** | O(n) |
| **LeetCode** | #739 |

---

## Quick Revision Questions

1. **How does Daily Temperatures differ from standard NGE?**
   > Instead of recording the greater value, we record the distance: `result[idx] = i - idx`.

2. **What value do days with no warmer temperature get?**
   > 0, since we initialize the result array with zeros, and those indices are never popped.

3. **What does the stack maintain in this problem?**
   > Indices of days whose "next warmer day" hasn't been found yet, in decreasing order of temperature.

4. **For temps = [30, 40, 50, 60], what is the output?**
   > [1, 1, 1, 0]. Each day has the next day as warmer, except the last.

5. **For temps = [60, 50, 40, 30], what is the output?**
   > [0, 0, 0, 0]. Temperatures are strictly decreasing, so no warmer day exists for any.

---

[← Previous: Next Smaller Element](05-next-smaller-element.md) | [Next: Stock Span Problem →](../06-Stock-Span-Problems/01-stock-span-problem.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)
