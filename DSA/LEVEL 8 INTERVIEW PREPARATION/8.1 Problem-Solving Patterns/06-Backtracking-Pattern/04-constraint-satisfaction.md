# Chapter 4: Constraint Satisfaction

## 📋 Chapter Overview
Backtracking shines in constraint satisfaction: place items while respecting rules. Classic examples: N-Queens, Sudoku, and word search.

---

## 🔍 Problem 1: N-Queens

**Place n queens on an n×n board so no two attack each other.**

```
  4-Queens solution:
  . Q . .        Constraints:
  . . . Q        - One queen per row
  Q . . .        - One queen per column
  . . Q .        - No two on same diagonal
```

### Solution

```
function solveNQueens(n):
    result = []
    board = n×n grid of '.'
    backtrack(board, 0, n, result)
    return result

function backtrack(board, row, n, result):
    if row == n:
        result.add(boardToStrings(board))
        return
    
    for col = 0 to n-1:
        if isValid(board, row, col, n):
            board[row][col] = 'Q'
            backtrack(board, row + 1, n, result)
            board[row][col] = '.'          // backtrack

function isValid(board, row, col, n):
    // Check column above
    for r = 0 to row-1:
        if board[r][col] == 'Q': return false
    
    // Check upper-left diagonal
    r, c = row-1, col-1
    while r >= 0 AND c >= 0:
        if board[r][c] == 'Q': return false
        r -= 1; c -= 1
    
    // Check upper-right diagonal
    r, c = row-1, col+1
    while r >= 0 AND c < n:
        if board[r][c] == 'Q': return false
        r -= 1; c += 1
    
    return true
```

### Optimized: Track with Sets

```
function backtrack(row, n, cols, diag1, diag2, board, result):
    if row == n:
        result.add(boardToStrings(board))
        return
    
    for col = 0 to n-1:
        d1 = row - col          // main diagonal identifier
        d2 = row + col          // anti-diagonal identifier
        
        if col in cols OR d1 in diag1 OR d2 in diag2:
            continue
        
        cols.add(col); diag1.add(d1); diag2.add(d2)
        board[row][col] = 'Q'
        backtrack(row + 1, n, cols, diag1, diag2, board, result)
        board[row][col] = '.'
        cols.remove(col); diag1.remove(d1); diag2.remove(d2)
```

### Diagonal Insight

```
  Row-Col = constant on each main diagonal (\)
  Row+Col = constant on each anti-diagonal (/)
  
  Board (row-col values):     Board (row+col values):
   0  -1  -2  -3              0   1   2   3
   1   0  -1  -2              1   2   3   4
   2   1   0  -1              2   3   4   5
   3   2   1   0              3   4   5   6
```

---

## 🔍 Problem 2: Sudoku Solver

```
function solveSudoku(board):
    solve(board)

function solve(board):
    for r = 0 to 8:
        for c = 0 to 8:
            if board[r][c] == '.':
                for num = '1' to '9':
                    if isValid(board, r, c, num):
                        board[r][c] = num
                        if solve(board): return true
                        board[r][c] = '.'      // backtrack
                return false                    // no valid number
    return true                                 // all cells filled

function isValid(board, row, col, num):
    for i = 0 to 8:
        if board[row][i] == num: return false   // row check
        if board[i][col] == num: return false   // col check
    
    // 3×3 box check
    boxRow = (row / 3) * 3
    boxCol = (col / 3) * 3
    for r = boxRow to boxRow+2:
        for c = boxCol to boxCol+2:
            if board[r][c] == num: return false
    
    return true
```

### Key Insight: Early Return

```
  When no digit 1-9 works for a cell → return false
  This triggers backtracking up the call stack.
  
  Without early return, we'd try ALL possibilities
  even when already impossible → exponential waste.
```

---

## 🔍 Problem 3: Word Search (LeetCode 79)

**Find if a word exists in a grid by moving to adjacent cells.**

```
  Board:                   Word: "ABCCED"
  A B C E
  S F C S                  Path: A(0,0)→B(0,1)→C(0,2)→C(1,2)→E(0,3)→D? ✗
  A D E E                  Path: A(0,0)→B(0,1)→C(0,2)→C(1,2)→E(2,3)→D(2,1) ✓
```

```
function exist(board, word):
    for r = 0 to rows-1:
        for c = 0 to cols-1:
            if dfs(board, word, r, c, 0):
                return true
    return false

function dfs(board, word, r, c, index):
    if index == len(word): return true
    if r < 0 OR r >= rows OR c < 0 OR c >= cols: return false
    if board[r][c] != word[index]: return false
    
    temp = board[r][c]
    board[r][c] = '#'              // mark visited
    
    found = dfs(board, word, r-1, c, index+1) OR
            dfs(board, word, r+1, c, index+1) OR
            dfs(board, word, r, c-1, index+1) OR
            dfs(board, word, r, c+1, index+1)
    
    board[r][c] = temp             // un-mark (backtrack)
    return found
```

---

## 🔍 Problem 4: Letter Combinations of Phone Number

```
  Mapping: 2="abc", 3="def", ..., 9="wxyz"
  Input: "23"
  Output: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

```
function letterCombinations(digits):
    if digits is empty: return []
    map = {"2":"abc", "3":"def", "4":"ghi", "5":"jkl",
           "6":"mno", "7":"pqrs", "8":"tuv", "9":"wxyz"}
    result = []
    backtrack(digits, 0, [], result, map)
    return result

function backtrack(digits, index, path, result, map):
    if index == len(digits):
        result.add(join(path))
        return
    
    letters = map[digits[index]]
    for letter in letters:
        path.add(letter)
        backtrack(digits, index + 1, path, result, map)
        path.removeLast()
```

---

## 📊 Constraint Satisfaction Pattern

```
  All constraint satisfaction problems follow:
  
  1. Find next empty slot
  2. Try each valid option for that slot
  3. If option leads to solution → return true
  4. If no option works → backtrack (return false)
  
  ┌──────────────────────────────────┐
  │  for each empty position:       │
  │    for each candidate:          │
  │      if valid(candidate):       │
  │        place(candidate)         │
  │        if solve(): return true  │
  │        remove(candidate)        │
  │    return false  ← backtrack    │
  │  return true     ← solved       │
  └──────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| N-Queens | Row-by-row, check col + 2 diagonals |
| Diagonal tracking | row-col for \, row+col for / |
| Sudoku | Try 1-9 per empty cell, backtrack on failure |
| Word Search | DFS on grid with in-place visited marking |
| Phone letters | Simple branching per digit |
| Early return | `return false` when no option works = triggers backtrack |

---

## ❓ Revision Questions

1. **N-Queens: how to check diagonals efficiently?** → Track `row - col` (main diagonal) and `row + col` (anti-diagonal) in sets.
2. **Sudoku: when does backtracking occur?** → When no digit 1-9 is valid for the current empty cell.
3. **Word Search: why mark and unmark cells?** → To prevent revisiting the same cell in one path, while allowing it for different paths.
4. **What's common to all constraint satisfaction?** → Find empty slot → try candidates → recurse → backtrack if no candidate works.
5. **Time complexity of N-Queens?** → O(n!) roughly — at most n choices for row 0, n-1 for row 1, etc. (with pruning, much less in practice).

---

[← Previous: Permutation Problems](03-permutation-problems.md) | [Next: Pruning Techniques →](05-pruning-techniques.md)
