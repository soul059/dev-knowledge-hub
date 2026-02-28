# Chapter 1: What is Two Pointer?

[← Previous Unit: Array Manipulation](../02-Basic-Array-Techniques/06-array-manipulation.md) | [Back to README](../README.md) | [Next: Opposite Direction Pointers →](02-opposite-direction-pointers.md)

---

## Overview

The two-pointer technique is one of the most powerful and frequently used patterns in array and string problems. It uses two variables (pointers) that traverse the data structure simultaneously, reducing time complexity from O(n²) to O(n) in many cases.

---

## Core Idea

Instead of using nested loops to examine pairs, use two pointers that move intelligently based on conditions.

```
  BRUTE FORCE (O(n²)):              TWO POINTER (O(n)):
  
  FOR i = 0 TO n-1:                 left = 0, right = n-1
    FOR j = i+1 TO n-1:             WHILE left < right:
      check(arr[i], arr[j])           check(arr[left], arr[right])
                                      move left or right
```

---

## Two Main Variants

### Variant 1: Opposite Direction (Converging)

Two pointers start from opposite ends and move toward each other.

```
  ┌───┬───┬───┬───┬───┬───┬───┐
  │   │   │   │   │   │   │   │
  └───┴───┴───┴───┴───┴───┴───┘
   L──────────────────────────R    Start
      L────────────────────R       Step 1
         L──────────────R          Step 2
            L────────R             Step 3
               L──R                Step 4
                LR                 Meet → done!

  Use when:
  • Array is sorted
  • Looking for pairs with a target sum
  • Palindrome checking
  • Container with most water
```

### Variant 2: Same Direction (Sliding/Fast-Slow)

Two pointers start from the same end, one moves faster.

```
  ┌───┬───┬───┬───┬───┬───┬───┐
  │   │   │   │   │   │   │   │
  └───┴───┴───┴───┴───┴───┴───┘
   SF                              Start (Slow, Fast together)
   S  F                            Step 1 (Fast moves ahead)
   S     F                         Step 2
      S     F                      Step 3 (Slow catches up)
      S        F                   Step 4
         S        F                Step 5

  Use when:
  • Removing duplicates in-place
  • Partitioning
  • Finding subarrays with conditions
  • Linked list cycle detection
```

---

## When to Recognize Two-Pointer Problems

```
  ┌─────────────────────────────────────────────────────┐
  │         SIGNALS THAT TWO POINTER WORKS               │
  ├─────────────────────────────────────────────────────┤
  │                                                      │
  │  ✓ Array is SORTED (or can be sorted)                │
  │  ✓ Need to find PAIRS with a property                │
  │  ✓ Need to compare ENDS of array/string              │
  │  ✓ Need to PARTITION or FILTER in-place              │
  │  ✓ Problem mentions "subarray" or "subsequence"      │
  │  ✓ Brute force is O(n²) with nested loops            │
  │  ✓ Need to reduce space from O(n) to O(1)            │
  │                                                      │
  │  ✗ Array is unsorted AND order matters               │
  │  ✗ Need to examine ALL pairs (no shortcut)           │
  │  ✗ Random access patterns needed                     │
  └─────────────────────────────────────────────────────┘
```

---

## Template: Opposite Direction

```
FUNCTION oppositePointers(arr, n):
    left = 0
    right = n - 1
    
    WHILE left < right:
        // Compute something with arr[left] and arr[right]
        result = process(arr[left], arr[right])
        
        IF result meets condition:
            // Found answer or update answer
            record(left, right)
        
        IF need to increase value:
            left = left + 1
        ELSE:
            right = right - 1
    
    RETURN answer
```

---

## Template: Same Direction

```
FUNCTION sameDirectionPointers(arr, n):
    slow = 0
    
    FOR fast = 0 TO n-1:
        IF arr[fast] meets condition:
            arr[slow] = arr[fast]     // or swap
            slow = slow + 1
    
    RETURN slow     // number of valid elements
```

---

## Quick Example: Is Array a Palindrome?

```
FUNCTION isPalindrome(arr, n):
    left = 0
    right = n - 1
    WHILE left < right:
        IF arr[left] != arr[right]:
            RETURN false
        left = left + 1
        right = right - 1
    RETURN true
```

```
  Check [1, 2, 3, 2, 1]:

  L=0, R=4: arr[0]=1, arr[4]=1 → match ✓, L=1, R=3
  L=1, R=3: arr[1]=2, arr[3]=2 → match ✓, L=2, R=2
  L=2, R=2: L not < R → STOP

  Result: true (palindrome!) ✓

  Check [1, 2, 3, 4, 5]:

  L=0, R=4: arr[0]=1, arr[4]=5 → mismatch ✗

  Result: false ✓
```

---

## Complexity Advantage

```
  Problem: Find pair with sum = target in sorted array

  BRUTE FORCE:                          TWO POINTER:
  for i = 0 to n-1:                     L=0, R=n-1
    for j = i+1 to n-1:                 while L < R:
      if arr[i]+arr[j] == target:         sum = arr[L]+arr[R]
        return (i, j)                     if sum == target: found
                                          elif sum < target: L++
  Time: O(n²)                            else: R--
  Space: O(1)                          
                                        Time: O(n)
                                        Space: O(1)
```

---

## Summary Table

| Aspect | Opposite Direction | Same Direction |
|--------|-------------------|----------------|
| Start positions | Both ends | Same end |
| Movement | Converge toward middle | Both move forward |
| Common use | Pair finding, palindrome | Filtering, partitioning |
| Requires sorted? | Usually yes | Not necessarily |
| Time | O(n) | O(n) |
| Space | O(1) | O(1) |

---

## Quick Revision Questions

1. **What are the two main variants of the two-pointer technique?** When do you use each?

2. **How does two-pointer reduce O(n²) to O(n)?** What assumption makes this possible?

3. **Can two pointers work on unsorted arrays?** Give an example where it can and where it can't.

4. **What is the key decision at each step** of the opposite-direction approach?

5. **How is the "remove duplicates" problem** a same-direction two-pointer problem?

6. **Name three classic problems** that use the two-pointer technique.

---

[← Previous Unit: Array Manipulation](../02-Basic-Array-Techniques/06-array-manipulation.md) | [Back to README](../README.md) | [Next: Opposite Direction Pointers →](02-opposite-direction-pointers.md)
