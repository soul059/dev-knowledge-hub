# Chapter 3: Next Permutation

## Overview
Given a permutation, find the **next lexicographically greater** permutation. This is an **O(n) in-place** algorithm — not backtracking, but essential for understanding permutation ordering and a common interview problem.

---

## The Lexicographic Order

```
All permutations of [1,2,3] in order:
  [1,2,3] → [1,3,2] → [2,1,3] → [2,3,1] → [3,1,2] → [3,2,1]
      1st       2nd       3rd       4th       5th       6th (last)
      ↑                                                    ↑
   smallest                                            largest

After the LAST permutation, wrap to the FIRST (circular).
[3,2,1] → next → [1,2,3]
```

---

## The Algorithm (4 Steps)

```
┌──────────────────────────────────────────────────────┐
│  Step 1: FIND the pivot                              │
│    Scan right-to-left, find first a[i] < a[i+1]     │
│    (the rightmost "ascending" pair)                  │
│                                                      │
│  Step 2: FIND the successor                          │
│    Scan right-to-left, find first a[j] > a[i]       │
│    (smallest element larger than pivot)              │
│                                                      │
│  Step 3: SWAP a[i] and a[j]                          │
│                                                      │
│  Step 4: REVERSE the suffix after position i         │
│    This turns the descending suffix into ascending   │
└──────────────────────────────────────────────────────┘
```

---

## Worked Example: [1, 3, 5, 4, 2]

```
Step 1: Find pivot (first a[i] < a[i+1] from right)
  [1, 3, 5, 4, 2]
              ↑ 4>2? yes (descending, continue)
           ↑ 5>4? yes (descending, continue)
        ↑ 3<5? YES! pivot found at i=1, a[i]=3

Step 2: Find successor (first a[j] > 3 from right)
  [1, 3, 5, 4, 2]
                 ↑ 2>3? no
              ↑ 4>3? YES! j=3, a[j]=4

Step 3: Swap a[1] and a[3]
  [1, 3, 5, 4, 2] → [1, 4, 5, 3, 2]
      ↑     ↑            ↑     ↑

Step 4: Reverse suffix after i=1
  [1, 4, | 5, 3, 2] → [1, 4, | 2, 3, 5]
          reverse →         ascending!

Result: [1, 4, 2, 3, 5]  ✓

Verify: [1,3,5,4,2] < [1,4,2,3,5] ✓
        No permutation exists between them ✓
```

---

## Pseudocode

```
FUNCTION nextPermutation(arr):
    n = len(arr)
    
    // Step 1: Find pivot
    i = n - 2
    WHILE i >= 0 AND arr[i] >= arr[i + 1]:
        i -= 1
    
    IF i >= 0:                      ← Not the last permutation
        // Step 2: Find successor
        j = n - 1
        WHILE arr[j] <= arr[i]:
            j -= 1
        
        // Step 3: Swap
        swap(arr, i, j)
    
    // Step 4: Reverse suffix (works for last perm too: full reverse)
    reverse(arr, i + 1, n - 1)
```

---

## Special Cases

```
Case 1: Already last permutation [3, 2, 1]
  Step 1: No pivot found (i = -1, fully descending)
  Skip Steps 2-3
  Step 4: Reverse entire array → [1, 2, 3] (first permutation)

Case 2: Single element [5]
  Step 1: i = -1 (no pair to compare)
  Step 4: Reverse → [5] (unchanged)

Case 3: Two elements [1, 2]
  Step 1: i = 0 (1 < 2)
  Step 2: j = 1 (2 > 1)
  Step 3: Swap → [2, 1]
  Step 4: Reverse nothing (suffix is empty)
  Result: [2, 1] ✓

Case 4: Duplicates [1, 5, 1]
  Works correctly! Step 2 finds SMALLEST value > pivot.
```

---

## Why This Works

```
Key insight: The suffix after the pivot is DESCENDING.

   [... , pivot, ↓descending suffix↓]
   [1,    3,     5, 4, 2]
                 ↑ descending from 5

The descending suffix is already at its LAST permutation.
To get the next overall permutation:
  1. Find the element just before this suffix (pivot)
  2. Replace it with the next larger value from the suffix
  3. After swapping, suffix is STILL descending
  4. Reverse suffix to make it ascending (FIRST permutation of suffix)

This gives the SMALLEST permutation that is LARGER than current.
```

---

## Complexity Analysis

```
┌────────────────────────────────────────────┐
│ Time: O(n)                                 │
│   Step 1: O(n) scan                        │
│   Step 2: O(n) scan                        │
│   Step 3: O(1) swap                        │
│   Step 4: O(n) reverse                     │
│   Total: O(n)                              │
│                                            │
│ Space: O(1) — in-place!                    │
│                                            │
│ Much better than generating all n!          │
│ permutations and finding the next one!     │
└────────────────────────────────────────────┘
```

---

## Applications

```
1. Generating all permutations in order
   Start with sorted array, call nextPermutation n! times

2. Finding the k-th permutation (iterative)
   Call nextPermutation k times from sorted

3. Previous permutation
   Same algorithm but reverse the comparisons
   (find first a[i] > a[i+1], etc.)

4. Permutation ranking
   Determine the position of a permutation in lexicographic order
```

---

## Summary Table

| Step | Action | Purpose |
|------|--------|---------|
| 1 | Find rightmost a[i] < a[i+1] | Locate where to make change |
| 2 | Find rightmost a[j] > a[i] | Find minimal replacement |
| 3 | Swap a[i] and a[j] | Increase at the right position |
| 4 | Reverse suffix after i | Make suffix smallest possible |

---

## Quick Revision Questions

1. **What does "lexicographically next"** mean?
2. **Why scan from the RIGHT** to find the pivot?
3. **Why is the suffix** always descending before the algorithm runs?
4. **What happens** when the array is fully descending?
5. **Trace the algorithm** on [2, 3, 1].
6. **How would you find** the PREVIOUS permutation?

---

[← Previous: Permutations With Duplicates](02-permutations-with-duplicates.md) | [Next: Permutation Sequence →](04-permutation-sequence.md)

[← Back to README](../README.md)
