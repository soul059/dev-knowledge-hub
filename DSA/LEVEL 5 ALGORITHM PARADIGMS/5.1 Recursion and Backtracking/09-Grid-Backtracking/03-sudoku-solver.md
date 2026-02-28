# Chapter 3: Sudoku Solver

## Overview
Sudoku solving is a **constraint satisfaction problem** that demonstrates backtracking at its finest. The algorithm tries digits 1-9 in empty cells, checks constraints, and backtracks when stuck — exactly like a human solver, but exhaustive.

---

## Rules / Constraints

```
Fill a 9×9 grid so that:
  1. Each ROW contains digits 1-9 exactly once
  2. Each COLUMN contains digits 1-9 exactly once
  3. Each 3×3 BOX contains digits 1-9 exactly once

┌───────┬───────┬───────┐
│ 5 3 . │ . 7 . │ . . . │
│ 6 . . │ 1 9 5 │ . . . │
│ . 9 8 │ . . . │ . 6 . │
├───────┼───────┼───────┤
│ 8 . . │ . 6 . │ . . 3 │
│ 4 . . │ 8 . 3 │ . . 1 │
│ 7 . . │ . 2 . │ . . 6 │
├───────┼───────┼───────┤
│ . 6 . │ . . . │ 2 8 . │
│ . . . │ 4 1 9 │ . . 5 │
│ . . . │ . 8 . │ . 7 9 │
└───────┴───────┴───────┘
  '.' = empty cell to fill
```

---

## Pseudocode

```
FUNCTION solveSudoku(board):
    solve(board)

FUNCTION solve(board):
    FOR r = 0 TO 8:
        FOR c = 0 TO 8:
            IF board[r][c] == '.':           ← Find empty cell
                FOR num = '1' TO '9':
                    IF isValid(board, r, c, num):
                        board[r][c] = num    ← PLACE digit
                        IF solve(board):     ← EXPLORE
                            RETURN TRUE      ← Solution found!
                        board[r][c] = '.'    ← UNDO (backtrack)
                RETURN FALSE                 ← No digit works here
    RETURN TRUE                              ← All cells filled!

FUNCTION isValid(board, row, col, num):
    // Check row
    FOR c = 0 TO 8:
        IF board[row][c] == num: RETURN FALSE
    
    // Check column
    FOR r = 0 TO 8:
        IF board[r][col] == num: RETURN FALSE
    
    // Check 3×3 box
    boxRow = (row / 3) * 3
    boxCol = (col / 3) * 3
    FOR r = boxRow TO boxRow + 2:
        FOR c = boxCol TO boxCol + 2:
            IF board[r][c] == num: RETURN FALSE
    
    RETURN TRUE
```

---

## Optimization: Precomputed Constraint Sets

```
Instead of scanning row/col/box each time (O(9) per check),
maintain SETS for O(1) lookups:

rowUsed[9][10]  ← rowUsed[r][d] = true if digit d is in row r
colUsed[9][10]  ← colUsed[c][d] = true if digit d is in col c
boxUsed[9][10]  ← boxUsed[b][d] = true if digit d is in box b

Box index: b = (r/3)*3 + (c/3)

  Box indices:
  ┌─────┬─────┬─────┐
  │  0  │  1  │  2  │
  ├─────┼─────┼─────┤
  │  3  │  4  │  5  │
  ├─────┼─────┼─────┤
  │  6  │  7  │  8  │
  └─────┴─────┴─────┘

isValid becomes O(1):
  RETURN NOT rowUsed[r][d] 
     AND NOT colUsed[c][d]
     AND NOT boxUsed[b][d]

MAKE: rowUsed[r][d]=true, colUsed[c][d]=true, boxUsed[b][d]=true
UNDO: rowUsed[r][d]=false, colUsed[c][d]=false, boxUsed[b][d]=false
```

---

## Solving Process Visualization

```
Board with 51 empty cells:
Step 1: Find first empty cell, say (0,2)
        Try 1: invalid (in box)
        Try 2: invalid (in column)
        Try 4: valid! Place 4.

Step 2: Find next empty cell (0,3)
        Try digits... find valid one.

Step 3: Continue filling...

Step ?: Reach a cell where NO digit is valid.
        → BACKTRACK to previous cell
        → Try next digit there
        → If no more digits, backtrack further

Eventually: All 51 cells filled → SOLVED!
Or: Impossible (malformed puzzle) → returns FALSE
```

---

## MRV Optimization (Most Constrained Variable)

```
Instead of scanning cells left-to-right, top-to-bottom:
  → Find the empty cell with FEWEST valid candidates
  → Fill THAT cell first

Why? If a cell has only 1 possible digit:
  → Fill it immediately (forced move)
  → No backtracking needed for that cell

If a cell has 2 possible digits:
  → Only 2 branches to explore (vs up to 9)
  → Fail faster → more pruning

This can reduce nodes explored by 10-100×!
```

---

## Complexity

```
┌────────────────────────────────────────────────────────┐
│ Worst case: O(9^(empty cells))                         │
│   For 51 empty cells: 9^51 ≈ 10^48 (theoretical max)  │
│                                                        │
│ Practical: constraint propagation reduces this         │
│   dramatically. Most Sudokus solve in ~1000 backtracks │
│                                                        │
│ With MRV + constraint propagation:                     │
│   Near-instant for standard puzzles                    │
│                                                        │
│ Space: O(81) for board + O(empty cells) stack          │
│   With sets: additional O(81×9) ≈ O(1)                 │
└────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Component | Sudoku Implementation |
|-----------|----------------------|
| State | 9×9 board |
| Empty cell marker | '.' or 0 |
| Choices | Digits 1-9 |
| Constraint | Row, column, 3×3 box uniqueness |
| Make | Place digit, update sets |
| Undo | Clear cell, revert sets |
| Goal | No empty cells remaining |
| Optimization | Constraint sets (O(1)), MRV heuristic |

---

## Quick Revision Questions

1. **What are the three constraints** in Sudoku?
2. **How do you calculate** the 3×3 box index from (row, col)?
3. **Why does the algorithm return FALSE** when no digit fits?
4. **What is MRV** and how does it help?
5. **What is the worst-case** complexity of Sudoku solving?
6. **How do constraint sets** improve validation from O(9) to O(1)?

---

[← Previous: N-Queens Problem](02-n-queens-problem.md) | [Next: Word Search →](04-word-search.md)

[← Back to README](../README.md)
