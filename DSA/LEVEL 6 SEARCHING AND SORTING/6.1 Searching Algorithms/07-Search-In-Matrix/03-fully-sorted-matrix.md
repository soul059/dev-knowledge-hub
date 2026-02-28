# Chapter 3: Search in Fully Sorted Matrix

[← Previous: Row-Column Sorted](02-row-col-sorted.md) | [Next: Kth Smallest in Matrix →](04-kth-smallest.md)

---

## Overview

When the matrix is **fully sorted** (reading left-to-right, top-to-bottom gives a sorted sequence), we can treat it as a **1D sorted array** and apply standard binary search in **O(log(m × n))**.

---

## Problem Statement (LeetCode 74)

> `m × n` matrix. Each row sorted. First integer of each row > last integer of previous row. Determine if target exists.

```
   ┌────┬────┬────┬────┐
   │  1 │  3 │  5 │  7 │    Row 0: [1, 7]
   ├────┼────┼────┼────┤
   │ 10 │ 11 │ 16 │ 20 │    Row 1: [10, 20]
   ├────┼────┼────┼────┤
   │ 23 │ 30 │ 34 │ 50 │    Row 2: [23, 50]
   └────┴────┴────┴────┘
   
   Flattened: [1,3,5,7,10,11,16,20,23,30,34,50]
```

---

## 1D ↔ 2D Index Mapping

```
   Matrix: m rows, n columns
   
   1D index → 2D:
   row = index / n
   col = index % n
   
   2D → 1D:
   index = row * n + col
   
   Example (n = 4):
   ┌────────────────────────────┐
   │ 1D:  0  1  2  3  4  5  6  │
   │ row: 0  0  0  0  1  1  1  │
   │ col: 0  1  2  3  0  1  2  │
   └────────────────────────────┘
   
   index 5 → row = 5/4 = 1, col = 5%4 = 1 → matrix[1][1]
```

---

## Implementation (Java)

```java
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int lo = 0, hi = m * n - 1;
    
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n];    // Convert 1D → 2D
        
        if (val == target) return true;
        else if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

---

## Implementation (Python)

```python
def searchMatrix(matrix, target):
    m, n = len(matrix), len(matrix[0])
    lo, hi = 0, m * n - 1
    
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        val = matrix[mid // n][mid % n]
        
        if val == target:
            return True
        elif val < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return False
```

---

## Implementation (C++)

```cpp
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
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

---

## Step-by-Step Trace

```
   Matrix:                    Target = 11
   ┌────┬────┬────┬────┐
   │  1 │  3 │  5 │  7 │
   ├────┼────┼────┼────┤
   │ 10 │ 11 │ 16 │ 20 │
   ├────┼────┼────┼────┤
   │ 23 │ 30 │ 34 │ 50 │
   └────┴────┴────┴────┘
   n = 4
   
   Step 1: lo=0 hi=11 mid=5
           row=5/4=1 col=5%4=1 → matrix[1][1]=11 = target → FOUND ✓
   
   Only 1 comparison needed! (lucky case)
```

```
   Target = 30:
   
   Step 1: lo=0 hi=11 mid=5 → val=11 < 30 → lo=6
   Step 2: lo=6 hi=11 mid=8 → val=23 < 30 → lo=9
   Step 3: lo=9 hi=11 mid=10 → val=34 > 30 → hi=9
   Step 4: lo=9 hi=9 mid=9 → val=30 = 30 → FOUND ✓
```

---

## Alternative: Two Binary Searches

```
   Step 1: Binary search for the correct ROW
           Find the row where row_first ≤ target ≤ row_last
   Step 2: Binary search within that row
   
   Same O(log(m) + log(n)) = O(log(m×n)) complexity
```

```java
public boolean searchMatrixTwoBS(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    
    // Step 1: Find the row
    int lo = 0, hi = m - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (matrix[mid][0] <= target && target <= matrix[mid][n - 1])
            return binarySearch(matrix[mid], target);
        else if (matrix[mid][0] > target)
            hi = mid - 1;
        else
            lo = mid + 1;
    }
    return false;
}

boolean binarySearch(int[] row, int target) {
    int lo = 0, hi = row.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (row[mid] == target) return true;
        else if (row[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

---

## Comparison of Approaches

| Approach | Time | Space | Code Complexity |
|----------|------|-------|-----------------|
| Flatten (1 BS) | O(log(mn)) | O(1) | Simple |
| Two BS | O(log m + log n) | O(1) | Moderate |
| Staircase | O(m + n) | O(1) | Simple |

Note: `O(log m + log n) = O(log(mn))` — mathematically same!

---

## Edge Cases

| Case | Input | Result |
|------|-------|--------|
| Single element, found | `[[5]]`, target=5 | true |
| Single element, not found | `[[5]]`, target=3 | false |
| Single row | `[[1,3,5,7]]`, target=3 | true |
| Single column | `[[1],[3],[5]]`, target=5 | true |
| Target before all | `[[2,4],[6,8]]`, target=1 | false |
| Target after all | `[[2,4],[6,8]]`, target=9 | false |

---

## Quick Revision Questions

1. **How do you convert 1D index to 2D coordinates?**
2. **Why does this approach NOT work for row-column sorted matrices?**
3. **What property distinguishes a fully sorted matrix from a row-column sorted one?**
4. **Trace the 1-BS approach for target = 7 in the example matrix.**
5. **Prove that O(log m + log n) = O(log(mn)).**

---

[← Previous: Row-Column Sorted](02-row-col-sorted.md) | [Next: Kth Smallest in Matrix →](04-kth-smallest.md)
