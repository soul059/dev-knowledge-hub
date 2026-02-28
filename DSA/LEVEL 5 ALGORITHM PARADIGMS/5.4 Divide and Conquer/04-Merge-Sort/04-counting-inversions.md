# Chapter 4: Counting Inversions

## Overview

**Counting inversions** is a classic application of Merge Sort's divide and conquer structure. An **inversion** is a pair (i, j) where i < j but arr[i] > arr[j]. The number of inversions measures how "far from sorted" an array is. Using a modified merge sort, we can count all inversions in **O(n log n)** instead of the brute-force **O(n²)**.

---

## What Is an Inversion?

```
Array: [2, 4, 1, 3, 5]

Pairs (i, j) where i < j and arr[i] > arr[j]:
  (2, 1): index 0 and 2 → 2 > 1 ✓ inversion
  (4, 1): index 1 and 2 → 4 > 1 ✓ inversion
  (4, 3): index 1 and 3 → 4 > 3 ✓ inversion

Total inversions = 3

Sorted array:   [1, 2, 3, 4, 5] → 0 inversions
Reversed array: [5, 4, 3, 2, 1] → 10 inversions = C(5,2) = n(n-1)/2
```

---

## Why Use Merge Sort?

```
Brute Force: Check all pairs → O(n²)

for i = 0 to n-2:
    for j = i+1 to n-1:
        if arr[i] > arr[j]:
            count++

Merge Sort approach: Count inversions DURING the merge step → O(n log n)

Key insight:
  When merging [left] and [right], if element from RIGHT
  is placed before remaining elements in LEFT, it forms
  inversions with ALL remaining left elements.
```

---

## Three Types of Inversions

```
Array: [a₁, a₂, ..., aₙ]   split at midpoint

┌──────────┐  ┌──────────┐
│   LEFT   │  │  RIGHT   │
└──────────┘  └──────────┘

Inversions are of three types:
1. LEFT inversions:  both elements in left half
2. RIGHT inversions: both elements in right half  
3. SPLIT inversions: one in left, one in right ← KEY!

Total = left_inv + right_inv + split_inv

Recursive sort handles #1 and #2.
Merge step counts #3.
```

---

## How Split Inversions Are Counted

```
During merge, when we pick from RIGHT before LEFT is exhausted:

Left:  [2, 4, 6]    Right: [1, 3, 5]
        i=0                  j=0

Step 1: Compare 2 vs 1 → pick 1 (from RIGHT)
  1 is less than ALL remaining left elements: {2, 4, 6}
  Split inversions += 3 (= mid - i + 1)

Step 2: Compare 2 vs 3 → pick 2 (from LEFT)
  No inversions counted.

Step 3: Compare 4 vs 3 → pick 3 (from RIGHT)
  3 is less than ALL remaining left elements: {4, 6}
  Split inversions += 2

Step 4: Compare 4 vs 5 → pick 4 (from LEFT)
  No inversions counted.

Step 5: Compare 6 vs 5 → pick 5 (from RIGHT)
  5 is less than ALL remaining left elements: {6}
  Split inversions += 1

Step 6: Copy 6 (from LEFT) → no inversions.

Total split inversions = 3 + 2 + 1 = 6
```

---

## Visual: Why It Works

```
When R[j] < L[i]:

Left:   [..., L[i], L[i+1], ..., L[mid]]
                ▲     ▲            ▲
                │     │            │
          All of these are > R[j]
          
Since Left is SORTED: L[i] ≤ L[i+1] ≤ ... ≤ L[mid]
And R[j] < L[i]
→ R[j] < ALL of {L[i], L[i+1], ..., L[mid]}
→ Number of inversions = (mid - i + 1)
```

---

## Complete Algorithm

```
function countInversions(arr, lo, hi):
    if lo >= hi:
        return 0
    
    mid = lo + (hi - lo) / 2
    
    leftInv  = countInversions(arr, lo, mid)
    rightInv = countInversions(arr, mid + 1, hi)
    splitInv = mergeCount(arr, lo, mid, hi)
    
    return leftInv + rightInv + splitInv

function mergeCount(arr, lo, mid, hi):
    temp = []
    i = lo, j = mid + 1
    inversions = 0
    
    while i <= mid AND j <= hi:
        if arr[i] <= arr[j]:
            temp.append(arr[i])
            i++
        else:
            temp.append(arr[j])
            inversions += (mid - i + 1)    // ← KEY LINE
            j++
    
    // Copy remaining
    while i <= mid: temp.append(arr[i++])
    while j <= hi:  temp.append(arr[j++])
    
    // Copy back
    for k = lo to hi:
        arr[k] = temp[k - lo]
    
    return inversions
```

---

## Full Trace

```
Input: [8, 4, 2, 1]

countInversions([8,4,2,1], 0, 3)
├── countInversions([8,4], 0, 1)
│   ├── countInversions([8], 0, 0) → 0
│   ├── countInversions([4], 1, 1) → 0
│   └── mergeCount([8,4], 0, 0, 1)
│       i=0, j=1: 8 > 4 → inv += (0-0+1) = 1
│       Result: [4,8], inversions = 1
│   Total: 0 + 0 + 1 = 1
│
├── countInversions([2,1], 2, 3)
│   ├── countInversions([2], 2, 2) → 0
│   ├── countInversions([1], 3, 3) → 0
│   └── mergeCount([2,1], 2, 2, 3)
│       i=2, j=3: 2 > 1 → inv += (2-2+1) = 1
│       Result: [1,2], inversions = 1
│   Total: 0 + 0 + 1 = 1
│
└── mergeCount([4,8,1,2], 0, 1, 3)
    i=0, j=2: 4 > 1 → inv += (1-0+1) = 2
    i=0, j=3: 4 > 2 → inv += (1-0+1) = 2
    i=0:      4 < ∞ → copy 4
    i=1:      8 < ∞ → copy 8
    Result: [1,2,4,8], inversions = 4
    
Total: 1 + 1 + 4 = 6

Verify: (8,4)(8,2)(8,1)(4,2)(4,1)(2,1) = 6 inversions ✓
```

---

## Python Implementation

```python
def count_inversions(arr):
    if len(arr) <= 1:
        return arr, 0
    
    mid = len(arr) // 2
    left, left_inv = count_inversions(arr[:mid])
    right, right_inv = count_inversions(arr[mid:])
    
    merged = []
    split_inv = 0
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            split_inv += len(left) - i    # KEY!
            j += 1
    
    merged.extend(left[i:])
    merged.extend(right[j:])
    
    return merged, left_inv + right_inv + split_inv
```

---

## Applications

```
┌─────────────────────────────────────────────────────────────┐
│  1. Measure of sortedness: 0 = sorted, n(n-1)/2 = reversed │
│  2. Kendall tau distance: similarity between two rankings   │
│  3. Collaborative filtering: compare user preferences       │
│  4. Genome analysis: count mutations / rearrangements       │
│  5. Counting smaller elements after self                    │
└─────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Inversion | Pair (i,j) where i < j but arr[i] > arr[j] |
| Brute force | O(n²) — check all pairs |
| D&C approach | Modified merge sort — O(n log n) |
| Three types | Left inversions, right inversions, split inversions |
| Key insight | When R[j] < L[i], count (mid - i + 1) inversions |
| Max inversions | n(n-1)/2 for reversed array |
| Min inversions | 0 for sorted array |

---

## Quick Revision Questions

1. **What is an inversion and what does the inversion count tell you about an array?**
2. **Why can't we count inversions in O(n)?**
3. **How does the merge step count split inversions?**
4. **When we pick from the right array during merge, why do we add (mid - i + 1)?**
5. **What is the maximum number of inversions for an array of size n?**

---

[← Previous: Complexity Analysis](03-complexity-analysis.md) | [Next: Merge Sort Variations →](05-variations.md)
