# Chapter 5: The Skyline Problem

## Overview

The **Skyline Problem** (LeetCode 218) is a classic D&C problem: given a set of rectangular buildings, compute the outline of their silhouette. The D&C solution splits buildings in half, computes each skyline recursively, and merges them in O(n) time — yielding O(n log n) overall. This problem beautifully illustrates how D&C applies to geometric merge operations.

---

## Problem Description

```
Input: List of buildings, each described as [left, right, height]

    Buildings: [2,9,10], [3,7,15], [5,12,12], [15,20,10], [19,24,8]

           15 │     ┌───┐
              │     │   │
              │ ┌───┤   │    12
              │ │   │   ├───────┐
           10 ├─┤   │   │       │         ┌───────┐
              │ │   │   │       │    10   │       │
            8 │ │   │   │       │    ┌────┤       │
              │ │   │   │       │    │    │       │
              │ │   │   │       │    │    │       │
              └─┴───┴───┴───────┴────┴────┴───────┴──
              0 2 3 5 7 9      12  15  19 20    24

Output: Skyline contour as list of [x, height] key points:
    [[2,10], [3,15], [7,12], [12,0], [15,10], [20,8], [24,0]]

           15 │     ┌───┐
              │     │   │
              │     │   │    12
              │     │   └───────┐
           10 ├─┐   │          │         ┌───────┐
              │ │   │          │    10   │       │
            8 │ │   │          │    ┌────┘       │
              │ │   │          │    │            │
              │ │   │          │    │            │
              └─┴───┴──────────┴────┴────────────┴──
                ▲   ▲          ▲    ▲    ▲       ▲
              (2,10)(3,15)  (12,0)(15,10)(20,8)(24,0)
                    (7,12)
```

---

## D&C Strategy

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  1. DIVIDE: Split buildings into two halves           │
│     Left half:  buildings[0..mid]                    │
│     Right half: buildings[mid+1..n-1]                │
│                                                      │
│  2. CONQUER: Recursively compute skyline for each    │
│     left_skyline  ← Skyline(left half)               │
│     right_skyline ← Skyline(right half)              │
│                                                      │
│  3. COMBINE: Merge two skylines into one             │
│     Walk through both skylines left-to-right         │
│     At each x-coordinate, height = max of both       │
│                                                      │
└──────────────────────────────────────────────────────┘

BASE CASE: Single building [l, r, h]
    → Skyline is [[l, h], [r, 0]]
```

---

## Merge Operation (Key Step)

```
Merging two skylines is like merging two sorted lists,
but tracking the MAXIMUM height at each position.

Left skyline:   [(2,10), (9,0)]
Right skyline:  [(3,15), (7,12), (12,0)]

Merge process:
    Maintain: h1 = current height in left skyline
              h2 = current height in right skyline
              max_h = max(h1, h2) at each event point

    x=2:  h1=10, h2=0  → max=10 → output (2,10)
    x=3:  h1=10, h2=15 → max=15 → output (3,15) ← height changed!
    x=7:  h1=10, h2=12 → max=12 → output (7,12) ← height changed!
    x=9:  h1=0,  h2=12 → max=12 → no change
    x=12: h1=0,  h2=0  → max=0  → output (12,0) ← height changed!

Result: [(2,10), (3,15), (7,12), (12,0)]

RULE: Only output a point when max height CHANGES.
```

---

## Detailed Walkthrough

```
Buildings: [2,9,10], [3,7,15], [5,12,12], [15,20,10], [19,24,8]

DIVIDE: 
    Left:  [2,9,10], [3,7,15], [5,12,12]
    Right: [15,20,10], [19,24,8]

CONQUER Left: recurse further
    Left-Left:  [2,9,10]
    Left-Right: [3,7,15], [5,12,12]
    
    LL skyline: [(2,10), (9,0)]
    
    LR: recurse
        LRL: [3,7,15] → [(3,15), (7,0)]
        LRR: [5,12,12] → [(5,12), (12,0)]
        
        Merge LRL + LRR:
            x=3: h1=15, h2=0  → max=15 → (3,15)
            x=5: h1=15, h2=12 → max=15 → no change
            x=7: h1=0,  h2=12 → max=12 → (7,12)
            x=12: h1=0, h2=0  → max=0  → (12,0)
        LR skyline: [(3,15), (7,12), (12,0)]
    
    Merge LL + LR:
        x=2:  h1=10, h2=0  → max=10 → (2,10)
        x=3:  h1=10, h2=15 → max=15 → (3,15)
        x=7:  h1=10, h2=12 → max=12 → no change (h1 still active)
              Wait — h2 drops to 12, but h1=10, so max=max(10,12)=12
              Previous max was 15, now 12 → (7,12)
        x=9:  h1=0,  h2=12 → max=12 → no change
        x=12: h1=0,  h2=0  → max=0  → (12,0)
    Left skyline: [(2,10), (3,15), (7,12), (12,0)]

CONQUER Right:
    RL: [15,20,10] → [(15,10), (20,0)]
    RR: [19,24,8]  → [(19,8), (24,0)]
    
    Merge:
        x=15: h1=10, h2=0 → max=10 → (15,10)
        x=19: h1=10, h2=8 → max=10 → no change
        x=20: h1=0,  h2=8 → max=8  → (20,8)
        x=24: h1=0,  h2=0 → max=0  → (24,0)
    Right skyline: [(15,10), (20,8), (24,0)]

FINAL MERGE:
    Left:  [(2,10), (3,15), (7,12), (12,0)]
    Right: [(15,10), (20,8), (24,0)]
    
    x=2:  h1=10, h2=0 → max=10 → (2,10)
    x=3:  h1=15, h2=0 → max=15 → (3,15)
    x=7:  h1=12, h2=0 → max=12 → (7,12)
    x=12: h1=0,  h2=0 → max=0  → (12,0)
    x=15: h1=0,  h2=10 → max=10 → (15,10)
    x=20: h1=0,  h2=8  → max=8  → (20,8)
    x=24: h1=0,  h2=0  → max=0  → (24,0)

ANSWER: [(2,10), (3,15), (7,12), (12,0), (15,10), (20,8), (24,0)]
```

---

## Pseudocode

```
SKYLINE(buildings):
    if len(buildings) == 0:
        return []
    if len(buildings) == 1:
        [l, r, h] ← buildings[0]
        return [[l, h], [r, 0]]
    
    mid ← len(buildings) / 2
    left  ← SKYLINE(buildings[0..mid])
    right ← SKYLINE(buildings[mid+1..n-1])
    
    return MERGE_SKYLINES(left, right)


MERGE_SKYLINES(sky1, sky2):
    result ← []
    h1 ← 0, h2 ← 0    // current heights
    i ← 0, j ← 0       // pointers
    
    while i < len(sky1) AND j < len(sky2):
        if sky1[i].x < sky2[j].x:
            x ← sky1[i].x
            h1 ← sky1[i].h
            i++
        elif sky1[i].x > sky2[j].x:
            x ← sky2[j].x
            h2 ← sky2[j].h
            j++
        else:  // same x
            x ← sky1[i].x
            h1 ← sky1[i].h
            h2 ← sky2[j].h
            i++, j++
        
        max_h ← max(h1, h2)
        
        // Only add if height changed
        if result is empty OR result.last.h ≠ max_h:
            result.append([x, max_h])
    
    // Append remaining
    while i < len(sky1): result.append(sky1[i]); i++
    while j < len(sky2): result.append(sky2[j]); j++
    
    return result
```

---

## Complexity Analysis

```
RECURRENCE:
    T(n) = 2T(n/2) + O(n)
    
    Merge step: Two-pointer merge is O(|sky1| + |sky2|) = O(n)
    (each building contributes at most 2 points to a skyline)
    
    Master Theorem: a=2, b=2, c=1 → Case 2
    T(n) = O(n log n)

SPACE: O(n) for the merged skyline at each level

ALTERNATIVE APPROACHES:
┌─────────────────────┬─────────────┬────────────────────┐
│ Algorithm           │ Time        │ Notes              │
├─────────────────────┼─────────────┼────────────────────┤
│ Brute force         │ O(n²)       │ Process each bldg  │
│ D&C (this chapter)  │ O(n log n)  │ Merge skylines     │
│ Sweep line + heap   │ O(n log n)  │ Event-based        │
│ Sweep + multiset    │ O(n log n)  │ Balanced BST       │
│ Segment tree        │ O(n log n)  │ Range update       │
└─────────────────────┴─────────────┴────────────────────┘
```

---

## Edge Cases

```
1. OVERLAPPING BUILDINGS (same x-coordinates):
    Buildings share left or right edges
    → Handle ties carefully in merge (process both)

2. BUILDINGS WITHIN BUILDINGS:
    [1, 10, 5], [3, 7, 3]
    Inner building completely hidden
    → Merge correctly handles (max height unchanged)

3. ADJACENT BUILDINGS (same height):
    [1, 5, 10], [5, 8, 10]
    → Should NOT produce a dip at x=5
    → Merge: no height change at x=5, so no output point

4. SINGLE BUILDING:
    → Base case: [[left, height], [right, 0]]

5. NO BUILDINGS:
    → Return empty list
```

---

## Python Implementation

```python
def get_skyline(buildings):
    """
    Divide and Conquer skyline solution.
    Input: List of [left, right, height]
    Output: List of [x, height] key points
    """
    if not buildings:
        return []
    
    if len(buildings) == 1:
        l, r, h = buildings[0]
        return [[l, h], [r, 0]]
    
    mid = len(buildings) // 2
    left = get_skyline(buildings[:mid])
    right = get_skyline(buildings[mid:])
    
    return merge_skylines(left, right)

def merge_skylines(sky1, sky2):
    """Merge two skylines using two-pointer technique."""
    result = []
    h1 = h2 = 0
    i = j = 0
    
    while i < len(sky1) and j < len(sky2):
        if sky1[i][0] < sky2[j][0]:
            x = sky1[i][0]
            h1 = sky1[i][1]
            i += 1
        elif sky1[i][0] > sky2[j][0]:
            x = sky2[j][0]
            h2 = sky2[j][1]
            j += 1
        else:
            x = sky1[i][0]
            h1 = sky1[i][1]
            h2 = sky2[j][1]
            i += 1
            j += 1
        
        max_h = max(h1, h2)
        
        if not result or result[-1][1] != max_h:
            result.append([x, max_h])
    
    while i < len(sky1):
        result.append(sky1[i])
        i += 1
    while j < len(sky2):
        result.append(sky2[j])
        j += 1
    
    return result

# Test
buildings = [[2,9,10], [3,7,15], [5,12,12], [15,20,10], [19,24,8]]
skyline = get_skyline(buildings)
print("Skyline:", skyline)
# Output: [[2, 10], [3, 15], [7, 12], [12, 0], [15, 10], [20, 8], [24, 0]]

# Edge case: overlapping
buildings2 = [[1,5,10], [2,7,10]]
print("Overlapping:", get_skyline(buildings2))
# Output: [[1, 10], [7, 0]]  ← merged seamlessly
```

---

## Connection to Merge Sort

```
The Skyline Problem's D&C structure is identical to Merge Sort:

MERGE SORT:                    SKYLINE:
    Split array in half         Split buildings in half
    Recursively sort            Recursively compute skyline
    Merge sorted halves         Merge skyline contours
    
    Merge: compare elements     Merge: compare x-coords,
           take smaller                track max height
    
    T(n) = 2T(n/2) + O(n)      T(n) = 2T(n/2) + O(n)
           = O(n log n)                = O(n log n)

This pattern — split, recurse, merge — appears across 
many D&C problems. The creativity is in the MERGE step.
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Compute silhouette of rectangular buildings |
| Input | List of [left, right, height] |
| Output | List of [x, height] critical points |
| Base case | Single building → [[l,h], [r,0]] |
| D&C split | Split buildings array at midpoint |
| Merge | Two-pointer, track max(h1, h2), emit on change |
| Recurrence | T(n) = 2T(n/2) + O(n) |
| Complexity | O(n log n) time, O(n) space |
| LeetCode | Problem 218 (Hard) |

---

## Quick Revision Questions

1. **What does a skyline key point represent?**
2. **How does the merge step decide whether to emit a point?**
3. **Why is the merge operation O(n)?**
4. **How do you handle buildings with the same left edge?**
5. **Explain the structural similarity between this problem and merge sort.**

---

[← Previous: Order Statistics](04-order-statistics.md) | [Next: Beautiful Array →](06-beautiful-array.md)
