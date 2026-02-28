# Chapter 5: Pruning Concept

## Overview
Pruning is what makes backtracking **dramatically faster** than brute force. By cutting off branches of the state space tree that **cannot lead to valid solutions**, we avoid exploring exponentially many dead-end paths.

---

## What Is Pruning?

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  PRUNING = Skipping entire subtrees when we can      │
│  PROVE they contain no valid solutions.              │
│                                                      │
│  Without pruning: Explore ALL 2^n or n! states       │
│  With pruning:    Explore only a FRACTION             │
│                                                      │
│  The smarter the pruning → the faster the algorithm  │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Pruning Visualized

```
FULL STATE SPACE TREE:           PRUNED TREE:

         ●                            ●
       / | \                        / | \
      ●  ●  ●                     ●  ✗  ●
     /|\ |  /|\                  /|\    /|\
    ● ● ● ● ● ● ●              ● ● ✗  ● ✗ ●
   ...(64 leaves)...           ...(only 12 explored)

  ✗ = pruned branch (never explored)

  64 leaves without pruning
  12 nodes explored with pruning
  → 81% of tree NEVER visited!
```

---

## Types of Pruning

### Type 1: Constraint Pruning (Feasibility)
```
"Is this choice even VALID?"

Example: N-Queens
  Before placing queen at (row, col):
  Check: Is column col already used?      → prune
  Check: Is diagonal already attacked?    → prune
  Check: Is anti-diagonal already used?   → prune

  IF any check fails → DON'T place, skip this choice

  FUNCTION isValid(row, col):
      RETURN col NOT IN usedCols
         AND (row-col) NOT IN usedDiags
         AND (row+col) NOT IN usedAntiDiags
```

### Type 2: Bound Pruning (Optimality)
```
"Can this branch POSSIBLY beat the best known solution?"

Example: Traveling Salesman
  Current path cost: 150
  Best complete tour found so far: 200
  Lower bound on remaining cost: 60
  
  150 + 60 = 210 > 200 → This branch CANNOT improve!
  → PRUNE entire subtree

  Also called: Branch and Bound
```

### Type 3: Symmetry Pruning
```
"Have I already explored an EQUIVALENT configuration?"

Example: Placing rooks on a chessboard
  Row [1,2,3,4] and row [2,1,3,4] might be equivalent
  by symmetry → only explore one

Example: N-Queens
  Solutions come in mirror pairs
  → Only explore first half, mirror the rest
```

### Type 4: Ordering Pruning
```
"Choose the most constrained variable first"

Example: Sudoku
  Instead of filling cells left-to-right:
  → Fill the cell with FEWEST possible values first
  
  Cell A: can be {1,3,7,9}      → 4 choices
  Cell B: can be {2,5}          → 2 choices  ← try this first!
  Cell C: can be {4}            → 1 choice   ← try this first!!
  
  Most Constrained Variable (MCV) heuristic
  Fail early = prune more!
```

---

## Pruning in Action: 4-Queens

```
Without pruning: try all 4⁴ = 256 placements

With constraint pruning:

Row 0: Try col 0
  Row 1: col 0 ✗(same col) col 1 ✗(diag) Try col 2
    Row 2: col 0 ✗(diag) col 1 ✗ col 2 ✗ col 3 ✗
    DEAD END → backtrack
  Row 1: Try col 3
    Row 2: Try col 1
      Row 3: col 0 ✗ col 1 ✗ col 2 ✗ col 3 ✗
      DEAD END → backtrack
    Row 2: col 2 ✗ col 3 ✗
    DEAD END → backtrack
  DEAD END → backtrack to row 0

Row 0: Try col 1
  Row 1: col 0 ✗ col 1 ✗ col 2 ✗ Try col 3
    Row 2: Try col 0
      Row 3: col 0 ✗ col 1 ✗ Try col 2  ✓✓✓ SOLUTION!
      
        . Q . .
        . . . Q
        Q . . .
        . . Q .

Nodes explored: ~44 (vs 256 without pruning)
```

---

## Measuring Pruning Effectiveness

```
Pruning Ratio = (Unpruned nodes - Explored nodes) / Unpruned nodes

Example: 8-Queens
  Total possible placements (no pruning): 8⁸ = 16,777,216
  With column constraint only:             8! = 40,320
  With full constraint pruning:            ~15,720 nodes explored
  
  Pruning ratio = (16,777,216 - 15,720) / 16,777,216 = 99.9%!
  
  Even better with smart ordering:
  With MCV heuristic: ~2,000 nodes explored → 99.99%!
```

---

## Implementing Pruning: Data Structures

```
For FAST constraint checking, maintain auxiliary data:

N-Queens:
  usedCols = set()        ← O(1) column check
  usedDiags = set()       ← O(1) diagonal check  (row-col)
  usedAntiDiags = set()   ← O(1) anti-diagonal   (row+col)

Sudoku:
  rowUsed[9][10] = bool   ← digit d used in row r?
  colUsed[9][10] = bool   ← digit d used in col c?
  boxUsed[9][10] = bool   ← digit d used in box b?

Graph Coloring:
  color[node] = int       ← current color assignment
  adjacency list          ← for neighbor checks
```

---

## Summary Table

| Pruning Type | Checks | When to Use | Savings |
|-------------|--------|-------------|---------|
| Constraint | Is choice valid? | Always | High |
| Bound | Can branch beat best? | Optimization | Medium-High |
| Symmetry | Equivalent explored? | Symmetric problems | Medium |
| Ordering | Most constrained first | Large search spaces | High |

---

## Quick Revision Questions

1. **What is the difference** between pruning and brute force?
2. **Give an example** of constraint pruning in N-Queens.
3. **What is bound pruning** and when is it used?
4. **How does ordering** (MCV heuristic) help with pruning?
5. **If a tree has 10^6 nodes** and pruning explores 10^3, what's the ratio?
6. **Why use sets** for constraint checking instead of arrays?

---

[← Previous: Make Choice, Explore, Undo](04-make-choice-explore-undo.md) | [Next: Backtracking vs Brute Force →](06-backtracking-vs-brute-force.md)

[← Back to README](../README.md)
