# Chapter 5: Pruning Techniques

## 📋 Chapter Overview
Pruning = cutting branches of the decision tree early. Without pruning, backtracking is just brute force. Good pruning makes the difference between TLE and AC.

---

## 🧠 Why Pruning Matters

```
  Without pruning:          With pruning:
  
      *                         *
     /|\                       / \
    * * *                     *   *
   /|\ ...                  /     \
  * * * * * ...             *       *
  
  Explore 3^10 = 59049      Explore ~100 nodes
  nodes (or worse)
```

---

## 📝 Pruning Strategy 1: Constraint Propagation

**Skip choices that immediately violate constraints.**

```
  N-Queens: Don't try columns already occupied

  Without: try all n columns, check validity after
  With:    skip occupied columns → fewer recursive calls

  // Slow
  for col = 0 to n-1:
      board[row][col] = 'Q'
      if isValid(board): backtrack(row + 1)    // check AFTER placing
      board[row][col] = '.'

  // Fast
  for col = 0 to n-1:
      if col in usedCols: continue             // check BEFORE placing
      if (row-col) in usedDiag1: continue
      if (row+col) in usedDiag2: continue
      // ... place and recurse
```

---

## 📝 Pruning Strategy 2: Sort + Early Termination

**Sort candidates; stop when remaining candidates can't help.**

```
  Combination Sum: candidates=[2,3,6,7], target=5
  
  Sorted → [2, 3, 6, 7]
  
  When candidate > remaining target → BREAK (not just skip)
  
  for i = start to len(candidates)-1:
      if candidates[i] > target:
          break                    // all future candidates also too large
      ...
```

### Impact

```
  target=5, candidates=[2,3,6,7,8,9,10,11,12]
  
  Without break: try 6,7,8,9,10,11,12 (all fail) → 7 wasted calls
  With break: stop at 6 → 0 wasted calls
```

---

## 📝 Pruning Strategy 3: Upper/Lower Bound

**Estimate best possible outcome; abandon if it can't beat current best.**

```
  Knapsack Backtracking with Bound:
  
  At each node, compute upper bound =
    current_value + (remaining capacity × best value-per-weight)
  
  If upper_bound <= best_so_far → PRUNE this branch

  function backtrack(items, index, weight, value, capacity, best):
      upperBound = value + estimateRemaining(items, index, capacity - weight)
      if upperBound <= best: return        // can't improve → PRUNE
      ...
```

---

## 📝 Pruning Strategy 4: Symmetry Breaking

**If symmetric solutions exist, only explore one representative.**

```
  N-Queens: The board has rotational and reflective symmetry
  
  First queen can be placed in columns 0 to n/2-1 only
  (mirror solutions are equivalent)
  
  Reduces search by ~50%
  
  for col = 0 to (n-1)/2:    // instead of 0 to n-1
      // place first queen
      ...
```

---

## 📝 Pruning Strategy 5: Remaining Count Check

**If remaining elements can't fill the requirement, prune.**

```
  Combinations C(n, k):
  
  Need k - len(path) more elements
  Only n - i elements remaining
  
  If n - i < k - len(path): prune!
  
  for i = start to n - (k - len(path)) + 1:    // tighter bound
      ...
```

### Impact

```
  C(20, 18) — choose 18 from 20
  
  Without pruning: explore from i=0 to 19 at every level
  With pruning: at each level, loop only 2-3 options
  
  Massive reduction in branches explored
```

---

## 📝 Pruning Strategy 6: Duplicate Skipping

**Avoid generating the same combination/permutation twice.**

```
  Already covered in previous chapters:
  
  // Subsets/Combinations with duplicates
  sort(nums)
  if i > start AND nums[i] == nums[i-1]: continue
  
  // Permutations with duplicates
  sort(nums)
  if i > 0 AND nums[i] == nums[i-1] AND !used[i-1]: continue
```

---

## 📊 Pruning Techniques Summary

| Technique | When to Use | Savings |
|-----------|------------|---------|
| Constraint propagation | Constraint satisfaction (N-Queens, Sudoku) | Skip invalid placements |
| Sort + early break | Target sum problems | Cut all-too-large branch |
| Upper/lower bound | Optimization (knapsack, TSP) | Skip suboptimal branches |
| Symmetry breaking | Symmetric solution spaces | ~50% reduction |
| Remaining count | Fixed-size combinations | Skip impossible branches |
| Duplicate skipping | Input has duplicates | Remove redundant branches |

---

## 🔍 Example: Maximum Pruning in Word Search II

```
  Word Search II: Find all words from dictionary in grid.
  
  Pruning techniques used:
  1. Trie for prefix checking → if prefix not in trie, PRUNE
  2. Mark visited cells → don't revisit in one path
  3. Remove found words from trie → don't search again
  4. Prune trie branches with no remaining words
```

```
function findWords(board, words):
    trie = buildTrie(words)
    result = []
    for r = 0 to rows-1:
        for c = 0 to cols-1:
            dfs(board, r, c, trie.root, result)
    return result

function dfs(board, r, c, node, result):
    if out of bounds OR board[r][c] == '#': return
    
    char = board[r][c]
    if char not in node.children: return    // PRUNE: prefix not in trie
    
    nextNode = node.children[char]
    if nextNode.word:
        result.add(nextNode.word)
        nextNode.word = null               // PRUNE: don't find again
    
    board[r][c] = '#'                      // mark visited
    dfs in 4 directions with nextNode
    board[r][c] = char                     // unmark
    
    if nextNode.children is empty:
        delete node.children[char]         // PRUNE: trim trie
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Pruning = smart skipping | Check BEFORE recursing, not after |
| Sort first | Enables early break and duplicate skip |
| Bound estimation | Skip branches that can't improve result |
| Symmetry | Only explore one of each symmetric pair |
| Trie pruning | For multi-word search, trim as you go |
| Impact | Can reduce exponential to near-polynomial in practice |

---

## ❓ Revision Questions

1. **Why prune before recursing, not after?** → Pruning after means you've already done the work (recursive call). Pruning before avoids the call entirely.
2. **How does sorting enable pruning?** → Groups duplicates together (skip) and orders values (break when too large).
3. **What is bound-based pruning?** → Estimate the best possible outcome from this branch; if it can't beat the current best, stop.
4. **Symmetry breaking in N-Queens?** → Only place the first queen in columns [0, n/2); mirror gives the rest.
5. **How does Trie help in Word Search II?** → Provides O(1) prefix validation, letting you prune branches not matching any word prefix.

---

[← Previous: Constraint Satisfaction](04-constraint-satisfaction.md) | [Next: When to Apply →](06-when-to-apply.md)
