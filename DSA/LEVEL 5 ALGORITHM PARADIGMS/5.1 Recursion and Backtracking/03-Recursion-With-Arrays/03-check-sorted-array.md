# Chapter 3: Check Sorted Array

## Overview
Checking whether an array is sorted recursively demonstrates the "verify a property across the entire array" pattern by checking adjacent pairs one step at a time.

---

## Problem Statement
Given an array, determine if it is sorted in non-decreasing order using recursion.

---

## Recursive Solution

### Pseudocode
```
function isSorted(arr, index=0):
    if index >= length(arr) - 1: return true      // Base: 0 or 1 elements left
    if arr[index] > arr[index + 1]: return false   // Found out-of-order pair
    return isSorted(arr, index + 1)                 // Check rest
```

### Trace: isSorted([1, 3, 5, 7, 9], 0) — Sorted

```
isSorted([1,3,5,7,9], 0)
│  arr[0]=1 <= arr[1]=3 ✓
│  isSorted(arr, 1)
│  │  arr[1]=3 <= arr[2]=5 ✓
│  │  isSorted(arr, 2)
│  │  │  arr[2]=5 <= arr[3]=7 ✓
│  │  │  isSorted(arr, 3)
│  │  │  │  arr[3]=7 <= arr[4]=9 ✓
│  │  │  │  isSorted(arr, 4)
│  │  │  │  │  index=4 >= length-1=4 → return true ← BASE
│  │  │  │  return true
│  │  │  return true
│  │  return true
│  return true
return true ✓

Check: [1≤3] ✓ [3≤5] ✓ [5≤7] ✓ [7≤9] ✓ → SORTED
```

### Trace: isSorted([1, 3, 8, 5, 9], 0) — NOT Sorted

```
isSorted([1,3,8,5,9], 0)
│  arr[0]=1 <= arr[1]=3 ✓
│  isSorted(arr, 1)
│  │  arr[1]=3 <= arr[2]=8 ✓
│  │  isSorted(arr, 2)
│  │  │  arr[2]=8 > arr[3]=5 ✗ → return false ← EARLY EXIT!
│  │  return false
│  return false
return false ✓

[1, 3, |8, 5|, 9]
        ↑  ↑
        Out of order! Stop immediately.
```

---

## Alternative: Check from End

```
function isSorted(arr, n):
    if n <= 1: return true                    // 0 or 1 element
    if arr[n-1] < arr[n-2]: return false      // Last pair out of order
    return isSorted(arr, n - 1)                // Check rest
```

---

## Divide and Conquer Approach

```
function isSorted(arr, low, high):
    if low >= high: return true
    mid = (low + high) / 2
    return isSorted(arr, low, mid) AND
           isSorted(arr, mid+1, high) AND
           arr[mid] <= arr[mid+1]         // Bridge check!
```

```
[1, 3, 5, 7, 9]
     /         \
[1, 3, 5]   [7, 9]       Check each half is sorted
    ↕           ↕          AND arr[2] <= arr[3] (5 ≤ 7) ✓
  sorted      sorted
```

---

## Complexity Analysis

```
Time:  O(n) — check each adjacent pair (worst case all)
Space: O(n) — recursive stack depth
Best case: O(1) — first pair is out of order

Recurrence: T(n) = T(n-1) + O(1) = O(n)
```

---

## Variations

| Variation | Modification |
|-----------|-------------|
| Strictly increasing | Change `<=` to `<` |
| Descending order | Change `<=` to `>=` |
| Check both directions | Return "ascending", "descending", or "neither" |
| Nearly sorted | Count out-of-order pairs |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Base case | 0 or 1 elements remaining → true |
| Early exit | Out-of-order pair → false |
| Recursive case | Check pair + recurse on rest |
| Time | O(n) |
| Space | O(n) |
| Key insight | Only need to check consecutive pairs |

---

## Quick Revision Questions

1. **Why is a single element** considered sorted?
2. **How does early termination** help in the unsorted case?
3. **What change** would check for strictly decreasing order?
4. **Trace isSorted([2, 4, 4, 7], 0)**. Is it sorted?
5. **What is the best case** time complexity and when does it occur?
6. **How does the divide-and-conquer** approach need a "bridge check"?

---

[← Previous: Binary Search Recursive](02-binary-search-recursive.md) | [Next: Array Sum →](04-array-sum.md)

[← Back to README](../README.md)
