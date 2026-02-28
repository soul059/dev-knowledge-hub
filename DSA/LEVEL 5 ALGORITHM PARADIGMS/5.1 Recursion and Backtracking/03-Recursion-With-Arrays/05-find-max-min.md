# Chapter 5: Find Max/Min Recursively

## Overview
Finding the maximum or minimum element in an array recursively demonstrates two approaches: linear recursion (compare with max of rest) and divide-and-conquer (tournament style).

---

## Approach 1: Linear Recursion

### Pseudocode — Find Maximum
```
function findMax(arr, index=0):
    if index == length(arr) - 1: return arr[index]    // Base: last element
    restMax = findMax(arr, index + 1)                  // Max of rest
    return max(arr[index], restMax)                     // Compare current with rest's max
```

### Trace: findMax([3, 7, 2, 9, 4], 0)

```
findMax(arr, 0): max(3, findMax(arr,1)) = max(3, 9) = 9
findMax(arr, 1): max(7, findMax(arr,2)) = max(7, 9) = 9
findMax(arr, 2): max(2, findMax(arr,3)) = max(2, 9) = 9
findMax(arr, 3): max(9, findMax(arr,4)) = max(9, 4) = 9
findMax(arr, 4): return 4  ← BASE (last element)

Unwinding: 4 → max(9,4)=9 → max(2,9)=9 → max(7,9)=9 → max(3,9)=9
```

---

## Approach 2: Divide and Conquer (Tournament)

```
function findMax(arr, low, high):
    if low == high: return arr[low]              // Single element
    mid = (low + high) / 2
    leftMax = findMax(arr, low, mid)             // Max of left half
    rightMax = findMax(arr, mid + 1, high)       // Max of right half
    return max(leftMax, rightMax)
```

```
Tournament style for [3, 7, 2, 9, 4]:

            max = 9
           /       \
       max=7      max=9
       / \        /   \
     [3] [7]   max=9  [4]
      3   7    / \     4
             [2] [9]
              2   9

Like a sports tournament — compare pairs, winners advance!
```

---

## Finding Both Max and Min

```
function findMinMax(arr, low, high):
    if low == high: return (arr[low], arr[low])    // Single element
    if high == low + 1:                             // Two elements
        return (min(arr[low],arr[high]), max(arr[low],arr[high]))
    
    mid = (low + high) / 2
    (lMin, lMax) = findMinMax(arr, low, mid)
    (rMin, rMax) = findMinMax(arr, mid+1, high)
    return (min(lMin, rMin), max(lMax, rMax))

// Uses 3n/2 - 2 comparisons (optimal!)
// vs 2(n-1) for separate min and max scans
```

---

## Complexity

| Approach | Time | Space | Comparisons |
|----------|------|-------|-------------|
| Linear | O(n) | O(n) | n-1 |
| Divide & conquer | O(n) | O(log n) | n-1 |
| Min AND Max (D&C) | O(n) | O(log n) | 3n/2 - 2 |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Linear approach | Compare current with max of rest |
| Tournament approach | Divide in half, compare winners |
| Both min & max | Return tuple, optimal 3n/2 comparisons |
| Key insight | max(a, max_of_rest) builds answer bottom-up |

---

## Quick Revision Questions

1. **What is the base case** for finding max in an array recursively?
2. **Why does the tournament approach** use O(log n) space?
3. **How many comparisons** does finding both min and max require optimally?
4. **Trace findMax([5, 1, 8, 3], 0, 3)** using divide and conquer.
5. **How would you modify** findMax to also return the index of the maximum?
6. **Which approach is better** for very large arrays and why?

---

[← Previous: Array Sum](04-array-sum.md) | [Next: First and Last Occurrence →](06-first-and-last-occurrence.md)

[← Back to README](../README.md)
