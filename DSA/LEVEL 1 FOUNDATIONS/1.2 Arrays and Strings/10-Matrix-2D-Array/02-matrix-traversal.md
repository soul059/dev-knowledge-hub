# Chapter 2: Matrix Traversal

[← Previous: Matrix Basics](01-matrix-basics.md) | [Back to README](../README.md) | [Next: Matrix Rotation →](03-matrix-rotation.md)

---

## Overview

Matrix traversal means visiting every element in a specific order. Different traversal patterns are required for different problems: row-wise, column-wise, diagonal, boundary, and snake/zigzag patterns.

---

## 1. Row-Wise Traversal

```
  ┌───┬───┬───┬───┐
  │ 1 → 2 → 3 → 4 │   Row 0: left to right
  ├───┼───┼───┼───┤
  │ 5 → 6 → 7 → 8 │   Row 1: left to right
  ├───┼───┼───┼───┤
  │ 9 →10 →11 →12 │   Row 2: left to right
  └───┴───┴───┴───┘

  Output: 1 2 3 4 5 6 7 8 9 10 11 12

  FOR i = 0 TO rows-1:
      FOR j = 0 TO cols-1:
          visit(matrix[i][j])
```

---

## 2. Column-Wise Traversal

```
  ┌───┬───┬───┬───┐
  │ 1 │ 4 │ 7 │10 │
  │ ↓ │ ↓ │ ↓ │ ↓ │
  │ 2 │ 5 │ 8 │11 │
  │ ↓ │ ↓ │ ↓ │ ↓ │
  │ 3 │ 6 │ 9 │12 │
  └───┴───┴───┴───┘

  Output: 1 2 3 4 5 6 7 8 9 10 11 12
  (Reading down each column)

  FOR j = 0 TO cols-1:
      FOR i = 0 TO rows-1:
          visit(matrix[i][j])
```

---

## 3. Snake/Zigzag Traversal

```
  ┌───┬───┬───┬───┐
  │ 1 → 2 → 3 → 4 │   Row 0: left → right
  ├───┼───┼───┼───┤
  │ 8 ← 7 ← 6 ← 5 │   Row 1: right → left
  ├───┼───┼───┼───┤
  │ 9 →10 →11 →12 │   Row 2: left → right
  └───┴───┴───┴───┘

  Output: 1 2 3 4 8 7 6 5 9 10 11 12

  FOR i = 0 TO rows-1:
      IF i is even:
          FOR j = 0 TO cols-1:
              visit(matrix[i][j])
      ELSE:
          FOR j = cols-1 DOWNTO 0:
              visit(matrix[i][j])
```

---

## 4. Diagonal Traversal

### Primary Diagonals (Top-Left to Bottom-Right)

```
  ┌───┬───┬───┬───┐
  │ a │ b │ c │ d │   Diagonals (by i - j):
  ├───┼───┼───┼───┤   d=0:  a, f, k
  │ e │ f │ g │ h │   d=1:  b, g, l
  ├───┼───┼───┼───┤   d=2:  c, h
  │ i │ j │ k │ l │   d=3:  d
  └───┴───┴───┴───┘   d=-1: e, j
                       d=-2: i

  Elements on same diagonal share same (i - j) value.
```

### Anti-Diagonals (Top-Right to Bottom-Left)

```
  ┌───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │   Anti-diagonals (by i + j):
  ├───┼───┼───┼───┤   s=0: 1
  │ 5 │ 6 │ 7 │ 8 │   s=1: 2, 5
  ├───┼───┼───┼───┤   s=2: 3, 6, 9
  │ 9 │10 │11 │12 │   s=3: 4, 7, 10
  └───┴───┴───┴───┘   s=4: 8, 11
                       s=5: 12

  Elements on same anti-diagonal share same (i + j) value.
```

### Diagonal Traversal Algorithm (LeetCode 498)

```
  Traverse in alternating diagonal order:

  ┌───┬───┬───┐        Output: 1, 2, 4, 7, 5, 3, 6, 8, 9
  │ 1 │ 2 │ 3 │        
  ├───┼───┼───┤        Diagonal 0 (sum=0): [1] ↗
  │ 4 │ 5 │ 6 │        Diagonal 1 (sum=1): [2,4] ↙
  ├───┼───┼───┤        Diagonal 2 (sum=2): [7,5,3] ↗
  │ 7 │ 8 │ 9 │        Diagonal 3 (sum=3): [6,8] ↙
  └───┴───┴───┘        Diagonal 4 (sum=4): [9] ↗

FUNCTION diagonalTraverse(matrix):
    m = rows, n = cols
    result = []

    FOR d = 0 TO m + n - 2:
        IF d is even:  // go up-right
            r = min(d, m-1)
            c = d - r
            WHILE r ≥ 0 AND c < n:
                result.append(matrix[r][c])
                r--, c++
        ELSE:          // go down-left
            c = min(d, n-1)
            r = d - c
            WHILE c ≥ 0 AND r < m:
                result.append(matrix[r][c])
                r++, c--

    RETURN result
```

---

## 5. Boundary Traversal

```
  ┌───┬───┬───┬───┬───┐
  │ 1 → 2 → 3 → 4 → 5 │   Top: left to right
  ├───┼───┼───┼───┼───┤
  │16 │           │ 6 │   Right: top to bottom
  ├───┤           ├───┤         (excluding corners)
  │15 │           │ 7 │
  ├───┤           ├───┤
  │14 │           │ 8 │
  ├───┼───┼───┼───┼───┤
  │13 ←12 ←11 ←10 ← 9 │   Bottom: right to left
  └───┴───┴───┴───┴───┘   Left: bottom to top
                                 (excluding corners)

  Output: 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16

FUNCTION boundaryTraversal(matrix):
    m = rows, n = cols
    result = []

    // Top row
    FOR j = 0 TO n-1:
        result.append(matrix[0][j])

    // Right column (skip top corner)
    FOR i = 1 TO m-1:
        result.append(matrix[i][n-1])

    // Bottom row (skip right corner, if different from top)
    IF m > 1:
        FOR j = n-2 DOWNTO 0:
            result.append(matrix[m-1][j])

    // Left column (skip both corners, if different from right)
    IF n > 1:
        FOR i = m-2 DOWNTO 1:
            result.append(matrix[i][0])

    RETURN result
```

---

## 6. Layer-by-Layer (Onion Peeling)

```
  Process matrix from outer layer inward:

  Layer 0:                Layer 1:
  ┌───┬───┬───┬───┐      
  │ * │ * │ * │ * │      
  ├───┼───┼───┼───┤      ┌───┬───┐
  │ * │   │   │ * │      │ * │ * │
  ├───┼───┼───┼───┤      └───┴───┘
  │ * │   │   │ * │
  ├───┼───┼───┼───┤
  │ * │ * │ * │ * │
  └───┴───┴───┴───┘

  Number of layers = ceil(min(m, n) / 2)

  For a 4×4 matrix: 2 layers
  For a 5×5 matrix: 3 layers (center is a single element)

  Layer k boundaries:
  - Top row: i = k
  - Bottom row: i = m - 1 - k
  - Left col: j = k
  - Right col: j = n - 1 - k
```

---

## Performance: Row vs Column Traversal

```
  Due to cache locality (row-major storage):

  ROW-WISE (cache-friendly):     COLUMN-WISE (cache-unfriendly):
  Memory: [1][2][3][4][5][6]     Memory: [1][2][3][4][5][6]
  Access: [1][2][3] → cache hit  Access: [1]...[4]...[7] → cache miss
          [4][5][6] → cache hit          [2]...[5]...[8] → cache miss

  For large matrices:
  Row-wise traversal can be 5-10× faster than column-wise
  due to CPU cache line optimization.

  ┌────────────────────┬─────────────────┐
  │ Traversal          │ Cache Behavior   │
  ├────────────────────┼─────────────────┤
  │ Row-wise           │ Excellent        │
  │ Column-wise        │ Poor             │
  │ Diagonal           │ Moderate         │
  │ Random access      │ Poor             │
  └────────────────────┴─────────────────┘
```

---

## Complexity

| Traversal | Time | Space |
|-----------|------|-------|
| Row/Column/Snake | O(m × n) | O(1) extra |
| Diagonal | O(m × n) | O(min(m,n)) buffer |
| Boundary | O(m + n) | O(1) extra |
| All elements | O(m × n) | Depends on output |

---

## Summary Table

| Pattern | Key Idea |
|---------|----------|
| Row-wise | Outer loop rows, inner loop cols |
| Column-wise | Outer loop cols, inner loop rows |
| Snake | Alternate direction on even/odd rows |
| Diagonal | Group by i+j (anti) or i-j (primary) |
| Boundary | Four edges: top→right→bottom→left |
| Layer-by-layer | Shrink boundaries inward |
| Cache | Row-wise is cache-friendly in row-major |

---

## Quick Revision Questions

1. **Write the snake traversal** for a 3×4 matrix. Show the output order.

2. **How do you identify** which anti-diagonal a cell (i,j) belongs to?

3. **Trace the diagonal traversal** (LeetCode 498) for a 3×3 matrix.

4. **Write boundary traversal** for a 1×5 matrix. What edge cases arise?

5. **Why is row-wise traversal** faster than column-wise for large matrices?

6. **How many layers** does a 6×3 matrix have? What are the boundary indices for each layer?

---

[← Previous: Matrix Basics](01-matrix-basics.md) | [Back to README](../README.md) | [Next: Matrix Rotation →](03-matrix-rotation.md)
