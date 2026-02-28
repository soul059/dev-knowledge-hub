# Chapter 3: Same Direction Pointers

[← Previous: Opposite Direction Pointers](02-opposite-direction-pointers.md) | [Back to README](../README.md) | [Next: Two Sum Problem →](04-two-sum-problem.md)

---

## Overview

Same-direction pointers (also called fast-slow pointers or read-write pointers) both start from the same end and move in the same direction, but at different speeds. This pattern is used for in-place filtering, partitioning, and cycle detection.

---

## Core Mechanism

```
  ┌───┬───┬───┬───┬───┬───┬───┐
  │   │   │   │   │   │   │   │
  └───┴───┴───┴───┴───┴───┴───┘
   S                               slow (write position)
   F                               fast (read/scan position)

  Fast pointer scans the array.
  Slow pointer marks where to write next valid element.
  Fast is always ≥ slow.

  ┌───┬───┬───┬───┬───┬───┬───┐
  │ ✓ │ ✓ │   │   │ ✓ │   │ ✓ │   (✓ = keep, blank = skip)
  └───┴───┴───┴───┴───┴───┴───┘
   S           F                    fast scans, slow collects valid
```

---

## Problem 1: Remove Duplicates from Sorted Array

Keep only unique elements in-place.

```
FUNCTION removeDuplicates(arr, n):
    IF n <= 1: RETURN n
    slow = 1                        // slow points to next write position
    FOR fast = 1 TO n-1:
        IF arr[fast] != arr[fast-1]:     // new unique element
            arr[slow] = arr[fast]
            slow = slow + 1
    RETURN slow                     // new length
```

### Step-by-Step Trace: [1, 1, 2, 2, 2, 3, 4, 4]

```
  S=1, F=1: arr[1]=1 == arr[0]=1 → skip
  S=1, F=2: arr[2]=2 ≠ arr[1]=1 → arr[1]=2, S=2
  S=2, F=3: arr[3]=2 == arr[2]=2 → skip
  S=2, F=4: arr[4]=2 == arr[3]=2 → skip
  S=2, F=5: arr[5]=3 ≠ arr[4]=2 → arr[2]=3, S=3
  S=3, F=6: arr[6]=4 ≠ arr[5]=3 → arr[3]=4, S=4
  S=4, F=7: arr[7]=4 == arr[6]=4 → skip

  ┌───┬───┬───┬───┬───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │ . │ . │ . │ . │   new length = 4
  └───┴───┴───┴───┴───┴───┴───┴───┘
   valid part ──────┘

  Original 8 elements → 4 unique elements
```

**Time: O(n)** | **Space: O(1)**

---

## Problem 2: Remove All Instances of a Value

```
FUNCTION removeElement(arr, n, val):
    slow = 0
    FOR fast = 0 TO n-1:
        IF arr[fast] != val:
            arr[slow] = arr[fast]
            slow = slow + 1
    RETURN slow
```

### Trace: arr = [0, 1, 2, 2, 3, 0, 4, 2], val = 2

```
  F=0: arr[0]=0 ≠ 2 → arr[0]=0, S=1
  F=1: arr[1]=1 ≠ 2 → arr[1]=1, S=2
  F=2: arr[2]=2 == 2 → skip
  F=3: arr[3]=2 == 2 → skip
  F=4: arr[4]=3 ≠ 2 → arr[2]=3, S=3
  F=5: arr[5]=0 ≠ 2 → arr[3]=0, S=4
  F=6: arr[6]=4 ≠ 2 → arr[4]=4, S=5
  F=7: arr[7]=2 == 2 → skip

  Result: [0, 1, 3, 0, 4, ...]   new length = 5
```

---

## Problem 3: Move Zeros to End (Preserving Order)

```
FUNCTION moveZeros(arr, n):
    slow = 0
    // Phase 1: Move non-zeros to front
    FOR fast = 0 TO n-1:
        IF arr[fast] != 0:
            SWAP(arr[slow], arr[fast])
            slow = slow + 1
```

### Trace: [0, 1, 0, 3, 12]

```
  F=0: arr[0]=0 → skip
  F=1: arr[1]=1 ≠ 0 → swap(arr[0], arr[1]) → [1, 0, 0, 3, 12], S=1
  F=2: arr[2]=0 → skip
  F=3: arr[3]=3 ≠ 0 → swap(arr[1], arr[3]) → [1, 3, 0, 0, 12], S=2
  F=4: arr[4]=12 ≠ 0 → swap(arr[2], arr[4]) → [1, 3, 12, 0, 0], S=3

  Result: [1, 3, 12, 0, 0] ✓

  Visualization of pointers:
  Step 0: [0, 1, 0, 3, 12]
           S
           F

  Step 1: [0, 1, 0, 3, 12]    0 is zero, skip
           S  F

  Step 2: [1, 0, 0, 3, 12]    1 ≠ 0, swap
              S     F

  Step 3: [1, 0, 0, 3, 12]    0 is zero, skip
              S        F

  Step 4: [1, 3, 0, 0, 12]    3 ≠ 0, swap
                 S        F

  Step 5: [1, 3, 12, 0, 0]    12 ≠ 0, swap ✓
```

---

## Problem 4: Sort Colors (Dutch National Flag — Revisited)

Three pointers: low (0s boundary), mid (current), high (2s boundary).

```
  ┌───┬───┬───┬───┬───┬───┬───┬───┐
  │ 0 │ 0 │ 0 │ 1 │ 1 │ ? │ 2 │ 2 │
  └───┴───┴───┴───┴───┴───┴───┴───┘
   all 0s ─┘     │     │     │ all 2s
              all 1s    │
                     unsorted
                    (mid scans)
         low        mid       high
```

This is a three-pointer variant of the same-direction pattern.

---

## Problem 5: Merge In-Place (Two Sorted Parts)

```
  Array has two sorted halves: [1, 3, 5, | 2, 4, 6]
  Merge them in-place (conceptually):

  Use extra space approach:
  ┌───┬───┬───┬───┬───┬───┐
  │ 1 │ 3 │ 5 │ 2 │ 4 │ 6 │
  └───┴───┴───┴───┴───┴───┘
   i───────►    j───────►
   
  Compare arr[i] and arr[j], write smaller to result
  Result: [1, 2, 3, 4, 5, 6]
```

---

## Fast-Slow Pointer for Cycle Detection (Floyd's)

While primarily for linked lists, the concept applies to arrays too:

```
  Array: [1, 3, 4, 2, 2]  (one duplicate — treated as linked list)

  Start = arr[0]
  Slow moves 1 step: slow = arr[slow]
  Fast moves 2 steps: fast = arr[arr[fast]]

  ┌───┐
  │ 0 │──► arr[0]=1
  └───┘         │
            ┌───┐
            │ 1 │──► arr[1]=3
            └───┘         │
                      ┌───┐
                      │ 3 │──► arr[3]=2
                      └───┘         │
                                ┌───┐
                                │ 2 │──► arr[2]=4
                                └───┘         │
                                          ┌───┐
                                          │ 4 │──► arr[4]=2 (CYCLE!)
                                          └───┘         │
                                                    back to 2
```

**Application:** Finding the duplicate number in an array where values are in range [1, n].

---

## Comparison: Swap vs Overwrite

```
  OVERWRITE approach (doesn't preserve skipped values):
  ┌───┬───┬───┬───┬───┐
  │ 1 │ 0 │ 3 │ 0 │ 5 │   remove zeros
  └───┴───┴───┴───┴───┘
  → [1, 3, 5, 0, 5]  then zero-fill: [1, 3, 5, 0, 0]

  SWAP approach (values rearranged, all preserved):
  ┌───┬───┬───┬───┬───┐
  │ 1 │ 0 │ 3 │ 0 │ 5 │   move zeros to end
  └───┴───┴───┴───┴───┘
  → [1, 3, 5, 0, 0]  (zeros naturally end up at end)

  Use SWAP when order of non-target elements matters.
  Use OVERWRITE when you just need the valid prefix.
```

---

## Summary Table

| Problem | Slow Pointer | Fast Pointer | Condition |
|---------|-------------|-------------|-----------|
| Remove duplicates | Next write position | Scans all elements | arr[F] ≠ arr[F-1] |
| Remove value | Next write position | Scans all elements | arr[F] ≠ val |
| Move zeros | Next non-zero position | Scans all elements | arr[F] ≠ 0 |
| Cycle detection | Moves 1 step | Moves 2 steps | slow == fast |
| Partition | Boundary of < pivot | Scans all elements | arr[F] < pivot |

---

## Quick Revision Questions

1. **What is the invariant maintained** by the slow pointer in the "remove duplicates" problem?

2. **Why use swap instead of overwrite** for "move zeros to end"? When does it matter?

3. **In Floyd's cycle detection, why does the fast pointer move 2 steps?** Would 3 steps work?

4. **How would you modify "remove duplicates" to allow at most 2 duplicates** instead of 0?

5. **What is the time complexity of same-direction two-pointer algorithms?** Can they ever be worse than O(n)?

6. **Compare same-direction vs opposite-direction** pointers: when is each appropriate?

---

[← Previous: Opposite Direction Pointers](02-opposite-direction-pointers.md) | [Back to README](../README.md) | [Next: Two Sum Problem →](04-two-sum-problem.md)
