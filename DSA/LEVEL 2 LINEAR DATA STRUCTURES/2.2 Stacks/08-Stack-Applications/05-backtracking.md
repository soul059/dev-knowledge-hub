# Chapter 5: Backtracking with Stack

[← Previous: Syntax Parsing](04-syntax-parsing.md) | [Next: Memory Stack →](06-memory-stack.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

---

## Overview

**Backtracking** systematically explores possibilities and "backtracks" upon hitting dead ends. While often implemented with recursion (which uses the call stack implicitly), backtracking can be explicitly modeled with a stack. The stack stores the current path, and popping "undoes" the last decision.

---

## How Backtracking Uses Stacks

```
┌──────────────────────────────────────────────────────────┐
│  BACKTRACKING = Explore + Undo + Retry                   │
│                                                          │
│  Stack = Current Path of Decisions                       │
│                                                          │
│  ┌──────────┐                                           │
│  │ Choice 3 │ ← most recent decision                    │
│  │ Choice 2 │                                           │
│  │ Choice 1 │ ← first decision                          │
│  └──────────┘                                           │
│                                                          │
│  Dead end? POP last choice (undo it)                    │
│  Try next alternative for that position.                 │
│  No alternatives? POP again (backtrack further)          │
│  Solution found? Record it, continue or stop.           │
└──────────────────────────────────────────────────────────┘
```

---

## Example 1: Maze Solving with DFS

```
Maze (0 = path, 1 = wall, S = start, E = end):

  ┌───┬───┬───┬───┬───┐
  │ S │   │ 1 │   │   │
  ├───┼───┼───┼───┼───┤
  │ 1 │   │ 1 │   │ 1 │
  ├───┼───┼───┼───┼───┤
  │   │   │   │   │   │
  ├───┼───┼───┼───┼───┤
  │ 1 │ 1 │   │ 1 │   │
  ├───┼───┼───┼───┼───┤
  │   │   │   │ 1 │ E │
  └───┴───┴───┴───┴───┘
```

### Stack-Based DFS Algorithm

```
FUNCTION solveMaze(maze, start, end):
    stack ← empty stack
    visited ← empty set
    stack.push((start, [start]))    // (position, path)
    
    WHILE stack NOT empty:
        (pos, path) ← stack.pop()
        
        IF pos == end:
            RETURN path    // Found solution!
        
        IF pos in visited:
            CONTINUE
        visited.add(pos)
        
        // Try all 4 directions
        FOR each neighbor (up, down, left, right) of pos:
            IF neighbor is valid AND not wall AND not visited:
                stack.push((neighbor, path + [neighbor]))
    
    RETURN null    // No path exists
```

### Trace (simplified):

```
Push (0,0)
Pop (0,0): explore neighbors
  Push (0,1)     (right)
  [can't go down — wall]

Pop (0,1): explore neighbors
  Push (1,1)     (down)
  [can't go right — wall]

Pop (1,1): explore neighbors
  Push (2,1)     (down)
  Push (1,1) already visited scenarios...

...eventually reaches (4,4) = END

Path found: (0,0)→(0,1)→(1,1)→(2,1)→(2,2)→(2,3)→(2,4)→...→(4,4)
```

---

## Example 2: N-Queens (Iterative Backtracking)

```
Place N queens on N×N board so no two attack each other.

Stack stores: column placement for each row.

FUNCTION solveNQueens(n):
    stack ← empty stack    // Stack of (row, col) placements
    row ← 0
    col ← 0
    solutions ← []
    
    WHILE row >= 0:
        placed ← false
        
        WHILE col < n:
            IF isSafe(stack, row, col):
                stack.push((row, col))
                
                IF row == n-1:
                    solutions.add(copy(stack))   // Found solution!
                    stack.pop()                  // Continue searching
                    col ← col + 1
                ELSE:
                    row ← row + 1
                    col ← 0
                    placed ← true
                    BREAK
            col ← col + 1
        
        IF NOT placed:
            // Backtrack!
            IF stack is empty:
                BREAK
            (prevRow, prevCol) ← stack.pop()
            row ← prevRow
            col ← prevCol + 1    // Try next column
    
    RETURN solutions
```

### 4-Queens Trace:

```
Try (0,0): safe ✓ → push
  Try (1,0): unsafe (same col)
  Try (1,1): unsafe (diagonal)
  Try (1,2): safe ✓ → push
    Try (2,0): unsafe
    Try (2,1): unsafe
    Try (2,2): unsafe
    Try (2,3): unsafe
    Backtrack! Pop (1,2), try col 3
  Try (1,3): safe ✓ → push
    Try (2,0): unsafe
    Try (2,1): safe ✓ → push
      Try (3,0): unsafe
      Try (3,1): unsafe
      Try (3,2): unsafe
      Try (3,3): unsafe
      Backtrack! Pop (2,1)
    Try (2,2): unsafe
    Try (2,3): unsafe
    Backtrack! Pop (1,3)
  No more cols for row 1. Backtrack! Pop (0,0)

Try (0,1): safe ✓ → push
  Try (1,3): safe ✓ → push
    Try (2,0): safe ✓ → push
      Try (3,2): safe ✓ → push
      SOLUTION FOUND! [(0,1), (1,3), (2,0), (3,2)]
      
  . Q . .
  . . . Q
  Q . . .
  . . Q .
```

---

## Example 3: Generate Parentheses

```
Generate all valid combinations of n pairs of parentheses.

FUNCTION generateParentheses(n):
    stack ← empty stack
    results ← []
    // State: (current_string, open_count, close_count)
    stack.push(("", 0, 0))
    
    WHILE stack NOT empty:
        (s, open, close) ← stack.pop()
        
        IF length(s) == 2*n:
            results.add(s)
            CONTINUE
        
        // Try adding ')'
        IF close < open:
            stack.push((s + ")", open, close + 1))
        
        // Try adding '('
        IF open < n:
            stack.push((s + "(", open + 1, close))
    
    RETURN results
```

---

## Recursion vs Explicit Stack

```
┌──────────────────────┬──────────────────────────────────┐
│ Recursive             │ Iterative (Explicit Stack)       │
├──────────────────────┼──────────────────────────────────┤
│ Shorter code          │ More verbose                     │
│ System call stack     │ Heap-allocated stack             │
│ Risk of stack overflow│ Can handle deeper exploration    │
│ Implicit backtracking │ Explicit push/pop                │
│ Easier to understand  │ More control over traversal      │
│ Harder to debug stack │ Easy to inspect stack state      │
└──────────────────────┴──────────────────────────────────┘

Convert recursion to stack when:
  1. Recursion depth may exceed system limits
  2. You need to pause/resume exploration
  3. You need explicit access to the decision path
```

---

## The Backtracking Template (Stack Version)

```
FUNCTION backtrack(problem):
    stack ← empty stack
    stack.push(initial_state)
    
    WHILE stack NOT empty:
        state ← stack.pop()
        
        IF state is solution:
            record(state)
            CONTINUE (or RETURN for first solution)
        
        IF state is invalid:
            CONTINUE    // Prune — don't explore further
        
        // Push next states (choices) onto stack
        FOR each choice from state:
            stack.push(state + choice)
    
    RETURN results
```

---

## Complexity Analysis

| Problem | Time | Stack Depth |
|---------|------|-------------|
| Maze DFS | O(V + E) | O(V) |
| N-Queens | O(N!) | O(N) |
| Generate Parens | O(4^n / √n) | O(2n) |
| General backtracking | O(b^d) | O(d) |

Where b = branching factor, d = depth.

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Backtracking = ** | Explore, dead end, undo, retry |
| **Stack stores** | Current path / sequence of decisions |
| **Push** | Make a choice, go deeper |
| **Pop** | Undo last choice (backtrack) |
| **Advantage over recursion** | No stack overflow, explicit state |
| **Applications** | Maze, N-Queens, permutations, constraint satisfaction |

---

## Quick Revision Questions

1. **How does a stack model backtracking?**
   > The stack stores the current path of decisions. Pushing = making a choice. Popping = undoing the last choice and trying alternatives.

2. **When should you use an explicit stack instead of recursion?**
   > When depth might cause stack overflow, when you need to pause/resume, or when you need to inspect the current decision path.

3. **What is "pruning" in backtracking?**
   > Skipping entire branches of exploration when we can determine they won't lead to a valid solution (checking `is_invalid` early).

4. **In maze solving, what does the stack hold?**
   > Current position and the path taken to reach it. Popping backtracks to the previous position.

5. **What is the relationship between DFS and backtracking?**
   > Backtracking IS essentially DFS on the decision tree. Both use LIFO (stack) ordering to explore depth-first and backtrack when needed.

---

[← Previous: Syntax Parsing](04-syntax-parsing.md) | [Next: Memory Stack →](06-memory-stack.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)
