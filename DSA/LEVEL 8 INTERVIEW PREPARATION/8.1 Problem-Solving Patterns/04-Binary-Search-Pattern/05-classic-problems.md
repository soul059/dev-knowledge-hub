# Chapter 5: Classic Problems

## 📋 Chapter Overview
Deep-dive into high-frequency binary search interview problems with full traces.

---

## 🔍 Problem 1: Median of Two Sorted Arrays (Hard)

**Problem:** Find the median of two sorted arrays in O(log(min(m,n))).

### Key Insight

```
  Binary search on the PARTITION of the smaller array.
  
  nums1: [1, 3, 8, 9, 15]    (m = 5)
  nums2: [7, 11, 18, 19, 21, 25]  (n = 6)
  
  Total = 11 elements → median is element at position 6
  
  We need to partition both arrays such that:
    left side has exactly (m+n+1)/2 elements
    max(left) <= min(right)
```

### Partition Visualization

```
  cut1 = 3 (take 3 from nums1)
  cut2 = (11+1)/2 - 3 = 3 (take 3 from nums2)
  
  nums1:  [1, 3, 8 | 9, 15]
  nums2:  [7, 11, 18 | 19, 21, 25]
  
  Left:  {1, 3, 7, 8, 11, 18}  → max = 18
  Right: {9, 15, 19, 21, 25}   → min = 9
  
  18 > 9 → invalid partition! nums1 contributes too many large elements.
  Move cut1 left.
```

### Solution

```
function findMedian(nums1, nums2):
    if len(nums1) > len(nums2):
        swap(nums1, nums2)          // search smaller array
    
    m = len(nums1), n = len(nums2)
    lo = 0, hi = m
    half = (m + n + 1) / 2

    while lo <= hi:
        cut1 = lo + (hi - lo) / 2
        cut2 = half - cut1

        L1 = nums1[cut1-1] if cut1 > 0 else -INF
        R1 = nums1[cut1]   if cut1 < m else +INF
        L2 = nums2[cut2-1] if cut2 > 0 else -INF
        R2 = nums2[cut2]   if cut2 < n else +INF

        if L1 <= R2 AND L2 <= R1:     // valid partition
            if (m + n) is odd:
                return max(L1, L2)
            else:
                return (max(L1, L2) + min(R1, R2)) / 2
        else if L1 > R2:
            hi = cut1 - 1             // too many from nums1
        else:
            lo = cut1 + 1             // too few from nums1
```

**Complexity:** O(log(min(m, n))) time, O(1) space.

---

## 🔍 Problem 2: Find K-th Smallest in Sorted Matrix

**Problem:** n×n matrix where each row and column is sorted. Find k-th smallest.

```
  ┌─────────────┐
  │  1  5  9    │   k = 8
  │ 10 11 13    │   Answer: 13
  │ 12 13 15    │
  └─────────────┘
```

### Approach: Binary Search on Answer

```
function kthSmallest(matrix, k):
    n = len(matrix)
    lo = matrix[0][0]
    hi = matrix[n-1][n-1]

    while lo < hi:
        mid = lo + (hi - lo) / 2
        count = countLessOrEqual(matrix, mid, n)
        if count < k:
            lo = mid + 1
        else:
            hi = mid
    return lo

function countLessOrEqual(matrix, target, n):
    count = 0
    row = n - 1
    col = 0
    while row >= 0 AND col < n:
        if matrix[row][col] <= target:
            count += row + 1       // all elements above in this col
            col += 1
        else:
            row -= 1
    return count
```

### Trace

```
  lo=1, hi=15, mid=8 → count(<=8)=2 < 8 → lo=9
  lo=9, hi=15, mid=12 → count(<=12)=5 < 8 → lo=13
  lo=13, hi=15, mid=14 → count(<=14)=8 >= 8 → hi=14
  lo=13, hi=14, mid=13 → count(<=13)=8 >= 8 → hi=13
  lo=13, hi=13 → return 13 ✓
```

**Complexity:** O(n × log(max - min)) time, O(1) space.

---

## 🔍 Problem 3: Aggressive Cows / Magnetic Balls

**Problem:** Place `c` cows in `n` stalls (positions given). Maximize the minimum distance between any two cows.

```
  Stalls: [1, 2, 4, 8, 9]    c = 3

  "Maximize the minimum distance" → Binary Search on Answer!
  
  Answer space: [1, 8]  (min gap = 1, max gap = 9-1 = 8)
```

### Solution

```
function aggressiveCows(stalls, c):
    sort(stalls)
    lo = 1
    hi = stalls[n-1] - stalls[0]

    while lo < hi:
        mid = lo + (hi - lo + 1) / 2    // ← round UP for maximize
        if canPlace(stalls, c, mid):
            lo = mid                      // mid works, try larger
        else:
            hi = mid - 1                 // mid too large
    return lo

function canPlace(stalls, c, minDist):
    count = 1
    lastPos = stalls[0]
    for i = 1 to len(stalls)-1:
        if stalls[i] - lastPos >= minDist:
            count += 1
            lastPos = stalls[i]
    return count >= c
```

### Trace: stalls = [1,2,4,8,9], c = 3

| Step | lo | hi | mid | Can place? | Action |
|------|----|----|-----|-----------|--------|
| 1 | 1 | 8 | 5 | Place at 1,8 → only 2 ✗ | hi = 4 |
| 2 | 1 | 4 | 3 | Place at 1,4,8 → 3 ✓ | lo = 3 |
| 3 | 3 | 4 | 4 | Place at 1,8 → only 2 ✗ | hi = 3 |
| 4 | 3 | 3 | - | - | **Return 3** ✓ |

**Note:** For "maximize" problems, use `mid = lo + (hi - lo + 1) / 2` (round up) with `lo = mid` to avoid infinite loops.

---

## 🔍 Problem 4: Find Peak Element (LeetCode 162)

```
function findPeakElement(nums):
    lo = 0, hi = len(nums) - 1

    while lo < hi:
        mid = lo + (hi - lo) / 2
        if nums[mid] < nums[mid + 1]:
            lo = mid + 1           // rising → peak to the right
        else:
            hi = mid               // falling → peak at mid or left
    return lo
```

### Trace: nums = [1, 2, 1, 3, 5, 6, 4]

| Step | lo | hi | mid | nums[mid] vs nums[mid+1] | Action |
|------|----|----|-----|--------------------------|--------|
| 1 | 0 | 6 | 3 | 3 < 5 (rising) | lo = 4 |
| 2 | 4 | 6 | 5 | 6 > 4 (falling) | hi = 5 |
| 3 | 4 | 5 | 4 | 5 < 6 (rising) | lo = 5 |
| 4 | 5 | 5 | - | - | **Return 5** (value 6) ✓ |

---

## 🔍 Problem 5: Single Element in Sorted Array (LeetCode 540)

**Every element appears twice except one. Find it.**

```
  [1, 1, 2, 3, 3, 4, 4, 8, 8]
         ↑ single = 2
  
  Key insight: Before the single element, pairs start at even index.
  After the single element, pairs start at odd index.
```

```
function singleNonDuplicate(nums):
    lo = 0, hi = len(nums) - 1

    while lo < hi:
        mid = lo + (hi - lo) / 2
        // Ensure mid is even (start of a pair)
        if mid % 2 == 1: mid -= 1
        
        if nums[mid] == nums[mid + 1]:
            lo = mid + 2           // pair intact → single is to the right
        else:
            hi = mid               // pair broken → single is at mid or left
    return nums[lo]
```

### Trace: [1,1,2,3,3,4,4,8,8]

| Step | lo | hi | mid (adjusted) | Pair check | Action |
|------|----|----|----------------|------------|--------|
| 1 | 0 | 8 | 4 | nums[4]=3 == nums[5]=4? No | hi = 4 |
| 2 | 0 | 4 | 2 | nums[2]=2 == nums[3]=3? No | hi = 2 |
| 3 | 0 | 2 | 0 | nums[0]=1 == nums[1]=1? Yes | lo = 2 |
| 4 | 2 | 2 | - | - | **Return nums[2] = 2** ✓ |

---

## 📊 Problem Comparison

| Problem | Type | Time | Key Trick |
|---------|------|------|-----------|
| Median of Two Arrays | Partition search | O(log min(m,n)) | Binary search on cut position |
| K-th in Sorted Matrix | Answer space | O(n log range) | Count ≤ mid using staircase |
| Aggressive Cows | Answer space (max) | O(n log range) | Round UP mid for maximize |
| Peak Element | Boundary | O(log n) | Compare mid with mid+1 |
| Single in Sorted | Boundary | O(log n) | Pair parity check |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Hard problems | Often combine binary search with clever condition |
| Partition-based | Median of two arrays — search cut point |
| Answer space | K-th smallest, aggressive cows — search value range |
| Property-based | Peak, single element — binary search on a property |
| Maximize answer | Round mid UP, use `lo = mid` |

---

## ❓ Revision Questions

1. **Median of Two Arrays: why search the smaller array?**
   → O(log(min(m,n))) instead of O(log(max(m,n))).

2. **Aggressive Cows: why round mid UP?**
   → When maximizing with `lo = mid`, rounding down can cause `lo = mid = lo` (infinite loop).

3. **Peak Element: why is O(log n) possible?**
   → If `nums[mid] < nums[mid+1]`, a peak must exist to the right (the sequence must come down eventually).

4. **K-th smallest in matrix: how does the staircase count work?**
   → Start at bottom-left; move right if ≤ target (add entire column above), move up if > target.

5. **Single element: why force mid to be even?**
   → Pairs normally start at even indices; checking `nums[mid] == nums[mid+1]` tells if the pair is intact.

---

[← Previous: Template and Variations](04-template-and-variations.md) | [Next: When to Apply →](06-when-to-apply.md)
