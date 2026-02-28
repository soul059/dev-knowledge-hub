# Surrounded Regions

[← Previous: Flood Fill](03-flood-fill.md) | [Back to TOC](../README.md) | [Next: Clone Graph →](05-clone-graph.md)

---

## Problem

Given an M×N board with 'X' and 'O', capture all 'O' regions that are **completely surrounded** by 'X'. An 'O' on the **border** cannot be captured, and any 'O' connected to a border 'O' is also safe.

## Key Insight: Reverse Thinking

```
    Instead of finding surrounded regions (hard),
    find UN-surrounded regions (easy)!
    
    Strategy:
    1. Start from border 'O's → mark all connected 'O's as SAFE
    2. Everything NOT safe → capture (flip to 'X')
    
    ┌───────────────────────────────────┐
    │ If an 'O' is connected to the    │
    │ border, it CANNOT be surrounded. │
    │ Everything else IS surrounded.   │
    └───────────────────────────────────┘
```

## Algorithm

```
FUNCTION solve(board):
    // Step 1: DFS from all border 'O's, mark as safe ('S')
    FOR each cell on the border:
        IF board[r][c] == 'O':
            DFS_Mark(board, r, c)     // marks connected O's as 'S'
    
    // Step 2: Process the entire board
    FOR each cell (r, c):
        IF board[r][c] == 'O':
            board[r][c] = 'X'         // surrounded → capture
        IF board[r][c] == 'S':
            board[r][c] = 'O'         // safe → restore

FUNCTION DFS_Mark(board, r, c):
    IF out of bounds OR board[r][c] != 'O':
        RETURN
    board[r][c] = 'S'                 // mark safe
    DFS_Mark in all 4 directions
```

## Trace

```
    Before:              Mark safe:           After:
    ┌───┬───┬───┬───┐   ┌───┬───┬───┬───┐   ┌───┬───┬───┬───┐
    │ X │ X │ X │ X │   │ X │ X │ X │ X │   │ X │ X │ X │ X │
    ├───┼───┼───┼───┤   ├───┼───┼───┼───┤   ├───┼───┼───┼───┤
    │ X │ O │ O │ X │   │ X │ O │ O │ X │   │ X │ X │ X │ X │
    ├───┼───┼───┼───┤   ├───┼───┼───┼───┤   ├───┼───┼───┼───┤
    │ X │ X │ O │ X │   │ X │ X │ O │ X │   │ X │ X │ X │ X │
    ├───┼───┼───┼───┤   ├───┼───┼───┼───┤   ├───┼───┼───┼───┤
    │ X │ O │ X │ X │   │ X │ S │ X │ X │   │ X │ O │ X │ X │
    └───┴───┴───┴───┘   └───┴───┴───┴───┘   └───┴───┴───┴───┘
    
    Border O at (3,1) → marked Safe
    Interior O's at (1,1)(1,2)(2,2) → NOT connected to border → captured!
```

---

[← Previous: Flood Fill](03-flood-fill.md) | [Back to TOC](../README.md) | [Next: Clone Graph →](05-clone-graph.md)
