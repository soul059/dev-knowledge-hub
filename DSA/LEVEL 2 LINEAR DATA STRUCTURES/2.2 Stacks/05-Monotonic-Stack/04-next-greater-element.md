# Chapter 4: Next Greater Element

[← Previous: Decreasing Stack](03-decreasing-stack.md) | [Next: Next Smaller Element →](05-next-smaller-element.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)

---

## Overview

The **Next Greater Element (NGE)** problem asks: for each element in an array, find the first element to its right that is **strictly greater**. This is the definitive monotonic stack problem and appears in many variations across coding interviews.

---

## Problem Statement

```
Given: arr = [4, 5, 2, 10, 8]

For each element, find the next element to the right that is greater:

  arr[0]=4  → next greater: 5   (at index 1)
  arr[1]=5  → next greater: 10  (at index 3)
  arr[2]=2  → next greater: 10  (at index 3)
  arr[3]=10 → next greater: -1  (none exists)
  arr[4]=8  → next greater: -1  (none exists)

Output: [5, 10, 10, -1, -1]
```

---

## Brute Force vs Monotonic Stack

```
BRUTE FORCE — O(n²):
  FOR each element i:
    FOR j = i+1 TO n-1:
      IF arr[j] > arr[i]:
        result[i] = arr[j]
        BREAK

MONOTONIC STACK — O(n):
  Use a decreasing stack.
  When element is popped, the current element is its NGE.
```

---

## Algorithm

```
FUNCTION nextGreaterElement(arr):
    n ← length(arr)
    result ← array of size n, fill with -1
    stack ← empty stack    // stores indices
    
    FOR i = 0 TO n-1:
        // Current element is NGE for all smaller stack elements
        WHILE stack is NOT empty AND arr[i] > arr[stack.top()]:
            idx ← stack.pop()
            result[idx] ← arr[i]
        
        stack.push(i)
    
    // Elements remaining in stack have no NGE → already -1
    RETURN result
```

---

## Detailed Trace

### Input: arr = [4, 5, 2, 10, 8]

```
═══════════════════════════════════════════════════════
  i=0, arr[0]=4
═══════════════════════════════════════════════════════
  Stack empty → Push 0
  Stack: [0]     Values: [4]
  
  ┌───┐
  │ 4 │ (idx 0)
  └───┘

═══════════════════════════════════════════════════════
  i=1, arr[1]=5
═══════════════════════════════════════════════════════
  5 > arr[0]=4? YES → Pop 0, result[0] = 5
  Stack empty → Push 1
  Stack: [1]     Values: [5]
  
  result = [5, -1, -1, -1, -1]

═══════════════════════════════════════════════════════
  i=2, arr[2]=2
═══════════════════════════════════════════════════════
  2 > arr[1]=5? NO → Push 2
  Stack: [1, 2]     Values: [5, 2]
  
  ┌───┐
  │ 2 │ (idx 2)
  │ 5 │ (idx 1)
  └───┘

═══════════════════════════════════════════════════════
  i=3, arr[3]=10
═══════════════════════════════════════════════════════
  10 > arr[2]=2? YES → Pop 2, result[2] = 10
  10 > arr[1]=5? YES → Pop 1, result[1] = 10
  Stack empty → Push 3
  Stack: [3]     Values: [10]
  
  result = [5, 10, 10, -1, -1]

═══════════════════════════════════════════════════════
  i=4, arr[4]=8
═══════════════════════════════════════════════════════
  8 > arr[3]=10? NO → Push 4
  Stack: [3, 4]     Values: [10, 8]

═══════════════════════════════════════════════════════
  END: Stack = [3, 4] → indices 3, 4 have no NGE
═══════════════════════════════════════════════════════

FINAL RESULT: [5, 10, 10, -1, -1]  ✓
```

---

## Variation 1: Next Greater Element in Circular Array

In a **circular array**, after the last element, we wrap around to the beginning.

```
Input: [1, 2, 1]
Circular: ..., 1, 2, 1, 1, 2, 1, ...

  arr[0]=1 → next greater: 2
  arr[1]=2 → next greater: -1 (no element > 2 even with wrap)
  arr[2]=1 → next greater: 2 (wrapping: next is arr[0]=1, then arr[1]=2)

Output: [2, -1, 2]

Trick: Iterate through the array TWICE (indices 0 to 2n-1)
       Use i % n as the actual index.
```

```
FUNCTION nextGreaterCircular(arr):
    n ← length(arr)
    result ← array of size n, fill with -1
    stack ← empty stack
    
    // Process 2 rounds
    FOR i = 0 TO 2*n - 1:
        actual ← i % n
        
        WHILE stack NOT empty AND arr[actual] > arr[stack.top()]:
            idx ← stack.pop()
            result[idx] ← arr[actual]
        
        // Only push indices from first round
        IF i < n:
            stack.push(i)
    
    RETURN result
```

### Trace for circular [1, 2, 1]:

```
Round 1 (i=0,1,2):
  i=0: Push 0                    Stack: [0]
  i=1: 2>1 → Pop 0, res[0]=2    Stack: [1]
       Push 1
  i=2: 1<2 → Push 2             Stack: [1, 2]

Round 2 (i=3,4,5):
  i=3, actual=0: arr[0]=1 < arr[2]=1, < arr[1]=2 → no pops
  i=4, actual=1: arr[1]=2 > arr[2]=1 → Pop 2, res[2]=2
       2 ≤ 2 → no more pops
  i=5, actual=2: no pops

Result: [2, -1, 2]  ✓
```

---

## Variation 2: Next Greater Element I (Two Arrays)

Given two arrays `nums1` (subset of `nums2`), find NGE of each `nums1[i]` in `nums2`.

```
nums1 = [4, 1, 2]
nums2 = [1, 3, 4, 2]

Find NGE of each nums1 element within nums2:
  4 → appears at index 2 in nums2, next greater: -1
  1 → appears at index 0 in nums2, next greater: 3
  2 → appears at index 3 in nums2, next greater: -1

Output: [-1, 3, -1]

FUNCTION nextGreaterI(nums1, nums2):
    // Build NGE map for all elements in nums2
    map ← empty hash map
    stack ← empty stack
    
    FOR each num in nums2:
        WHILE stack NOT empty AND num > stack.top():
            map[stack.pop()] ← num
        stack.push(num)
    
    // Look up results for nums1
    result ← []
    FOR each num in nums1:
        result.append(map.getOrDefault(num, -1))
    
    RETURN result
```

---

## Variation 3: Number of NGEs to the Right

Count how many elements to the right are greater (not just the first one).

```
This is NOT a standard monotonic stack problem.
For counting all greater elements, use:
  - Merge sort-based counting: O(n log n)
  - BIT/Fenwick tree: O(n log n)
```

---

## Complexity Analysis

| Variant | Time | Space |
|---------|------|-------|
| **Standard NGE** | O(n) | O(n) |
| **Circular NGE** | O(n) | O(n) |
| **NGE I (two arrays)** | O(m + n) | O(n) |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Find first greater element to the right |
| **Stack Type** | Monotonically decreasing |
| **Time** | O(n) |
| **Space** | O(n) |
| **Circular Variant** | Iterate 2n times, use i%n |
| **Key Insight** | When popped, incoming element is the answer |

---

## Quick Revision Questions

1. **What type of monotonic stack solves Next Greater Element?**
   > Monotonically decreasing stack — pop when incoming is greater.

2. **What value do elements remaining in the stack at the end get?**
   > -1, since no greater element exists to their right.

3. **How do you handle circular arrays?**
   > Iterate 2n times using `i % n` as the index. Only push during the first n iterations.

4. **For arr = [3, 1, 2, 4], what is the NGE array?**
   > [4, 2, 4, -1]. 3→4, 1→2, 2→4, 4→none.

5. **Why is the while loop inside the for loop still O(n) overall?**
   > Each element is pushed once and popped at most once, so total operations across all iterations ≤ 2n.

---

[← Previous: Decreasing Stack](03-decreasing-stack.md) | [Next: Next Smaller Element →](05-next-smaller-element.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)
