# Chapter 4: Splitting & Partitioning Problems

[← Previous: Capacity Problems](03-capacity-problems.md) | [Next: Aggressive Cows →](05-aggressive-cows.md)

---

## Overview

Splitting problems ask: **"Partition an array into groups to minimize the maximum sum (or some cost)."** These are among the most classic and challenging BS-on-answer problems.

---

## Problem 1: Split Array Largest Sum (LeetCode 410)

> Split array into `m` subarrays to **minimize the largest subarray sum**.

```
   nums = [7, 2, 5, 10, 8], m = 2
   
   Possible splits:
   [7] [2, 5, 10, 8]      → max(7, 25) = 25
   [7, 2] [5, 10, 8]      → max(9, 23) = 23
   [7, 2, 5] [10, 8]      → max(14, 18) = 18  ← minimum!
   [7, 2, 5, 10] [8]      → max(24, 8) = 24
   
   Answer: 18
```

### Search Space

```
   lo = max(nums) = 10     // Each subarray has at least 1 element
   hi = sum(nums) = 32     // All in one subarray
   
   maxSum:    10  11  12  13  14  15  16  17  18  ...  32
   splits:    5   4   4   4   3   3   3   3   2   ...   1
   ≤ m=2?     F   F   F   F   F   F   F   F   T   ...   T
                                                ↑
                                          minimum = 18
```

### Feasibility Function

```
   canSplit(maxSum):
       Count how many subarrays needed if each ≤ maxSum.
       Return count ≤ m.
   
   Greedy: Go left to right, accumulate sum.
   When adding next element would exceed maxSum, start new subarray.
```

### Solution (Java)

```java
public int splitArray(int[] nums, int m) {
    int lo = 0, hi = 0;
    for (int num : nums) {
        lo = Math.max(lo, num);
        hi += num;
    }
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canSplit(nums, mid, m)) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    return lo;
}

boolean canSplit(int[] nums, int maxSum, int m) {
    int count = 1, currentSum = 0;
    for (int num : nums) {
        if (currentSum + num > maxSum) {
            count++;
            currentSum = num;
        } else {
            currentSum += num;
        }
    }
    return count <= m;
}
```

### Solution (Python)

```python
def splitArray(nums, m):
    def canSplit(maxSum):
        count, current = 1, 0
        for num in nums:
            if current + num > maxSum:
                count += 1
                current = num
            else:
                current += num
        return count <= m
    
    lo, hi = max(nums), sum(nums)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if canSplit(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

### Detailed Trace

```
   nums = [7, 2, 5, 10, 8], m = 2
   lo = 10, hi = 32
   
   mid = 21: canSplit(21)?
     current=0+7=7, +2=9, +5=14, +10=24>21 → count=2, current=10, +8=18
     count=2 ≤ 2 → TRUE → hi=21
   
   mid = 15: canSplit(15)?
     7, +2=9, +5=14, +10=24>15 → count=2, 10, +8=18>15 → count=3
     count=3 > 2 → FALSE → lo=16
   
   mid = 18: canSplit(18)?
     7, +2=9, +5=14, +10=24>18 → count=2, 10, +8=18
     count=2 ≤ 2 → TRUE → hi=18
   
   mid = 17: canSplit(17)?
     7, +2=9, +5=14, +10=24>17 → count=2, 10, +8=18>17 → count=3
     count=3 > 2 → FALSE → lo=18
   
   lo=18 hi=18 → return 18 ✓
```

---

## Problem 2: Painter's Partition (Book Allocation)

> `n` books with pages `[p1, p2, ..., pn]`. Allocate to `m` students. Each student gets a **contiguous** set of books. **Minimize the maximum pages** any student reads.

This is **identical** to Split Array Largest Sum!

```
   pages = [12, 34, 67, 90], students = 2
   
   [12] [34, 67, 90]  → max(12, 191) = 191
   [12, 34] [67, 90]  → max(46, 157) = 157
   [12, 34, 67] [90]  → max(113, 90) = 113  ← minimum!
   
   Answer: 113
```

---

## Problem 3: Divide Chocolate (LeetCode 1231)

> Cut a chocolate bar into `k+1` pieces. **Maximize the minimum** piece size.

```
   sweetness = [1, 2, 3, 4, 5, 6, 7, 8, 9], k = 5
   
   Here k+1 = 6 pieces.
   Cut to maximize the minimum piece sweetness.
```

### Key Difference: This is a MAXIMIZE problem!

```
   Find MAXIMUM where canCut(mid) = true
   → Use lo = mid template (right-biased)
   
   minSweet:  1   2   3   4   5   6   7   8   9
   pieces:    9   6   5   4   3   3   2   2   1
   ≥ 6 pcs?   T   T   F   F   F   F   F   F   F
                  ↑ WRONG! We need ≥ k+1 pieces
   
   Wait — reversed! Larger minSweet → fewer pieces.
   
   canCut(minSweet): count pieces ≥ minSweet → return count ≥ k+1
```

### Solution (Python)

```python
def maximizeSweetness(sweetness, k):
    def canCut(minSweet):
        pieces, current = 0, 0
        for s in sweetness:
            current += s
            if current >= minSweet:
                pieces += 1
                current = 0
        return pieces >= k + 1
    
    lo, hi = 1, sum(sweetness) // (k + 1)
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2    # right-biased!
        if canCut(mid):
            lo = mid          # mid works, try bigger
        else:
            hi = mid - 1      # mid too big
    return lo
```

---

## Minimize vs Maximize Problems

```
   ┌────────────────────────────────────────────────────────────────┐
   │                                                                │
   │  MINIMIZE the maximum (Split Array, Ship, Painter):           │
   │    lo = max(element)                                          │
   │    hi = sum(all)                                              │
   │    Template: hi = mid (find first feasible)                   │
   │                                                                │
   │  MAXIMIZE the minimum (Divide Chocolate, Aggressive Cows):    │
   │    lo = min_possible                                          │
   │    hi = max_possible                                          │
   │    Template: lo = mid (find last feasible, right-biased)      │
   │                                                                │
   └────────────────────────────────────────────────────────────────┘
```

---

## Complexity

| Problem | Binary Search Range | Feasibility | Total |
|---------|-------------------|-------------|-------|
| Split Array | O(log(sum - max)) | O(n) | O(n log S) |
| Painter's Partition | O(log(sum - max)) | O(n) | O(n log S) |
| Divide Chocolate | O(log(sum/(k+1))) | O(n) | O(n log S) |

---

## Quick Revision Questions

1. **How is Split Array Largest Sum related to Painter's Partition?**
2. **Why is lo = max(element) for minimize-the-maximum problems?**
3. **How does the template differ for maximize-the-minimum problems?**
4. **In canSplit, why do we start count at 1, not 0?**
5. **What's the time complexity of Split Array Largest Sum?**
6. **Trace `splitArray([1, 4, 4], 3)` step by step.**

---

[← Previous: Capacity Problems](03-capacity-problems.md) | [Next: Aggressive Cows →](05-aggressive-cows.md)
