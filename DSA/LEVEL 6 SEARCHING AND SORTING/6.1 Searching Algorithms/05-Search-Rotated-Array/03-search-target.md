# Chapter 3: Search Target in Rotated Sorted Array

[← Previous: Find Minimum](02-find-minimum.md) | [Next: Handling Duplicates →](04-handling-duplicates.md)

---

## Overview

Searching for a specific target in a rotated sorted array requires identifying **which half is sorted**, then deciding whether the target lies in that sorted half. This achieves **O(log n)** time.

---

## Problem Statement

> Given a rotated sorted array of **distinct** integers, find the index of `target`. Return `-1` if not found.

```
   Input:  nums = [4, 5, 6, 7, 0, 1, 2], target = 0
   Output: 4
   
   Input:  nums = [4, 5, 6, 7, 0, 1, 2], target = 3
   Output: -1
```

---

## Algorithm

```
   At each step, at least one half [lo..mid] or [mid..hi] is sorted.
   
   1. Find mid
   2. If arr[mid] == target → FOUND
   3. If LEFT half is sorted (arr[lo] <= arr[mid]):
        a. If target is in [arr[lo], arr[mid]) → search left
        b. Else → search right
   4. Else RIGHT half is sorted:
        a. If target is in (arr[mid], arr[hi]] → search right
        b. Else → search left
```

---

## Visual Walkthrough

```
   arr = [4, 5, 6, 7, 0, 1, 2], target = 0
   
   Step 1: lo=0 hi=6 mid=3
   ┌───┬───┬───┬───┬───┬───┬───┐
   │ 4 │ 5 │ 6 │ 7 │ 0 │ 1 │ 2 │
   └───┴───┴───┴───┴───┴───┴───┘
    lo          mid             hi
   
   arr[lo]=4 ≤ arr[mid]=7 → LEFT is sorted [4,5,6,7]
   Is 0 in [4, 7]? No → search RIGHT
   lo = mid + 1 = 4
   
   Step 2: lo=4 hi=6 mid=5
   ┌───┬───┬───┐
   │ 0 │ 1 │ 2 │
   └───┴───┴───┘
    lo  mid  hi
   
   arr[lo]=0 ≤ arr[mid]=1 → LEFT is sorted [0,1]
   Is 0 in [0, 1]? Yes → search LEFT
   hi = mid - 1 = 4
   
   Step 3: lo=4 hi=4 mid=4
   ┌───┐
   │ 0 │
   └───┘
   lo,mid,hi
   
   arr[mid] = 0 == target → FOUND at index 4 ✓
```

---

## Decision Tree

```
                         arr[mid] == target?
                        /        \
                      Yes         No
                      |            |
                   RETURN       arr[lo] <= arr[mid]?
                    mid        /           \
                             Yes            No
                    (left sorted)     (right sorted)
                        |                  |
               target in                target in
              [arr[lo], arr[mid])?     (arr[mid], arr[hi]]?
              /          \             /          \
            Yes           No         Yes           No
            |              |          |              |
          hi=mid-1      lo=mid+1   lo=mid+1      hi=mid-1
```

---

## Implementation (Java)

```java
public int search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (nums[mid] == target) {
            return mid;
        }
        
        // Left half is sorted
        if (nums[lo] <= nums[mid]) {
            if (nums[lo] <= target && target < nums[mid]) {
                hi = mid - 1;   // target in left sorted half
            } else {
                lo = mid + 1;   // target in right half
            }
        }
        // Right half is sorted
        else {
            if (nums[mid] < target && target <= nums[hi]) {
                lo = mid + 1;   // target in right sorted half
            } else {
                hi = mid - 1;   // target in left half
            }
        }
    }
    
    return -1;
}
```

---

## Implementation (Python)

```python
def search(nums, target):
    lo, hi = 0, len(nums) - 1
    
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        
        if nums[mid] == target:
            return mid
        
        # Left half is sorted
        if nums[lo] <= nums[mid]:
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        # Right half is sorted
        else:
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    
    return -1
```

---

## Implementation (C++)

```cpp
int search(vector<int>& nums, int target) {
    int lo = 0, hi = nums.size() - 1;
    
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (nums[mid] == target)
            return mid;
        
        if (nums[lo] <= nums[mid]) {
            if (nums[lo] <= target && target < nums[mid])
                hi = mid - 1;
            else
                lo = mid + 1;
        } else {
            if (nums[mid] < target && target <= nums[hi])
                lo = mid + 1;
            else
                hi = mid - 1;
        }
    }
    
    return -1;
}
```

---

## Why `nums[lo] <= nums[mid]` (Not `<`)?

```
   When lo == mid (only 1 or 2 elements left):
   
   arr = [3, 1], target = 1
   lo = 0, hi = 1, mid = 0
   
   arr[lo] = 3, arr[mid] = 3 → they're equal!
   
   If we use < :  3 < 3 → FALSE → goes to "right sorted" branch
   If we use <= : 3 <= 3 → TRUE → goes to "left sorted" branch
   
   With <=: Is 1 in [3, 3)? No → lo = mid + 1 = 1
            Now lo=1 hi=1 mid=1 → arr[1]=1 == target → FOUND ✓
   
   Both work here, but <= is more intuitive: a single element IS sorted.
```

---

## Alternative: Two-Step Approach

```
   Step 1: Find the index of minimum (pivot) using findMin
   Step 2: Determine which half to binary search
   Step 3: Standard binary search on that half
   
   Pros: Easier to understand
   Cons: Two passes (still O(log n) but constant factor is larger)
```

```java
public int searchTwoStep(int[] nums, int target) {
    int pivot = findMinIndex(nums);
    int n = nums.length;
    
    // Determine which half to search
    if (target >= nums[pivot] && target <= nums[n - 1]) {
        return binarySearch(nums, pivot, n - 1, target);
    } else {
        return binarySearch(nums, 0, pivot - 1, target);
    }
}
```

---

## Detailed Trace Table

```
   arr = [6, 7, 0, 1, 2, 4, 5], target = 4
   
   ┌──────┬────┬────┬─────┬──────────┬────────────┬──────────┐
   │ Step │ lo │ hi │ mid │ arr[mid] │ Sorted?    │ Action   │
   ├──────┼────┼────┼─────┼──────────┼────────────┼──────────┤
   │  1   │ 0  │ 6  │  3  │    1     │ R sorted   │ 4 in     │
   │      │    │    │     │          │ [1,2,4,5]  │ (1,5]?   │
   │      │    │    │     │          │            │ Yes→lo=4 │
   ├──────┼────┼────┼─────┼──────────┼────────────┼──────────┤
   │  2   │ 4  │ 6  │  5  │    4     │ FOUND!     │ return 5 │
   └──────┴────┴────┴─────┴──────────┴────────────┴──────────┘
```

---

## Edge Cases

| Case | Example | Result |
|------|---------|--------|
| Not rotated | `[1,2,3,4,5]`, target=3 | 2 |
| Target at pivot | `[4,5,1,2,3]`, target=1 | 2 |
| Target is first | `[4,5,1,2,3]`, target=4 | 0 |
| Target is last | `[4,5,1,2,3]`, target=3 | 4 |
| Target absent | `[4,5,1,2,3]`, target=6 | -1 |
| Single element hit | `[1]`, target=1 | 0 |
| Single element miss | `[1]`, target=0 | -1 |
| Two elements | `[2,1]`, target=1 | 1 |

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(log n) |
| Space | O(1) |
| Comparisons | At most 2×⌈log₂ n⌉ (two comparisons per iteration) |

---

## Quick Revision Questions

1. **How do you determine if the left half `[lo..mid]` is sorted?**
2. **Why do we check `nums[lo] <= target < nums[mid]` (not `<=` on right)?**
3. **What happens when the array is not rotated?**
4. **Trace the algorithm on `[5, 1, 2, 3, 4]` with target = 1.**
5. **Compare single-pass vs two-step approach. Which do you prefer?**

---

[← Previous: Find Minimum](02-find-minimum.md) | [Next: Handling Duplicates →](04-handling-duplicates.md)
