# Chapter 4: Spiral Order

[← Previous: Matrix Rotation](03-matrix-rotation.md) | [Back to README](../README.md) | [Next: Search in Sorted Matrix →](05-search-sorted-matrix.md)

---

## Overview

Spiral order traversal visits all elements of a matrix in a clockwise spiral from outside to inside. This is a popular interview problem (LeetCode 54, 59) that tests careful boundary management.

---

## Spiral Matrix (LeetCode 54)

```
  ┌───┬───┬───┬───┐
  │ 1 → 2 → 3 → 4 │  ──→ Right along top
  ├───┼───┼───┼───┤
  │12 │13 →14 │ 5 │      ↓ Down along right
  ├───┼───┼───┼───┤       │
  │11 │16 ←15 │ 6 │      ↓
  ├───┼───┼───┼───┤
  │10 ← 9 ← 8 ← 7 │  ←── Left along bottom
  └───┴───┴───┴───┘

  Output: 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16

  Pattern: Right → Down → Left → Up → Right → Down → ...
  Shrink boundaries after each edge.
```

---

## Algorithm

```
FUNCTION spiralOrder(matrix):
    result = []
    top = 0, bottom = rows - 1
    left = 0, right = cols - 1

    WHILE top ≤ bottom AND left ≤ right:

        // Traverse Right (along top row)
        FOR j = left TO right:
            result.append(matrix[top][j])
        top++

        // Traverse Down (along right column)
        FOR i = top TO bottom:
            result.append(matrix[i][right])
        right--

        // Traverse Left (along bottom row)
        IF top ≤ bottom:
            FOR j = right DOWNTO left:
                result.append(matrix[bottom][j])
            bottom--

        // Traverse Up (along left column)
        IF left ≤ right:
            FOR i = bottom DOWNTO top:
                result.append(matrix[i][left])
            left++

    RETURN result
```

---

## Detailed Trace

```
  Matrix 3×4:
  ┌───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │
  ├───┼───┼───┼───┤
  │ 5 │ 6 │ 7 │ 8 │
  ├───┼───┼───┼───┤
  │ 9 │10 │11 │12 │
  └───┴───┴───┴───┘

  Initial: top=0, bottom=2, left=0, right=3

  Iteration 1:
    Right: [1, 2, 3, 4]           top→1
    Down:  [8, 12]                right→2
    Left:  [11, 10, 9]            bottom→1
    Up:    [5]                    left→1

  State: top=1, bottom=1, left=1, right=2

  Iteration 2:
    Right: [6, 7]                 top→2
    Down:  (top > bottom, skip)
    Left:  (skipped)
    Up:    (skipped)

  Result: [1,2,3,4,8,12,11,10,9,5,6,7]

  ┌───┬───┬───┬───┐
  │ 1 → 2 → 3 → 4 │  Step 1: Right
  ├───┼───┼───┤ ↓ ┤
  │ ↑ │ 6 → 7 │ 8 │  Step 2: Down (8, 12)
  ├───┼───┼───┤ ↓ ┤
  │ ↑ │   │   │12 │
  ├ ↑─┼───┼───┼───┤
  │ 9 ←10 ←11 │   │  Step 3: Left (11, 10, 9)
  └───┴───┴───┴───┘  Step 4: Up (5)
                      Step 5: Right (6, 7)
```

---

## Edge Cases

```
  Single Row (1×4):         Single Column (4×1):
  ┌───┬───┬───┬───┐        ┌───┐
  │ 1 │ 2 │ 3 │ 4 │        │ 1 │
  └───┴───┴───┴───┘        ├───┤
  Output: [1,2,3,4]        │ 2 │
  Only "Right" step.        ├───┤
                            │ 3 │
                            ├───┤
  Single Element (1×1):     │ 4 │
  ┌───┐                    └───┘
  │ 1 │                    Output: [1,2,3,4]
  └───┘                    Only "Down" step after first "Right".
  Output: [1]

  Why we need the IF checks:
  - "IF top ≤ bottom" before Left traversal
  - "IF left ≤ right" before Up traversal
  - Without these, single row/column gets double-counted!
```

---

## Generate Spiral Matrix (LeetCode 59)

```
  Given n, generate an n×n matrix filled in spiral order.

  n = 3:
  ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │
  ├───┼───┼───┤
  │ 8 │ 9 │ 4 │
  ├───┼───┼───┤
  │ 7 │ 6 │ 5 │
  └───┴───┴───┘
```

### Algorithm

```
FUNCTION generateMatrix(n):
    matrix = n × n matrix filled with 0
    num = 1
    top = 0, bottom = n-1
    left = 0, right = n-1

    WHILE num ≤ n × n:

        // Right
        FOR j = left TO right:
            matrix[top][j] = num++
        top++

        // Down
        FOR i = top TO bottom:
            matrix[i][right] = num++
        right--

        // Left
        FOR j = right DOWNTO left:
            matrix[bottom][j] = num++
        bottom--

        // Up
        FOR i = bottom DOWNTO top:
            matrix[i][left] = num++
        left++

    RETURN matrix
```

### Trace (n = 3)

```
  Start: num=1, top=0, bot=2, left=0, right=2

  Right: matrix[0][0..2] = 1,2,3    top→1
  Down:  matrix[1..2][2] = 4,5      right→1
  Left:  matrix[2][1..0] = 6,7      bot→1
  Up:    matrix[1][0]    = 8        left→1

  Right: matrix[1][1]    = 9        top→2

  ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │
  │ 8 │ 9 │ 4 │
  │ 7 │ 6 │ 5 │
  └───┴───┴───┘ ✓
```

---

## Spiral Matrix III (LeetCode 885)

```
  Start from (rStart, cStart), walk in spiral order outward.
  Visit ALL cells in an R×C grid.

  Unlike the standard spiral:
  - Start from any position
  - Expand outward
  - Steps increase: 1,1,2,2,3,3,4,4,...

  Direction: East → South → West → North → East → ...

  Step pattern:
  1 step East, 1 step South,
  2 steps West, 2 steps North,
  3 steps East, 3 steps South,
  4 steps West, 4 steps North, ...
```

---

## Direction Array Approach (Alternative)

```
  Instead of explicit boundaries, use direction cycling:

  dirs = [(0,1), (1,0), (0,-1), (-1,0)]  // R, D, L, U
  
  Steps per direction for n=3:
  R:3, D:2, L:2, U:1, R:1  (decreasing pattern)

  For n=4:
  R:4, D:3, L:3, U:2, R:2, D:1, L:1

  After the first direction, steps decrease:
  n, n-1, n-1, n-2, n-2, ...
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Spiral traversal | O(m × n) | O(1) extra |
| Generate spiral | O(n²) | O(n²) output |
| Spiral from point | O(R × C) | O(R × C) output |

---

## Summary Table

| Concept | Key Insight |
|---------|-------------|
| Traversal order | Right → Down → Left → Up, repeat |
| Boundary variables | top, bottom, left, right |
| After each edge | Shrink the corresponding boundary |
| Edge case: single row | Need IF check before Left traversal |
| Edge case: single col | Need IF check before Up traversal |
| Generate spiral | Same logic, but write num++ instead of read |
| Total elements | Must visit exactly m × n elements |

---

## Quick Revision Questions

1. **Trace spiral order** for a 2×3 matrix [[1,2,3],[4,5,6]].

2. **Why are the IF checks** (top ≤ bottom, left ≤ right) necessary? Give a case that fails without them.

3. **Generate a 4×4 spiral matrix.** Show the result.

4. **How would you do counter-clockwise** spiral traversal? What changes?

5. **What happens** if the matrix is 5×1? Walk through the algorithm step by step.

6. **How many times** does each boundary variable change for an m×n matrix?

---

[← Previous: Matrix Rotation](03-matrix-rotation.md) | [Back to README](../README.md) | [Next: Search in Sorted Matrix →](05-search-sorted-matrix.md)
