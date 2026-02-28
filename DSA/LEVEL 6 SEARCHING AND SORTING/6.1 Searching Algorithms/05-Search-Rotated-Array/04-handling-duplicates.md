# Chapter 4: Handling Duplicates in Rotated Sorted Array

[← Previous: Search Target](03-search-target.md) | [Next: Practice Problems →](05-practice-problems.md)

---

## Overview

When the rotated sorted array contains **duplicates**, the worst-case time complexity degrades from O(log n) to **O(n)**. This chapter explains why and how to handle it.

---

## The Problem with Duplicates

```
   arr = [2, 5, 6, 0, 0, 1, 2]
   
   Standard approach:
   lo=0 hi=6 mid=3
   arr[lo]=2, arr[mid]=0, arr[hi]=2
   
   arr[lo] <= arr[mid]? 2 <= 0? No
   → Right half sorted? arr[mid] <= arr[hi]? 0 <= 2? Yes
   
   This case works fine. But what about...
   
   arr = [1, 0, 1, 1, 1]
   lo=0 hi=4 mid=2
   arr[lo]=1, arr[mid]=1, arr[hi]=1
   
   arr[lo] <= arr[mid]? 1 <= 1? Yes → Left sorted?
   BUT [1, 0, 1] is NOT sorted! WRONG!
   
   The problem: When arr[lo] == arr[mid] == arr[hi],
   we CANNOT determine which half is sorted.
```

---

## Visual: The Ambiguous Case

```
   Case A: [1, 1, 1, 1, 2]  (sorted, minimum at start)
   
   Value
     2 │                ●
     1 │ ● ● ● ●
       └─────────────────
         0 1 2 3 4
   
   Case B: [1, 1, 2, 1, 1]  (rotated, minimum at index 3)
   
   Value
     2 │       ●
     1 │ ● ●     ● ●
       └─────────────────
         0 1 2 3 4
   
   In both cases: arr[0] = 1, arr[2] = 1, arr[4] = 1
   They look IDENTICAL to binary search! Cannot distinguish.
```

---

## Solution Strategy

When `arr[lo] == arr[mid] == arr[hi]`, we **cannot** determine which half is sorted. The only safe move is to **shrink both ends by 1**:

```
   lo++, hi--    (skip the ambiguous duplicates)
```

This is worst-case O(n) — consider all elements the same except one.

---

## Find Minimum with Duplicates

### Pseudocode

```
   FIND-MIN-DUPLICATES(arr):
       lo = 0, hi = n - 1
       while lo < hi:
           mid = lo + (hi - lo) / 2
           if arr[mid] > arr[hi]:
               lo = mid + 1          // min is right of mid
           else if arr[mid] < arr[hi]:
               hi = mid              // mid could be min
           else:
               hi--                  // arr[mid] == arr[hi], can't decide
       return arr[lo]
```

### Why `hi--` instead of `lo++`?

```
   arr = [3, 3, 1, 3]
   lo=0 hi=3 mid=1
   arr[mid]=3 == arr[hi]=3 → ambiguous
   
   If lo++: lo=1 → search [3, 1, 3] → still works
   If hi--: hi=2 → search [3, 3, 1] → still works
   
   But hi-- is safer because the minimum could be at lo.
   Doing lo++ might skip the minimum if it's at the current lo.
   hi-- is safe because arr[hi] == arr[mid], so even if arr[hi] 
   was the minimum, arr[mid] has the same value.
```

---

## Implementation: Find Minimum (Java)

```java
public int findMin(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) {
            lo = mid + 1;
        } else if (nums[mid] < nums[hi]) {
            hi = mid;
        } else {
            hi--;           // handle duplicates
        }
    }
    return nums[lo];
}
```

---

## Search Target with Duplicates

### Implementation (Python)

```python
def search(nums, target):
    lo, hi = 0, len(nums) - 1
    
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        
        if nums[mid] == target:
            return True   # Note: returns bool, not index
        
        # Cannot determine sorted half
        if nums[lo] == nums[mid] == nums[hi]:
            lo += 1
            hi -= 1
        # Left half is sorted
        elif nums[lo] <= nums[mid]:
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
    
    return False
```

### Implementation (C++)

```cpp
bool search(vector<int>& nums, int target) {
    int lo = 0, hi = nums.size() - 1;
    
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (nums[mid] == target)
            return true;
        
        if (nums[lo] == nums[mid] && nums[mid] == nums[hi]) {
            lo++;
            hi--;
        } else if (nums[lo] <= nums[mid]) {
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
    
    return false;
}
```

---

## Trace: Find Min with Duplicates

```
   arr = [2, 2, 2, 0, 1, 2]
   
   Step 1: lo=0 hi=5 mid=2
           arr[2]=2 == arr[5]=2 → hi-- → hi=4
   
   Step 2: lo=0 hi=4 mid=2
           arr[2]=2 > arr[4]=1 → lo = 3
   
   Step 3: lo=3 hi=4 mid=3
           arr[3]=0 < arr[4]=1 → hi = 3
   
   Step 4: lo=3 hi=3 → STOP
           return arr[3] = 0 ✓
```

---

## Worst Case Scenario

```
   arr = [1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1]
                                ↑
                              target
   
   Every comparison: arr[lo] == arr[mid] == arr[hi] == 1
   Must do lo++ / hi-- repeatedly → O(n) total
   
   This is UNAVOIDABLE — no algorithm can do better than O(n)
   for this input because we must examine every element to
   distinguish [1,1,...,0,...,1] from [1,1,...,1,...,1].
```

---

## Comparison: With vs Without Duplicates

| Aspect | Without Duplicates | With Duplicates |
|--------|-------------------|----------------|
| Best case | O(1) | O(1) |
| Average case | O(log n) | O(log n) |
| Worst case | O(log n) | **O(n)** |
| Always determines sorted half? | Yes | **No** |
| Extra handling | None | `lo++, hi--` for ambiguous |
| Space | O(1) | O(1) |

---

## When Does O(n) Actually Happen?

```
   The worst case requires MANY duplicates at both ends
   equal to mid. In practice, this is rare.
   
   Average case with random duplicates: O(log n)
   
   Only adversarial inputs cause O(n):
   - [k, k, k, ..., k, x, k, k, ..., k]  where x ≠ k
```

---

## Quick Revision Questions

1. **Why can't we determine the sorted half when `arr[lo] == arr[mid] == arr[hi]`?**
2. **Why do we use `hi--` instead of `lo++` in findMin with duplicates?**
3. **What is the worst-case input for find min with duplicates?**
4. **Can we do better than O(n) for the worst case? Why or why not?**
5. **In the search function, why does the return type change to `boolean` instead of returning an index?**
6. **Trace find minimum on `[10, 1, 10, 10, 10]`.**

---

[← Previous: Search Target](03-search-target.md) | [Next: Practice Problems →](05-practice-problems.md)
