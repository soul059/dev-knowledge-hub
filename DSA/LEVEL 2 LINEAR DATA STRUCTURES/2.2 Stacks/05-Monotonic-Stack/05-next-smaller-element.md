# Chapter 5: Next Smaller Element

[← Previous: Next Greater Element](04-next-greater-element.md) | [Next: Daily Temperatures →](06-daily-temperatures.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)

---

## Overview

The **Next Smaller Element (NSE)** problem is the mirror of NGE: for each element, find the first element to its right that is **strictly smaller**. This uses a **monotonically increasing stack**.

---

## Problem Statement

```
Given: arr = [4, 8, 5, 2, 25]

For each element, find the first smaller element to the right:

  arr[0]=4  → next smaller: 2   (at index 3)
  arr[1]=8  → next smaller: 5   (at index 2)
  arr[2]=5  → next smaller: 2   (at index 3)
  arr[3]=2  → next smaller: -1  (none)
  arr[4]=25 → next smaller: -1  (none)

Output: [2, 5, 2, -1, -1]
```

---

## Algorithm

```
FUNCTION nextSmallerElement(arr):
    n ← length(arr)
    result ← array of size n, fill with -1
    stack ← empty stack    // stores indices (increasing stack)
    
    FOR i = 0 TO n-1:
        WHILE stack NOT empty AND arr[i] < arr[stack.top()]:
            idx ← stack.pop()
            result[idx] ← arr[i]
        stack.push(i)
    
    RETURN result
```

---

## Trace: arr = [13, 7, 6, 12]

```
┌──────┬───────┬────────────────────────────┬──────────┬──────────────────┐
│ i    │ arr[i]│ Action                     │ Stack(i) │ result           │
├──────┼───────┼────────────────────────────┼──────────┼──────────────────┤
│  0   │  13   │ Push 0                     │ [0]      │ [-1,-1,-1,-1]    │
│  1   │  7    │ 7<13 → Pop 0, res[0]=7     │ [1]      │ [ 7,-1,-1,-1]    │
│      │       │ Push 1                     │          │                  │
│  2   │  6    │ 6<7 → Pop 1, res[1]=6      │ [2]      │ [ 7, 6,-1,-1]    │
│      │       │ Push 2                     │          │                  │
│  3   │  12   │ 12>6 → Push 3              │ [2,3]    │ [ 7, 6,-1,-1]    │
└──────┴───────┴────────────────────────────┴──────────┴──────────────────┘

Remaining: 2,3 → -1

Result: [7, 6, -1, -1]

Verify:
  13 → first smaller to right: 7  ✓
  7  → first smaller to right: 6  ✓ 
  6  → none smaller to right: -1  ✓
  12 → none smaller to right: -1  ✓
```

---

## All Four Variants at a Glance

```
┌──────────────────────────┬─────────────┬──────────────┬────────────────┐
│ Problem                  │ Stack Type  │ Scan         │ Pop Condition  │
├──────────────────────────┼─────────────┼──────────────┼────────────────┤
│ Next Greater Element     │ Decreasing  │ L → R        │ incoming > top │
│ Next Smaller Element     │ Increasing  │ L → R        │ incoming < top │
│ Previous Greater Element │ Decreasing  │ L → R        │ top ≤ incoming │
│ Previous Smaller Element │ Increasing  │ L → R        │ top ≥ incoming │
└──────────────────────────┴─────────────┴──────────────┴────────────────┘

Key Relationships:
  Next ____:     Answer found when element is POPPED
  Previous ____: Answer is the REMAINING stack top after popping
  Greater:       Use DECREASING stack
  Smaller:       Use INCREASING stack
```

---

## Combined Example: All Four for One Array

```
arr = [3, 7, 1, 5, 4, 2, 6]

┌───────┬─────┬─────┬─────┬─────┐
│ Index │ NGE │ NSE │ PGE │ PSE │
├───────┼─────┼─────┼─────┼─────┤
│ 3     │  7  │  1  │  -1 │  -1 │
│ 7     │ -1  │  1  │  -1 │  3  │
│ 1     │  5  │ -1  │   7 │  -1 │
│ 5     │  6  │  4  │   7 │  1  │
│ 4     │  6  │  2  │   5 │  1  │
│ 2     │  6  │ -1  │   4 │  1  │
│ 6     │ -1  │ -1  │   7 │  2  │
└───────┴─────┴─────┴─────┴─────┘
```

---

## Variation: Next Smaller or Equal

Change `<` to `<=` in the condition:

```
WHILE stack NOT empty AND arr[i] <= arr[stack.top()]:
    //              ^^
    // Now pops for equal elements too
```

---

## Application: Sum of Subarray Minimums

A powerful application of NSE and PSE: find the sum of minimum values across all subarrays.

```
Idea:
  For each element arr[i], determine how many subarrays
  have arr[i] as their minimum.
  
  left[i]  = distance to Previous Smaller Element
  right[i] = distance to Next Smaller Element
  
  Count of subarrays where arr[i] is minimum:
    count = left[i] * right[i]
  
  Contribution of arr[i] = arr[i] * left[i] * right[i]
  
  Total sum = Σ arr[i] * left[i] * right[i]

Example: arr = [3, 1, 2, 4]
  
  3: PSE=-1(idx -1), NSE=1(idx 1) → left=1, right=1 → 3*1*1=3
  1: PSE=-1(idx -1), NSE=-1(idx 4)→ left=2, right=3 → 1*2*3=6
  2: PSE=1(idx 1),   NSE=-1(idx 4)→ left=1, right=2 → 2*1*2=4
  4: PSE=2(idx 2),   NSE=-1(idx 4)→ left=1, right=1 → 4*1*1=4
  
  Total = 3+6+4+4 = 17

  Verify all subarrays:
  [3]=3, [1]=1, [2]=2, [4]=4
  [3,1]=1, [1,2]=1, [2,4]=2
  [3,1,2]=1, [1,2,4]=1
  [3,1,2,4]=1
  Sum = 3+1+2+4+1+1+2+1+1+1 = 17 ✓
```

---

## Complexity Analysis

| Aspect | Complexity |
|--------|-----------|
| **Time** | O(n) |
| **Space** | O(n) |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Find first smaller element to the right |
| **Stack Type** | Monotonically increasing |
| **Pop Condition** | incoming < stack top |
| **Time** | O(n) |
| **Space** | O(n) |
| **Application** | Sum of subarray minimums, histogram areas |

---

## Quick Revision Questions

1. **What type of monotonic stack solves Next Smaller Element?**
   > Monotonically increasing stack — pop when incoming is smaller.

2. **For arr = [5, 3, 4, 2, 7], what is the NSE array?**
   > [3, 2, 2, -1, -1]. 5→3, 3→2, 4→2, 2→none, 7→none.

3. **How does NSE differ from PSE in implementation?**
   > Both use increasing stack scanning L→R. NSE: answer found on pop (incoming is NSE). PSE: answer is the remaining top after popping (top is PSE of current).

4. **How is NSE used in the Sum of Subarray Minimums problem?**
   > NSE gives the right boundary, PSE gives the left boundary. The product of distances tells how many subarrays have each element as minimum.

5. **What changes to find Next Smaller or Equal (≤)?**
   > Change the pop condition from `arr[i] < arr[stack.top()]` to `arr[i] <= arr[stack.top()]`.

---

[← Previous: Next Greater Element](04-next-greater-element.md) | [Next: Daily Temperatures →](06-daily-temperatures.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)
