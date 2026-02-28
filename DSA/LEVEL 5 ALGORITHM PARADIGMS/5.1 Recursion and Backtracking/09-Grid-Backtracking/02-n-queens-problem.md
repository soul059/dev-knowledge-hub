# Chapter 2: N-Queens Problem

## Overview
Place $n$ queens on an $n×n$ chessboard so that **no two queens attack each other**. This is the **poster child** of backtracking — beautiful constraint propagation, heavy pruning, and elegant implementation.

---

## Constraints

```
Queens attack along:
  • Same ROW     ─────
  • Same COLUMN  │
  • Diagonals    ╲ ╱

Two queens at (r1,c1) and (r2,c2) attack if:
  • r1 == r2        (same row)
  • c1 == c2        (same column)
  • |r1-r2| == |c1-c2|  (same diagonal)

Diagonal trick:
  • Same diagonal:      r - c is constant
  • Same anti-diagonal: r + c is constant
```

---

## Strategy: Row-by-Row Placement

```
Since each row can have AT MOST ONE queen,
place one queen per row, choosing the column.

Row 0: try columns 0, 1, ..., n-1
Row 1: try valid columns (not attacked)
Row 2: try valid columns
...
Row n-1: try valid columns

If all rows filled → SOLUTION!
If no valid column → BACKTRACK!
```

---

## Pseudocode

```
FUNCTION solveNQueens(n):
    result = []
    cols = set()         ← occupied columns
    diags = set()        ← occupied diagonals (r - c)
    antiDiags = set()    ← occupied anti-diagonals (r + c)
    board = n×n grid of '.'
    
    backtrack(0, n, board, cols, diags, antiDiags, result)
    RETURN result

FUNCTION backtrack(row, n, board, cols, diags, antiDiags, result):
    IF row == n:
        result.add(boardToStrings(board))
        RETURN
    
    FOR col = 0 TO n - 1:
        IF col IN cols: CONTINUE
        IF (row - col) IN diags: CONTINUE
        IF (row + col) IN antiDiags: CONTINUE
        
        // PLACE queen
        board[row][col] = 'Q'
        cols.add(col)
        diags.add(row - col)
        antiDiags.add(row + col)
        
        backtrack(row + 1, n, board, cols, diags, antiDiags, result)
        
        // REMOVE queen (backtrack)
        board[row][col] = '.'
        cols.remove(col)
        diags.remove(row - col)
        antiDiags.remove(row + col)
```

---

## Solved: 4-Queens

```
Start with empty 4×4 board:

Row 0: Try col 0
  . . . .     Q . . .
  . . . .  →  . . . .
  . . . .     . . . .
  . . . .     . . . .

Row 1: col0 ✗(col) col1 ✗(diag) col2 ✓
  Q . . .
  . . Q .
  . . . .
  . . . .

Row 2: col0 ✗(diag) col1 ✗(col) col2 ✗(col) col3 ✗(diag)
  ALL BLOCKED → BACKTRACK to Row 1!

Row 1: Try col 3
  Q . . .
  . . . Q
  . . . .
  . . . .

Row 2: col0 ✗(diag) col1 ✓
  Q . . .
  . . . Q
  . Q . .
  . . . .

Row 3: col0 ✗(col) col1 ✗(col) col2 ✗(diag) col3 ✗(col)
  ALL BLOCKED → BACKTRACK!

(...continue trying Row 0 col 1...)

Row 0, col 1:
  . Q . .
  . . . Q       SOLUTION 1:
  Q . . .   →   . Q . .
  . . Q .       . . . Q
                Q . . .
                . . Q .

Row 0, col 2:
  . . Q .       SOLUTION 2:
  Q . . .   →   . . Q .
  . . . Q       Q . . .
  . Q . .       . . . Q
                . Q . .

4-Queens has exactly 2 solutions ✓
```

---

## Constraint Checking with Sets: O(1)

```
Using three sets for O(1) conflict checking:

Queen at (2, 1):
  cols = {..., 1}           ← column 1 occupied
  diags = {..., 2-1=1}     ← diagonal 1 occupied  
  antiDiags = {..., 2+1=3} ← anti-diagonal 3 occupied

Checking (3, 1):
  col 1 in cols? YES → CONFLICT!

Checking (3, 2):
  col 2 in cols? no
  3-2=1 in diags? YES → CONFLICT! (same diagonal)

Checking (3, 0):
  col 0 in cols? no
  3-0=3 in diags? no
  3+0=3 in antiDiags? YES → CONFLICT!

All O(1) operations!
```

---

## N-Queens Solutions Count

```
┌─────┬────────────┐
│  n  │ Solutions   │
├─────┼────────────┤
│  1  │     1       │
│  2  │     0       │
│  3  │     0       │
│  4  │     2       │
│  5  │    10       │
│  6  │     4       │
│  7  │    40       │
│  8  │    92       │
│  9  │   352       │
│ 10  │   724       │
│ 12  │ 14,200      │
│ 14  │ 365,596     │
└─────┴────────────┘
```

---

## Complexity

```
┌────────────────────────────────────────────────────────┐
│ Without pruning: O(n^n) — try all n cols for n rows    │
│ With row constraint: O(n!) — permutations              │
│ With full pruning: much less than O(n!)                │
│                                                        │
│ Time: O(n!) upper bound (with pruning, much less)      │
│ Space: O(n) for the sets + board                       │
│                                                        │
│ 8-Queens: worst case ~40,320 but explores ~15,720      │
│           with smart pruning: ~2,000 nodes              │
└────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Component | Implementation |
|-----------|---------------|
| State | Board + three constraint sets |
| Choose | Pick column for current row |
| Constraint | Column, diagonal, anti-diagonal checks |
| Make | Place 'Q', add to sets |
| Undo | Remove 'Q', remove from sets |
| Goal | All n rows have queens |

---

## Quick Revision Questions

1. **Why place queens row by row** instead of trying all positions?
2. **What do r-c and r+c** represent geometrically?
3. **How many solutions** does the 8-Queens problem have?
4. **Why are sets preferred** over scanning the board for conflicts?
5. **What is the pruning ratio** for 8-Queens?
6. **Write the isValid check** without sets (scanning approach).

---

[← Previous: Rat in a Maze](01-rat-in-a-maze.md) | [Next: Sudoku Solver →](03-sudoku-solver.md)

[← Back to README](../README.md)
