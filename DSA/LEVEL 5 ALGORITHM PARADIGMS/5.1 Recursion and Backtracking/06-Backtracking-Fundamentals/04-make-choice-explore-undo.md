# Chapter 4: Make Choice, Explore, Undo

## Overview
The **choose-explore-unchoose** pattern is the beating heart of backtracking. This chapter examines the mechanics of each phase in detail, common pitfalls, and how to implement the pattern correctly.

---

## The Three Phases

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│   PHASE 1: MAKE CHOICE                                 │
│   ──────────────────                                   │
│   • Modify the state to reflect the decision           │
│   • Mark resources as "used"                           │
│   • Add element to partial solution                    │
│                                                        │
│   PHASE 2: EXPLORE                                     │
│   ──────────────────                                   │
│   • Recurse with the modified state                    │
│   • The recursive call explores all possibilities      │
│     that follow this choice                            │
│                                                        │
│   PHASE 3: UNDO CHOICE                                 │
│   ──────────────────                                   │
│   • REVERSE the modification exactly                   │
│   • Unmark resources                                   │
│   • Remove element from partial solution               │
│   • State must be IDENTICAL to before Phase 1          │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Visualizing the Pattern

```
State before:  [A, B, _, _, _]   used = {A, B}

MAKE CHOICE:   Choose C
State:         [A, B, C, _, _]   used = {A, B, C}
                        ↑ added

EXPLORE:       backtrack(state)
               → tries all possibilities after [A,B,C,...]
               → finds: [A,B,C,D,E] ✓, [A,B,C,E,D] ✓

UNDO CHOICE:   Remove C
State:         [A, B, _, _, _]   used = {A, B}
                        ↑ removed

Now try next:  Choose D
State:         [A, B, D, _, _]   used = {A, B, D}
...
```

---

## Common Make/Undo Operations

```
┌───────────────────┬────────────────────┬──────────────────┐
│ Problem           │ MAKE Choice        │ UNDO Choice      │
├───────────────────┼────────────────────┼──────────────────┤
│ Subsets           │ list.add(elem)     │ list.removeLast()│
│ Permutations      │ swap(arr, i, j)    │ swap(arr, i, j)  │
│ N-Queens          │ board[r][c] = 'Q'  │ board[r][c] = '.'│
│ Sudoku            │ grid[r][c] = num   │ grid[r][c] = 0   │
│ Maze              │ visited[r][c]=true │ visited[r][c]=false│
│ Graph coloring    │ color[node] = c    │ color[node] = 0  │
│ Word Search       │ mark cell visited  │ unmark cell      │
│ Parentheses       │ append '(' or ')'  │ remove last char │
└───────────────────┴────────────────────┴──────────────────┘
```

---

## Detailed Example: Finding Paths in a Grid

```
Problem: Find all paths from (0,0) to (2,2) moving right or down

Grid:
  ┌───┬───┬───┐
  │ S │   │   │
  ├───┼───┼───┤
  │   │   │   │
  ├───┼───┼───┤
  │   │   │ G │
  └───┴───┴───┘

FUNCTION findPaths(r, c, path):
    // Base case: reached goal
    IF r == 2 AND c == 2:
        PRINT path
        RETURN
    
    // Try moving RIGHT
    IF c + 1 <= 2:
        path.add("R")              ← MAKE
        findPaths(r, c + 1, path)  ← EXPLORE
        path.removeLast()          ← UNDO
    
    // Try moving DOWN
    IF r + 1 <= 2:
        path.add("D")              ← MAKE
        findPaths(r + 1, c, path)  ← EXPLORE
        path.removeLast()          ← UNDO

Trace:
Call findPaths(0, 0, [])
  MAKE: path = [R]
  Call findPaths(0, 1, [R])
    MAKE: path = [R, R]
    Call findPaths(0, 2, [R, R])
      MAKE: path = [R, R, D]
      Call findPaths(1, 2, [R, R, D])
        MAKE: path = [R, R, D, D]
        Call findPaths(2, 2, [R, R, D, D])
          → PRINT "RRDD" ←  SOLUTION 1
        UNDO: path = [R, R, D]
      UNDO: path = [R, R]
    UNDO: path = [R]
    MAKE: path = [R, D]
    Call findPaths(1, 1, [R, D])
      ...continues finding "RDRD", "RDDR"
    UNDO: path = [R]
  UNDO: path = []
  ...continues finding "DRRD", "DRDR", "DDRR"

All 6 paths found: RRDD, RDRD, RDDR, DRRD, DRDR, DDRR
```

---

## Critical Rule: State Integrity

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  GOLDEN RULE: After UNDO, the state MUST be          │
│  EXACTLY the same as before MAKE.                    │
│                                                      │
│  State_before_make == State_after_undo               │
│                                                      │
│  If this is violated, subsequent branches will        │
│  explore WRONG states → WRONG solutions!             │
│                                                      │
└──────────────────────────────────────────────────────┘

Common State Integrity Violations:

1. FORGETTING TO UNDO
   list.add(x)
   backtrack(...)
   // Oops! forgot list.remove(x)
   → List keeps growing, wrong solutions

2. PARTIAL UNDO
   visited[r][c] = true
   board[r][c] = 'Q'
   backtrack(...)
   visited[r][c] = false
   // Oops! forgot board[r][c] = '.'
   → Board corrupted

3. WRONG UNDO ORDER
   a = old_a; b = old_b  // Must undo in reverse order
   // if MAKE was: a=new_a then b=new_b
   // UNDO should be: b=old_b then a=old_a
```

---

## Implicit vs Explicit Undo

```
EXPLICIT UNDO (modify state in place):
  path.add(x)          ← modify
  backtrack(path)
  path.removeLast()    ← undo
  
  Pro: Memory efficient (one shared state)
  Con: Must remember to undo

IMPLICIT UNDO (pass new state):
  backtrack(path + [x])  ← new copy created
  // No undo needed — original path unchanged
  
  Pro: No undo bugs possible
  Con: Creates new objects (more memory, slower)

RECOMMENDATION:
  Use EXPLICIT for performance-critical code
  Use IMPLICIT when learning or debugging
```

---

## Summary Table

| Phase | Purpose | Must Be | Common Bug |
|-------|---------|---------|------------|
| MAKE | Apply decision | Reversible | Forgetting what to undo |
| EXPLORE | Recurse deeper | After MAKE | Exploring without making |
| UNDO | Restore state | Exact reverse | Partial or missing undo |

---

## Quick Revision Questions

1. **Why must UNDO** produce the exact same state as before MAKE?
2. **What is the difference** between implicit and explicit undo?
3. **In the grid path example**, why add and remove from the path list?
4. **What happens** if you forget the UNDO step in N-Queens?
5. **For swap-based permutations**, why is swap its own inverse?
6. **Write the make/undo** for a Sudoku solver placing digit d at position (r,c).

---

[← Previous: State Space Tree](03-state-space-tree.md) | [Next: Pruning Concept →](05-pruning-concept.md)

[← Back to README](../README.md)
