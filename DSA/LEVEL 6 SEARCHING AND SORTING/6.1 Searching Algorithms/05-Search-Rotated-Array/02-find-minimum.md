# Chapter 2: Find Minimum in Rotated Sorted Array

[← Previous: Rotated Array Concept](01-rotated-array-concept.md) | [Next: Search Target →](03-search-target.md)

---

## Overview

Finding the minimum element in a rotated sorted array is equivalent to finding the **pivot point** — the index where the array "breaks" from descending back to ascending. This can be solved in **O(log n)** using binary search.

---

## Problem Statement

> Given a rotated sorted array of **distinct** elements, find the minimum element.

```
   Input:  [4, 5, 6, 7, 0, 1, 2]
   Output: 0
   
   Input:  [3, 4, 5, 1, 2]
   Output: 1
   
   Input:  [1, 2, 3, 4, 5]   (not rotated)
   Output: 1
```

---

## Key Insight

```
   Compare arr[mid] with arr[hi]:
   
   Case 1: arr[mid] > arr[hi]
   ─────────────────────────────
     Value
       │     ●
       │   ●
       │ ●         mid is here
       │                 ●      ← minimum is somewhere here
       │               ●
       └──────┬──────┬──────
            lo  mid    hi
     
     Minimum is in RIGHT half → lo = mid + 1
   
   Case 2: arr[mid] < arr[hi]
   ─────────────────────────────
     Value
       │                 ●
       │               ●
       │ ●         mid is here (or left of it) might be min
       │   ●
       │     ●
       └──────┬──────┬──────
            lo  mid    hi
     
     Minimum is at mid or LEFT → hi = mid
   
   Case 3: arr[mid] == arr[hi]
   ─────────────────────────────
     Only possible with distinct elements if lo == mid == hi
     → already converged
```

---

## Why Compare with `arr[hi]` and Not `arr[lo]`?

```
   Array: [1, 2, 3, 4, 5]  (not rotated)
   
   Compare with arr[lo]:
   arr[mid] = 3, arr[lo] = 1
   3 > 1 → left is sorted → min "might be right"? 
   But min is actually at lo! This logic fails.
   
   Compare with arr[hi]:
   arr[mid] = 3, arr[hi] = 5
   3 < 5 → hi = mid → keeps shrinking toward index 0 ✓
```

---

## Pseudocode

```
   FIND-MINIMUM(arr):
       lo = 0, hi = n - 1
       while lo < hi:                    // Template 3
           mid = lo + (hi - lo) / 2
           if arr[mid] > arr[hi]:
               lo = mid + 1             // min is strictly right of mid
           else:
               hi = mid                 // mid could be the min
       return arr[lo]                   // lo == hi → minimum
```

---

## Step-by-Step Trace

```
   arr = [4, 5, 6, 7, 0, 1, 2]
   
   Step 1: lo=0 hi=6 mid=3
           arr[3]=7 > arr[6]=2 → lo = 4
           
   Step 2: lo=4 hi=6 mid=5
           arr[5]=1 < arr[6]=2 → hi = 5
           
   Step 3: lo=4 hi=5 mid=4
           arr[4]=0 < arr[5]=1 → hi = 4
           
   Step 4: lo=4 hi=4 → STOP
           return arr[4] = 0 ✓
```

```
   Visualizing search space shrinkage:
   
   [4, 5, 6, 7, 0, 1, 2]     Full array
                [0, 1, 2]     After step 1
                [0, 1]        After step 2
                [0]           After step 3 → answer!
```

---

## Implementation (Java)

```java
public int findMin(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) {
            lo = mid + 1;       // min is to the right
        } else {
            hi = mid;           // mid could be min
        }
    }
    return nums[lo];
}
```

---

## Implementation (Python)

```python
def findMin(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] > nums[hi]:
            lo = mid + 1
        else:
            hi = mid
    return nums[lo]
```

---

## Implementation (C++)

```cpp
int findMin(vector<int>& nums) {
    int lo = 0, hi = nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi])
            lo = mid + 1;
        else
            hi = mid;
    }
    return nums[lo];
}
```

---

## Edge Cases

```
   ┌──────────────────────┬─────────────┬──────────────────────┐
   │ Case                 │ Input       │ Output               │
   ├──────────────────────┼─────────────┼──────────────────────┤
   │ Not rotated          │ [1,2,3,4,5] │ 1 (index 0)          │
   │ Fully rotated (n-1)  │ [2,3,4,5,1] │ 1 (index 4)          │
   │ Single element       │ [5]         │ 5 (index 0)          │
   │ Two elements         │ [2, 1]      │ 1 (index 1)          │
   │ Two elements (sorted)│ [1, 2]      │ 1 (index 0)          │
   │ Large rotation       │ [3,1,2]     │ 1 (index 1)          │
   └──────────────────────┴─────────────┴──────────────────────┘
```

---

## Finding Rotation Count

The rotation count = index of the minimum element:

```java
public int findRotationCount(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi])
            lo = mid + 1;
        else
            hi = mid;
    }
    return lo;   // return index, not value
}
```

```
   [4, 5, 6, 7, 0, 1, 2] → returns 4 (rotated 4 times)
   [1, 2, 3, 4, 5]       → returns 0 (not rotated)
```

---

## Complexity Analysis

| Metric | Value |
|--------|-------|
| Time | O(log n) — halving each iteration |
| Space | O(1) — only lo, hi, mid |
| Comparisons | At most ⌈log₂ n⌉ |

---

## Quick Revision Questions

1. **Why do we compare `arr[mid]` with `arr[hi]` instead of `arr[lo]`?**
2. **Why is `lo = mid + 1` safe when `arr[mid] > arr[hi]`?**
3. **Why is `hi = mid` (not `mid - 1`) in the else branch?**
4. **What happens when the array is not rotated at all?**
5. **How do you find the rotation count from the minimum?**
6. **Trace the algorithm on `[3, 1, 2]`.**

---

[← Previous: Rotated Array Concept](01-rotated-array-concept.md) | [Next: Search Target →](03-search-target.md)
