# Chapter 5: Practice Problems — Search in Matrix

[← Previous: Kth Smallest](04-kth-smallest.md) | [Next Unit: Ternary Search →](../08-Ternary-Search/01-concept.md)

---

## Overview

Practice problems covering all types of matrix search.

---

## Problem 1: Search a 2D Matrix (LeetCode 74) — Fully Sorted

```java
// Treat as 1D sorted array
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int lo = 0, hi = m * n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n];
        if (val == target) return true;
        else if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

**Time:** O(log(mn))

---

## Problem 2: Search a 2D Matrix II (LeetCode 240) — Row+Col Sorted

```python
def searchMatrix(matrix, target):
    row, col = 0, len(matrix[0]) - 1   # top-right
    while row < len(matrix) and col >= 0:
        if matrix[row][col] == target:
            return True
        elif matrix[row][col] > target:
            col -= 1
        else:
            row += 1
    return False
```

**Time:** O(m + n)

---

## Problem 3: Sorted Matrix — Count Negatives (LeetCode 1351)

> Row-column sorted (non-increasing). Count negative numbers.

```
   ┌────┬────┬────┬────┐
   │  4 │  3 │  2 │ -1 │
   ├────┼────┼────┼────┤
   │  3 │  2 │  1 │ -1 │
   ├────┼────┼────┼────┤
   │  1 │  1 │ -1 │ -2 │
   ├────┼────┼────┼────┤
   │ -1 │ -1 │ -2 │ -3 │
   └────┴────┴────┴────┘
   Answer: 8 negatives
```

### O(m + n) Solution

```python
def countNegatives(grid):
    m, n = len(grid), len(grid[0])
    count = 0
    row, col = 0, n - 1    # top-right
    
    while row < m and col >= 0:
        if grid[row][col] < 0:
            count += m - row    # all below are also negative
            col -= 1
        else:
            row += 1
    return count
```

---

## Problem 4: Row with Maximum 1s (GFG) 

> Binary matrix, each row sorted. Find the row with maximum number of 1s.

```
   ┌───┬───┬───┬───┐
   │ 0 │ 0 │ 1 │ 1 │  → 2 ones
   ├───┼───┼───┼───┤
   │ 0 │ 1 │ 1 │ 1 │  → 3 ones ← maximum!
   ├───┼───┼───┼───┤
   │ 0 │ 0 │ 0 │ 1 │  → 1 one
   └───┴───┴───┴───┘
   Answer: row 1
```

### O(m + n) Solution (Staircase)

```java
public int rowWithMax1s(int[][] mat) {
    int m = mat.length, n = mat[0].length;
    int maxRow = -1, col = n - 1;
    
    for (int row = 0; row < m; row++) {
        while (col >= 0 && mat[row][col] == 1) {
            col--;
            maxRow = row;
        }
    }
    return maxRow;
}
```

**Key insight:** Start at top-right. Move left while seeing 1s. Move down to next row. The column pointer only moves left, so total work = O(m + n).

---

## Problem 5: Median in Row-Wise Sorted Matrix (GFG)

> Find median of all elements. Guaranteed odd number of elements.

```
   ┌───┬───┬───┐
   │ 1 │ 3 │ 5 │
   ├───┼───┼───┤
   │ 2 │ 6 │ 9 │
   ├───┼───┼───┤
   │ 3 │ 6 │ 9 │
   └───┴───┴───┘
   Sorted: [1,2,3,3,5,6,6,9,9] → median = 5
```

### BS on Answer + Binary Search per Row

```python
import bisect

def findMedian(matrix):
    m, n = len(matrix), len(matrix[0])
    desired = (m * n) // 2 + 1    # position of median (1-indexed)
    
    lo = min(row[0] for row in matrix)
    hi = max(row[-1] for row in matrix)
    
    while lo < hi:
        mid = lo + (hi - lo) // 2
        count = sum(bisect.bisect_right(row, mid) for row in matrix)
        if count >= desired:
            hi = mid
        else:
            lo = mid + 1
    return lo
```

**Time:** O(m × log n × log(max - min))  
**Key idea:** Binary search on value, for each candidate count elements ≤ it using binary search on each row.

---

## Summary Table

| # | Problem | Matrix Type | Approach | Time |
|---|---------|-------------|----------|------|
| 1 | Search Matrix I | Fully sorted | Flatten + BS | O(log(mn)) |
| 2 | Search Matrix II | Row+col sorted | Staircase | O(m+n) |
| 3 | Count negatives | Non-increasing | Staircase | O(m+n) |
| 4 | Row with max 1s | Binary rows | Staircase | O(m+n) |
| 5 | Median | Row-sorted | BS on answer | O(m log n log V) |

---

## Quick Revision Questions

1. **When should you use staircase vs flatten for matrix search?**
2. **Why does counting negatives from top-right work in O(m+n)?**
3. **For "row with max 1s", why does the column pointer only move left?**
4. **How does the median problem combine BS-on-answer with per-row BS?**
5. **What is `desired` in the median problem and why `(m*n)//2 + 1`?**

---

[← Previous: Kth Smallest](04-kth-smallest.md) | [Next Unit: Ternary Search →](../08-Ternary-Search/01-concept.md)
