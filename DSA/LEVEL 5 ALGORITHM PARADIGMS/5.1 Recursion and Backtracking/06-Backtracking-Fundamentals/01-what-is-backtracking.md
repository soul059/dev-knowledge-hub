# Chapter 1: What Is Backtracking?

## Overview
Backtracking is a **systematic method for exploring all possible solutions** by building candidates incrementally and **abandoning ("backtracking")** a candidate as soon as it is determined that it cannot lead to a valid solution. It's recursion with a **"try, check, undo"** philosophy.

---

## The Core Idea

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  BRUTE FORCE: Try EVERYTHING, check at the end       │
│  BACKTRACKING: Try step-by-step, PRUNE early         │
│                                                      │
│  Think of it as:                                     │
│  "Exploring a maze — when you hit a dead end,        │
│   go BACK to the last fork and try another path"     │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### Maze Analogy
```
START → ? → ? → ? → GOAL
         ↓
    ┌─── A ───┐
    │         │
    B    ──── C ──── DEAD END ✗  ← backtrack!
    │
    D ──── E ──── GOAL ✓  ← found solution!
```

---

## Backtracking vs Plain Recursion

```
PLAIN RECURSION:                BACKTRACKING:
─────────────────              ──────────────────
• Breaks problem into          • Builds solution incrementally
  smaller subproblems          • Makes a CHOICE at each step
• Combines results             • CHECKS if choice is valid
• No "undoing"                 • UNDOES choice if it fails
• Always reaches base case     • May PRUNE entire branches

Examples:                       Examples:
  Factorial                      N-Queens
  Fibonacci                      Sudoku
  Merge Sort                     Maze solving
  Binary Search                  Subset generation
```

---

## The Three Pillars of Backtracking

```
┌────────────────────────────────────────────┐
│                                            │
│   1. CHOOSE    → Pick an option            │
│   2. EXPLORE   → Recurse with that choice  │
│   3. UNCHOOSE  → Undo the choice (backtrack│)
│                                            │
└────────────────────────────────────────────┘

Also known as:
  MAKE → CHECK → UNDO
  TRY  → RECURSE → REVERT
  PLACE → SOLVE → REMOVE
```

---

## Pseudocode Template

```
FUNCTION backtrack(state, choices):
    IF isGoal(state):
        recordSolution(state)
        RETURN
    
    FOR each choice in choices:
        IF isValid(choice, state):      ← PRUNE invalid
            makeChoice(choice, state)    ← CHOOSE
            backtrack(state, remaining)  ← EXPLORE
            undoChoice(choice, state)    ← UNCHOOSE
```

---

## Simple Example: Generate All Binary Strings of Length n

```
Problem: Generate all binary strings of length 3
Answer:  000, 001, 010, 011, 100, 101, 110, 111

FUNCTION generateBinary(n, current):
    IF length(current) == n:
        PRINT current
        RETURN
    
    // CHOOSE '0', EXPLORE, UNCHOOSE (implicit with new string)
    generateBinary(n, current + "0")
    
    // CHOOSE '1', EXPLORE, UNCHOOSE (implicit)
    generateBinary(n, current + "1")

Trace for n=3:
                    ""
                /        \
            "0"           "1"
           /   \         /   \
        "00"  "01"    "10"  "11"
        / \   / \     / \   / \
     000 001 010 011 100 101 110 111

8 solutions = 2³  ✓
```

---

## When to Use Backtracking

```
USE BACKTRACKING when:
┌──────────────────────────────────────────────────┐
│ ✓ You need to find ALL solutions (or count them) │
│ ✓ You need to find ANY valid solution            │
│ ✓ The problem has CONSTRAINTS to check           │
│ ✓ Solutions are built INCREMENTALLY              │
│ ✓ There are CHOICES at each step                 │
│ ✓ Wrong choices can be DETECTED early            │
└──────────────────────────────────────────────────┘

DO NOT use backtracking when:
┌──────────────────────────────────────────────────┐
│ ✗ A greedy approach works                        │
│ ✗ Dynamic programming applies (overlapping       │
│   subproblems with optimal substructure)         │
│ ✗ The problem can be solved in polynomial time   │
│   without exploring all possibilities            │
└──────────────────────────────────────────────────┘
```

---

## Classic Backtracking Problems

```
Category          Problems
─────────────────────────────────────────────
Constraint        N-Queens, Sudoku, Graph Coloring
Satisfaction

Combinatorial     Subsets, Permutations, Combinations,
Enumeration       Parentheses Generation

Path Finding      Maze Solving, Hamiltonian Path,
                  Knight's Tour

Partitioning      Palindrome Partitioning,
                  Equal Sum Partition

String            Word Break, Regular Expression
Matching          Matching, Wildcard Matching
```

---

## Summary Table

| Aspect | Brute Force | Backtracking |
|--------|------------|-------------|
| Strategy | Try everything | Try + prune |
| When to check | After complete solution | At each step |
| Efficiency | Always worst case | Often much better |
| Early termination | No | Yes (pruning) |
| Undo mechanism | Not needed | Essential |
| Typical problems | Simple enumeration | Constraint satisfaction |

---

## Quick Revision Questions

1. **What is the key difference** between backtracking and brute force?
2. **What are the three pillars** of backtracking?
3. **Why is "unchoose"** necessary in backtracking?
4. **Give an example** where backtracking is better than brute force.
5. **What types of problems** are best suited for backtracking?
6. **Write the general pseudocode** template for backtracking.

---

[← Previous Unit: Leading to DP](../05-Understanding-Recursion-Tree/06-leading-to-dp.md) | [Next: Backtracking Template →](02-backtracking-template.md)

[← Back to README](../README.md)
