# Chapter 6: Matrix Problems

[← Previous: Search in Sorted Matrix](05-search-sorted-matrix.md) | [Back to README](../README.md) | [Next: Back to Start →](../README.md)

---

## Overview

This chapter covers classic matrix problems frequently asked in interviews: Set Matrix Zeroes, Word Search, Island Counting, and Game of Life. Each tests different skills — in-place marking, DFS/BFS on grids, and simulation.

---

## 1. Set Matrix Zeroes (LeetCode 73)

```
  If an element is 0, set its entire row and column to 0.

  Input:              Output:
  ┌───┬───┬───┐      ┌───┬───┬───┐
  │ 1 │ 1 │ 1 │      │ 1 │ 0 │ 1 │
  ├───┼───┼───┤      ├───┼───┼───┤
  │ 1 │ 0 │ 1 │  →   │ 0 │ 0 │ 0 │
  ├───┼───┼───┤      ├───┼───┼───┤
  │ 1 │ 1 │ 1 │      │ 1 │ 0 │ 1 │
  └───┴───┴───┘      └───┴───┴───┘

  Challenge: Can't zero as you go (would cascade incorrectly).
```

### Approach 1: Extra Space O(m + n)

```
FUNCTION setZeroes_v1(matrix):
    zeroRows = set()
    zeroCols = set()

    // Pass 1: Find all zero positions
    FOR i = 0 TO m-1:
        FOR j = 0 TO n-1:
            IF matrix[i][j] == 0:
                zeroRows.add(i)
                zeroCols.add(j)

    // Pass 2: Zero out marked rows and columns
    FOR i = 0 TO m-1:
        FOR j = 0 TO n-1:
            IF i in zeroRows OR j in zeroCols:
                matrix[i][j] = 0
```

### Approach 2: O(1) Space — Use First Row/Column as Markers

```
FUNCTION setZeroes_v2(matrix):
    m = rows, n = cols
    firstRowZero = false
    firstColZero = false

    // Check if first row/column have zeros
    FOR j = 0 TO n-1:
        IF matrix[0][j] == 0: firstRowZero = true
    FOR i = 0 TO m-1:
        IF matrix[i][0] == 0: firstColZero = true

    // Use first row/col as markers for rest
    FOR i = 1 TO m-1:
        FOR j = 1 TO n-1:
            IF matrix[i][j] == 0:
                matrix[i][0] = 0    // mark row
                matrix[0][j] = 0    // mark col

    // Zero out marked rows and columns (skip first row/col)
    FOR i = 1 TO m-1:
        FOR j = 1 TO n-1:
            IF matrix[i][0] == 0 OR matrix[0][j] == 0:
                matrix[i][j] = 0

    // Handle first row and column
    IF firstRowZero:
        FOR j = 0 TO n-1: matrix[0][j] = 0
    IF firstColZero:
        FOR i = 0 TO m-1: matrix[i][0] = 0
```

### Trace (O(1) Approach)

```
  ┌───┬───┬───┐
  │ 0 │ 1 │ 2 │    firstRowZero = true (matrix[0][0]=0)
  │ 3 │ 4 │ 5 │    firstColZero = true (matrix[0][0]=0)
  │ 0 │ 6 │ 7 │
  └───┴───┴───┘

  Mark: matrix[2][0] already 0 → mark matrix[0][0]=0 (already 0)

  Zero from markers: no additional changes (row 0, col 0 already flagged)

  Handle first row: [0,0,0]
  Handle first col: [0,_,0]

  Result:
  ┌───┬───┬───┐
  │ 0 │ 0 │ 0 │
  │ 0 │ 4 │ 5 │
  │ 0 │ 0 │ 0 │
  └───┴───┴───┘
```

---

## 2. Word Search (LeetCode 79)

```
  Find if a word exists in the grid by moving to adjacent cells.
  Each cell can be used only once per word.

  Board:                    Word: "ABCCED"
  ┌───┬───┬───┬───┐
  │ A │ B │ C │ E │        A → B → C
  ├───┼───┼───┼───┤                ↓
  │ S │ F │ C │ S │            C ← C
  ├───┼───┼───┼───┤            ↓
  │ A │ D │ E │ E │            E → D ✓
  └───┴───┴───┴───┘
```

### Algorithm (DFS + Backtracking)

```
FUNCTION exist(board, word):
    FOR i = 0 TO m-1:
        FOR j = 0 TO n-1:
            IF dfs(board, word, i, j, 0):
                RETURN true
    RETURN false

FUNCTION dfs(board, word, i, j, k):
    // k = current index in word
    IF k == length(word): RETURN true
    IF i < 0 OR i ≥ m OR j < 0 OR j ≥ n: RETURN false
    IF board[i][j] ≠ word[k]: RETURN false

    // Mark visited
    temp = board[i][j]
    board[i][j] = '#'

    // Explore all 4 directions
    found = dfs(board, word, i+1, j, k+1) OR
            dfs(board, word, i-1, j, k+1) OR
            dfs(board, word, i, j+1, k+1) OR
            dfs(board, word, i, j-1, k+1)

    // Backtrack (restore)
    board[i][j] = temp

    RETURN found
```

### Key Points

```
  - Time: O(m × n × 4^L) where L = word length
  - Space: O(L) recursion stack
  - Mark visited by changing cell to '#' (backtrack to restore)
  - No separate visited array needed — modify board in-place
```

---

## 3. Number of Islands (LeetCode 200)

```
  Count connected groups of '1's (land).
  '0' is water. Connections: up/down/left/right.

  ┌───┬───┬───┬───┬───┐
  │ 1 │ 1 │ 1 │ 1 │ 0 │    Island 1: top-left block
  ├───┼───┼───┼───┼───┤
  │ 1 │ 1 │ 0 │ 1 │ 0 │
  ├───┼───┼───┼───┼───┤
  │ 1 │ 1 │ 0 │ 0 │ 0 │
  ├───┼───┼───┼───┼───┤
  │ 0 │ 0 │ 0 │ 0 │ 0 │
  ├───┼───┼───┼───┼───┤
  │ 0 │ 0 │ 0 │ 1 │ 1 │    Island 2: bottom-right
  └───┴───┴───┴───┴───┘

  Answer: 2 islands
```

### Algorithm (DFS Flood Fill)

```
FUNCTION numIslands(grid):
    count = 0
    FOR i = 0 TO m-1:
        FOR j = 0 TO n-1:
            IF grid[i][j] == '1':
                count++
                dfs(grid, i, j)    // sink the island
    RETURN count

FUNCTION dfs(grid, i, j):
    IF i < 0 OR i ≥ m OR j < 0 OR j ≥ n: RETURN
    IF grid[i][j] ≠ '1': RETURN

    grid[i][j] = '0'    // mark visited (sink)

    dfs(grid, i+1, j)
    dfs(grid, i-1, j)
    dfs(grid, i, j+1)
    dfs(grid, i, j-1)
```

### Trace

```
  Start scanning:
  (0,0) = '1' → count=1, DFS sinks entire first island
  
  After sinking island 1:
  ┌───┬───┬───┬───┬───┐
  │ 0 │ 0 │ 0 │ 0 │ 0 │
  │ 0 │ 0 │ 0 │ 0 │ 0 │
  │ 0 │ 0 │ 0 │ 0 │ 0 │
  │ 0 │ 0 │ 0 │ 0 │ 0 │
  │ 0 │ 0 │ 0 │ 1 │ 1 │
  └───┴───┴───┴───┴───┘

  Continue scanning...
  (4,3) = '1' → count=2, DFS sinks second island

  Time: O(m × n)
  Space: O(m × n) worst case recursion stack
```

---

## 4. Game of Life (LeetCode 289)

```
  Cellular automaton. Each cell is alive (1) or dead (0).
  Simultaneously update all cells based on 8 neighbors:

  Rules:
  1. Live cell, < 2 live neighbors → dies (underpopulation)
  2. Live cell, 2-3 live neighbors → lives
  3. Live cell, > 3 live neighbors → dies (overpopulation)
  4. Dead cell, exactly 3 live neighbors → becomes alive

  Challenge: Update simultaneously (can't use partially updated values).
```

### In-Place Solution: Use State Encoding

```
  Encode transitions to avoid needing a copy:
  
  Current → Next   Code   Meaning
    0    →   0      0     was dead, stays dead
    1    →   0      1     was alive, now dead
    0    →   1      2     was dead, now alive
    1    →   1      3     was alive, stays alive

  Original state = code % 2
  New state = code / 2
```

### Algorithm

```
FUNCTION gameOfLife(board):
    m = rows, n = cols
    dx = [-1,-1,-1, 0, 0, 1, 1, 1]
    dy = [-1, 0, 1,-1, 1,-1, 0, 1]

    FOR i = 0 TO m-1:
        FOR j = 0 TO n-1:
            // Count live neighbors (using original state)
            live = 0
            FOR d = 0 TO 7:
                ni = i + dx[d]
                nj = j + dy[d]
                IF isValid(ni, nj) AND board[ni][nj] % 2 == 1:
                    live++

            // Apply rules
            IF board[i][j] == 1:  // currently alive
                IF live == 2 OR live == 3:
                    board[i][j] = 3  // alive → alive
                // else stays 1 (alive → dead, code already 1)
            ELSE:                  // currently dead
                IF live == 3:
                    board[i][j] = 2  // dead → alive

    // Final pass: extract new state
    FOR i = 0 TO m-1:
        FOR j = 0 TO n-1:
            board[i][j] = board[i][j] / 2
```

### Trace

```
  Board:
  ┌───┬───┬───┐
  │ 0 │ 1 │ 0 │
  │ 0 │ 0 │ 1 │
  │ 1 │ 1 │ 1 │
  │ 0 │ 0 │ 0 │
  └───┴───┴───┘

  Cell (0,1)=1: neighbors alive=1 → dies → stays 1
  Cell (1,2)=1: neighbors alive=3 → lives → becomes 3
  Cell (2,0)=1: neighbors alive=1 → dies → stays 1
  Cell (2,1)=1: neighbors alive=3 → lives → becomes 3
  Cell (2,2)=1: neighbors alive=2 → lives → becomes 3
  Cell (1,0)=0: neighbors alive=3 → born → becomes 2
  Cell (3,1)=0: neighbors alive=2 → stays dead → stays 0

  After encoding:
  ┌───┬───┬───┐
  │ 0 │ 1 │ 0 │
  │ 2 │ 0 │ 3 │     2 = was dead, now alive
  │ 1 │ 3 │ 3 │     3 = was alive, stays alive
  │ 0 │ 0 │ 0 │     1 = was alive, now dead
  └───┴───┴───┘

  After /2:
  ┌───┬───┬───┐
  │ 0 │ 0 │ 0 │
  │ 1 │ 0 │ 1 │
  │ 0 │ 1 │ 1 │
  │ 0 │ 0 │ 0 │
  └───┴───┴───┘
```

---

## 5. Rotate Image (Recap)

```
  See Chapter 3 for full details.
  Quick reminder: Rotate 90° CW = Transpose + Reverse rows
```

---

## Complexity Summary

| Problem | Time | Space |
|---------|------|-------|
| Set Matrix Zeroes (optimal) | O(m × n) | O(1) |
| Word Search | O(m × n × 4^L) | O(L) |
| Number of Islands | O(m × n) | O(m × n) stack |
| Game of Life | O(m × n) | O(1) in-place |

---

## Summary Table

| Problem | Key Technique |
|---------|---------------|
| Set Matrix Zeroes | Use first row/col as markers |
| Word Search | DFS + backtracking, mark '#' |
| Number of Islands | DFS flood fill, sink visited |
| Game of Life | State encoding (0,1,2,3), two passes |
| Common pattern | In-place marking to avoid extra space |
| Grid DFS | Direction arrays, boundary checks |

---

## Quick Revision Questions

1. **Why can't you zero cells** as you find them in Set Matrix Zeroes? Show a counterexample.

2. **Trace a Word Search** for "SEE" in the board from the example.

3. **What is the BFS alternative** for Number of Islands? When would you prefer it over DFS?

4. **Explain the state encoding** in Game of Life. Why does `% 2` give the original state?

5. **How would you handle** an infinite board for Game of Life?

6. **Compare DFS and BFS** for grid problems: when does each approach shine?

---

[← Previous: Search in Sorted Matrix](05-search-sorted-matrix.md) | [Back to README](../README.md) | [Next: Back to Start →](../README.md)
