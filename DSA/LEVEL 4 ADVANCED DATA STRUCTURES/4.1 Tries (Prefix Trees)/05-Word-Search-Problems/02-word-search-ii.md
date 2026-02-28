# Word Search II (LeetCode 212)

## Overview
Given an m×n grid and a list of words, find all words that exist in the grid. This is the definitive trie + backtracking problem — you build a trie from the word list and explore the grid once, matching all words simultaneously.

---

## Problem Statement

```
  board = [['o','a','a','n'],
           ['e','t','a','e'],
           ['i','h','k','r'],
           ['i','f','l','v']]
  
  words = ["oath","pea","eat","rain"]
  
  Output: ["eat","oath"]
  
  "oath": o(0,0)→a(0,1)→t(1,1)→h(1,0) ✓
  "eat":  e(1,0)→a(0,1)→t(1,1)  — WAIT, wrong... 
          e(1,3)→a(1,2)→t(1,1) ✓
```

---

## Why Trie?

```
  Without trie: Search each word independently
  → 4 words × O(M×N×4^L) each = very slow
  
  With trie: ONE grid traversal finds ALL words
  → Build trie from words
  → At each grid cell, follow trie
  → Shared prefixes explored ONCE
  
  Example: words = ["oath", "oat", "oar"]
  All share prefix "oa" — trie explores "oa" path ONCE,
  then branches to "th", "t", "r"
```

---

## Solution

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = None  # Store complete word at terminal

class Solution:
    def findWords(self, board, words):
        # Build trie
        root = TrieNode()
        for word in words:
            node = root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.word = word  # Store word at end node
        
        rows, cols = len(board), len(board[0])
        result = []
        
        def backtrack(r, c, node):
            ch = board[r][c]
            
            if ch not in node.children:
                return  # This path doesn't match any word prefix
            
            next_node = node.children[ch]
            
            # Found a word!
            if next_node.word:
                result.append(next_node.word)
                next_node.word = None  # Avoid duplicates
            
            # Mark visited
            board[r][c] = '#'
            
            # Explore 4 directions
            for dr, dc in [(0,1),(0,-1),(1,0),(-1,0)]:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] != '#':
                    backtrack(nr, nc, next_node)
            
            # Restore
            board[r][c] = ch
            
            # Optimization: prune empty trie branches
            if not next_node.children:
                del node.children[ch]
        
        # Start backtracking from every cell
        for r in range(rows):
            for c in range(cols):
                backtrack(r, c, root)
        
        return result
```

---

## Step-by-Step Trace

```
  board = [['o','a','n'],
           ['e','t','a']]
  words = ["oat","eat","eta"]
  
  ── Build Trie ──
  
  root ── o ── a ── t [word="oat"]
       ├── e ── a ── t [word="eat"]
       │        └── t ── a [word="eta"]
  
  ── Grid Search ──
  
  Cell (0,0)='o':
    'o' in root.children? Yes → follow
    next_node has word? No
    Mark (0,0)='#'
    
    Explore neighbors:
    (0,1)='a': 'a' in o.children? Yes → follow
      next_node has word? No
      Mark (0,1)='#'
      
      Explore neighbors:
      (1,1)='t': 't' in a.children? Yes → follow
        next_node.word = "oat" → ADD TO RESULT! Set word=None
        Mark (1,1)='#'
        
        No more matching children → backtrack
        Restore (1,1)='t'
        a.children now has no 't' children → prune 't' from trie
      
      Restore (0,1)='a'
    Restore (0,0)='o'
  
  Cell (1,0)='e':
    'e' in root.children? Yes → follow
    Mark (1,0)='#'
    
    (0,0)='o': 'o' not in e.children → skip
    (1,1)='t': 't' not in e.children → skip
    
    ... no 'a' neighbor adjacent → dead end
    Restore (1,0)='e'
  
  Cell (1,1)='t': 't' not in root → skip
  
  ... continue for remaining cells...
  
  Result: ["oat"]
```

---

## Key Optimizations

### 1. Store Word at Terminal (Avoid Path Reconstruction)

```python
# Instead of:
node.is_end = True
# Then reconstruct word from path during DFS

# Do this:
node.word = word
# Direct access: result.append(next_node.word)
```

### 2. Remove Found Words (Avoid Duplicates)

```python
if next_node.word:
    result.append(next_node.word)
    next_node.word = None  # ← Don't find "oath" twice!
```

### 3. Prune Empty Trie Branches

```python
# After backtracking, if a trie node has no children left:
if not next_node.children:
    del node.children[ch]  # ← Remove dead branch
```

```
  Why prune?
  
  After finding all words with prefix "oat":
  Trie branch o→a→t has no more words
  → Delete it so future grid cells don't explore dead paths
  
  Massive speedup for grids with many starting positions!
```

### 4. Character Count Pre-check

```python
from collections import Counter

def findWords(self, board, words):
    board_count = Counter()
    for row in board:
        for ch in row:
            board_count[ch] += 1
    
    # Filter words that can't possibly exist
    valid_words = []
    for word in words:
        word_count = Counter(word)
        if all(board_count[ch] >= count for ch, count in word_count.items()):
            valid_words.append(word)
    
    # Build trie from valid_words only
    # ... rest of solution
```

---

## Complexity Analysis

| Aspect | Value |
|--------|:-----:|
| Build trie | O(W × L) |
| Grid search | O(M × N × 4^L) |
| Total | O(W×L + M×N×4^L) |
| Space | O(W × L) for trie |

Where: W = number of words, L = max word length, M×N = grid size

```
  With pruning optimization:
  - As words are found, trie branches are removed
  - Search progressively gets faster
  - In practice, MUCH faster than worst case
```

---

## Common Mistakes

```
  1. Forgetting to handle duplicates
     ✗ Same word found via different paths → added twice
     ✓ Set word=None after first find (or use a set for results)
  
  2. Not restoring board cell
     ✗ board[r][c] = '#' but forgetting to restore
     → Subsequent searches can't visit this cell
  
  3. Not pruning trie
     ✗ Searching blind trie branches after all their words found
     → Much slower on large inputs
  
  4. Checking is_end instead of storing word
     ✗ Must reconstruct the word from the path
     → Error-prone and verbose
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Pattern | Trie + Grid Backtracking |
| Build | Insert all words into trie |
| Search | DFS from every cell, following trie |
| Word found | `node.word` is not None |
| Dedup | Set `node.word = None` after finding |
| Pruning | Delete empty trie branches after backtrack |
| Time | O(M×N×4^L) with pruning improvements |

---

## Quick Revision Questions

1. **Why is a trie better than running Word Search I multiple times?**
   > Trie shares prefix exploration. All words with the same prefix are explored in a single grid traversal. Without a trie, each word requires a separate full grid search.

2. **Why store the complete word at terminal nodes?**
   > To avoid maintaining a path list and reconstructing the word during DFS. When `node.word` is set, we can directly add it to results.

3. **How does trie pruning help?**
   > After finding all words in a trie branch, we delete that branch. Future grid cells skip this branch entirely, reducing unnecessary exploration.

4. **How do we prevent finding the same word twice?**
   > Set `node.word = None` after the first find. Since the word is only stored at one terminal node, nullifying it prevents duplicate additions.

5. **What's the worst-case scenario for this algorithm?**
   > A grid filled with the same character (e.g., all 'a's) with words like "aaa", "aaaa", etc. Every cell matches, causing maximum branching. Pruning helps significantly here.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Word Search in Grid](01-word-search-in-grid.md) | [Next: Add and Search Word ▶](03-add-and-search-word.md) |
