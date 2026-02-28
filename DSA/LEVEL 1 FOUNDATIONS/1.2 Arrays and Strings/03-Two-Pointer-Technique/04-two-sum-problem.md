# Chapter 4: Two Sum Problem

[← Previous: Same Direction Pointers](03-same-direction-pointers.md) | [Back to README](../README.md) | [Next: Three Sum Problem →](05-three-sum-problem.md)

---

## Overview

The Two Sum problem is one of the most famous algorithm problems (LeetCode #1). Given an array and a target, find two numbers that add up to the target. We'll explore multiple approaches from brute force to optimal.

---

## Problem Statement

```
Given: Array of integers and a target sum
Find:  Two indices i, j such that arr[i] + arr[j] = target

Example:
  arr = [2, 7, 11, 15], target = 9
  Output: (0, 1) because arr[0] + arr[1] = 2 + 7 = 9
```

---

## Approach 1: Brute Force — O(n²)

```
FUNCTION twoSum_bruteForce(arr, n, target):
    FOR i = 0 TO n-2:
        FOR j = i+1 TO n-1:
            IF arr[i] + arr[j] == target:
                RETURN (i, j)
    RETURN "not found"
```

```
  arr = [2, 7, 11, 15], target = 9

  i=0, j=1: 2+7  = 9  == 9 → FOUND! (0, 1)

  Worst case: checks all n(n-1)/2 pairs
```

**Time: O(n²)** | **Space: O(1)**

---

## Approach 2: Hash Map — O(n)

For each element, check if its complement (target - arr[i]) exists in a hash map.

```
FUNCTION twoSum_hashMap(arr, n, target):
    map = empty hash map     // value → index

    FOR i = 0 TO n-1:
        complement = target - arr[i]
        IF complement IN map:
            RETURN (map[complement], i)
        map[arr[i]] = i      // store current element

    RETURN "not found"
```

### Step-by-Step Trace: arr = [2, 7, 11, 15], target = 9

```
  map = {}

  i=0: complement = 9-2 = 7
       7 in map? No
       map = {2: 0}

  i=1: complement = 9-7 = 2
       2 in map? YES → map[2] = 0
       RETURN (0, 1) ✓

  ┌─────────────────────────────────────────────┐
  │  Why it works:                               │
  │                                              │
  │  For each number X, we ask:                  │
  │  "Have I seen (target - X) before?"          │
  │                                              │
  │  arr[0] = 2:  "Have I seen 7?" No.           │
  │  arr[1] = 7:  "Have I seen 2?" YES → answer! │
  └─────────────────────────────────────────────┘
```

**Time: O(n)** | **Space: O(n)**

---

## Approach 3: Sorting + Two Pointers — O(n log n)

If we're okay losing original indices (or can track them):

```
FUNCTION twoSum_twoPointer(arr, n, target):
    SORT(arr)                // O(n log n)
    L = 0, R = n - 1
    WHILE L < R:
        sum = arr[L] + arr[R]
        IF sum == target:
            RETURN (L, R)    // indices in sorted array
        ELSE IF sum < target:
            L = L + 1
        ELSE:
            R = R - 1
    RETURN "not found"
```

### Trace: arr = [11, 2, 15, 7], target = 9

```
  Sort: [2, 7, 11, 15]

  L=0, R=3: 2+15 = 17 > 9 → R=2
  L=0, R=2: 2+11 = 13 > 9 → R=1
  L=0, R=1: 2+7  = 9  == 9 → FOUND! ✓

  Note: Returns indices in sorted array, not original.
  To preserve original indices, store (value, originalIndex) pairs.
```

**Time: O(n log n)** | **Space: O(1)** (ignoring sort space)

---

## Comparison of All Approaches

```
  ┌───────────────────────────────────────────────────────┐
  │              TWO SUM APPROACHES                        │
  ├─────────────────────┬─────────────┬───────────────────┤
  │   Approach          │    Time     │      Space        │
  ├─────────────────────┼─────────────┼───────────────────┤
  │   Brute Force       │   O(n²)    │      O(1)         │
  │   Hash Map          │   O(n)     │      O(n)         │
  │   Sort + 2 Pointer  │ O(n log n) │      O(1)*        │
  └─────────────────────┴─────────────┴───────────────────┘
  * O(1) if you don't need original indices
```

---

## Variations of Two Sum

### Variation 1: Two Sum — Sorted Array
Already sorted → use two pointers directly. O(n) time, O(1) space.

### Variation 2: Two Sum — Multiple Pairs
Find ALL pairs (not just one).

```
FUNCTION twoSumAllPairs(arr, n, target):
    SORT(arr)
    L = 0, R = n - 1
    pairs = []
    WHILE L < R:
        sum = arr[L] + arr[R]
        IF sum == target:
            pairs.add((arr[L], arr[R]))
            L = L + 1
            R = R - 1
            // Skip duplicates
            WHILE L < R AND arr[L] == arr[L-1]: L++
            WHILE L < R AND arr[R] == arr[R+1]: R--
        ELSE IF sum < target:
            L = L + 1
        ELSE:
            R = R - 1
    RETURN pairs
```

### Variation 3: Two Sum — Closest Pair
Find pair whose sum is closest to target.

```
FUNCTION twoSumClosest(arr, n, target):
    SORT(arr)
    L = 0, R = n - 1
    closestDiff = INFINITY
    bestPair = (0, 0)

    WHILE L < R:
        sum = arr[L] + arr[R]
        diff = ABS(sum - target)
        IF diff < closestDiff:
            closestDiff = diff
            bestPair = (arr[L], arr[R])
        IF sum < target:
            L = L + 1
        ELSE:
            R = R - 1

    RETURN bestPair
```

### Variation 4: Two Sum — Count Pairs

```
  arr = [1, 1, 2, 3, 3], target = 4
  Pairs: (1,3), (1,3), (1,3), (1,3) → count = 4
  Unique pairs: (1,3) → count = 1
```

---

## Decision Tree: Which Approach to Use

```
  Two Sum Problem?
        │
        ├── Need original indices?
        │     ├── Yes → HASH MAP (O(n) time, O(n) space)
        │     └── No  → Is array sorted?
        │                ├── Yes → TWO POINTERS (O(n), O(1))
        │                └── No  → SORT + TWO POINTERS (O(n log n), O(1))
        │                         or HASH MAP (O(n), O(n))
        │
        ├── Multiple queries with same array?
        │     └── SORT once + TWO POINTERS for each query
        │
        └── Find all pairs / count pairs?
              └── SORT + TWO POINTERS with duplicate handling
```

---

## Summary Table

| Approach | Time | Space | Preserves Indices | Best For |
|----------|------|-------|-------------------|----------|
| Brute force | O(n²) | O(1) | Yes | Small arrays |
| Hash map | O(n) | O(n) | Yes | General case |
| Sort + 2 pointer | O(n log n) | O(1) | No | Already sorted |
| Sorted + 2 pointer | O(n) | O(1) | No | Pre-sorted data |

---

## Quick Revision Questions

1. **Why is the hash map approach O(n)?** What's stored in the hash map?

2. **When would you prefer two pointers over hash map?** What's the trade-off?

3. **How do you handle duplicates** when finding all pairs with a given sum?

4. **Can Two Sum be solved in O(n) time AND O(1) space?** Why or why not?

5. **Modify the hash map approach** to return all pairs, not just the first one.

6. **What's the complement concept?** Why is checking for `target - arr[i]` the key insight?

---

[← Previous: Same Direction Pointers](03-same-direction-pointers.md) | [Back to README](../README.md) | [Next: Three Sum Problem →](05-three-sum-problem.md)
