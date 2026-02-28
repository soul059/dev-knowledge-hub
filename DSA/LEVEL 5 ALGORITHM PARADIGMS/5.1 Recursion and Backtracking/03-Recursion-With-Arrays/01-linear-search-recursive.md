# Chapter 1: Linear Search - Recursive

## Overview
Linear search checks each element sequentially. The recursive version demonstrates the "check current, recurse on rest" pattern — one of the most fundamental recursive patterns for array processing.

---

## Problem Statement
Given an array and a target value, find the index of the target. Return -1 if not found.

---

## Recursive Solution

### Pseudocode
```
function linearSearch(arr, target, index=0):
    if index >= length(arr): return -1     // Base: not found
    if arr[index] == target: return index   // Base: found!
    return linearSearch(arr, target, index + 1)  // Check next
```

### Trace: linearSearch([4, 2, 7, 1, 9], 7, 0)

```
linearSearch(arr, 7, 0)
│  arr[0]=4, not 7
│  linearSearch(arr, 7, 1)
│  │  arr[1]=2, not 7
│  │  linearSearch(arr, 7, 2)
│  │  │  arr[2]=7, FOUND! return 2  ← BASE CASE
│  │  return 2
│  return 2
return 2  ✓

Visualization:
[4, 2, 7, 1, 9]
 ↓
 4≠7 → check rest → [2, 7, 1, 9]
                      ↓
                      2≠7 → check rest → [7, 1, 9]
                                          ↓
                                          7==7 → FOUND at index 2!
```

### Trace: Not Found

```
linearSearch([4, 2, 7], 5, 0)
│  arr[0]=4, not 5 → linearSearch(arr, 5, 1)
│  │  arr[1]=2, not 5 → linearSearch(arr, 5, 2)
│  │  │  arr[2]=7, not 5 → linearSearch(arr, 5, 3)
│  │  │  │  index=3 >= length=3 → return -1  ← NOT FOUND
│  │  │  return -1
│  │  return -1
│  return -1
return -1
```

---

## Finding All Occurrences

```
function findAll(arr, target, index=0):
    if index >= length(arr): return []     // Base: empty list
    rest = findAll(arr, target, index + 1) // Find in rest first
    if arr[index] == target:
        return [index] + rest              // Prepend current index
    return rest

// Alternative: build result going down
function findAll(arr, target, index=0, result=[]):
    if index >= length(arr): return result
    if arr[index] == target:
        result.append(index)
    return findAll(arr, target, index + 1, result)
```

### Trace: findAll([3, 5, 3, 7, 3], 3, 0)

```
findAll(arr, 3, 0)
│  arr[0]=3, match! → [0] + findAll(arr, 3, 1)
│                              │  arr[1]=5, no → findAll(arr, 3, 2)
│                              │                   │  arr[2]=3, match! → [2] + findAll(arr, 3, 3)
│                              │                   │                              │  arr[3]=7, no → findAll(arr, 3, 4)
│                              │                   │                              │                   │  arr[4]=3, match! → [4] + findAll(arr, 3, 5)
│                              │                   │                              │                   │                              │  BASE: return []
│                              │                   │                              │                   │  return [4]
│                              │                   │                              │  return [4]
│                              │                   │  return [2, 4]
│                              │  return [2, 4]
│  return [0, 2, 4]  ✓
```

---

## Complexity Analysis

```
┌──────────────────────────────────────────────┐
│         COMPLEXITY ANALYSIS                  │
│                                              │
│  Time:  O(n) — worst case checks all n       │
│  Space: O(n) — n stack frames                │
│                                              │
│  Best case:  O(1) — found at index 0         │
│  Worst case: O(n) — not found or at end      │
│  Average:    O(n/2) = O(n)                   │
│                                              │
│  vs Iterative: Same time, but O(1) space     │
│                                              │
│  Recurrence: T(n) = T(n-1) + O(1) = O(n)    │
└──────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Base cases | Found (return index) or exhausted (return -1) |
| Recursive case | Check arr[i], recurse on i+1 |
| Time | O(n) |
| Space | O(n) recursive, O(1) iterative |
| Pattern | Check current + recurse rest |
| Variant | Find all occurrences returns list |

---

## Quick Revision Questions

1. **What are the two base cases** in recursive linear search?
2. **What is the space overhead** compared to iterative linear search?
3. **How do you modify** the algorithm to find ALL occurrences?
4. **What is the recurrence relation** for recursive linear search?
5. **Trace the search** for value 8 in [1, 3, 8, 5, 2].
6. **Can linear search be faster** than O(n) in worst case? Why or why not?

---

[← Previous: Reverse a String](../02-Basic-Recursion/06-reverse-a-string.md) | [Next: Binary Search Recursive →](02-binary-search-recursive.md)

[← Back to README](../README.md)
