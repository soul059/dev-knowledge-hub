# Chapter 6: Searching in Rotated Sorted Arrays

## Overview

A **rotated sorted array** is created by taking a sorted array and rotating it at some pivot. Searching in such an array using binary search is a classic interview problem that elegantly applies the Divide and Conquer principle.

---

## What Is a Rotated Sorted Array?

```
Original Sorted:  [1, 2, 3, 4, 5, 6, 7]

Rotated by 3:     [4, 5, 6, 7, 1, 2, 3]
                              ▲
                              pivot (minimum element)

Rotated by 5:     [6, 7, 1, 2, 3, 4, 5]
                        ▲
                        pivot
```

**Key Insight**: At least one half of the array (left or right of mid) is always sorted!

```
Array:  [4, 5, 6, 7, 1, 2, 3]
                  mid
         ╰─sorted─╯   ╰─sorted─╯
         
    Left half [4,5,6,7]: sorted (arr[lo] <= arr[mid])
    Right half [1,2,3]:  sorted (arr[mid+1] <= arr[hi])
    
    One of these always holds true regardless of where mid falls!
```

---

## Problem 1: Find the Minimum Element

```
function findMin(arr, n):
    lo = 0, hi = n - 1
    
    while lo < hi:
        mid = lo + (hi - lo) / 2
        if arr[mid] > arr[hi]:
            lo = mid + 1      // min is in right half
        else:
            hi = mid           // mid could be the min
    
    return arr[lo]             // lo == hi == index of min
```

### Trace: arr = [4, 5, 6, 7, 1, 2, 3]

```
lo=0, hi=6, mid=3: arr[3]=7 > arr[6]=3  → lo=4
lo=4, hi=6, mid=5: arr[5]=2 ≤ arr[6]=3  → hi=5
lo=4, hi=5, mid=4: arr[4]=1 ≤ arr[5]=2  → hi=4
lo=4, hi=4: STOP → arr[4] = 1 ✓
```

### Visual: Decision at Each Step

```
[4, 5, 6, 7, 1, 2, 3]
 lo        mid      hi
 
arr[mid]=7 > arr[hi]=3
→ The rotation point is to the RIGHT
→ Move lo to mid+1

          [1, 2, 3]
           lo mid hi
           
arr[mid]=2 ≤ arr[hi]=3  
→ min is at mid or LEFT
→ Move hi to mid
```

---

## Problem 2: Search in Rotated Array (No Duplicates)

```
function searchRotated(arr, n, target):
    lo = 0, hi = n - 1
    
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        
        if arr[mid] == target:
            return mid
        
        // Left half is sorted
        if arr[lo] <= arr[mid]:
            if arr[lo] <= target < arr[mid]:
                hi = mid - 1     // target in sorted left half
            else:
                lo = mid + 1     // target in right half
        
        // Right half is sorted
        else:
            if arr[mid] < target <= arr[hi]:
                lo = mid + 1     // target in sorted right half
            else:
                hi = mid - 1     // target in left half
    
    return -1                    // not found
```

### Trace: arr = [4, 5, 6, 7, 0, 1, 2], target = 0

```
Step 1: lo=0, hi=6, mid=3
  arr[3]=7, not target
  arr[0]=4 <= arr[3]=7 → left half [4,5,6,7] sorted
  Is 4 <= 0 < 7? NO
  → lo=4

Step 2: lo=4, hi=6, mid=5
  arr[5]=1, not target
  arr[4]=0 <= arr[5]=1 → left half [0,1] sorted
  Is 0 <= 0 < 1? YES
  → hi=4

Step 3: lo=4, hi=4, mid=4
  arr[4]=0 == target ✓ → return 4
```

### Decision Tree Visualization

```
                    arr = [4,5,6,7,0,1,2], target = 0
                    
                    mid = arr[3] = 7 ≠ 0
                    Left sorted? arr[0]≤arr[3] → 4≤7 YES
                    ┌────────────────────────────────┐
                    │ Target in [4..7)? NO           │
                    │ → Search RIGHT                  │
                    └────────────────────────────────┘
                              │
                       mid = arr[5] = 1 ≠ 0
                       Left sorted? arr[4]≤arr[5] → 0≤1 YES
                       ┌────────────────────────────┐
                       │ Target in [0..1)? YES       │
                       │ → Search LEFT               │
                       └────────────────────────────┘
                              │
                        mid = arr[4] = 0 ✓ FOUND!
```

---

## Problem 3: Search with Duplicates

When duplicates exist, `arr[lo] == arr[mid] == arr[hi]` makes it impossible to determine which half is sorted.

```
Example: [2, 2, 2, 3, 2, 2]  target = 3
          lo     mid       hi
          
arr[lo]=2 == arr[mid]=2 == arr[hi]=2
→ Can't determine which half is sorted!
→ Solution: shrink from both ends
```

```
function searchRotatedDuplicates(arr, n, target):
    lo = 0, hi = n - 1
    
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        
        if arr[mid] == target:
            return true
        
        // Handle duplicates: can't determine sorted half
        if arr[lo] == arr[mid] && arr[mid] == arr[hi]:
            lo += 1
            hi -= 1
            continue
        
        // Left half is sorted
        if arr[lo] <= arr[mid]:
            if arr[lo] <= target < arr[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        
        // Right half is sorted
        else:
            if arr[mid] < target <= arr[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    
    return false
```

**Note**: Worst case becomes O(n) with duplicates (e.g., [2,2,2,2,2,2,3,2,2,2]).

---

## Problem 4: Find Rotation Count

The rotation count equals the index of the minimum element.

```
[4, 5, 6, 7, 1, 2, 3]
                ▲
    min at index 4 → rotated 4 times

[1, 2, 3, 4, 5]       → rotation count = 0 (not rotated)
[5, 1, 2, 3, 4]       → rotation count = 1
[3, 4, 5, 1, 2]       → rotation count = 3
```

```
function rotationCount(arr, n):
    return findMinIndex(arr, n)   // same as finding min index
```

---

## Complexity Analysis

| Variant | Time | Space |
|---------|------|-------|
| Find minimum | O(log n) | O(1) |
| Search (no duplicates) | O(log n) | O(1) |
| Search (with duplicates) | O(log n) avg, O(n) worst | O(1) |
| Find rotation count | O(log n) | O(1) |

---

## Why This Is Divide and Conquer

```
Divide:   Split array at mid
Conquer:  Determine which half is sorted
          → Decide which half can contain the target
Combine:  No combination needed (just return result)

Key D&C insight: At each step, we eliminate HALF the search space
by using the structural property of the rotated array.
```

---

## Edge Cases

```
┌────────────────────────────────────────────────────┐
│  1. Array not rotated: [1,2,3,4,5]                 │
│     → Works correctly (degenerates to normal BS)   │
│                                                    │
│  2. Array of size 1: [5]                           │
│     → lo = hi = 0, found immediately               │
│                                                    │
│  3. Array of size 2: [3,1]                         │
│     → Works: mid=0, check both possibilities       │
│                                                    │
│  4. Target not present:                            │
│     → Returns -1 or false                          │
│                                                    │
│  5. All elements same: [2,2,2,2]                   │
│     → O(n) in worst case with duplicates           │
└────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Rotated array | Sorted, then shifted; one half always sorted |
| Decision key | Check which half is sorted via arr[lo] vs arr[mid] |
| Find minimum | Compare arr[mid] with arr[hi] to decide direction |
| Search logic | If target in sorted half → search there, else other half |
| Duplicates | When arr[lo]=arr[mid]=arr[hi], shrink ends by 1 |
| Worst case with dups | O(n) — can't eliminate half |

---

## Quick Revision Questions

1. **Why is at least one half of a rotated sorted array always sorted?**
2. **How do you determine which half is sorted?**
3. **What condition tells you the target is in the sorted left half?**
4. **Why does the presence of duplicates degrade worst-case to O(n)?**
5. **How is finding the rotation count related to finding the minimum?**

---

[← Previous: Lower and Upper Bounds](05-lower-and-upper-bounds.md) | [Back to Unit 3](../README.md)
