# Chapter 1: Pruning Strategies

## Overview
**Pruning** eliminates branches of the search tree that cannot lead to valid or optimal solutions. It transforms brute-force exponential search into practical backtracking. This chapter catalogs all major pruning techniques with examples, showing how each technique targets a different source of wasted computation.

---

## The Pruning Mindset

```
Full search tree (brute force):
          *
        / | \
       *  *  *
      /|  |  |\
     * * * * * *        ← Explore EVERYTHING
    ...............

Pruned search tree:
          *
        / | \
       *  ✗  *          ← Cut early
      /|     |
     * ✗     *          ← Cut deeper
    ...     ...

Goal: Same correct answers, FAR fewer nodes explored.
```

---

## Taxonomy of Pruning Strategies

```
┌─────────────────────────────────────────────────────────┐
│  1. CONSTRAINT PRUNING  ← Violates problem rules       │
│  2. BOUND PRUNING       ← Can't beat current best      │
│  3. SYMMETRY PRUNING    ← Equivalent to explored path  │
│  4. ORDERING PRUNING    ← Smart choice order → fail fast│
│  5. FEASIBILITY PRUNING ← Remaining resources can't win│
│  6. DOMINANCE PRUNING   ← Another option always better │
│  7. DUPLICATE PRUNING   ← Same state already visited   │
└─────────────────────────────────────────────────────────┘
```

---

## 1. Constraint Pruning

```
IDEA: Skip choices that immediately violate constraints.

Example — N-Queens:
  Before: try all n columns, check validity afterward
  After:  only try columns not in (cols ∪ diags ∪ antiDiags)

  IF col IN cols OR (row-col) IN diags OR (row+col) IN antiDiags:
      SKIP                    ← Don't even recurse

Example — Sudoku:
  Only try digits not already in row, column, or box.

Effect: Transforms O(n^n) → O(n!) for N-Queens
        (from trying all cells to only valid ones)
```

---

## 2. Bound Pruning (Branch and Bound)

```
IDEA: If current partial solution can't possibly 
      beat the best known solution, abandon it.

Example — Shortest path backtracking:
  bestSoFar = 50
  currentCost = 35
  remainingMinCost = 20   ← optimistic estimate
  35 + 20 = 55 > 50       → PRUNE (can't improve)

Implementation:
  IF currentCost + lowerBound(remaining) >= bestSoFar:
      RETURN                ← Abandon this branch

Key: The lower bound must be:
  1. ADMISSIBLE: never overestimate the true cost
  2. FAST to compute: O(1) or O(n), not O(exponential)
  
  Common lower bounds:
    • Greedy estimate
    • Relaxation (remove constraints, solve easier problem)
    • Linear programming relaxation
```

---

## 3. Symmetry Pruning

```
IDEA: If two branches produce equivalent results
      (rotations, reflections, relabeling), explore one.

Example — N-Queens symmetry:
  Board has 8 symmetries (4 rotations × 2 reflections).
  → Only place first queen in left half of first row.
  → Multiply valid solutions by 2 (or 8 for full symmetry).

Example — Subset sum:
  [1, 1, 1, 2] — three identical 1s.
  Don't generate [1a, 1b] and [1b, 1a] separately.
  → Sort + "skip duplicates at same level"

  FOR i = start TO n-1:
      IF i > start AND nums[i] == nums[i-1]:
          CONTINUE          ← Skip duplicate at same level
```

---

## 4. Ordering Pruning (Fail-First Principle)

```
IDEA: Try the most constrained choice first.
      If it fails, you fail EARLIER → less wasted work.

Example — Sudoku (MRV heuristic):
  Fill the cell with FEWEST valid digits first.
  Cell with 1 option: forced, no branching.
  Cell with 2 options: 2 branches (not 9).

Example — Graph coloring:
  Color the vertex with MOST neighbors first.
  → Most constrained → fails fastest if no coloring exists.

Value ordering (opposite):
  Try the MOST LIKELY value first.
  → Succeed earlier → less backtracking.

  Knight's Tour + Warnsdorff:
    Move to square with fewest onward moves.
    → Nearly eliminates backtracking.
```

---

## 5. Feasibility Pruning

```
IDEA: Check if remaining resources CAN complete the task.

Example — Combination sum:
  Target = 10, current sum = 7, remaining numbers = [1, 1, 1]
  Max possible = 7 + 3 = 10 → feasible, continue
  
  Target = 10, current sum = 7, remaining numbers = [1, 1]
  Max possible = 7 + 2 = 9 < 10 → PRUNE!

Example — K-combinations:
  Need k items, have chosen 2, index at position 7, n=10.
  Can pick at most n - 7 = 3 more items.
  Need k - 2 more. If k - 2 > 3 → PRUNE!

  FOR i = start TO n - (k - current.length) + 1:
                     ↑ upper bound pruning
```

---

## 6. Dominance Pruning

```
IDEA: If choice A is always at least as good as 
      choice B, never explore B.

Example — Coin change:
  If coin A > coin B and both valid, using A
  first covers more value → fewer coins → dominated.

Example — Job scheduling:
  If job A finishes before B and has more profit,
  any schedule with B but not A is dominated.

This requires problem-specific analysis to identify
dominance relationships between partial solutions.
```

---

## 7. Duplicate State Pruning (Memoization)

```
IDEA: If we reach the same state via different paths,
      only solve it once.

Example — Fibonacci recursion tree:
  fib(5) calls fib(3) from two different paths.
  → Cache result: memo[3] = 2

Example — Grid paths:
  Cell (2,3) reachable from (1,3)↓ and (2,2)→
  → Cache: paths[2][3] = k

When applicable:
  ✓ State fully described by small set of parameters
  ✓ Same state always gives same result
  ✗ State depends on entire path (e.g., exact path listing)
```

---

## Pruning Effectiveness by Problem

```
┌────────────────────┬─────────────────┬───────────────┐
│ Problem            │ Key Pruning     │ Speedup       │
├────────────────────┼─────────────────┼───────────────┤
│ N-Queens           │ Constraint      │ n^n → ~n!     │
│ Subset Sum         │ Bound + Sort    │ 2^n → varies  │
│ Sudoku             │ Constraint+MRV  │ 9^81 → ~1000  │
│ TSP                │ Bound (B&B)     │ n! → varies   │
│ Knight's Tour      │ Ordering (Warn) │ 8^64 → ~64    │
│ Subsets w/ dupes   │ Symmetry/Dedup  │ 2^n → 2^unique│
│ Combination Sum    │ Feasibility     │ ∞ → bounded   │
│ Graph Coloring     │ Fail-first      │ k^n → varies  │
└────────────────────┴─────────────────┴───────────────┘
```

---

## Implementing Pruning: Checklist

```
When designing a backtracking solution:

□ 1. Can I CHECK CONSTRAINTS before recursing?
      → Add IF statements before recursive call

□ 2. Can I BOUND the remaining cost/benefit?
      → Track best-so-far, compare with estimate

□ 3. Are there SYMMETRIC configurations?
      → Sort input, skip duplicates, fix first choice

□ 4. Can I ORDER choices to fail fast?
      → Most constrained first, most promising first

□ 5. Can I check FEASIBILITY of remaining state?
      → Verify enough resources to complete

□ 6. Do I revisit DUPLICATE states?
      → Add memoization/visited set

□ 7. Does any choice DOMINATE others?
      → Eliminate dominated options upfront
```

---

## Complexity Impact

```
┌──────────────────────────────────────────────┐
│ Without pruning: typically O(b^d)            │
│   b = branching factor, d = depth            │
│                                              │
│ With good pruning:                           │
│   Effective branching factor b' < b          │
│   Runtime: O(b'^d) where b' << b             │
│                                              │
│ Best case:                                   │
│   Pruning reduces b to 1 → O(d) linear!     │
│   (e.g., Warnsdorff on knight's tour)        │
│                                              │
│ Typical case:                                │
│   2-10× speedup per pruning technique        │
│   Multiple techniques compound multiplicatively│
└──────────────────────────────────────────────┘
```

---

## Summary Table

| Pruning Type | What It Cuts | When to Use |
|-------------|-------------|-------------|
| Constraint | Rule-violating branches | Always (first line of defense) |
| Bound | Suboptimal branches | Optimization problems |
| Symmetry | Equivalent configurations | Inputs with duplicates/symmetry |
| Ordering | Branches doomed to fail | High branching factor problems |
| Feasibility | Impossible completions | Resource-constrained problems |
| Dominance | Inferior alternatives | Choice comparison possible |
| Duplicate | Repeated states | Overlapping subproblems |

---

## Quick Revision Questions

1. **What is the difference** between constraint and bound pruning?
2. **Why does the fail-first** principle reduce backtracking?
3. **How does sorting** enable symmetry pruning?
4. **What makes a lower bound** admissible for bound pruning?
5. **When can duplicate pruning** (memoization) be applied?
6. **How do multiple pruning** techniques compound?

---

[← Previous: Unit 10 - Restore IP Addresses](../10-Advanced-Backtracking/06-restore-ip-addresses.md) | [Next: Early Termination →](02-early-termination.md)

[← Back to README](../README.md)
