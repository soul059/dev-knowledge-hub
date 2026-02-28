# Chapter 4: Word Search

## Overview
The **Word Search** problem finds if a word exists in a 2D grid of characters by tracing adjacent cells (up, down, left, right). Each cell may be used only once per word. This is a classic grid DFS + backtracking problem — explore a path character by character, backtrack when the match fails.

---

## Problem Statement

```
Given an m×n board of characters and a target word,
return TRUE if the word exists in the grid.

The word can be formed from sequentially adjacent cells
(horizontally or vertically). Same cell cannot be reused.

Board:              Word: "ABCCED"
┌───┬───┬───┬───┐
│ A │ B │ C │ E │   A → B → C
│───┼───┼───┼───│             ↓
│ S │ F │ C │ S │         C ← C
│───┼───┼───┼───│         ↓
│ A │ D │ E │ E │     D ← E
└───┴───┴───┴───┘
Result: TRUE (path shown with arrows)
```

---

## Algorithm

```
1. Scan every cell in the grid
2. If cell matches first character of word → start DFS
3. DFS: mark current cell visited, check 4 neighbors
4. If all characters matched → return TRUE
5. If no neighbor matches next char → BACKTRACK (unmark cell)

FUNCTION exist(board, word):
    FOR r = 0 TO rows-1:
        FOR c = 0 TO cols-1:
            IF board[r][c] == word[0]:
                IF dfs(board, word, r, c, 0):
                    RETURN TRUE
    RETURN FALSE

FUNCTION dfs(board, word, r, c, idx):
    IF idx == length(word):
        RETURN TRUE                          ← All chars matched!
    
    IF r < 0 OR r >= rows OR c < 0 OR c >= cols:
        RETURN FALSE                         ← Out of bounds
    IF board[r][c] != word[idx]:
        RETURN FALSE                         ← Character mismatch
    
    temp = board[r][c]
    board[r][c] = '#'                        ← Mark visited (in-place)
    
    found = dfs(board, word, r+1, c, idx+1)  ← Down
          OR dfs(board, word, r-1, c, idx+1)  ← Up
          OR dfs(board, word, r, c+1, idx+1)  ← Right
          OR dfs(board, word, r, c-1, idx+1)  ← Left
    
    board[r][c] = temp                       ← UNDO (restore cell)
    
    RETURN found
```

---

## Step-by-Step Trace

```
Board:           Word: "BCF"
┌───┬───┬───┐
│ A │ B │ C │
│ D │ E │ F │
│ G │ H │ I │
└───┴───┴───┘

Scan → cell (0,1) = 'B' matches word[0]

DFS from (0,1), idx=0:
  board[0][1] = '#' (mark B visited)
  
  Try neighbors for word[1]='C':
    Down (1,1) = 'E' ≠ 'C'  → skip
    Up (-1,1) → out of bounds → skip
    Right (0,2) = 'C' == 'C' ✓
    
    DFS from (0,2), idx=1:
      board[0][2] = '#' (mark C visited)
      
      Try neighbors for word[2]='F':
        Down (1,2) = 'F' == 'F' ✓
        
        DFS from (1,2), idx=2:
          board[1][2] = '#' (mark F visited)
          
          idx+1 == 3 == length("BCF")
          RETURN TRUE ✓
        
        Restore board[1][2] = 'F'
      Restore board[0][2] = 'C'
  Restore board[0][1] = 'B'

Result: TRUE
```

---

## Why In-Place Marking?

```
Instead of a separate visited[][] array:
  → Temporarily replace board[r][c] with '#'
  → No extra O(m×n) space needed
  → Automatically prevents revisiting (won't match any letter)
  → Restore original character on backtrack

This is a SPACE OPTIMIZATION trick:
  With visited array → O(m×n) extra space
  With in-place mark → O(1) extra space (only stack)
```

---

## Optimizations & Pruning

```
1. EARLY TERMINATION:
   If word has more of a character than the board → FALSE immediately
   Count characters before DFS.

2. REVERSE SEARCH:
   If word's first char appears more often than last char,
   search for the REVERSED word (fewer starting points).

3. SHORT-CIRCUIT OR:
   Stop exploring directions once TRUE is found:
   IF dfs(down) → return TRUE immediately
   (Don't check up/right/left)

4. FREQUENCY PRUNING:
   Count each char frequency in board.
   If any char in word exceeds board frequency → FALSE.
```

---

## Complexity

```
┌───────────────────────────────────────────────┐
│ Time:  O(m × n × 3^L)                        │
│   m×n starting cells                          │
│   At each step, 3 choices (4 dirs - 1 parent) │
│   L = length of word                          │
│                                               │
│ Space: O(L) for recursion stack               │
│   In-place marking avoids visited array       │
│                                               │
│ Note: 3^L not 4^L because we never revisit    │
│   the cell we just came from                  │
└───────────────────────────────────────────────┘
```

---

## Variations

| Variant | Difference |
|---------|-----------|
| Word Search I | Find if single word exists |
| Word Search II | Find all words from dictionary (use Trie) |
| 8-directional | Include diagonal neighbors (8 instead of 4) |
| With obstacles | Some cells are blocked |
| Longest word | Find longest formable word (DFS + pruning) |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Pattern | Grid DFS + Backtracking |
| Start | Any cell matching word[0] |
| Move | 4 directions (up/down/left/right) |
| Constraint | Visit each cell once per path |
| Mark | In-place '#' replacement |
| Undo | Restore original character |
| Time | O(m × n × 3^L) |
| Space | O(L) recursion stack |

---

## Quick Revision Questions

1. **Why do we try every cell** as a starting point?
2. **How does in-place marking** prevent revisiting?
3. **Why is the branching factor 3** and not 4?
4. **When should you search** the reversed word?
5. **What frequency check** can prune before DFS starts?
6. **How does Word Search II** differ in approach?

---

[← Previous: Sudoku Solver](03-sudoku-solver.md) | [Next: Unique Paths with Backtracking →](05-unique-paths-with-backtrack.md)

[← Back to README](../README.md)
