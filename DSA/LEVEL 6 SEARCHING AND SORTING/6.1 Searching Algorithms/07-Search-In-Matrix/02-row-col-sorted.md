# Chapter 2: Search in Row-Column Sorted Matrix (Staircase Search)

[← Previous: Sorted Matrix Types](01-sorted-matrix-types.md) | [Next: Fully Sorted Matrix →](03-fully-sorted-matrix.md)

---

## Overview

The **staircase search** (also called **step-wise search**) is an elegant O(m + n) algorithm for searching in a matrix where each row and each column is sorted.

---

## Key Insight: Start from a Corner

```
   ┌────┬────┬────┬────┐
   │  1 │  4 │  7 │ 11 │  ← Start from top-right: 11
   ├────┼────┼────┼────┤     
   │  2 │  5 │  8 │ 12 │     If target < 11 → go LEFT (eliminate column)
   ├────┼────┼────┼────┤     If target > 11 → go DOWN (eliminate row)
   │  3 │  6 │  9 │ 16 │
   ├────┼────┼────┼────┤
   │ 10 │ 13 │ 14 │ 17 │
   └────┴────┴────┴────┘
   
   Top-right corner is special:
   - It's the LARGEST in its row → go LEFT to find smaller
   - It's the SMALLEST in its column → go DOWN to find larger
   
   Each step eliminates one entire row OR column!
```

---

## Two Valid Starting Corners

```
   ┌──────────────────────────────────────────────────┐
   │  Top-Right Corner:                               │
   │    target < value → move LEFT                    │
   │    target > value → move DOWN                    │
   │                                                  │
   │  Bottom-Left Corner:                             │
   │    target < value → move UP                      │
   │    target > value → move RIGHT                   │
   │                                                  │
   │  Top-Left: both directions increase → BAD        │
   │  Bottom-Right: both directions decrease → BAD    │
   └──────────────────────────────────────────────────┘
```

---

## Walkthrough: Top-Right Start

```
   Matrix:                         Target = 6
   ┌────┬────┬────┬────┐
   │  1 │  4 │  7 │ 11 │
   ├────┼────┼────┼────┤
   │  2 │  5 │  8 │ 12 │
   ├────┼────┼────┼────┤
   │  3 │  6 │  9 │ 16 │
   ├────┼────┼────┼────┤
   │ 10 │ 13 │ 14 │ 17 │
   └────┴────┴────┴────┘
   
   Step 1: (0,3) = 11 > 6 → LEFT
   Step 2: (0,2) = 7  > 6 → LEFT
   Step 3: (0,1) = 4  < 6 → DOWN
   Step 4: (1,1) = 5  < 6 → DOWN
   Step 5: (2,1) = 6  = 6 → FOUND at (2,1) ✓
   
   Path visualization:
   ┌────┬────┬────┬────┐
   │    │    │ ←← │ ★  │  ★ = start
   ├────┼────┼────┼────┤
   │    │ ↓  │    │    │
   ├────┼────┼────┼────┤
   │    │ ●  │    │    │  ● = found!
   ├────┼────┼────┼────┤
   │    │    │    │    │
   └────┴────┴────┴────┘
```

---

## Pseudocode

```
   STAIRCASE-SEARCH(matrix, target):
       row = 0
       col = n - 1          // Start top-right
       
       while row < m AND col >= 0:
           if matrix[row][col] == target:
               return (row, col)
           else if matrix[row][col] > target:
               col--        // Eliminate this column
           else:
               row++        // Eliminate this row
       
       return NOT_FOUND
```

---

## Implementation (Java)

```java
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int row = 0, col = n - 1;   // top-right
    
    while (row < m && col >= 0) {
        if (matrix[row][col] == target) {
            return true;
        } else if (matrix[row][col] > target) {
            col--;
        } else {
            row++;
        }
    }
    return false;
}
```

---

## Implementation (Python)

```python
def searchMatrix(matrix, target):
    if not matrix:
        return False
    m, n = len(matrix), len(matrix[0])
    row, col = 0, n - 1    # top-right
    
    while row < m and col >= 0:
        if matrix[row][col] == target:
            return True
        elif matrix[row][col] > target:
            col -= 1
        else:
            row += 1
    return False
```

---

## Implementation (C++)

```cpp
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
    int row = 0, col = n - 1;
    
    while (row < m && col >= 0) {
        if (matrix[row][col] == target)
            return true;
        else if (matrix[row][col] > target)
            col--;
        else
            row++;
    }
    return false;
}
```

---

## Bottom-Left Alternative

```java
// Starting from bottom-left: (m-1, 0)
public boolean searchMatrix(int[][] matrix, int target) {
    int row = matrix.length - 1, col = 0;
    
    while (row >= 0 && col < matrix[0].length) {
        if (matrix[row][col] == target)
            return true;
        else if (matrix[row][col] > target)
            row--;      // go UP
        else
            col++;      // go RIGHT
    }
    return false;
}
```

---

## Why O(m + n)?

```
   Each step either:
   - Decreases col by 1 (at most n times), OR
   - Increases row by 1 (at most m times)
   
   Total steps ≤ m + n
   
   For a 4×4 matrix:
   Worst case: 4 + 4 = 8 comparisons
   vs brute force: 16 comparisons
   vs binary search per row: 4 × log(4) = 8 comparisons (same!)
   
   For 1000×1000:
   Staircase: 2000 comparisons
   Brute force: 1,000,000 comparisons  
   BS per row: 1000 × 10 ≈ 10,000 comparisons
```

---

## Counting Elements ≤ Target

A useful extension: count how many elements are ≤ target.

```python
def countLessEqual(matrix, target):
    n = len(matrix)
    count = 0
    row, col = n - 1, 0    # bottom-left
    
    while row >= 0 and col < n:
        if matrix[row][col] <= target:
            count += row + 1    # entire column above is ≤ target
            col += 1
        else:
            row -= 1
    return count
```

```
   Used in: Kth smallest element in sorted matrix (BS on answer)
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(m + n) |
| Space | O(1) |
| Best case | O(1) — target is at the starting corner |
| Worst case | O(m + n) — traverse full diagonal path |

---

## Quick Revision Questions

1. **Why must we start from top-right or bottom-left?**
2. **Why can't we start from top-left?**
3. **What does each step eliminate (row or column)?**
4. **Why is the time complexity O(m + n)?**
5. **Trace the search for target = 14 in the example matrix.**
6. **How would you modify the algorithm to find the index, not just boolean?**

---

[← Previous: Sorted Matrix Types](01-sorted-matrix-types.md) | [Next: Fully Sorted Matrix →](03-fully-sorted-matrix.md)
