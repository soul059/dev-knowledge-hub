# Chapter 5: Search in Sorted Matrix

[← Previous: Spiral Order](04-spiral-order.md) | [Back to README](../README.md) | [Next: Matrix Problems →](06-matrix-problems.md)

---

## Overview

Searching in a sorted matrix efficiently is a classic interview problem. The approach depends on the type of sorting: row-sorted, fully sorted (row-major), or sorted rows and columns. Each variant has a different optimal strategy.

---

## Type 1: Each Row Sorted Independently

```
  ┌────┬────┬────┬────┐
  │  2 │  5 │  8 │ 12 │   Each row sorted left to right
  ├────┼────┼────┼────┤   But NO guarantee between rows
  │  1 │  7 │ 11 │ 15 │
  ├────┼────┼────┼────┤
  │  3 │  6 │  9 │ 16 │
  └────┴────┴────┴────┘

  Approach: Binary search on EACH row
  Time: O(m × log n)
```

---

## Type 2: Fully Sorted (Row-Major Order) — LeetCode 74

```
  ┌────┬────┬────┬────┐
  │  1 │  3 │  5 │  7 │   Row 0
  ├────┼────┼────┼────┤
  │ 10 │ 11 │ 16 │ 20 │   Row 1  (10 > 7)
  ├────┼────┼────┼────┤
  │ 23 │ 30 │ 34 │ 60 │   Row 2  (23 > 20)
  └────┴────┴────┴────┘

  First element of each row > last element of previous row.
  Flattened: 1, 3, 5, 7, 10, 11, 16, 20, 23, 30, 34, 60

  Treat as a single sorted array → Binary search!
  
  Index mapping:
  1D index k → row = k / cols, col = k % cols
  2D (i, j) → 1D = i × cols + j
```

### Algorithm

```
FUNCTION searchMatrix(matrix, target):
    m = rows, n = cols
    lo = 0, hi = m × n - 1

    WHILE lo ≤ hi:
        mid = lo + (hi - lo) / 2
        row = mid / n
        col = mid % n
        val = matrix[row][col]

        IF val == target:
            RETURN true
        ELSE IF val < target:
            lo = mid + 1
        ELSE:
            hi = mid - 1

    RETURN false
```

### Trace

```
  Search for 11 in:
  ┌───┬───┬───┬───┐
  │ 1 │ 3 │ 5 │ 7 │
  │10 │11 │16 │20 │
  │23 │30 │34 │60 │
  └───┴───┴───┴───┘
  m=3, n=4

  lo=0, hi=11
  mid=5 → (5/4, 5%4) = (1,1) → matrix[1][1] = 11 = target ✓

  Time: O(log(m × n))
  Space: O(1)
```

---

## Type 3: Rows and Columns Sorted (Staircase Search) — LeetCode 240

```
  ┌────┬────┬────┬────┐
  │  1 │  4 │  7 │ 11 │   Each row sorted left→right
  ├────┼────┼────┼────┤   Each column sorted top→bottom
  │  2 │  5 │  8 │ 12 │
  ├────┼────┼────┼────┤   But NOT fully sorted
  │  3 │  6 │  9 │ 16 │   (2 > 1 but appears below 1,
  ├────┼────┼────┼────┤    not after 11)
  │ 10 │ 13 │ 14 │ 17 │
  └────┴────┴────┴────┘
```

### Key Insight: Start from Top-Right (or Bottom-Left)

```
  Start at top-right corner (0, n-1):

  ┌────┬────┬────┬────┐
  │    │    │    │[11]│  ← Start here
  ├────┼────┼────┼────┤
  │    │    │    │    │  If target < current: move LEFT
  ├────┼────┼────┼────┤  If target > current: move DOWN
  │    │    │    │    │  If target = current: FOUND
  ├────┼────┼────┼────┤
  │    │    │    │    │
  └────┴────┴────┴────┘

  Why this works:
  - Moving LEFT → everything smaller (row is sorted)
  - Moving DOWN → everything larger (column is sorted)
  - Eliminates one row or one column at each step!
```

### Algorithm

```
FUNCTION searchSortedMatrix(matrix, target):
    m = rows, n = cols
    i = 0           // start at top
    j = n - 1       // start at right

    WHILE i < m AND j ≥ 0:
        IF matrix[i][j] == target:
            RETURN true
        ELSE IF matrix[i][j] > target:
            j--     // move left (eliminate column)
        ELSE:
            i++     // move down (eliminate row)

    RETURN false
```

### Trace

```
  Search for 9:
  ┌────┬────┬────┬────┐
  │  1 │  4 │  7 │[11]│  11 > 9 → move left
  ├────┼────┼────┼────┤
  │  2 │  5 │  8 │ 12 │
  ├────┼────┼────┼────┤
  │  3 │  6 │  9 │ 16 │
  ├────┼────┼────┼────┤
  │ 10 │ 13 │ 14 │ 17 │
  └────┴────┴────┴────┘

  (0,3)=11 > 9 → j=2
  (0,2)= 7 < 9 → i=1
  (1,2)= 8 < 9 → i=2
  (2,2)= 9 = 9 → FOUND! ✓

  Path visualized:
  ┌────┬────┬────┬────┐
  │    │    │  ← │[11]│  Step 1: Go left
  ├────┼────┼────┼────┤
  │    │    │[8] │    │  Step 3: Go down (from 7)
  ├────┼────┼────┼────┤          Step 4: At 8, go down
  │    │    │[9] │    │  Step 5: FOUND!
  ├────┼────┼────┼────┤
  │    │    │    │    │
  └────┴────┴────┴────┘

  Steps: 4 (at most m + n - 1 steps)
```

### Why Not Start from Top-Left?

```
  At top-left (0,0):
  - Right → larger
  - Down → larger
  Both directions go to larger values!
  Can't decide which way to eliminate.

  At top-right (0, n-1):
  - Left → smaller (row sorted)
  - Down → larger (col sorted)
  Opposite directions → can make decisions!

  Same logic applies for bottom-left:
  - Right → larger
  - Up → smaller
```

---

## Comparison of Approaches

```
  ┌──────────────┬──────────────┬──────────┬─────────────────┐
  │ Matrix Type  │ Algorithm    │ Time     │ Space           │
  ├──────────────┼──────────────┼──────────┼─────────────────┤
  │ Each row     │ Binary search│ O(m·logn)│ O(1)            │
  │ sorted       │ per row      │          │                 │
  ├──────────────┼──────────────┼──────────┼─────────────────┤
  │ Fully sorted │ Single binary│ O(log mn)│ O(1)            │
  │ (row-major)  │ search       │          │                 │
  ├──────────────┼──────────────┼──────────┼─────────────────┤
  │ Rows & cols  │ Staircase    │ O(m + n) │ O(1)            │
  │ sorted       │ search       │          │                 │
  ├──────────────┼──────────────┼──────────┼─────────────────┤
  │ Unsorted     │ Linear scan  │ O(m × n) │ O(1)            │
  └──────────────┴──────────────┴──────────┴─────────────────┘
```

---

## Count Negatives in Sorted Matrix (LeetCode 1351)

```
  Rows sorted descending, cols sorted descending.
  Count all negative numbers.

  ┌────┬────┬────┬────┐
  │  4 │  3 │  2 │ -1 │   1 negative
  ├────┼────┼────┼────┤
  │  3 │  2 │  1 │ -1 │   1 negative
  ├────┼────┼────┼────┤
  │  1 │  1 │ -1 │ -2 │   2 negatives
  ├────┼────┼────┼────┤
  │ -1 │ -1 │ -2 │ -3 │   4 negatives
  └────┴────┴────┴────┘

  Total: 8 negatives

  Staircase approach: start bottom-left
  Time: O(m + n)
```

---

## Kth Smallest in Sorted Matrix (LeetCode 378)

```
  Rows and columns sorted. Find kth smallest.

  Approach 1: Min-heap
  - Add first column to heap
  - Extract min k times, add right neighbor

  Approach 2: Binary search on value
  - lo = matrix[0][0], hi = matrix[m-1][n-1]
  - Count elements ≤ mid using staircase
  - Narrow range

  Time: O(n × log(max-min)) for binary search
        O(k × log n) for heap
```

---

## Summary Table

| Concept | Key Insight |
|---------|-------------|
| Fully sorted matrix | Treat as 1D sorted array, binary search O(log mn) |
| Row & col sorted | Staircase from top-right or bottom-left O(m+n) |
| Index mapping | k → (k/cols, k%cols) for flattened view |
| Why top-right? | Left = smaller, down = larger → can decide |
| Why not top-left? | Both directions larger → can't eliminate |
| Count negatives | Same staircase idea, count from boundary |

---

## Quick Revision Questions

1. **Search for 16** in [[1,3,5,7],[10,11,16,20],[23,30,34,60]] using 1D binary search. Show each step.

2. **Why does the staircase search work?** Explain the elimination principle.

3. **Search for 14** in a 4×4 row-col sorted matrix. Trace the staircase path.

4. **Can you use binary search** on a row-and-column sorted matrix? What's the time complexity?

5. **How would you find the kth smallest** element in a row-col sorted matrix?

6. **What's the maximum number of steps** in staircase search for an m×n matrix?

---

[← Previous: Spiral Order](04-spiral-order.md) | [Back to README](../README.md) | [Next: Matrix Problems →](06-matrix-problems.md)
