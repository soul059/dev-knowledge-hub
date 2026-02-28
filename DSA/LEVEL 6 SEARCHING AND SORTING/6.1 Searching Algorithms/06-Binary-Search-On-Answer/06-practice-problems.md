# Chapter 6: Practice Problems — Binary Search on Answer

[← Previous: Aggressive Cows](05-aggressive-cows.md) | [Next Unit: Search in Matrix →](../07-Search-In-Matrix/01-sorted-matrix-types.md)

---

## Overview

Consolidation problems covering BS-on-answer. Each includes the search space, feasibility function, and complete solution.

---

## Problem 1: Minimum Time to Complete Trips (LeetCode 2187)

> `n` buses with trip times `time[i]`. Find minimum total time for at least `totalTrips`.

```
   time = [1, 2, 3], totalTrips = 5
   
   At t=3: bus1 does 3 trips, bus2 does 1, bus3 does 1 → total = 5 ✓
   Answer: 3
```

### Solution

```python
def minimumTime(time, totalTrips):
    def canComplete(t):
        return sum(t // ti for ti in time) >= totalTrips
    
    lo, hi = 1, min(time) * totalTrips
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if canComplete(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

---

## Problem 2: Find the Smallest Divisor (LeetCode 1283)

> Given array and threshold, find smallest divisor such that sum of `ceil(nums[i]/divisor)` ≤ threshold.

```
   nums = [1, 2, 5, 9], threshold = 6
   
   divisor=4: ceil(1/4)+ceil(2/4)+ceil(5/4)+ceil(9/4) = 1+1+2+3 = 7 > 6
   divisor=5: 1+1+1+2 = 5 ≤ 6 ✓
   Answer: 5
```

### Solution (Java)

```java
public int smallestDivisor(int[] nums, int threshold) {
    int lo = 1, hi = 0;
    for (int n : nums) hi = Math.max(hi, n);
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        int sum = 0;
        for (int n : nums) sum += (n + mid - 1) / mid;
        if (sum <= threshold)
            hi = mid;
        else
            lo = mid + 1;
    }
    return lo;
}
```

---

## Problem 3: Median of Two Sorted Arrays (LeetCode 4) — Hard

> Find the median of two sorted arrays in O(log(min(m,n))).

```
   nums1 = [1, 3], nums2 = [2]
   merged = [1, 2, 3] → median = 2
```

### Key Insight: Binary Search on Partition

```
   nums1: [1, 3, 8, 9, 15]    (m = 5)
   nums2: [7, 11, 18, 19, 21, 25]  (n = 6)
   
   Total = 11 elements → median at position 6
   
   We binary search on how many elements come from nums1's left half:
   partition1 = i → take i elements from nums1
   partition2 = (m+n+1)/2 - i → take rest from nums2
   
   Valid partition:
   nums1[i-1] ≤ nums2[j]   AND   nums2[j-1] ≤ nums1[i]
   (left1 ≤ right2)              (left2 ≤ right1)
```

### Solution (Python)

```python
def findMedianSortedArrays(nums1, nums2):
    if len(nums1) > len(nums2):
        nums1, nums2 = nums2, nums1   # BS on shorter array
    
    m, n = len(nums1), len(nums2)
    lo, hi = 0, m
    
    while lo <= hi:
        i = lo + (hi - lo) // 2       # partition1
        j = (m + n + 1) // 2 - i      # partition2
        
        left1 = nums1[i-1] if i > 0 else float('-inf')
        right1 = nums1[i] if i < m else float('inf')
        left2 = nums2[j-1] if j > 0 else float('-inf')
        right2 = nums2[j] if j < n else float('inf')
        
        if left1 <= right2 and left2 <= right1:
            if (m + n) % 2 == 1:
                return max(left1, left2)
            return (max(left1, left2) + min(right1, right2)) / 2
        elif left1 > right2:
            hi = i - 1
        else:
            lo = i + 1
```

### Visual

```
   nums1:  ... left1 | right1 ...
   nums2:  ... left2 | right2 ...
   
   Valid partition:
   left1 ≤ right2    (nums1's max-left ≤ nums2's min-right)
   left2 ≤ right1    (nums2's max-left ≤ nums1's min-right)
   
                    left half        |        right half
   nums1:  [1, 3]                   | [8, 9, 15]
   nums2:  [7, 11, 18, 19]         | [21, 25]
   
   left1=3, right1=8, left2=19, right2=21
   3 ≤ 21 ✓ but 19 > 8 ✗ → need more from nums1 → lo = i + 1
```

---

## Problem 4: Kth Smallest Element in Sorted Matrix (LeetCode 378)

> `n × n` matrix sorted row-wise and column-wise. Find kth smallest.

### BS-on-answer approach

```python
def kthSmallest(matrix, k):
    n = len(matrix)
    lo, hi = matrix[0][0], matrix[n-1][n-1]
    
    while lo < hi:
        mid = lo + (hi - lo) // 2
        count = countLessEqual(matrix, mid)
        if count >= k:
            hi = mid
        else:
            lo = mid + 1
    return lo

def countLessEqual(matrix, target):
    n = len(matrix)
    count = 0
    row, col = n - 1, 0    # Start bottom-left
    while row >= 0 and col < n:
        if matrix[row][col] <= target:
            count += row + 1
            col += 1
        else:
            row -= 1
    return count
```

---

## Problem 5: Maximum Running Time of N Computers (LeetCode 2141) — Hard

> `n` computers, `batteries[i]` minutes each. Run all `n` computers simultaneously. Find max running time.

### Key Insight

```
   If we run for T minutes total:
   - Each battery with charge ≥ T can power one computer for the full T
   - Remaining batteries (charge < T) contribute their full charge
   - Total power needed: n * T
   
   Check: Can we run for T minutes?
   → sum of min(b, T) for all batteries ≥ n * T
```

### Solution (Python)

```python
def maxRunTime(n, batteries):
    lo, hi = 1, sum(batteries) // n
    
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2    # right-biased
        total = sum(min(b, mid) for b in batteries)
        if total >= n * mid:
            lo = mid
        else:
            hi = mid - 1
    return lo
```

---

## Master Problem Table

| # | Problem | Type | lo | hi | Check |
|---|---------|------|----|----|-------|
| 1 | Min Time Trips | Minimize | 1 | min×total | trips ≥ total |
| 2 | Smallest Divisor | Minimize | 1 | max(arr) | sum ≤ threshold |
| 3 | Median Two Arrays | Find partition | 0 | m | valid partition |
| 4 | Kth in Matrix | Minimize | min | max | count ≥ k |
| 5 | Max Run Time | Maximize | 1 | sum/n | energy ≥ n×T |

---

## Quick Revision Questions

1. **For "Min Time Trips", why is `hi = min(time) * totalTrips`?**
2. **How does the staircase counting work for kth in sorted matrix?**
3. **Why do we binary search on the shorter array for median of two arrays?**
4. **For max run time, why is `min(b, T)` the contribution of each battery?**
5. **Categorize each problem as minimize/maximize.**

---

[← Previous: Aggressive Cows](05-aggressive-cows.md) | [Next Unit: Search in Matrix →](../07-Search-In-Matrix/01-sorted-matrix-types.md)
