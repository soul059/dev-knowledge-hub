# Chapter 3: Matrix Rotation

[← Previous: Matrix Traversal](02-matrix-traversal.md) | [Back to README](../README.md) | [Next: Spiral Order →](04-spiral-order.md)

---

## Overview

Matrix rotation is a classic interview problem (LeetCode 48). The key insight is that rotation can be decomposed into simpler operations: transpose + reverse. This avoids complex index math and makes the solution elegant and memorable.

---

## Rotate 90° Clockwise (LeetCode 48)

```
  Original:          Rotated 90° CW:
  ┌───┬───┬───┐      ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │      │ 7 │ 4 │ 1 │
  ├───┼───┼───┤      ├───┼───┼───┤
  │ 4 │ 5 │ 6 │  →   │ 8 │ 5 │ 2 │
  ├───┼───┼───┤      ├───┼───┼───┤
  │ 7 │ 8 │ 9 │      │ 9 │ 6 │ 3 │
  └───┴───┴───┘      └───┴───┴───┘

  Formula: rotated[j][n-1-i] = original[i][j]
```

### Method: Transpose + Reverse Each Row

```
  Step 1: Transpose (swap across main diagonal)
  ┌───┬───┬───┐      ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │      │ 1 │ 4 │ 7 │
  │ 4 │ 5 │ 6 │  →   │ 2 │ 5 │ 8 │
  │ 7 │ 8 │ 9 │      │ 3 │ 6 │ 9 │
  └───┴───┴───┘      └───┴───┴───┘

  Step 2: Reverse each row
  ┌───┬───┬───┐      ┌───┬───┬───┐
  │ 1 │ 4 │ 7 │      │ 7 │ 4 │ 1 │
  │ 2 │ 5 │ 8 │  →   │ 8 │ 5 │ 2 │
  │ 3 │ 6 │ 9 │      │ 9 │ 6 │ 3 │
  └───┴───┴───┘      └───┴───┴───┘
```

### Algorithm

```
FUNCTION rotate90CW(matrix):
    n = length(matrix)

    // Step 1: Transpose
    FOR i = 0 TO n-1:
        FOR j = i+1 TO n-1:
            SWAP(matrix[i][j], matrix[j][i])

    // Step 2: Reverse each row
    FOR i = 0 TO n-1:
        left = 0, right = n-1
        WHILE left < right:
            SWAP(matrix[i][left], matrix[i][right])
            left++, right--
```

---

## Rotate 90° Counter-Clockwise

```
  Original:          Rotated 90° CCW:
  ┌───┬───┬───┐      ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │      │ 3 │ 6 │ 9 │
  ├───┼───┼───┤      ├───┼───┼───┤
  │ 4 │ 5 │ 6 │  →   │ 2 │ 5 │ 8 │
  ├───┼───┼───┤      ├───┼───┼───┤
  │ 7 │ 8 │ 9 │      │ 1 │ 4 │ 7 │
  └───┴───┴───┘      └───┴───┴───┘

  Method: Transpose + Reverse each COLUMN
  OR: Reverse each row + Transpose
```

### Algorithm

```
FUNCTION rotate90CCW(matrix):
    n = length(matrix)

    // Step 1: Transpose
    FOR i = 0 TO n-1:
        FOR j = i+1 TO n-1:
            SWAP(matrix[i][j], matrix[j][i])

    // Step 2: Reverse each column
    FOR j = 0 TO n-1:
        top = 0, bottom = n-1
        WHILE top < bottom:
            SWAP(matrix[top][j], matrix[bottom][j])
            top++, bottom--
```

---

## Rotate 180°

```
  Original:          Rotated 180°:
  ┌───┬───┬───┐      ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │      │ 9 │ 8 │ 7 │
  ├───┼───┼───┤      ├───┼───┼───┤
  │ 4 │ 5 │ 6 │  →   │ 6 │ 5 │ 4 │
  ├───┼───┼───┤      ├───┼───┼───┤
  │ 7 │ 8 │ 9 │      │ 3 │ 2 │ 1 │
  └───┴───┴───┘      └───┴───┴───┘

  Method: Reverse each row + Reverse each column
  OR: Apply 90° rotation twice
```

### Algorithm

```
FUNCTION rotate180(matrix):
    n = length(matrix)

    // Reverse each row
    FOR i = 0 TO n-1:
        reverse(matrix[i])

    // Reverse row order (swap top and bottom rows)
    top = 0, bottom = n-1
    WHILE top < bottom:
        SWAP(matrix[top], matrix[bottom])
        top++, bottom--
```

---

## Rotation Cheat Sheet

```
  ┌─────────────────┬─────────────────────────────────┐
  │ Rotation        │ Method                          │
  ├─────────────────┼─────────────────────────────────┤
  │ 90° CW          │ Transpose → Reverse rows        │
  │ 90° CCW         │ Transpose → Reverse columns     │
  │                 │ (OR Reverse rows → Transpose)    │
  │ 180°            │ Reverse rows → Reverse row order │
  │                 │ (OR 90° CW twice)               │
  │ 270° CW = 90°CCW│ Same as 90° CCW                │
  └─────────────────┴─────────────────────────────────┘
```

---

## Layer-by-Layer Rotation (Alternative Approach)

```
  Rotate groups of 4 elements at a time per layer:

  For a 4×4 matrix, layer 0:

  ┌───┬───┬───┬───┐
  │ A │ . │ . │ B │    A → B position
  ├───┼───┼───┼───┤    B → D position
  │ . │   │   │ . │    D → C position
  ├───┼───┼───┼───┤    C → A position
  │ . │   │   │ . │
  ├───┼───┼───┼───┤    4-way swap in a cycle
  │ C │ . │ . │ D │
  └───┴───┴───┴───┘

FUNCTION rotateLayerByLayer(matrix):
    n = length(matrix)
    FOR layer = 0 TO n/2 - 1:
        first = layer
        last = n - 1 - layer
        FOR i = first TO last - 1:
            offset = i - first
            // Save top
            top = matrix[first][i]
            // Left → Top
            matrix[first][i] = matrix[last-offset][first]
            // Bottom → Left
            matrix[last-offset][first] = matrix[last][last-offset]
            // Right → Bottom
            matrix[last][last-offset] = matrix[i][last]
            // Top → Right
            matrix[i][last] = top
```

### Trace (4×4 matrix, layer 0, first element)

```
  ┌───┬───┬───┬───┐      Cycle for (0,0):
  │ 1 │ . │ . │ 4 │      top = matrix[0][0] = 1
  ├───┼───┼───┼───┤      matrix[0][0] = matrix[3][0] = 13
  │ . │   │   │ . │      matrix[3][0] = matrix[3][3] = 16
  ├───┼───┼───┼───┤      matrix[3][3] = matrix[0][3] = 4
  │ . │   │   │ . │      matrix[0][3] = top = 1
  ├───┼───┼───┼───┤
  │13 │ . │ . │16 │      Result: 1→4, 4→16, 16→13, 13→1
  └───┴───┴───┴───┘
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Rotate 90° (transpose+reverse) | O(n²) | O(1) in-place |
| Rotate 180° | O(n²) | O(1) in-place |
| Layer-by-layer rotation | O(n²) | O(1) in-place |
| Rotation with new matrix | O(n²) | O(n²) |

---

## Summary Table

| Concept | Key Insight |
|---------|-------------|
| 90° CW | Transpose + reverse each row |
| 90° CCW | Transpose + reverse each column |
| 180° | Reverse rows + reverse row order |
| Layer-by-layer | 4-way cyclic swap per layer |
| In-place | All rotations achievable in O(1) space |
| Transpose | Swap matrix[i][j] ↔ matrix[j][i] |
| Only square | In-place rotation only works for n×n |

---

## Quick Revision Questions

1. **Rotate the matrix** [[1,2],[3,4]] by 90° clockwise using transpose + reverse.

2. **What is the difference** between 90° CW and 90° CCW in terms of operations?

3. **Trace the layer-by-layer approach** on a 3×3 matrix with values 1-9.

4. **Can you rotate a non-square matrix** (3×4) in-place? Why or why not?

5. **How many 4-way swaps** are needed to rotate one layer of an n×n matrix?

6. **Prove that** transpose + reverse rows = 90° clockwise rotation using index mapping.

---

[← Previous: Matrix Traversal](02-matrix-traversal.md) | [Back to README](../README.md) | [Next: Spiral Order →](04-spiral-order.md)
