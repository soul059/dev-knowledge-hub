# Chapter 5: Three Sum Problem

[← Previous: Two Sum Problem](04-two-sum-problem.md) | [Back to README](../README.md) | [Next: Container with Most Water →](06-container-with-most-water.md)

---

## Overview

Three Sum extends Two Sum: find three numbers that add to a target (usually 0). This problem is a staple in coding interviews and demonstrates how sorting + two pointers can reduce O(n³) to O(n²).

---

## Problem Statement

```
Given: Array of integers
Find:  All unique triplets (a, b, c) such that a + b + c = 0

Example:
  arr = [-1, 0, 1, 2, -1, -4]
  Output: [(-1, -1, 2), (-1, 0, 1)]
```

---

## Approach 1: Brute Force — O(n³)

```
FUNCTION threeSum_brute(arr, n):
    result = []
    FOR i = 0 TO n-3:
        FOR j = i+1 TO n-2:
            FOR k = j+1 TO n-1:
                IF arr[i] + arr[j] + arr[k] == 0:
                    result.add(SORT(arr[i], arr[j], arr[k]))
    RETURN unique(result)
```

**Time: O(n³)** — too slow for large inputs.

---

## Approach 2: Sort + Two Pointers — O(n²)

**Key insight:** Fix one element, then solve Two Sum on the rest.

```
FUNCTION threeSum(arr, n):
    SORT(arr)
    result = []

    FOR i = 0 TO n-3:
        // Skip duplicate values of arr[i]
        IF i > 0 AND arr[i] == arr[i-1]:
            CONTINUE

        // Two pointer for remaining
        L = i + 1
        R = n - 1
        target = -arr[i]     // we need arr[L] + arr[R] = -arr[i]

        WHILE L < R:
            sum = arr[L] + arr[R]
            IF sum == target:
                result.add((arr[i], arr[L], arr[R]))
                // Skip duplicates
                WHILE L < R AND arr[L] == arr[L+1]: L++
                WHILE L < R AND arr[R] == arr[R-1]: R--
                L++; R--
            ELSE IF sum < target:
                L++
            ELSE:
                R--

    RETURN result
```

---

## Step-by-Step Trace

### Input: [-1, 0, 1, 2, -1, -4]

```
  Step 1: Sort → [-4, -1, -1, 0, 1, 2]

  ──── i = 0, arr[i] = -4, target = 4 ────
  
  [-4, -1, -1,  0,  1,  2]
    i   L                 R
    
  L=1, R=5: -1+2 = 1 < 4 → L++
  L=2, R=5: -1+2 = 1 < 4 → L++
  L=3, R=5:  0+2 = 2 < 4 → L++
  L=4, R=5:  1+2 = 3 < 4 → L++
  L=5, R=5: L not < R → done with i=0
  
  No triplets with -4.

  ──── i = 1, arr[i] = -1, target = 1 ────
  
  [-4, -1, -1,  0,  1,  2]
        i   L              R

  L=2, R=5: -1+2 = 1 == target → FOUND! (-1, -1, 2)
    Skip duplicates: L→3, R→4
  
  L=3, R=4: 0+1 = 1 == target → FOUND! (-1, 0, 1)
    L→4, R→3, L not < R → done

  ──── i = 2, arr[i] = -1, arr[1] also -1 → SKIP (duplicate) ────

  ──── i = 3, arr[i] = 0, target = 0 ────

  L=4, R=5: 1+2 = 3 > 0 → R--
  L=4, R=4: L not < R → done
  
  No more triplets.

  RESULT: [(-1, -1, 2), (-1, 0, 1)] ✓
```

---

## Duplicate Handling — Why and How

The hardest part of Three Sum is avoiding duplicate triplets.

```
  Array: [-1, -1, -1, 0, 1, 1, 2]

  Without duplicate handling:
  (-1, -1, 2) appears 3 times!
  (-1, 0, 1) appears 4 times!

  TWO levels of deduplication:

  Level 1: Skip duplicate arr[i]
  ─────────────────────────────────
  IF i > 0 AND arr[i] == arr[i-1]: CONTINUE
  
  This ensures we don't fix the same first element twice.

  Level 2: Skip duplicate arr[L] and arr[R] after finding a triplet
  ─────────────────────────────────────────────────────────────────
  WHILE L < R AND arr[L] == arr[L+1]: L++
  WHILE L < R AND arr[R] == arr[R-1]: R--
  
  This ensures we don't find the same second/third elements.
```

---

## Visualization of the Algorithm

```
  Sorted array: [-4, -1, -1, 0, 1, 2]

  Round 1: Fix -4, find pairs summing to 4
  ┌────┬────┬────┬────┬────┬────┐
  │ -4 │ -1 │ -1 │  0 │  1 │  2 │
  └────┴────┴────┴────┴────┴────┘
    ▲    L──────────────────── R
   fixed       two-sum region

  Round 2: Fix -1, find pairs summing to 1
  ┌────┬────┬────┬────┬────┬────┐
  │ -4 │ -1 │ -1 │  0 │  1 │  2 │
  └────┴────┴────┴────┴────┴────┘
          ▲    L──────────── R
         fixed   two-sum region

  Round 3: Fix -1 → same as round 2, SKIP!

  Round 4: Fix 0, find pairs summing to 0
  ┌────┬────┬────┬────┬────┬────┐
  │ -4 │ -1 │ -1 │  0 │  1 │  2 │
  └────┴────┴────┴────┴────┴────┘
                    ▲    L── R
                  fixed
```

---

## Complexity Analysis

```
  Outer loop: O(n) iterations
  Inner two-pointer: O(n) per iteration
  Sorting: O(n log n)

  Total: O(n log n) + O(n × n) = O(n²)
  Space: O(1) extra (not counting output)

  ┌──────────────────────────────────────────┐
  │  For n = 10,000:                          │
  │  Brute force: 10,000³ = 10¹² ops  (TLE!)│
  │  Optimized:   10,000² = 10⁸ ops   (OK!) │
  └──────────────────────────────────────────┘
```

---

## Variations

### 3Sum Closest
Find triplet whose sum is closest to a given target.

```
FUNCTION threeSumClosest(arr, n, target):
    SORT(arr)
    closest = arr[0] + arr[1] + arr[2]

    FOR i = 0 TO n-3:
        L = i + 1, R = n - 1
        WHILE L < R:
            sum = arr[i] + arr[L] + arr[R]
            IF ABS(sum - target) < ABS(closest - target):
                closest = sum
            IF sum < target: L++
            ELSE IF sum > target: R--
            ELSE: RETURN sum    // exact match
    RETURN closest
```

### 3Sum Smaller
Count triplets with sum < target.

### 4Sum
Fix two elements, then Two Sum on the rest → O(n³).

```
  k-Sum generalization:
  2Sum → O(n)        (with sort: O(n log n))
  3Sum → O(n²)
  4Sum → O(n³)
  kSum → O(n^(k-1))
```

---

## Summary Table

| Approach | Time | Space | Handles Duplicates |
|----------|------|-------|-------------------|
| Brute force | O(n³) | O(1) | Need post-processing |
| Sort + Two Pointers | O(n²) | O(1) | Skip duplicates inline |
| Hash Map based | O(n²) | O(n) | Need set for dedup |

---

## Quick Revision Questions

1. **Why do we sort the array first?** What two benefits does sorting provide?

2. **How is Three Sum reduced to Two Sum?** Explain the "fix one, solve for two" strategy.

3. **Walk through the duplicate skipping logic.** Why do we skip at two levels?

4. **What is the time complexity of kSum?** Express in terms of n and k.

5. **How would you modify Three Sum to find all triplets summing to a target T** (not just 0)?

6. **Why can't we achieve O(n log n) for Three Sum?** What's the lower bound argument?

---

[← Previous: Two Sum Problem](04-two-sum-problem.md) | [Back to README](../README.md) | [Next: Container with Most Water →](06-container-with-most-water.md)
