# Chapter 5: When to Use Recursion

## Overview
Choosing between recursion and iteration isn't arbitrary. Certain problem structures **naturally lend themselves** to recursive solutions. This chapter teaches you to recognize those patterns and make informed decisions.

---

## The Decision Framework

```
┌────────────────────────────────────────────────────────┐
│              SHOULD I USE RECURSION?                    │
│                                                        │
│  Ask these questions about your problem:               │
│                                                        │
│  1. Can it be broken into SAME-TYPE subproblems? ──────┤► YES → Recursion candidate
│  2. Is the data structure RECURSIVE (tree/graph)? ─────┤► YES → Use recursion
│  3. Does it involve CHOICES + BACKTRACKING? ───────────┤► YES → Use recursion
│  4. Is it a simple linear scan? ───────────────────────┤► YES → Use iteration
│  5. Is stack depth a concern (n > 10000)? ─────────────┤► YES → Consider iteration
│  6. Need to generate ALL combinations? ────────────────┤► YES → Use recursion
└────────────────────────────────────────────────────────┘
```

---

## Pattern 1: Naturally Recursive Data Structures

When the **data itself** is recursive, use recursion:

```
TREES — Each node's children are also trees
┌─────────┐
│    5    │
│   / \   │
│  3   8  │ ← Left subtree is a tree, right subtree is a tree
│ / \   \ │
│1   4   9│
└─────────┘

function height(node):
    if node == null: return 0
    return 1 + max(height(node.left), height(node.right))
    // Each subtree is the SAME problem, just smaller

LINKED LISTS — Each node points to another list
┌───┬───┐   ┌───┬───┐   ┌───┬───┐
│ 1 │ ──┼──►│ 2 │ ──┼──►│ 3 │ / │
└───┴───┘   └───┴───┘   └───┴───┘

function length(node):
    if node == null: return 0
    return 1 + length(node.next)

DIRECTORIES — Folders contain subfolders
📁 root/
├── 📁 src/
│   ├── 📁 utils/
│   │   └── 📄 helper.js
│   └── 📄 main.js
└── 📁 docs/
    └── 📄 readme.md
```

---

## Pattern 2: Divide and Conquer

Problems that split into **independent subproblems**:

```
┌─────────────────────────────────┐
│     DIVIDE AND CONQUER          │
│                                 │
│  Problem(n)                     │
│    ├── Solve(n/2)  ← left half  │
│    ├── Solve(n/2)  ← right half │
│    └── Combine results          │
│                                 │
│  Examples:                      │
│  • Merge Sort                   │
│  • Quick Sort                   │
│  • Binary Search                │
│  • Closest pair of points       │
│  • Strassen's matrix multiply   │
└─────────────────────────────────┘

Merge Sort Visualization:
[38, 27, 43, 3, 9, 82, 10]
        /                \
[38, 27, 43, 3]    [9, 82, 10]
    /       \         /      \
[38, 27] [43, 3]  [9, 82]  [10]
 /   \    /   \    /   \      |
[38] [27][43] [3] [9] [82]  [10]
 \   /    \   /    \   /      |
[27, 38] [3, 43]  [9, 82]  [10]
    \       /         \      /
[3, 27, 38, 43]    [9, 10, 82]
        \                /
[3, 9, 10, 27, 38, 43, 82]
```

---

## Pattern 3: Explore All Possibilities

When you need to try **every combination**:

```
┌──────────────────────────────────────────┐
│     EXPLORATION / ENUMERATION            │
│                                          │
│  • Generate all subsets                  │
│  • Generate all permutations             │
│  • Find all paths in a graph             │
│  • Solve puzzles (Sudoku, N-Queens)      │
│  • Password cracking (all combos)        │
│                                          │
│  Key insight:                            │
│  At each step, you have CHOICES          │
│  Each choice leads to MORE choices       │
│  → This is a TREE of possibilities       │
│  → Recursion naturally walks this tree   │
└──────────────────────────────────────────┘

Subset generation for {a, b, c}:
                    {}
           /                  \
     include a            exclude a
        {a}                  {}
       /    \              /    \
    inc b  exc b       inc b  exc b
    {a,b}   {a}        {b}     {}
    / \    / \        / \    / \
  +c  -c +c  -c    +c  -c +c  -c
{a,b,c}{a,b}{a,c}{a}{b,c}{b}{c}  {}
```

---

## Pattern 4: Problems with Optimal Substructure

When the **optimal solution** contains optimal solutions to subproblems:

```
USE RECURSION (then optimize with DP):
• Shortest path problems
• Knapsack problem
• Longest common subsequence
• Matrix chain multiplication
• Edit distance

Optimal Substructure:
┌─────────────────────────────────────────┐
│ Best solution for problem of size n     │
│ = Best of:                              │
│   • [choice 1] + Best solution(n-1)     │
│   • [choice 2] + Best solution(n-2)     │
│   • ...                                 │
│                                         │
│ Start with recursion → Add memoization  │
│ → Convert to DP (bottom-up iteration)   │
└─────────────────────────────────────────┘
```

---

## When NOT to Use Recursion

```
┌────────────────────────────────────────────────────┐
│            AVOID RECURSION FOR:                     │
│                                                    │
│  ✗ Simple counting/accumulation                    │
│    sum = 0; for i in range(n): sum += i            │
│                                                    │
│  ✗ Linear array processing                        │
│    for item in array: process(item)                │
│                                                    │
│  ✗ When n can be very large (n > 10000)            │
│    Risk of stack overflow                          │
│                                                    │
│  ✗ When iterative solution is equally clear        │
│    Don't use recursion just to be "clever"         │
│                                                    │
│  ✗ Performance-critical tight loops                │
│    Function call overhead adds up                  │
│                                                    │
│  ✗ Simple state machines                           │
│    while loops with switch/if are clearer          │
└────────────────────────────────────────────────────┘
```

---

## Quick Reference: Problem → Approach

| Problem Type | Approach | Why |
|-------------|----------|-----|
| Tree traversal | Recursion | Data is recursive |
| Graph DFS | Recursion | Naturally nested exploration |
| Graph BFS | Iteration (queue) | Level-by-level processing |
| Array sum | Iteration | Simple accumulation |
| Sort (merge/quick) | Recursion | Divide and conquer |
| Sort (bubble/insertion) | Iteration | Simple comparison loops |
| All subsets | Recursion | Exponential branching |
| All permutations | Recursion | Branching choices |
| Fibonacci | Iteration (or memoized) | Avoid repeated work |
| Factorial | Either | Both equally clear |
| Binary search | Either | Recursion slightly cleaner |
| Linked list reverse | Either | Recursion short but tricky |
| Sudoku solving | Recursion + backtrack | Need to undo choices |
| Matrix operations | Iteration | Regular structure |

---

## The Recursion Readiness Checklist

```
Before writing a recursive solution, verify:

□ Can I clearly define what f(input) returns?
□ Can I identify the base case(s)?
□ Does the problem naturally decompose into smaller same-type problems?
□ Will the recursion depth be manageable (not too deep)?
□ Is the recursive solution clearer than iterative?
□ If overlapping subproblems exist, will I add memoization?
```

---

## Summary Table

| Signal | Use Recursion? | Example |
|--------|---------------|---------|
| Recursive data structure | Yes | Trees, graphs, nested lists |
| Divide and conquer | Yes | Merge sort, binary search |
| All combinations/permutations | Yes | Subsets, arrangements |
| Backtracking needed | Yes | Sudoku, N-Queens |
| Simple linear scan | No | Array sum, max finding |
| Very large input (n>10K) | Careful | Consider iterative + stack |
| Same-type subproblems | Yes | Most recursive problems |

---

## Quick Revision Questions

1. **List three data structures** that naturally suit recursive solutions.
2. **What is divide and conquer**, and why does it use recursion?
3. **Why should you avoid recursion** for very large inputs (n > 10000)?
4. **When generating all subsets**, why is recursion preferred over iteration?
5. **What is optimal substructure**, and how does it connect to recursion?
6. **Given a problem of finding the max element in an array**, would you use recursion or iteration? Why?

---

[← Previous: Recursion vs Iteration](04-recursion-vs-iteration.md) | [Next: Thinking Recursively →](06-thinking-recursively.md)

[← Back to README](../README.md)
