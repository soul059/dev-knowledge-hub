# Chapter 6: When to Apply Backtracking

## 📋 Chapter Overview
Recognize when a problem requires backtracking and when a different approach is better.

---

## ✅ Signals for Backtracking

| Signal | Example |
|--------|---------|
| "Generate ALL solutions" | All subsets, all permutations |
| "Find ANY valid solution" | Sudoku solver, N-Queens |
| "Count solutions" (small n) | N-Queens count |
| Constraint satisfaction | Place items with rules |
| Decision at each step | Include/exclude, place digit |
| Small input size (n ≤ 15-20) | Exponential is acceptable |

---

## ❌ When NOT Backtracking

| Scenario | Better Approach |
|----------|----------------|
| Optimize (min/max) over subsets | DP (e.g., knapsack) |
| Count solutions (large n) | DP / combinatorics |
| Single optimal answer | Greedy / DP |
| Shortest path | BFS |
| n > 20 | Almost never pure backtracking |

---

## 🗺️ Decision Flowchart

```
  Need ALL solutions or ANY valid config?
  │
  ├── YES, n ≤ 15?
  │   └── Backtracking ✓
  │
  ├── YES, n ≤ 20 with good pruning?
  │   └── Backtracking ✓ (with careful pruning)
  │
  ├── Need COUNT of solutions?
  │   ├── n small → Backtracking
  │   └── n large → DP / Math
  │
  └── Need OPTIMAL value?
      └── DP or Greedy (not backtracking)
```

---

## 📊 Backtracking Problem Categories

| Category | Problems | Template |
|----------|----------|----------|
| Subsets/Combos | Subsets, CombSum | `start, path, result` |
| Permutations | Permutations, Next Perm | `used[], path, result` |
| Constraint | N-Queens, Sudoku | `find empty, try options` |
| Path in Grid | Word Search | `visited, 4-dir DFS` |
| Partitioning | Palindrome Partition | `start, check valid` |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| When | ALL solutions, ANY valid config, small n |
| When not | Optimization, large n, counting |
| Key indicator | Multiple choices at each step, need to explore all/any |
| Always | Add pruning to avoid TLE |
| Complexity | Exponential (2^n, n!, etc.) |

---

## ❓ Revision Questions

1. **"Find minimum cost subset" → backtracking?** → No, use DP (subset DP).
2. **"Generate all valid parentheses" → backtracking?** → Yes — generate all, prune invalid incrementally.
3. **n = 25 elements, generate all subsets → feasible?** → 2^25 ≈ 33M — borderline; depends on work per node.
4. **n = 10 permutations → feasible?** → 10! = 3.6M — yes.
5. **Backtracking vs DFS: what's the difference?** → Backtracking is DFS on a decision tree with explicit undo (un-choose) step. DFS is general graph/tree traversal.

---

[← Previous: Pruning Techniques](05-pruning-techniques.md)

[← Back to README](../README.md) | [Next Unit: Dynamic Programming →](../07-Dynamic-Programming-Patterns/01-dp-fundamentals.md)
