# Chapter 6: Knight's Tour

## Overview
The **Knight's Tour** problem asks: can a chess knight visit every square on an N×N board exactly once? This is one of the oldest mathematical puzzles (Euler, 1759) and a beautiful application of backtracking with heuristic optimization.

---

## Problem Statement

```
Place a knight on any square of an N×N chessboard.
Find a sequence of moves such that the knight visits
every square exactly once.

Knight moves in an "L" shape:
  2 squares in one direction + 1 square perpendicular

    . . X . X . .
    . X . . . X .       8 possible moves
    . . . K . . .       from position K
    . X . . . X .
    . . X . X . .

8 possible offsets:
  (-2,-1), (-2,+1), (-1,-2), (-1,+2)
  (+1,-2), (+1,+2), (+2,-1), (+2,+1)
```

---

## Pseudocode

```
FUNCTION knightTour(n):
    board[n][n] = -1 (all cells unvisited)
    board[0][0] = 0                     ← Start at (0,0), move #0
    
    IF solve(board, 0, 0, 1, n):        ← Next move is #1
        printBoard(board)
    ELSE:
        PRINT "No solution exists"

FUNCTION solve(board, r, c, moveNum, n):
    IF moveNum == n*n:
        RETURN TRUE                     ← All squares visited!
    
    FOR each (dr, dc) in knightMoves:
        nr = r + dr
        nc = c + dc
        IF isValid(nr, nc, board, n):
            board[nr][nc] = moveNum     ← PLACE (record move #)
            IF solve(board, nr, nc, moveNum+1, n):
                RETURN TRUE             ← Solution found
            board[nr][nc] = -1          ← UNDO (backtrack)
    
    RETURN FALSE                        ← Dead end

FUNCTION isValid(r, c, board, n):
    RETURN r >= 0 AND r < n 
       AND c >= 0 AND c < n 
       AND board[r][c] == -1            ← Not yet visited
```

---

## Trace for 5×5 Board (Partial)

```
Start: Knight at (0,0), move 0

Step 0:                    Step 1: move to (1,2)
┌────┬────┬────┬────┬────┐ ┌────┬────┬────┬────┬────┐
│  0 │  . │  . │  . │  . │ │  0 │  . │  . │  . │  . │
├────┼────┼────┼────┼────┤ ├────┼────┼────┼────┼────┤
│  . │  . │  . │  . │  . │ │  . │  . │  1 │  . │  . │
├────┼────┼────┼────┼────┤ ├────┼────┼────┼────┼────┤
│  . │  . │  . │  . │  . │ │  . │  . │  . │  . │  . │
├────┼────┼────┼────┼────┤ ├────┼────┼────┼────┼────┤
│  . │  . │  . │  . │  . │ │  . │  . │  . │  . │  . │
├────┼────┼────┼────┼────┤ ├────┼────┼────┼────┼────┤
│  . │  . │  . │  . │  . │ │  . │  . │  . │  . │  . │
└────┴────┴────┴────┴────┘ └────┴────┴────┴────┴────┘

... many steps later (with backtracking) ...

Complete 5×5 solution:
┌────┬────┬────┬────┬────┐
│  0 │ 11 │  8 │ 19 │  2 │
├────┼────┼────┼────┼────┤
│  9 │ 18 │  1 │ 12 │  7 │
├────┼────┼────┼────┼────┤
│ 16 │ 23 │ 10 │  3 │ 20 │
├────┼────┼────┼────┼────┤
│ 13 │  4 │ 17 │ 22 │ 11 │   (← actual numbers may vary)
├────┼────┼────┼────┼────┤
│ 24 │ 15 │  6 │ 13 │ 14 │
└────┴────┴────┴────┴────┘
Total moves: 25 (all squares visited)
```

---

## Warnsdorff's Rule (Heuristic)

```
IDEA: At each step, move to the neighbor that has
      the FEWEST onward moves (most constrained first).

Why? Visiting "corner" squares early prevents dead-ends.
     Squares with few exits become inaccessible if left.

FUNCTION getNextMoves(board, r, c, n):
    moves = []
    FOR each (dr, dc) in knightMoves:
        nr, nc = r+dr, c+dc
        IF isValid(nr, nc, board, n):
            degree = countValidMoves(board, nr, nc, n)
            moves.add((nr, nc, degree))
    
    SORT moves BY degree ASCENDING      ← Fewest exits first!
    RETURN moves

Performance comparison on 8×8 board:
  ┌────────────────────┬──────────────────────┐
  │ Plain Backtracking │ ~200 million attempts │
  │ With Warnsdorff    │ ~64 attempts (linear!)│
  └────────────────────┴──────────────────────┘

Warnsdorff's rule finds a tour in O(n²) time for
most board sizes — practically no backtracking needed!
```

---

## Open vs Closed Tours

```
OPEN Tour:                      CLOSED Tour:
  Start and end on               End square is a knight's
  different squares               move away from start
  
  ┌──┬──┬──┬──┐                  ┌──┬──┬──┬──┐
  │S │  │  │  │                  │S │  │  │  │
  ├──┼──┼──┼──┤                  ├──┼──┼──┼──┤
  │  │  │  │  │                  │  │  │  │  │←─┐
  ├──┼──┼──┼──┤                  ├──┼──┼──┼──┤  │ knight
  │  │  │  │  │                  │  │  │  │  │  │ move
  ├──┼──┼──┼──┤                  ├──┼──┼──┼──┤  │
  │  │  │E │  │                  │  │  │E │  │──┘
  └──┴──┴──┴──┘                  └──┴──┴──┴──┘

Closed Tour Check:
  After finding open tour ending at (er, ec),
  verify that (er, ec) → (sr, sc) is a valid knight move.
```

---

## Board Size Analysis

```
┌──────┬──────────────────────────────────────┐
│ Size │ Tour Exists?                         │
├──────┼──────────────────────────────────────┤
│ 1×1  │ Trivially yes (1 square)            │
│ 2×2  │ No                                   │
│ 3×3  │ No                                   │
│ 4×4  │ No (open), No (closed)              │
│ 5×5  │ Yes (open), No (closed)             │
│ 6×6  │ Yes (open), Yes (closed)            │
│ 7×7  │ Yes (open), No (closed)             │
│ 8×8  │ Yes (open), Yes (closed)            │
│ n≥5  │ Open tour always exists              │
└──────┴──────────────────────────────────────┘

Key theorem: For n ≥ 5, an open knight's tour
always exists on an n×n board.
```

---

## Complexity

```
┌──────────────────────────────────────────┐
│ Plain Backtracking:                      │
│   Time:  O(8^(n²))  — up to 8 choices   │
│          per cell, n² cells to fill      │
│   Space: O(n²) for board + stack         │
│                                          │
│ With Warnsdorff's Rule:                  │
│   Time:  O(n² × 8 log 8) ≈ O(n²)       │
│          (practically linear)            │
│   Space: O(n²) for board                 │
│                                          │
│ For 8×8: n²=64, 8^64 ≈ 10^57 (brute)   │
│          vs ~64 steps (Warnsdorff)       │
└──────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Grid | N×N chessboard |
| Piece | Knight (L-shaped moves) |
| Goal | Visit every cell exactly once |
| Choices | Up to 8 knight moves |
| Constraint | Not visited, in bounds |
| Make | board[nr][nc] = moveNum |
| Undo | board[nr][nc] = -1 |
| Key Optimization | Warnsdorff's rule (minimum degree first) |
| Open vs Closed | Closed = end connects back to start |

---

## Quick Revision Questions

1. **What are the 8 possible** knight move offsets?
2. **What does Warnsdorff's rule** prioritize and why?
3. **For what board sizes** does a knight's tour NOT exist?
4. **What's the difference** between an open and closed tour?
5. **Why is naive backtracking** impractical for 8×8 boards?
6. **How does move ordering** reduce backtracking to near-zero?

---

[← Previous: Unique Paths with Backtracking](05-unique-paths-with-backtrack.md) | [Next: Unit 10 - Palindrome Partitioning →](../10-Advanced-Backtracking/01-palindrome-partitioning.md)

[← Back to README](../README.md)
