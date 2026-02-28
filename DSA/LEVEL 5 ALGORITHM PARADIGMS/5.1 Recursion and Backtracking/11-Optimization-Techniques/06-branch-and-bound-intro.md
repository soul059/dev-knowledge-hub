# Chapter 6: Branch and Bound Introduction

## Overview
**Branch and Bound (B&B)** is a systematic optimization technique that extends backtracking with *bounding functions* to find the optimal solution without exploring the entire search tree. While backtracking prunes based on feasibility ("can this work?"), B&B prunes based on optimality ("can this be the BEST?"). It's the most powerful general-purpose technique for solving NP-hard optimization problems exactly.

---

## Backtracking vs Branch and Bound

```
BACKTRACKING:
  "Is this partial solution VALID?"
  → Prune if constraints violated
  → Finds ALL (or any) feasible solutions
  → No concept of "best"

BRANCH AND BOUND:
  "Can this partial solution lead to a BETTER solution?"
  → Prune if bound worse than best known
  → Finds the OPTIMAL solution
  → Maintains best-so-far + bounding function

┌────────────────────┬──────────────────────────────┐
│                    │     Feasible?                │
│                    │    YES         NO            │
├────────────────────┼──────────────┬───────────────┤
│ Can be optimal?    │              │               │
│   YES              │  EXPLORE     │  PRUNE (BT)   │
│   NO               │  PRUNE (B&B) │  PRUNE (both) │
└────────────────────┴──────────────┴───────────────┘
  B&B prunes the bottom-left cell that backtracking can't!
```

---

## The Three Components

```
1. BRANCHING: Split problem into subproblems
   (same as backtracking's "make choices")

2. BOUNDING: Compute bound on best achievable 
   solution from this partial state
   
   For MINIMIZATION: compute LOWER bound
     If lowerBound ≥ bestSoFar → prune
   
   For MAXIMIZATION: compute UPPER bound
     If upperBound ≤ bestSoFar → prune

3. BEST-SO-FAR: Track the best complete solution found
   (called the "incumbent")
   
   Initially: ∞ for min, -∞ for max
   Updated whenever a better complete solution is found.

The tighter the bound, the more branches pruned.
```

---

## General Framework

```
FUNCTION branchAndBound(problem):
    bestSoFar = INFINITY             ← (for minimization)
    bestSolution = NULL
    solve(initialState, bestSoFar, bestSolution)
    RETURN bestSolution

FUNCTION solve(state, bestSoFar, bestSolution):
    // Is this state a complete solution?
    IF isComplete(state):
        cost = evaluate(state)
        IF cost < bestSoFar:
            bestSoFar = cost
            bestSolution = copy(state)
        RETURN
    
    // BOUND: Can this branch beat the best?
    bound = lowerBound(state)
    IF bound >= bestSoFar:
        RETURN                       ← PRUNE! Can't improve.
    
    // BRANCH: Try each choice
    FOR each choice IN getChoices(state):
        make(choice)
        solve(newState, bestSoFar, bestSolution)
        undo(choice)
```

---

## Example: 0/1 Knapsack with B&B

```
Items: [(weight=2, value=40), (w=3, v=50), (w=5, v=100)]
Capacity: 7

Sort by value/weight ratio (descending):
  Item 0: 40/2 = 20.0
  Item 2: 100/5 = 20.0  
  Item 1: 50/3 = 16.7

Upper bound (fractional relaxation):
  Greedily fill remaining capacity with fractions of items.

                     (items included, capacity left)
                              ([], 7)
                        bound = 40+100+50×(0/3) = 150
                        best = -∞
                       /                    \
              Include item 0              Exclude item 0
              ([ i0], cap=5)               ([], cap=7)
              bound = 40+100+50×(0/3)=140   bound = 100+50×(2/3)=133.3
             /            \                /            \
        Include i2      Exclude i2    Include i2       ...
        ([i0,i2], cap=0) ([i0], cap=5) ([i2], cap=2)
        value = 140      bound=40+50×(5/3) value=100
        best = 140       =40+83.3=123.3   bound=100+50×(2/3)
        ✓ New best!      > 140? NO → PRUNE ✗ = 133 < 140 → PRUNE ✗

Answer: 140 (items 0 and 2)
```

---

## Common Bounding Functions

```
┌─────────────────┬──────────────────────────────────────┐
│ Problem         │ Lower/Upper Bound Technique          │
├─────────────────┼──────────────────────────────────────┤
│ Knapsack (max)  │ Fractional relaxation (upper bound)  │
│ TSP (min)       │ Min spanning tree of remaining nodes │
│ Assignment      │ Reduce cost matrix (row+col min)     │
│ Job scheduling  │ Greedy lower bound on makespan       │
│ Graph coloring  │ Clique number (lower bound on colors)│
│ Set cover       │ LP relaxation                        │
│ Integer prog.   │ LP relaxation                        │
└─────────────────┴──────────────────────────────────────┘

GOOD bounds are:
  1. TIGHT — close to actual optimal
  2. CHEAP — fast to compute (not another NP-hard problem)
  3. MONOTONE — get tighter as partial solution grows
```

---

## Example: TSP with B&B

```
Cities: 4 cities with distance matrix:
       A    B    C    D
  A  [ 0   10   15   20 ]
  B  [ 10   0   35   25 ]
  C  [ 15  35    0   30 ]
  D  [ 20  25   30    0 ]

Lower bound: sum of minimum edge from each unvisited city / 2
(MST-based bounds are tighter but more complex)

Starting from A:
                        A
                    /   |    \
                  B     C      D
                (10)  (15)   (20)
                / \    / \    / \
              C   D  B   D  B   C
              ...  ... ...  ... ...

When bound(partial path) ≥ bestTour → prune entire subtree.

With n=20 cities:
  Brute force: 19! ≈ 10^17 paths
  B&B with good bounds: typically ~10^6 nodes (practical!)
```

---

## Search Strategy Variants

```
B&B can explore the tree in different orders:

1. DFS (Depth-First Search):
   → Find complete solutions quickly
   → Update bestSoFar early → better pruning
   → Low memory: O(depth)
   → DEFAULT — most common for backtracking-style B&B

2. BFS (Breadth-First Search):
   → Explore level by level
   → High memory: O(branching^depth)
   → Rarely used alone

3. Best-First Search:
   → Expand node with BEST BOUND first
   → Most promising branches explored first
   → Uses priority queue
   → Finds optimal early (if bound is good)
   → Memory: can be large (stored nodes)

4. Least-Cost Search (LC-Search):
   → Expand node with lowest actual cost + bound
   → A* search is a special case
   → Optimal and efficient with admissible heuristic

┌────────────────┬──────────┬───────────┬──────────┐
│ Strategy       │ Memory   │ Speed     │ Best for │
├────────────────┼──────────┼───────────┼──────────┤
│ DFS            │ O(d)     │ Moderate  │ General  │
│ Best-first     │ O(large) │ Fastest   │ Tight bd │
│ Iterative deep │ O(d)     │ Moderate  │ Memory   │
└────────────────┴──────────┴───────────┴──────────┘
```

---

## Improving B&B Performance

```
1. START WITH A GOOD INCUMBENT:
   Run a greedy algorithm first to get a reasonable bestSoFar.
   Better starting point → more pruning from the start.

2. ORDER CHOICES BY PROMISE:
   Try most likely optimal choice first.
   → Find better solutions early → update bestSoFar → prune more.

3. USE TIGHTER BOUNDS:
   Invest more time computing bounds → prune more branches.
   Trade-off: bound computation time vs nodes saved.

4. COMBINE WITH HEURISTICS:
   At each node, run a local search heuristic.
   If it finds a solution better than bestSoFar → update.

5. PREPROCESS:
   Reduce problem size before B&B.
   Fix forced assignments, remove dominated options.
```

---

## B&B in the Backtracking Toolkit

```
Spectrum of search techniques:

  Brute Force → Backtracking → Branch and Bound → DP
  
  Brute Force:    Try everything
  Backtracking:   + feasibility pruning
  Branch & Bound: + optimality pruning
  DP:             + overlapping subproblems

When to use B&B:
  ✓ OPTIMIZATION problem (find min/max)
  ✓ Can compute a good BOUND function
  ✓ Problem is NP-hard (no polynomial algorithm)
  ✓ Need EXACT optimal (not approximate)
  ✓ Size is moderate (n ≤ 20-30 typically)

When NOT to use:
  ✗ Feasibility problem (use backtracking)
  ✗ Has optimal substructure (use DP)
  ✗ Approximate solution acceptable (use heuristics)
  ✗ Problem too large (use metaheuristics)
```

---

## Complexity

```
┌───────────────────────────────────────────────────┐
│ Worst case: same as backtracking (no pruning)     │
│   O(b^d) where b=branching, d=depth               │
│                                                   │
│ With good bounds:                                 │
│   Effective branching factor b' << b              │
│   In practice: explores tiny fraction of tree     │
│                                                   │
│ Space:                                            │
│   DFS: O(d) — same as backtracking                │
│   Best-first: O(nodes stored) — can be large      │
│                                                   │
│ Key insight:                                      │
│   Performance depends almost entirely on           │
│   BOUND QUALITY. Tight bounds → exponential speedup│
└───────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Purpose | Find optimal solution |
| Extension of | Backtracking |
| Key addition | Bounding function |
| Prune condition | bound ≥ bestSoFar (min) |
| Components | Branch + Bound + Best-so-far |
| Bound quality | Tight = fewer nodes, more pruning |
| Search order | DFS (default), best-first, BFS |
| Applications | Knapsack, TSP, assignment, scheduling |
| Limitation | Exponential worst case (NP-hard) |

---

## Quick Revision Questions

1. **What distinguishes** B&B from plain backtracking?
2. **What are the three** components of branch and bound?
3. **What makes a bounding function** "good"?
4. **Why search DFS** for B&B instead of BFS?
5. **How does a good starting** incumbent improve performance?
6. **When should you use** B&B vs DP vs heuristics?

---

[← Previous: Symmetry Breaking](05-symmetry-breaking.md) | [Back to Unit 1 →](../01-Recursion-Fundamentals/01-what-is-recursion.md)

[← Back to README](../README.md)
