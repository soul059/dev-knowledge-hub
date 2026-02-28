# Chapter 5.5: Find the Duplicate Number

> **Unit 5 — XOR Applications**
> Given n+1 numbers from 1 to n with one duplicate, find it using XOR.

---

## Overview

Given an array of n+1 integers where each integer is in the range [1, n], there is exactly one repeated number. Find it without modifying the array, in O(1) extra space.

---

## XOR Approach (When Exactly One Duplicate Exists)

```
  ┌──────────────────────────────────────────────────────────┐
  │  XOR all array elements with all numbers 1..n.           │
  │  Every number appears once in range, once in array       │
  │  → they cancel. The duplicate appears TWICE in array,    │
  │  once in range → one copy survives!                      │
  └──────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Trace

### Example: arr = [1, 3, 4, 2, 2], n = 4

```
  Range 1..4:    1, 2, 3, 4
  Array:         1, 3, 4, 2, 2
  Duplicate:     2

  XOR of 1..4:       1 ^ 2 ^ 3 ^ 4  = 4
  XOR of array:      1 ^ 3 ^ 4 ^ 2 ^ 2

  Combined:
  (1^1) ^ (2^2^2) ^ (3^3) ^ (4^4)
  =  0  ^    2    ^   0   ^   0
  = 2  ✓

  The duplicate appears 2 times in array + 1 time in range = 3 times.
  Other elements appear 1+1 = 2 times. Pairs cancel, triple = one survives.
```

---

## Pseudocode

```
FUNCTION findDuplicate(arr[]):
    n = LENGTH(arr) - 1
    result = 0
    FOR i = 1 TO n:
        result = result ^ i
    FOR each x in arr:
        result = result ^ x
    RETURN result
```

Or combined in one loop:

```
FUNCTION findDuplicate(arr[]):
    n = LENGTH(arr) - 1
    result = 0
    FOR i = 0 TO n-1:
        result = result ^ (i + 1) ^ arr[i]
    result = result ^ arr[n]    // one extra element in array
    RETURN result
```

---

## Important Limitation

```
  ┌─────────────────────────────────────────────────────────┐
  │  ⚠ XOR approach works ONLY when there is exactly ONE   │
  │  duplicate and it appears exactly TWICE.                │
  │                                                         │
  │  If a number can appear MORE than twice, or if there    │
  │  can be MULTIPLE different duplicates, XOR doesn't      │
  │  directly apply. Use Floyd's cycle detection instead.   │
  └─────────────────────────────────────────────────────────┘
```

---

## Floyd's Cycle Detection (General Case)

For the general problem (LeetCode #287) where multiple duplicates or higher multiplicities are possible:

```
  Treat arr as a linked list: index → arr[index]
  The duplicate creates a cycle.

  Phase 1: Find meeting point (fast/slow pointers)
  Phase 2: Find cycle entrance = duplicate number

  FUNCTION findDuplicate(arr[]):
      slow = arr[0]
      fast = arr[arr[0]]
      WHILE slow != fast:
          slow = arr[slow]
          fast = arr[arr[fast]]
      
      slow = 0
      WHILE slow != fast:
          slow = arr[slow]
          fast = arr[fast]
      RETURN slow
```

---

## Comparison

| Method | Time | Space | Constraint |
|--------|------|-------|------------|
| **XOR** | O(n) | O(1) | Exactly one duplicate, appears exactly twice |
| **Floyd's** | O(n) | O(1) | Works for any single duplicate |
| Sorting | O(n log n) | O(1)* | Modifies array |
| Hash set | O(n) | O(n) | General case |
| Bit counting | O(n log n) | O(1) | General case |

---

## XOR vs Missing Number

```
  Missing Number:                    Duplicate Number:
  n elements from 0..n              n+1 elements from 1..n
  One is ABSENT                     One is EXTRA
  XOR range ^ XOR array             XOR range ^ XOR array
  = missing value                   = duplicate value

  Same technique, opposite problem!
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Core trick | XOR range 1..n with array; pairs cancel |
| Result | Duplicate survives (odd count after combining) |
| Limitation | Only works for exactly one duplicate appearing exactly twice |
| General solution | Floyd's cycle detection for arbitrary duplicates |
| Relation to missing # | Same XOR technique, mirror problem |

---

## Quick Revision Questions

1. **Find the duplicate in [3, 1, 3, 2], n = 3.**
   <details><summary>Answer</summary>XOR(1..3) = 1^2^3 = 0. XOR(array) = 3^1^3^2 = 3. Combined: 0^3 = 3. Duplicate is 3.</details>

2. **Why does XOR fail if 2 appears three times and 3 is missing?**
   <details><summary>Answer</summary>Array might be [1,2,2,2]. XOR range = 1^2^3^4. XOR array = 1^2^2^2 = 1^2. Combined = 3^4^2 = 5 — which is not a valid answer. The assumption of exactly one duplicate twice is violated.</details>

3. **Can we combine the missing and duplicate problems?**
   <details><summary>Answer</summary>Yes! Given [1..n] with one missing and one duplicate: XOR all gives missing ^ duplicate. Then use the two-unique-numbers technique (Chapter 5.3) to separate them.</details>

4. **How does Floyd's cycle detection relate to bit manipulation?**
   <details><summary>Answer</summary>It doesn't use bit manipulation — it's a pointer/graph technique. But it solves the same problem more generally. XOR is preferred when its preconditions hold due to simplicity.</details>

5. **What is the time complexity of the XOR approach?**
   <details><summary>Answer</summary>O(n) — one pass through the range 1..n and one pass through the array. Can be combined into a single loop for O(n).</details>

---

[← Previous: Find Missing Number](04-find-missing-number.md) | [Next: XOR of a Range →](06-xor-of-range.md)
