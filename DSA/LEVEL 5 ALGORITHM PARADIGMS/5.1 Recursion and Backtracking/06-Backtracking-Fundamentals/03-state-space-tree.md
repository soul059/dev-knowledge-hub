# Chapter 3: State Space Tree

## Overview
The **state space tree** is the conceptual tree of **all possible states** a backtracking algorithm can explore. Understanding this tree is essential for analyzing complexity, designing pruning strategies, and debugging backtracking solutions.

---

## What Is a State Space Tree?

```
┌──────────────────────────────────────────────────────┐
│  A State Space Tree represents:                       │
│                                                      │
│  • ROOT = Initial (empty) state                      │
│  • EDGES = Choices/decisions                         │
│  • NODES = Partial solutions                         │
│  • LEAVES = Complete solutions (or dead ends)        │
│  • INTERNAL NODES = Intermediate states              │
│                                                      │
│  Backtracking = Depth-First Search of this tree      │
│  with PRUNING of invalid branches                    │
└──────────────────────────────────────────────────────┘
```

---

## Example: Subsets of {1, 2, 3}

```
Decision at each level: Include or Exclude element?

                            {}
                     /              \
              {1}                     {}
            /      \              /       \
        {1,2}      {1}        {2}         {}
        /   \      / \       /  \        / \
   {1,2,3} {1,2} {1,3} {1} {2,3} {2}  {3}  {}
     ↑       ↑     ↑    ↑    ↑    ↑    ↑    ↑
   LEAF    LEAF   LEAF LEAF LEAF LEAF LEAF LEAF

8 leaves = 2³ = all possible subsets ✓

Tree properties:
  Depth: 3 (one level per element)
  Branching factor: 2 (include/exclude)
  Total leaves: 2³ = 8
  Total nodes: 2⁴ - 1 = 15
```

---

## Example: Permutations of {A, B, C}

```
Decision at each level: Which remaining element to place?

                           []
                     /     |      \
                  [A]     [B]     [C]
                 / \     / \      / \
             [AB] [AC] [BA] [BC] [CA] [CB]
              |    |    |    |    |    |
           [ABC][ACB][BAC][BCA][CAB][CBA]

6 leaves = 3! = all permutations ✓

Tree properties:
  Depth: 3
  Branching factor: 3 at level 0, 2 at level 1, 1 at level 2
  Total leaves: 3! = 6
  Total nodes: 1 + 3 + 6 + 6 = 16
```

---

## State Space Tree with Pruning (4-Queens)

```
Place queens row by row. Prune if queen is attacked.

Level 0 (row 0):    Try columns 0,1,2,3

     Q at col 0         Q at col 1        Q at col 2       Q at col 3
    [Q . . .]         [. Q . .]          [. . Q .]        [. . . Q]
       |                  |                  |                |
Level 1 (row 1):
  col0:✗ col1:✗       col0:✗ col1:✗     col0:Q            col0:Q
  col2:Q col3:Q       col2:✗ col3:Q     col1:✗            col1:Q
    |      |               |             col3:✗              |
    ↓      ↓               ↓                              ✗ all pruned
  ...    ...             ...

  ✗ = PRUNED (queen attacked, don't explore further)
  
Without pruning: 4⁴ = 256 nodes to check
With pruning: Only ~60 nodes explored
Savings: ~76% of nodes pruned!
```

---

## Terminology

```
┌────────────────────────────────────────────────────────┐
│ TERM              │ MEANING                            │
├───────────────────┼────────────────────────────────────┤
│ Live node         │ Generated but children not fully   │
│                   │ explored yet                       │
│ Dead node         │ All children explored or pruned    │
│ E-node           │ Currently being expanded            │
│ Solution node     │ Leaf representing valid solution   │
│ Answer node       │ Solution satisfying constraints    │
│ Bounding function │ Rule that decides if branch is     │
│                   │ worth exploring                    │
│ Feasible solution │ Satisfies all constraints          │
│ Pruned branch     │ Subtree skipped due to constraint  │
│ Backtrack point   │ Node where we return after pruning │
└───────────────────┴────────────────────────────────────┘
```

---

## Types of State Space Trees

```
TYPE 1: FIXED-LENGTH (Subset/Binary)
  Each level = decision about one element
  Branching factor = fixed (usually 2)
  Depth = n (number of elements)
  Leaves = 2^n
  
  Example: Subsets, Binary strings

TYPE 2: VARIABLE-LENGTH (Permutation)
  Each level = choose from remaining
  Branching factor = decreasing (n, n-1, ..., 1)
  Depth = n
  Leaves = n!
  
  Example: Permutations, Arrangements

TYPE 3: VARIABLE-DEPTH
  Depth varies per branch
  Some branches terminate early
  Branching may vary
  
  Example: Combination Sum (variable amounts),
           Maze paths (variable length)
```

---

## Reading the State Space Tree

```
Information you can extract:

1. SOLUTION COUNT
   Count the leaf nodes (answer nodes)
   
2. TIME COMPLEXITY
   Count total nodes visited (with pruning)
   Upper bound: total nodes without pruning
   
3. SPACE COMPLEXITY  
   Maximum depth × state size per node
   (Only one root-to-leaf path in memory at a time)
   
4. PRUNING EFFECTIVENESS
   (Total nodes - Visited nodes) / Total nodes × 100%
   
5. BRANCHING PATTERNS
   Which levels have most pruning?
   Which choices lead to most solutions?
```

---

## DFS Traversal of State Space Tree

```
The backtracking algorithm performs DFS:

Visit order (subset tree, n=3):

    1. {} 
    2. → {1}
    3. → → {1,2}
    4. → → → {1,2,3}  LEAF! Record. Backtrack.
    5. → → {1,2}       LEAF! Record. Backtrack.
    6. → {1,3}
    7. → → {1,3}       LEAF! Record. Backtrack.  
    8. → {1}            LEAF! Record. Backtrack.
    9. {2}
   10. → {2,3}
   ...and so on

Key: At each backtrack, we UNDO the last choice
and try the next sibling branch.
```

---

## Summary Table

| Tree Type | Branching | Depth | Leaves | Total Nodes | Example |
|-----------|----------|-------|--------|-------------|---------|
| Binary (subset) | 2 | n | 2^n | 2^(n+1)-1 | Subsets |
| Permutation | n,n-1,...,1 | n | n! | ~e·n! | Permutations |
| k-ary fixed | k | n | k^n | (k^(n+1)-1)/(k-1) | k-coloring |
| Variable | varies | varies | varies | varies | Combination Sum |

---

## Quick Revision Questions

1. **What does each level** of a state space tree represent?
2. **What is the difference** between a live node and a dead node?
3. **How many total nodes** are in a binary state space tree of depth 5?
4. **What is the branching pattern** for a permutation tree of n=4?
5. **How does DFS traversal** relate to backtracking?
6. **What percentage of nodes** can pruning eliminate in N-Queens?

---

[← Previous: Backtracking Template](02-backtracking-template.md) | [Next: Make Choice, Explore, Undo →](04-make-choice-explore-undo.md)

[← Back to README](../README.md)
