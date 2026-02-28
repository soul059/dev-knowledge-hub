# Chapter 4: Kth Smallest Element in Sorted Matrix

[← Previous: Fully Sorted Matrix](03-fully-sorted-matrix.md) | [Next: Practice Problems →](05-practice-problems.md)

---

## Overview

Finding the kth smallest element in a **row-column sorted** matrix combines staircase counting with binary search on answer. This is LeetCode 378.

---

## Problem Statement

> `n × n` matrix sorted row-wise and column-wise. Find the kth smallest element.

```
   ┌────┬────┬────┐
   │  1 │  5 │  9 │
   ├────┼────┼────┤
   │ 10 │ 11 │ 13 │
   ├────┼────┼────┤
   │ 12 │ 13 │ 15 │
   └────┴────┴────┘
   
   Sorted: [1, 5, 9, 10, 11, 12, 13, 13, 15]
   k = 8 → answer = 13
```

---

## Approach 1: Min-Heap (O(k log n))

```
   1. Push first element of each row into a min-heap: [(1,0,0), (10,1,0), (12,2,0)]
   2. Pop minimum k times
   3. After popping (val, row, col), push (matrix[row][col+1]) if it exists
```

This works but BS-on-Answer is more elegant and efficient for large k.

---

## Approach 2: Binary Search on Answer (O(n × log(max - min)))

### Key Idea

```
   Answer range: [matrix[0][0], matrix[n-1][n-1]]
   
   For a candidate value mid:
     Count how many elements ≤ mid
     If count ≥ k → answer is ≤ mid → hi = mid
     If count < k → answer is > mid → lo = mid + 1
   
   The counting uses staircase technique: O(n)
```

### Counting Elements ≤ mid

```
   Start at bottom-left corner.
   
   ┌────┬────┬────┐
   │  1 │  5 │  9 │   target mid = 11
   ├────┼────┼────┤
   │ 10 │ 11 │ 13 │   Start at (2, 0) = 12
   ├────┼────┼────┤
   │ 12 │ 13 │ 15 │   12 > 11 → move UP
   └────┴────┴────┘
   
   (1, 0) = 10 ≤ 11 → count += 2 (row 0,1), move RIGHT
   (1, 1) = 11 ≤ 11 → count += 2, move RIGHT
   (1, 2) = 13 > 11 → move UP
   (0, 2) = 9 ≤ 11  → count += 1, move RIGHT → out of bounds
   
   Total count = 2 + 2 + 1 = 5
```

```
   Visual: counting elements ≤ 11
   
   ┌────┬────┬────┐
   │ ✓1 │ ✓5 │ ✓9 │     Elements ≤ 11: {1, 5, 9, 10, 11}
   ├────┼────┼────┤      Count = 5
   │✓10 │✓11 │ 13 │
   ├────┼────┼────┤
   │ 12 │ 13 │ 15 │
   └────┴────┴────┘
```

---

## Implementation (Java)

```java
public int kthSmallest(int[][] matrix, int k) {
    int n = matrix.length;
    int lo = matrix[0][0], hi = matrix[n-1][n-1];
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        int count = countLessEqual(matrix, mid);
        if (count >= k) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    return lo;
}

int countLessEqual(int[][] matrix, int target) {
    int n = matrix.length;
    int count = 0;
    int row = n - 1, col = 0;    // bottom-left
    
    while (row >= 0 && col < n) {
        if (matrix[row][col] <= target) {
            count += row + 1;     // all elements in this column above are ≤
            col++;
        } else {
            row--;
        }
    }
    return count;
}
```

---

## Implementation (Python)

```python
def kthSmallest(matrix, k):
    n = len(matrix)
    lo, hi = matrix[0][0], matrix[n-1][n-1]
    
    while lo < hi:
        mid = lo + (hi - lo) // 2
        count = countLessEqual(matrix, mid, n)
        if count >= k:
            hi = mid
        else:
            lo = mid + 1
    return lo

def countLessEqual(matrix, target, n):
    count = 0
    row, col = n - 1, 0
    while row >= 0 and col < n:
        if matrix[row][col] <= target:
            count += row + 1
            col += 1
        else:
            row -= 1
    return count
```

---

## Implementation (C++)

```cpp
int kthSmallest(vector<vector<int>>& matrix, int k) {
    int n = matrix.size();
    int lo = matrix[0][0], hi = matrix[n-1][n-1];
    
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        int count = countLessEqual(matrix, mid, n);
        if (count >= k)
            hi = mid;
        else
            lo = mid + 1;
    }
    return lo;
}

int countLessEqual(vector<vector<int>>& matrix, int target, int n) {
    int count = 0, row = n - 1, col = 0;
    while (row >= 0 && col < n) {
        if (matrix[row][col] <= target) {
            count += row + 1;
            col++;
        } else {
            row--;
        }
    }
    return count;
}
```

---

## Detailed Trace

```
   Matrix:                k = 5
   ┌────┬────┬────┐
   │  1 │  5 │  9 │
   ├────┼────┼────┤
   │ 10 │ 11 │ 13 │
   ├────┼────┼────┤
   │ 12 │ 13 │ 15 │
   └────┴────┴────┘
   
   lo = 1, hi = 15
   
   Step 1: mid = 8
           count(≤8) = 2 (just 1 and 5) < 5 → lo = 9
   
   Step 2: mid = 12
           count(≤12):
             (2,0)=12 ≤ 12 → count+=3, col=1
             (2,1)=13 > 12 → row=1
             (1,1)=11 ≤ 12 → count+=2, col=2
             (1,2)=13 > 12 → row=0
             (0,2)=9 ≤ 12 → count+=1
           count = 3+2+1 = 6 ≥ 5 → hi = 12
   
   Step 3: mid = 10
           count(≤10) = 4 < 5 → lo = 11
   
   Step 4: mid = 11
           count(≤11) = 5 ≥ 5 → hi = 11
   
   lo = 11 = hi → return 11 ✓
   
   Verification: sorted = [1,5,9,10,11,12,13,13,15], 5th = 11 ✓
```

---

## Why Does BS Converge to an Actual Matrix Element?

```
   Consider mid = 11.5 (not in matrix):
   count(≤11.5) = count(≤11) = 5
   
   And for mid = 11 (in matrix):
   count(≤11) = 5
   
   BS finds the SMALLEST value where count ≥ k.
   This value must be a matrix element because:
   - If mid is between two consecutive matrix values a < b,
     then count(≤mid) = count(≤a).
   - BS would keep shrinking until lo = hi = an actual element.
```

---

## Comparison of Approaches

| Approach | Time | Space | When to Use |
|----------|------|-------|-------------|
| Min-Heap | O(k log n) | O(n) | k is small |
| BS on Answer | O(n log(max-min)) | O(1) | k is large |
| Sort all | O(n² log(n²)) | O(n²) | Never (too slow) |

---

## Quick Revision Questions

1. **Why do we start counting from the bottom-left corner?**
2. **Why does `count += row + 1` give the correct count for a column?**
3. **How do we know the answer is always an actual matrix element?**
4. **What is the time complexity of the BS approach?**
5. **When would the min-heap approach be better than BS?**
6. **Trace kthSmallest for k = 3 in the example matrix.**

---

[← Previous: Fully Sorted Matrix](03-fully-sorted-matrix.md) | [Next: Practice Problems →](05-practice-problems.md)
