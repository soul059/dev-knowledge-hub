# Chapter 1: Rotated Sorted Array — Concept

[← Previous Unit: Practice Problems](../04-Binary-Search-Template/06-practice-problems.md) | [Next: Find Minimum →](02-find-minimum.md)

---

## Overview

A **rotated sorted array** is a sorted array that has been "rotated" around a **pivot point**. Binary search still works on these arrays — you just need to determine which half is sorted.

---

## What Is Rotation?

```
   Original sorted array:
   [1, 2, 3, 4, 5, 6, 7]
   
   Rotate by 3 positions (move first 3 to the end):
   [4, 5, 6, 7, 1, 2, 3]
         ↑              
        pivot (where the break occurs)
   
   Rotation count = index of the minimum element = 4
```

### All Rotations of [1, 2, 3, 4, 5]

```
   Rotation 0: [1, 2, 3, 4, 5]   ← original (not rotated)
   Rotation 1: [2, 3, 4, 5, 1]
   Rotation 2: [3, 4, 5, 1, 2]
   Rotation 3: [4, 5, 1, 2, 3]
   Rotation 4: [5, 1, 2, 3, 4]
```

---

## Key Properties

```
   [4, 5, 6, 7, 1, 2, 3]
    ─────────  ────────
     Part A      Part B
    (sorted)    (sorted)
   
   Property 1: Both parts are individually sorted
   Property 2: Every element in A > every element in B
   Property 3: arr[0] of A > arr[last] of B (unless not rotated)
   Property 4: There is exactly ONE point where arr[i] > arr[i+1]
               (the "break point" or "pivot")
```

---

## Visual Model

```
     Value
       │
    7  │         ●
    6  │      ●
    5  │   ●
    4  │ ●                          ← Part A (ascending)
    3  │                      ●
    2  │                   ●        ← Part B (ascending)
    1  │                ●
       │                   
       └──┬──┬──┬──┬──┬──┬──┬──
          0  1  2  3  4  5  6       Index
                      ↑
                    pivot
              (drop from 7 to 1)
```

---

## How to Detect Rotation

```
   Not rotated: arr[0] <= arr[n-1]
   Rotated:     arr[0] > arr[n-1]
   
   Example:
   [1, 2, 3, 4, 5] → arr[0] = 1 ≤ arr[4] = 5 → NOT rotated
   [4, 5, 1, 2, 3] → arr[0] = 4 > arr[4] = 3 → ROTATED
```

---

## Why Standard Binary Search Fails

```
   arr = [4, 5, 6, 7, 1, 2, 3], target = 2
   
   Standard BS:
   lo=0 hi=6 mid=3 arr[3]=7
   7 > 2 → hi = 2
   
   New range: [4, 5, 6] ← target 2 is NOT here! WRONG!
   
   The problem: Standard BS assumes the entire array is sorted.
   In a rotated array, we need to figure out WHICH HALF is sorted,
   then decide which half to search.
```

---

## The Key Insight

```
   [4, 5, 6, 7, 1, 2, 3]
    lo       mid       hi
   
   Compare arr[mid] with arr[lo] or arr[hi]:
   
   Case 1: arr[lo] <= arr[mid]  →  LEFT half [lo..mid] is sorted
           [4, 5, 6, 7, ...]
            sorted ────→
   
   Case 2: arr[mid] <= arr[hi]  →  RIGHT half [mid..hi] is sorted
           [..., 1, 2, 3]
                sorted ────→
   
   Once you know which half is sorted, check if target lies in it:
   - If yes → search that half
   - If no  → search the other half
```

---

## Identifying the Sorted Half

```
   Example 1: [4, 5, 6, 7, 1, 2, 3]
               lo       mid       hi
               4   ≤   7  →  LEFT is sorted
               Is target in [4..7]?
                 Yes → search left
                 No  → search right
   
   Example 2: [6, 7, 1, 2, 3, 4, 5]
               lo       mid       hi
               6   >   2  →  RIGHT is sorted
               Is target in [2..5]?
                 Yes → search right
                 No  → search left
```

---

## Types of Problems on Rotated Arrays

```
   ┌──────────────────────────────────────────────────────────┐
   │  1. Find the minimum element (= find pivot)             │
   │     → O(log n)                                          │
   │                                                         │
   │  2. Search for a target                                 │
   │     → O(log n)                                          │
   │                                                         │
   │  3. Find rotation count                                 │
   │     → Index of minimum element                          │
   │                                                         │
   │  4. Handle duplicates                                   │
   │     → Worst case O(n), average O(log n)                 │
   └──────────────────────────────────────────────────────────┘
```

---

## Rotation Count

```
   Number of rotations = index of the minimum element
   
   [4, 5, 6, 7, 1, 2, 3]
                ↑
                index 4 → rotated 4 times
   
   [1, 2, 3, 4, 5]
    ↑
    index 0 → rotated 0 times (not rotated)
```

---

## Summary

| Concept | Detail |
|---------|--------|
| What | Sorted array broken into two sorted halves |
| Property | Both halves sorted, all of left > all of right |
| Detection | `arr[0] > arr[n-1]` means rotated |
| Key trick | Identify which half is sorted |
| Applications | Find min, search target, count rotations |
| Complexity | O(log n) for distinct elements |

---

## Quick Revision Questions

1. **Given `[3, 4, 5, 1, 2]`, what is the rotation count?**
2. **How do you check if an array is rotated or not?**
3. **Why does standard binary search fail on rotated arrays?**
4. **What are the two key properties of a rotated sorted array?**
5. **In `[4, 5, 6, 7, 1, 2, 3]`, if mid=3, which half is sorted?**

---

[← Previous Unit: Practice Problems](../04-Binary-Search-Template/06-practice-problems.md) | [Next: Find Minimum →](02-find-minimum.md)
