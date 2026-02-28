# Chapter 3: Sum of Array

## Overview
Computing the sum of an array recursively demonstrates how to process **collections** one element at a time. This pattern generalizes to many array processing problems.

---

## Problem Statement
Given an array of numbers, find the sum of all elements using recursion.

---

## Approach 1: Process from Left

### Pseudocode
```
function arraySum(arr, index):
    if index >= length(arr):    // Base case: past the end
        return 0
    return arr[index] + arraySum(arr, index + 1)
```

### Trace: arraySum([3, 7, 2, 5], 0)

```
arraySum([3,7,2,5], 0)
│  return 3 + arraySum([3,7,2,5], 1)
│              │
│              return 7 + arraySum([3,7,2,5], 2)
│                          │
│                          return 2 + arraySum([3,7,2,5], 3)
│                                      │
│                                      return 5 + arraySum([3,7,2,5], 4)
│                                                  │
│                                                  index=4 >= length=4
│                                                  return 0  ← BASE
│                                      return 5 + 0 = 5
│                          return 2 + 5 = 7
│              return 7 + 7 = 14
│  return 3 + 14 = 17  ✓

Visualization:
  [3, 7, 2, 5]
   ↓
   3 + sum([7, 2, 5])
        ↓
        7 + sum([2, 5])
             ↓
             2 + sum([5])
                  ↓
                  5 + sum([])
                       ↓
                       0  (base)
```

---

## Approach 2: Process from Right

### Pseudocode
```
function arraySum(arr, n):    // n = number of elements to consider
    if n <= 0:                // Base case: no elements
        return 0
    return arr[n-1] + arraySum(arr, n - 1)
```

### Trace: arraySum([3, 7, 2, 5], 4)

```
arraySum(arr, 4): return arr[3] + arraySum(arr, 3) = 5 + 12 = 17
arraySum(arr, 3): return arr[2] + arraySum(arr, 2) = 2 + 10 = 12
arraySum(arr, 2): return arr[1] + arraySum(arr, 1) = 7 + 3  = 10
arraySum(arr, 1): return arr[0] + arraySum(arr, 0) = 3 + 0  = 3
arraySum(arr, 0): return 0  ← BASE CASE
```

---

## Approach 3: Divide and Conquer

### Pseudocode
```
function arraySum(arr, low, high):
    if low > high: return 0           // Empty range
    if low == high: return arr[low]   // Single element
    mid = (low + high) / 2
    leftSum = arraySum(arr, low, mid)
    rightSum = arraySum(arr, mid + 1, high)
    return leftSum + rightSum
```

### Trace: arraySum([3, 7, 2, 5], 0, 3)

```
                    sum(0,3)
                   /        \
              sum(0,1)      sum(2,3)
             /      \      /      \
         sum(0,0) sum(1,1) sum(2,2) sum(3,3)
            |        |        |        |
            3        7        2        5
             \      /          \      /
               10                7
                 \              /
                      17

Divide and conquer: split array in half each time
```

---

## Call Stack Visualization

For `arraySum([3, 7, 2, 5], 0)` at maximum depth:

```
┌─────────────────────┐
│ arraySum(arr, 4)    │ ← TOP (returns 0)
│ index=4, BASE CASE  │
├─────────────────────┤
│ arraySum(arr, 3)    │
│ index=3, arr[3]=5   │
├─────────────────────┤
│ arraySum(arr, 2)    │
│ index=2, arr[2]=2   │
├─────────────────────┤
│ arraySum(arr, 1)    │
│ index=1, arr[1]=7   │
├─────────────────────┤
│ arraySum(arr, 0)    │ ← BOTTOM (initial call)
│ index=0, arr[0]=3   │
└─────────────────────┘
```

---

## Tail-Recursive Version

```
function arraySum(arr, index, accumulator=0):
    if index >= length(arr):
        return accumulator
    return arraySum(arr, index + 1, accumulator + arr[index])
```

```
Trace: arraySum([3,7,2,5], 0, 0)
  → arraySum([3,7,2,5], 1, 3)     // 0+3
  → arraySum([3,7,2,5], 2, 10)    // 3+7
  → arraySum([3,7,2,5], 3, 12)    // 10+2
  → arraySum([3,7,2,5], 4, 17)    // 12+5
  → return 17  (base: index >= length)
```

---

## Complexity Analysis

```
┌──────────────────────────────────────────────┐
│         COMPLEXITY ANALYSIS                  │
│                                              │
│  Linear approach (left/right):               │
│    Time:  O(n) — visit each element once     │
│    Space: O(n) — n recursive stack frames    │
│                                              │
│  Divide and conquer:                         │
│    Time:  O(n) — still visit each element    │
│    Space: O(log n) — tree depth is log n     │
│                                              │
│  Tail-recursive:                             │
│    Time:  O(n)                               │
│    Space: O(n) without TCO, O(1) with TCO    │
│                                              │
│  Iterative (comparison):                     │
│    Time:  O(n)                               │
│    Space: O(1)                               │
└──────────────────────────────────────────────┘
```

---

## Pattern: Recursive Array Processing

This same pattern works for many array problems:

```
function process(arr, index):
    if index >= length(arr): return IDENTITY
    return COMBINE(arr[index], process(arr, index + 1))

Where:
  Sum:     IDENTITY = 0,    COMBINE = +
  Product: IDENTITY = 1,    COMBINE = ×
  Max:     IDENTITY = -INF, COMBINE = max()
  Min:     IDENTITY = +INF, COMBINE = min()
  Count:   IDENTITY = 0,    COMBINE = (condition ? 1 : 0) +
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Sum all elements in array |
| Base case | Empty range → return 0 |
| Recursive case | arr[i] + sum(rest) |
| Time | O(n) |
| Space | O(n) linear, O(log n) divide-and-conquer |
| Pattern | Process one element + recurse on rest |
| Generalization | Works for product, max, min, count |

---

## Quick Revision Questions

1. **What is the base case** for recursive array sum?
2. **What is the advantage** of the divide-and-conquer approach for array sum?
3. **Convert the linear recursive sum** to a tail-recursive version.
4. **How would you modify** the pattern to find the product of array elements?
5. **What is the space complexity** of the linear recursive approach and why?
6. **Trace arraySum([1, 4, 6, 2, 8], 0)** showing each recursive call.

---

[← Previous: Fibonacci](02-fibonacci.md) | [Next: Power Function →](04-power-function.md)

[← Back to README](../README.md)
