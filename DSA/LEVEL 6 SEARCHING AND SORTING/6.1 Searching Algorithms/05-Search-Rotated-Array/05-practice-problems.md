# Chapter 5: Practice Problems — Rotated Sorted Array

[← Previous: Handling Duplicates](04-handling-duplicates.md) | [Next Unit: Binary Search on Answer →](../06-Binary-Search-On-Answer/01-concept.md)

---

## Overview

This chapter provides **practice problems** on rotated sorted arrays, ranging from easy to hard.

---

## Problem 1: Find Rotation Count (Easy)

**Statement:** Given a rotated sorted array of distinct integers, find how many times it was rotated.

```
   [4, 5, 6, 7, 0, 1, 2] → 4 rotations
   [1, 2, 3, 4, 5]       → 0 rotations
```

### Solution

```python
def countRotations(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] > nums[hi]:
            lo = mid + 1
        else:
            hi = mid
    return lo   # index of minimum = rotation count
```

**Key insight:** Rotation count = index of minimum element.

---

## Problem 2: Check If Array Is Sorted and Rotated (Easy)

**Statement:** Given an array, check if it's a sorted array that has been rotated.

```
   [3, 4, 5, 1, 2] → True (sorted and rotated)
   [2, 1, 3, 4]    → False (not sorted when unrotated)
   [1, 2, 3, 4]    → True (sorted, 0 rotations counts)
```

### Solution

```python
def check(nums):
    n = len(nums)
    count = 0
    for i in range(n):
        if nums[i] > nums[(i + 1) % n]:
            count += 1
    return count <= 1   # At most one "break" point
```

**Key insight:** A sorted-and-rotated array has at most one index where `arr[i] > arr[i+1]`.

---

## Problem 3: Find Element in Rotated Array (No Duplicates) (Medium)

**Statement:** LeetCode 33 — standard problem.

```java
public int search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        
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

### Trace

```
   nums = [6, 7, 0, 1, 2, 4, 5], target = 4
   
   lo=0 hi=6 mid=3: arr[3]=1
     arr[0]=6 > arr[3]=1 → RIGHT sorted
     4 in (1, 5]? Yes → lo=4
   
   lo=4 hi=6 mid=5: arr[5]=4 == target → FOUND at 5 ✓
```

---

## Problem 4: Search in Rotated Array II (With Duplicates) (Medium)

**Statement:** LeetCode 81 — return `true`/`false`.

```python
def search(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] == target:
            return True
        if nums[lo] == nums[mid] == nums[hi]:
            lo += 1
            hi -= 1
        elif nums[lo] <= nums[mid]:
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return False
```

---

## Problem 5: Find Minimum in Rotated Sorted Array II (Hard)

**Statement:** LeetCode 154 — find minimum with duplicates.

```cpp
int findMin(vector<int>& nums) {
    int lo = 0, hi = nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi])
            lo = mid + 1;
        else if (nums[mid] < nums[hi])
            hi = mid;
        else
            hi--;     // safe: arr[hi] == arr[mid], not losing min
    }
    return nums[lo];
}
```

### Trace

```
   nums = [3, 3, 1, 3]
   
   lo=0 hi=3 mid=1: arr[1]=3 == arr[3]=3 → hi=2
   lo=0 hi=2 mid=1: arr[1]=3 > arr[2]=1  → lo=2
   lo=2 hi=2 → return arr[2] = 1 ✓
```

---

## Problem 6: Search in Nearly Sorted Array (Medium)

**Statement:** Array is sorted but each element may be swapped with the adjacent element. Find target.

```
   Sorted:      [1, 2, 3, 4, 5]
   Nearly sorted: [2, 1, 3, 5, 4]   (1↔2 swapped, 4↔5 swapped)
   
   For element at index i, it could actually be at i-1, i, or i+1.
```

### Solution

```java
public int searchNearlySorted(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        
        if (arr[mid] == target) return mid;
        if (mid > lo && arr[mid - 1] == target) return mid - 1;
        if (mid < hi && arr[mid + 1] == target) return mid + 1;
        
        if (arr[mid] < target)
            lo = mid + 2;      // Skip mid and mid+1 (already checked)
        else
            hi = mid - 2;      // Skip mid and mid-1 (already checked)
    }
    return -1;
}
```

**Key insight:** Check `mid-1`, `mid`, and `mid+1` each iteration. Move by 2 instead of 1.

---

## Problem Summary Table

| # | Problem | Difficulty | Duplicates? | Time |
|---|---------|-----------|-------------|------|
| 1 | Rotation count | Easy | No | O(log n) |
| 2 | Check sorted-and-rotated | Easy | Yes | O(n) |
| 3 | Search target | Medium | No | O(log n) |
| 4 | Search target II | Medium | Yes | O(n) worst |
| 5 | Find min II | Hard | Yes | O(n) worst |
| 6 | Nearly sorted search | Medium | No | O(log n) |

---

## Quick Revision Questions

1. **What's the relationship between rotation count and minimum index?**
2. **Why does the nearly sorted search move lo/hi by 2 instead of 1?**
3. **How many "break points" does a valid sorted-and-rotated array have?**
4. **Why is LeetCode 81 harder than LeetCode 33?**
5. **Can we find the minimum in O(log n) when duplicates exist? Why or why not?**

---

[← Previous: Handling Duplicates](04-handling-duplicates.md) | [Next Unit: Binary Search on Answer →](../06-Binary-Search-On-Answer/01-concept.md)
