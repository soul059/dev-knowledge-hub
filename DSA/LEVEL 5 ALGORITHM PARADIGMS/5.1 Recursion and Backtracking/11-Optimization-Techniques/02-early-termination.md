# Chapter 2: Early Termination

## Overview
**Early termination** stops the search as soon as the answer is found — or as soon as we know no better answer exists. While pruning cuts branches *before* exploring them, early termination cuts the entire remaining search *after* a condition is met. The two are complementary.

---

## Types of Early Termination

```
┌─────────────────────────────────────────────────────┐
│ 1. FIRST SOLUTION FOUND  → stop immediately         │
│ 2. SUFFICIENT SOLUTION   → meets quality threshold  │
│ 3. PROVEN OPTIMAL        → can't improve further    │
│ 4. INFEASIBILITY DETECTED→ no solution possible      │
│ 5. RESOURCE LIMIT HIT    → time/memory exhausted    │
└─────────────────────────────────────────────────────┘
```

---

## 1. Stop at First Solution

```
Many problems ask: "Does a solution EXIST?" or "Find ANY solution."
→ Return TRUE/the solution immediately, don't explore further.

WITHOUT early termination:
  FUNCTION solve(state):
      IF isGoal(state):
          solutions.add(state)       ← Keeps searching!
      FOR each choice:
          solve(nextState)           ← Explores EVERYTHING

WITH early termination:
  FUNCTION solve(state):
      IF isGoal(state):
          RETURN TRUE                ← STOP HERE!
      FOR each choice:
          IF solve(nextState):
              RETURN TRUE            ← Propagate stop upward!
      RETURN FALSE

The RETURN TRUE propagates up the call stack,
unwinding ALL recursive calls immediately.

Example: Sudoku solver
  → Only need ONE valid filling
  → Return TRUE as soon as board is complete
  → Saves exploring all 6.67×10²¹ possible fillings
```

---

## 2. Threshold-Based Termination

```
"Find a solution with cost ≤ K"
→ Stop when first qualifying solution is found.

FUNCTION solve(state, cost, threshold):
    IF isGoal(state):
        IF cost <= threshold:
            RETURN state             ← Good enough!
        RETURN NULL
    
    FOR each choice (sorted by promise):
        result = solve(nextState, newCost, threshold)
        IF result != NULL:
            RETURN result            ← Propagate up!
    
    RETURN NULL

Use case: Approximate solutions for NP-hard problems.
  Instead of finding THE best, find a "good enough" answer.
```

---

## 3. Proven Optimal Termination

```
"I found a solution with cost X. Can I prove no better exists?"

Technique: Compare current best with LOWER BOUND of remaining.

IF lowerBound(allRemaining) >= bestFoundSoFar:
    RETURN bestFoundSoFar           ← Proven optimal!

Example — Knapsack with branch and bound:
  Current best value = 100
  Fractional relaxation upper bound = 95
  → Since relaxation (best possible) < current best,
     95 < 100, remaining branches can't improve.
  → But wait, that means other branches beat this one.
  
  Correct logic:
  IF upperBound(this branch) <= bestSoFar:
      PRUNE this branch (not terminate entirely)
  
  Terminate entirely when:
      ALL remaining branches have upper bound ≤ bestSoFar

This is the core of BRANCH AND BOUND algorithms.
```

---

## 4. Infeasibility Detection

```
"Can this partial solution ever be completed?"

If we can PROVE no valid completion exists → backtrack immediately.

Example — Graph coloring:
  IF an uncolored vertex has no available colors:
      RETURN FALSE                   ← Dead end!
  
  Even stronger — forward checking:
  After coloring vertex v:
    FOR each uncolored neighbor u:
        Remove color(v) from u's domain
        IF u's domain is EMPTY:
            RETURN FALSE             ← u can't be colored!
            (Undo domain changes)

Example — Constraint propagation in Sudoku:
  After placing digit d in cell (r,c):
    Remove d from peers' candidates.
    IF any peer has 0 candidates:
        UNDO and backtrack immediately.
    IF any peer has 1 candidate:
        Place it (forced move) and propagate.
```

---

## 5. Resource-Limited Termination

```
Practical systems have finite time/memory.

FUNCTION solve(state, startTime, timeLimit):
    IF currentTime() - startTime > timeLimit:
        RETURN bestSoFar             ← Time's up!
    
    // ... normal backtracking ...

FUNCTION solve(state, nodesExplored, nodeLimit):
    nodesExplored += 1
    IF nodesExplored > nodeLimit:
        RETURN bestSoFar             ← Node budget exhausted!

Use in:
  • Competitive programming (time limits)
  • Game AI (move time limits)
  • Anytime algorithms (return best-so-far on interrupt)
```

---

## Implementation Patterns

```
Pattern 1: Boolean return with short-circuit
─────────────────────────────────────────────
  FUNCTION solve(state):
      IF isGoal(state): RETURN TRUE
      FOR each choice:
          make(choice)
          IF solve(nextState): RETURN TRUE   ← Short circuit!
          undo(choice)
      RETURN FALSE

Pattern 2: Global flag
─────────────────────────────────────────────
  found = FALSE
  
  FUNCTION solve(state):
      IF found: RETURN                       ← Global check
      IF isGoal(state):
          found = TRUE
          saveSolution(state)
          RETURN
      FOR each choice:
          IF found: RETURN                   ← Check again
          make(choice)
          solve(nextState)
          undo(choice)

Pattern 3: Exception/early exit
─────────────────────────────────────────────
  FUNCTION solve(state):
      IF isGoal(state):
          THROW SolutionFound(state)         ← Unwind all frames
      FOR each choice:
          make(choice)
          solve(nextState)
          undo(choice)
  
  TRY: solve(initialState)
  CATCH SolutionFound as e: RETURN e.state

Pattern 1 is cleanest. Pattern 2 for complex cases.
Pattern 3 rarely needed (language-dependent).
```

---

## Find-All vs Find-First Comparison

```
Find ALL solutions:
┌──────────────────────────────────────┐
│ FUNCTION solve(state):              │
│     IF isGoal(state):               │
│         results.add(copy(state))    │
│         RETURN  ← Don't return TRUE │
│     FOR each choice:                │
│         make(choice)                │
│         solve(nextState)  ← Always  │
│         undo(choice)                │
└──────────────────────────────────────┘

Find FIRST solution:
┌──────────────────────────────────────┐
│ FUNCTION solve(state):              │
│     IF isGoal(state):               │
│         RETURN TRUE                 │
│     FOR each choice:                │
│         make(choice)                │
│         IF solve(nextState):        │
│             RETURN TRUE ← Stop!    │
│         undo(choice)                │
│     RETURN FALSE                    │
└──────────────────────────────────────┘

Find BEST solution (optimization):
┌──────────────────────────────────────┐
│ FUNCTION solve(state, cost):        │
│     IF isGoal(state):               │
│         bestSoFar = min(bestSoFar,  │
│                         cost)       │
│         RETURN                      │
│     IF cost + lb >= bestSoFar:      │
│         RETURN  ← Prune + terminate │
│     FOR each choice:                │
│         make(choice)                │
│         solve(nextState, newCost)   │
│         undo(choice)                │
└──────────────────────────────────────┘
```

---

## Complexity Impact

```
┌────────────────────────┬──────────────────────────────┐
│ Scenario               │ Impact of Early Termination  │
├────────────────────────┼──────────────────────────────┤
│ Find first of many     │ From O(explore all) → O(1st) │
│ N-Queens (any sol.)    │ ~1000× faster than find-all  │
│ Sudoku                 │ Find 1 vs find all: huge     │
│ SAT (satisfiable)      │ Returns on first assignment  │
│ Optimization + tight   │ Prunes most branches via     │
│ bound                  │ proven optimality             │
│ No solution exists     │ No benefit (must check all)  │
└────────────────────────┴──────────────────────────────┘

Important: Early termination helps MOST when:
  1. Solutions exist and are plentiful
  2. We only need one (or a "good enough" one)
  3. Good ordering puts solutions near the top of the tree
```

---

## Summary Table

| Type | Condition | Action |
|------|-----------|--------|
| First found | Goal reached | Return TRUE up call stack |
| Threshold | Cost ≤ K | Return solution immediately |
| Proven optimal | Lower bound ≥ best | Stop all remaining branches |
| Infeasible | No valid completion | Backtrack immediately |
| Resource limit | Time/nodes exceeded | Return best-so-far |

---

## Quick Revision Questions

1. **How does `RETURN TRUE`** propagate to stop all recursion?
2. **What's the difference** between pruning and early termination?
3. **When does early termination** NOT help?
4. **How does forward checking** detect infeasibility early?
5. **What's the advantage** of boolean return over global flag?
6. **Why does choice ordering** amplify early termination's benefit?

---

[← Previous: Pruning Strategies](01-pruning-strategies.md) | [Next: Memoization in Backtracking →](03-memoization-in-backtracking.md)

[← Back to README](../README.md)
