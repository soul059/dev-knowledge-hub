# Chapter 6: Practice Problems — Binary Search Templates

[← Previous: Template Comparison](05-template-comparison.md) | [Next Unit: Search in Rotated Sorted Array →](../05-Search-Rotated-Array/01-rotated-array-concept.md)

---

## Overview

This chapter provides **graded practice problems** to cement your understanding of binary search templates. Each problem includes hints, the recommended template, and a full solution.

---

## Problem 1: First Bad Version (Easy)

**Statement:** Versions `1..n`. API `isBadVersion(v)`. Find the first bad version.

```
   Version:  1   2   3   4   5   6   7
   Bad?:     F   F   F   T   T   T   T
                         ↑ first bad = 4
```

**Template:** 3 (Shrinking Window)

### Solution (Java)

```java
public int firstBadVersion(int n) {
    int lo = 1, hi = n;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;    // left-biased (hi = mid branch)
        if (isBadVersion(mid))
            hi = mid;          // mid could be first bad
        else
            lo = mid + 1;      // mid is good, skip it
    }
    return lo;                 // lo == hi → first bad version
}
```

### Solution (Python)

```python
def firstBadVersion(n):
    lo, hi = 1, n
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if isBadVersion(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

### Trace

```
   n = 7, first bad = 4
   
   lo=1  hi=7  mid=4  isBad(4)=T → hi=4
   lo=1  hi=4  mid=2  isBad(2)=F → lo=3
   lo=3  hi=4  mid=3  isBad(3)=F → lo=4
   lo=4  hi=4  → STOP → return 4 ✓
```

---

## Problem 2: Guess Number Higher or Lower (Easy)

**Statement:** Pick a number from 1 to n. API `guess(num)` returns -1 (lower), 0 (correct), 1 (higher).

**Template:** 1 (Standard)

### Solution (Java)

```java
public int guessNumber(int n) {
    int lo = 1, hi = n;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int result = guess(mid);
        if (result == 0) return mid;
        else if (result == 1) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1; // Should not reach here
}
```

---

## Problem 3: Find Smallest Letter Greater Than Target (Easy)

**Statement:** Given a sorted array of chars, find the smallest char > target. Wraps around.

```
   letters = ['c', 'f', 'j'], target = 'f'
   Answer: 'j'   (next letter strictly greater)
```

**Template:** 3 (Shrinking Window)

### Solution (Python)

```python
def nextGreatestLetter(letters, target):
    lo, hi = 0, len(letters)   # hi = n (for wraparound)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if letters[mid] <= target:
            lo = mid + 1
        else:
            hi = mid
    return letters[lo % len(letters)]  # Wrap if lo == n
```

---

## Problem 4: Find First and Last Position (Medium)

**Statement:** Given sorted array, find starting and ending position of target. O(log n).

```
   arr = [5, 7, 7, 8, 8, 10], target = 8
   Output: [3, 4]
```

**Template:** 2 (Boundary Finding)

### Solution (Java)

```java
public int[] searchRange(int[] nums, int target) {
    return new int[]{findFirst(nums, target), findLast(nums, target)};
}

int findFirst(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1, result = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) {
            result = mid;
            hi = mid - 1;       // keep searching left
        } else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return result;
}

int findLast(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1, result = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) {
            result = mid;
            lo = mid + 1;       // keep searching right
        } else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return result;
}
```

---

## Problem 5: Sqrt(x) (Easy)

**Statement:** Compute `floor(sqrt(x))` without using `sqrt` function.

```
   x = 17
   sqrt(17) = 4.123...
   floor = 4
   
   Binary search: find largest mid where mid*mid <= x
```

**Template:** 2 (Boundary — find last satisfying)

### Solution (Python)

```python
def mySqrt(x):
    if x < 2:
        return x
    lo, hi = 1, x // 2
    result = 1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if mid * mid <= x:
            result = mid       # candidate
            lo = mid + 1       # try larger
        else:
            hi = mid - 1
    return result
```

### Solution (C++)

```cpp
int mySqrt(int x) {
    if (x < 2) return x;
    long lo = 1, hi = x / 2, result = 1;
    while (lo <= hi) {
        long mid = lo + (hi - lo) / 2;
        if (mid * mid <= (long)x) {
            result = mid;
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return (int)result;
}
```

---

## Problem 6: Search a 2D Matrix (Medium)

**Statement:** `m × n` matrix, each row sorted, first element of each row > last of previous. Find target.

**Template:** 1 (Standard — treat as 1D array)

### Solution (Java)

```java
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int lo = 0, hi = m * n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n];    // 1D → 2D conversion
        if (val == target) return true;
        else if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

```
   Mapping:  1D index → 2D:
   mid = 7, n = 4
   row = 7 / 4 = 1
   col = 7 % 4 = 3
   → matrix[1][3]
```

---

## Problem 7: Find Peak Element (Medium)

**Statement:** Array where `nums[i] ≠ nums[i+1]`. Find a peak (element > both neighbors).

**Template:** 3 (Shrinking Window)

### Solution (Python)

```python
def findPeakElement(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] < nums[mid + 1]:
            lo = mid + 1       # peak is on the right
        else:
            hi = mid           # mid could be peak
    return lo
```

---

## Problem 8: Single Element in Sorted Array (Medium)

**Statement:** Sorted array where every element appears twice except one. Find the single element in O(log n).

```
   [1, 1, 2, 3, 3, 4, 4, 8, 8]
            ↑ single = 2
```

**Template:** 3 (Shrinking Window)

### Key Insight

```
   Before single: pairs start at even indices
   [1, 1, 2, 3, 3, 4, 4, 8, 8]
    0  1  2  3  4  5  6  7  8
    ↑  ↑        ← pair at (0,1), then disrupted at index 2
   
   If mid is even and arr[mid] == arr[mid+1] → single is to the right
   If mid is even and arr[mid] != arr[mid+1] → single is at mid or left
```

### Solution (Java)

```java
public int singleNonDuplicate(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (mid % 2 == 1) mid--;   // Ensure mid is even
        if (nums[mid] == nums[mid + 1])
            lo = mid + 2;          // Pair intact → single is right
        else
            hi = mid;              // Pair broken → single is here or left
    }
    return nums[lo];
}
```

---

## Problem Difficulty Progression

```
   Easy                    Medium                   Hard
   ─────────────────────────────────────────────────────────
   First Bad Version       Find First & Last        Median of
   Guess Number            Search 2D Matrix         Two Sorted
   Sqrt(x)                 Peak Element             Arrays
   Smallest Letter         Single Element           Split Array
                                                    Largest Sum
```

---

## Summary Table

| # | Problem | Template | Key Trick |
|---|---------|----------|-----------|
| 1 | First Bad Version | 3 | `hi = mid` |
| 2 | Guess Number | 1 | Standard exact match |
| 3 | Smallest Letter | 3 | Wraparound with `%` |
| 4 | First & Last Position | 2 | Two passes: left + right |
| 5 | Sqrt(x) | 2 | Find last `mid² ≤ x` |
| 6 | Search 2D Matrix | 1 | Flatten `mid/n`, `mid%n` |
| 7 | Peak Element | 3 | Compare `mid` vs `mid+1` |
| 8 | Single Element | 3 | Check pair alignment |

---

## Quick Revision Questions

1. **For "First Bad Version", why do we use `lo < hi` instead of `lo <= hi`?**
2. **In Sqrt(x), why do we track a `result` variable?**
3. **How do you convert a 1D index to 2D coordinates in a matrix?**
4. **What's the key observation that makes Single Element in Sorted Array work with BS?**
5. **For each problem above, state the time and space complexity.**

---

[← Previous: Template Comparison](05-template-comparison.md) | [Next Unit: Search in Rotated Sorted Array →](../05-Search-Rotated-Array/01-rotated-array-concept.md)
