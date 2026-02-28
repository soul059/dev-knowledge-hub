# Word Search in Grid (LeetCode 79)

## Overview
Given an m×n grid of characters and a word, determine if the word exists in the grid. The word can be constructed from sequentially adjacent cells (horizontal or vertical), and each cell may be used at most once. While this problem can be solved with pure backtracking, a trie becomes essential when searching for multiple words simultaneously.

---

## Problem Statement

```
  board = [['A','B','C','E'],
           ['S','F','C','S'],
           ['A','D','E','E']]
  
  word = "ABCCED" → True
  
  A → B → C → C → E → D
  ↓   →   →   ↓   ←   ↑
  [A] [B] [C] [E]
  [S] [F] [C] [S]
  [A] [D] [E] [E]
  
  Path: (0,0)→(0,1)→(0,2)→(1,2)→(2,2)→(2,1)
```

---

## Backtracking Solution (Single Word)

```python
class Solution:
    def exist(self, board, word):
        rows, cols = len(board), len(board[0])
        
        def backtrack(r, c, idx):
            if idx == len(word):
                return True  # All characters matched
            
            if (r < 0 or r >= rows or c < 0 or c >= cols or
                board[r][c] != word[idx]):
                return False
            
            # Mark visited
            temp = board[r][c]
            board[r][c] = '#'
            
            # Explore 4 directions
            found = (backtrack(r+1, c, idx+1) or
                     backtrack(r-1, c, idx+1) or
                     backtrack(r, c+1, idx+1) or
                     backtrack(r, c-1, idx+1))
            
            # Restore
            board[r][c] = temp
            return found
        
        for r in range(rows):
            for c in range(cols):
                if backtrack(r, c, 0):
                    return True
        return False
```

### Trace

```
  board = [['A','B','C','E'],
           ['S','F','C','S'],
           ['A','D','E','E']]
  word = "SEE"
  
  Start: Try every cell as starting point
  
  (0,0)='A' ≠ 'S' → skip
  (0,1)='B' ≠ 'S' → skip
  ...
  (1,3)='S' = 'S' → match! idx=0
    Mark (1,3)='#'
    
    Try (2,3)='E' = 'E' → match! idx=1
      Mark (2,3)='#'
      
      Try (2,2)='E' = 'E' → match! idx=2
        idx=3 == len("SEE") → return True! ✓
```

---

## Why Trie for Word Search?

```
  Single word: Backtracking is sufficient
  
  Multiple words: Brute force = run backtracking N times
  
  ┌───────────────────────────────────────────────────────────┐
  │  Grid: 15×15, Words: 10,000                              │
  │                                                           │
  │  Brute force: 10,000 × O(M × N × 4^L) = TLE!           │
  │                                                           │
  │  With Trie: ONE traversal of the grid                     │
  │  Build trie from all words, then search all simultaneously│
  │  O(M × N × 4^L) total — shared prefix pruning            │
  └───────────────────────────────────────────────────────────┘
  
  This is exactly what Word Search II (LeetCode 212) does.
  → See the next chapter for the full trie-based solution.
```

---

## Optimizations for Single Word

### 1. Frequency Pruning

```python
from collections import Counter

def exist(self, board, word):
    # Count characters in board
    board_count = Counter()
    for row in board:
        for ch in row:
            board_count[ch] += 1
    
    word_count = Counter(word)
    
    # Quick check: does board have enough of each character?
    for ch, count in word_count.items():
        if board_count[ch] < count:
            return False  # Impossible!
    
    # Optimization: if last char is rarer, reverse the word
    if board_count[word[0]] > board_count[word[-1]]:
        word = word[::-1]
    
    # ... proceed with backtracking
```

### 2. Early Termination

```python
def backtrack(r, c, idx):
    if idx == len(word):
        return True
    
    # Bounds + character check in one
    if not (0 <= r < rows and 0 <= c < cols) or board[r][c] != word[idx]:
        return False
    
    # ... rest of backtracking
```

---

## Complexity Analysis

| Aspect | Value |
|--------|:-----:|
| Time | O(M × N × 4^L) |
| Space | O(L) recursion stack |
| M, N | Grid dimensions |
| L | Word length |
| 4^L | 4 directions at each step |

```
  In practice, much better than O(4^L) because:
  - Character mismatches prune most branches immediately
  - Visited cell marking prevents revisiting
  - Average branching factor is closer to 3 (can't go back)
```

---

## Connection to Word Search II

```
  LeetCode 79 (this problem):  Search ONE word → Backtracking
  LeetCode 212 (next chapter): Search MANY words → Trie + Backtracking
  
  Skill progression:
  1. Master backtracking on grid (this chapter)
  2. Add trie to handle multiple words (next chapter)
  3. Add pruning optimizations (remove found words from trie)
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Algorithm | Backtracking with visited marking |
| Time | O(M × N × 4^L) worst case |
| Space | O(L) for recursion stack |
| Visited tracking | Modify board in-place (temp = board[r][c]) |
| Key optimization | Frequency check + reverse word if needed |
| Multiple words | Use Trie (→ Word Search II) |

---

## Quick Revision Questions

1. **Why do we modify the board in-place for visited tracking?**
   > To avoid creating a separate visited set (saves space). We temporarily replace the cell with '#', explore, then restore it. This is O(1) space per cell vs O(M×N) for a visited array.

2. **Why is the actual time better than O(M × N × 4^L)?**
   > Most branches are pruned early by character mismatches. The visited marking also prevents going back, reducing the branching factor from 4 to effectively 3. Short words search very quickly.

3. **When should you reverse the search word?**
   > When the first character appears more frequently in the grid than the last character. Starting from the rarer character means fewer starting points and faster pruning.

4. **Can the same cell be used twice in one path?**
   > No. Each cell can be used at most once per path. This is enforced by marking cells as visited during exploration.

5. **What's the key difference between Word Search I and II?**
   > Word Search I finds ONE word (backtracking suffices). Word Search II finds MULTIPLE words simultaneously using a trie to share prefix traversals and avoid redundant grid searches.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Prefix Matching](../04-Prefix-Based-Problems/06-prefix-matching.md) | [Next: Word Search II ▶](02-word-search-ii.md) |
