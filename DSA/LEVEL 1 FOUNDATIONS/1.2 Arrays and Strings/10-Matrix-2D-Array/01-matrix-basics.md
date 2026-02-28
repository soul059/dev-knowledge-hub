# Chapter 1: Matrix Basics

[← Previous: String Transformation](../09-String-Manipulation/06-string-transformation.md) | [Back to README](../README.md) | [Next: Matrix Traversal →](02-matrix-traversal.md)

---

## Overview

A matrix (2D array) is a rectangular grid of elements arranged in rows and columns. Matrices are fundamental in computer science for representing grids, images, graphs (adjacency matrices), game boards, and dynamic programming tables.

---

## What Is a Matrix?

```
  A matrix with m rows and n columns (m × n matrix):

  ┌─────────────────────────────┐
  │  matrix[0][0]  matrix[0][1]  matrix[0][2]  │   Row 0
  │  matrix[1][0]  matrix[1][1]  matrix[1][2]  │   Row 1
  │  matrix[2][0]  matrix[2][1]  matrix[2][2]  │   Row 2
  │  matrix[3][0]  matrix[3][1]  matrix[3][2]  │   Row 3
  └─────────────────────────────┘
       Col 0         Col 1         Col 2

  This is a 4 × 3 matrix (4 rows, 3 columns).
  Total elements: m × n = 12
```

---

## Memory Layout

### Row-Major Order (C, C++, Python, Java)

```
  Matrix:         Memory (1D):
  ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │   [1][2][3][4][5][6][7][8][9]
  ├───┼───┼───┤    ─────── ─────── ───────
  │ 4 │ 5 │ 6 │    Row 0   Row 1   Row 2
  ├───┼───┼───┤
  │ 7 │ 8 │ 9 │
  └───┴───┴───┘

  Address of matrix[i][j] = base + (i × numCols + j) × elementSize
  
  matrix[2][1] = base + (2 × 3 + 1) × size = base + 7 × size
  The element is '8' ✓
```

### Column-Major Order (Fortran, MATLAB, Julia)

```
  Matrix:         Memory (1D):
  ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │   [1][4][7][2][5][8][3][6][9]
  ├───┼───┼───┤    ─────── ─────── ───────
  │ 4 │ 5 │ 6 │    Col 0   Col 1   Col 2
  ├───┼───┼───┤
  │ 7 │ 8 │ 9 │
  └───┴───┴───┘

  Address of matrix[i][j] = base + (j × numRows + i) × elementSize
```

---

## Declaration Patterns

```
  // Static 2D array
  int matrix[3][4]          // 3 rows, 4 columns

  // Dynamic (array of arrays)
  int[][] matrix = new int[m][n]

  // Initialize with values
  matrix = [[1, 2, 3],
            [4, 5, 6],
            [7, 8, 9]]

  // Get dimensions
  rows = length(matrix)        // m
  cols = length(matrix[0])     // n
```

---

## Access Patterns

```
  Accessing matrix[i][j]:

  Step 1: Go to row i   → matrix[i]     → a 1D array
  Step 2: Go to col j   → matrix[i][j]  → the element

  Time: O(1) for random access

  Common bounds: 0 ≤ i < rows, 0 ≤ j < cols
```

---

## Neighbors in a Grid

```
  For cell (i, j), the 4-directional neighbors:

        (i-1, j)
           ↑
  (i, j-1) ← (i,j) → (i, j+1)
           ↓
        (i+1, j)

  Direction arrays:
  dx = [-1, 0, 1, 0]     // row changes
  dy = [0, 1, 0, -1]     // col changes

  For 8-directional (including diagonals):
  dx = [-1, -1, -1, 0, 0, 1, 1, 1]
  dy = [-1, 0, 1, -1, 1, -1, 0, 1]
```

### Boundary Check

```
FUNCTION isValid(i, j, rows, cols):
    RETURN i ≥ 0 AND i < rows AND j ≥ 0 AND j < cols
```

---

## Square Matrix Properties

```
  A square matrix (n × n):

  ┌───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │   Main diagonal: i == j
  ├───┼───┼───┼───┤   Anti-diagonal: i + j == n-1
  │ 5 │ 6 │ 7 │ 8 │
  ├───┼───┼───┼───┤   Upper triangle: i < j
  │ 9 │10 │11 │12 │   Lower triangle: i > j
  ├───┼───┼───┼───┤
  │13 │14 │15 │16 │
  └───┴───┴───┴───┘

  Main diagonal: (0,0), (1,1), (2,2), (3,3) → 1, 6, 11, 16
  Anti-diagonal: (0,3), (1,2), (2,1), (3,0) → 4, 7, 10, 13
```

---

## Transpose

```
  Transpose swaps rows and columns: matrix[i][j] ↔ matrix[j][i]

  Original:        Transposed:
  ┌───┬───┬───┐    ┌───┬───┐
  │ 1 │ 2 │ 3 │    │ 1 │ 4 │
  ├───┼───┼───┤    ├───┼───┤
  │ 4 │ 5 │ 6 │    │ 2 │ 5 │
  └───┴───┴───┘    ├───┼───┤
  (2 × 3)          │ 3 │ 6 │
                    └───┴───┘
                    (3 × 2)

  For square matrices, transpose can be done in-place:
  FOR i = 0 TO n-1:
      FOR j = i+1 TO n-1:
          SWAP(matrix[i][j], matrix[j][i])
```

---

## Common Matrix Types

```
  Identity Matrix (n × n):     Diagonal Matrix:
  ┌───┬───┬───┐                ┌───┬───┬───┐
  │ 1 │ 0 │ 0 │                │ 3 │ 0 │ 0 │
  ├───┼───┼───┤                ├───┼───┼───┤
  │ 0 │ 1 │ 0 │                │ 0 │ 5 │ 0 │
  ├───┼───┼───┤                ├───┼───┼───┤
  │ 0 │ 0 │ 1 │                │ 0 │ 0 │ 7 │
  └───┴───┴───┘                └───┴───┴───┘

  Symmetric: matrix[i][j] == matrix[j][i]
  Sparse: Most elements are zero (use special storage)
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Access element | O(1) | - |
| Set element | O(1) | - |
| Create m × n matrix | O(m × n) | O(m × n) |
| Transpose (in-place, square) | O(n²) | O(1) |
| Transpose (general m × n) | O(m × n) | O(m × n) |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Dimensions | m rows × n columns |
| Access | matrix[i][j] → O(1) |
| Row-major | Elements stored row by row |
| Column-major | Elements stored column by column |
| Address formula | base + (i × cols + j) × size |
| Neighbors | 4-directional: dx/dy arrays |
| Diagonal | Main: i == j, Anti: i+j == n-1 |
| Transpose | Swap matrix[i][j] ↔ matrix[j][i] |

---

## Quick Revision Questions

1. **What is the address** of matrix[3][2] in a 5×4 row-major matrix with base address 1000 and element size 4 bytes?

2. **How does column-major differ** from row-major order? Why does it matter for performance?

3. **List all 4-directional neighbors** of cell (2, 3) in a 5×6 matrix. Which are valid?

4. **Transpose the matrix** [[1,2,3],[4,5,6]] in-place. Is in-place possible for non-square matrices?

5. **What condition defines** elements on the anti-diagonal of an n × n matrix?

6. **Why are direction arrays** (dx, dy) useful? How do you extend them to 8 directions?

---

[← Previous: String Transformation](../09-String-Manipulation/06-string-transformation.md) | [Back to README](../README.md) | [Next: Matrix Traversal →](02-matrix-traversal.md)
