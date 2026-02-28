# Chapter 1: Visualizing Recursion

## Overview
Visualizing recursion as a **tree** is the most powerful technique for understanding how recursive algorithms work. Every recursive function call creates a node, and the recursive calls it makes become its children. This mental model helps you analyze time complexity, space complexity, and correctness.

---

## The Recursion Tree Concept

```
┌──────────────────────────────────────────────────────┐
│              RECURSION TREE                           │
│                                                      │
│  • Every function call = a NODE                      │
│  • Recursive calls from that function = CHILDREN     │
│  • Base case calls = LEAF nodes                      │
│  • The first call = ROOT node                        │
│                                                      │
│  f(n)                                                │
│  ├── f(n-1)    ← child (recursive call 1)            │
│  │   ├── f(n-2)                                      │
│  │   └── ...                                         │
│  └── f(n-2)    ← child (recursive call 2)            │
│      └── ...                                         │
└──────────────────────────────────────────────────────┘
```

---

## Classification by Shape

```
LINEAR RECURSION TREE            BINARY RECURSION TREE
(1 recursive call)              (2 recursive calls)

f(4)                              f(4)
│                                /     \
f(3)                           f(3)    f(2)
│                              / \      / \
f(2)                        f(2) f(1) f(1) f(0)
│                           / \
f(1)                     f(1) f(0)
│
f(0) ← leaf

Depth: n                   Depth: n
Nodes: n+1                 Nodes: up to 2^(n+1)-1
Examples:                  Examples:
  Factorial                  Fibonacci
  Linear search              Merge sort
  Array sum                  Subsequences


K-ARY RECURSION TREE
(k recursive calls)

        f(n)
    /   |   |   \
  f() f() f() f()        ← k children per node
  ...  ... ... ...

Depth: depends on reduction
Nodes: up to k^d where d = depth
Examples:
  Permutations (n branches)
  N-Queens (n branches)
```

---

## How to Draw a Recursion Tree

### Step-by-Step Process

```
1. Write the ROOT — the initial function call
2. For each recursive call in the function, draw a CHILD
3. Label each node with its PARAMETER values
4. Mark LEAF nodes (base cases) with their return values
5. Annotate INTERNAL nodes with work done at that level
6. Sum up work per level to get total time complexity
```

### Example: sum(arr, 0) for arr = [3, 1, 4, 2]

```
Linear recursion tree:

sum(arr, 0)  → 3 + 7 = 10
│  work: arr[0]=3
│
sum(arr, 1)  → 1 + 6 = 7
│  work: arr[1]=1
│
sum(arr, 2)  → 4 + 2 = 6
│  work: arr[2]=4
│
sum(arr, 3)  → return 2
│  work: arr[3]=2
│
sum(arr, 4)  → return 0  ← base case (leaf)

Total work: O(1) per node × (n+1) nodes = O(n)
```

---

## Anatomy of a Recursion Tree

```
                    f(input)          ← ROOT (Level 0)
                   /        \
           f(smaller1)   f(smaller2)  ← Level 1
           /     \         /    \
       f(...)  f(...)  f(...)  f(...) ← Level 2
       ...       ...    ...     ...
        ↓         ↓      ↓       ↓
     [leaves]  [leaves] [leaves] [leaves]  ← Level d (base cases)

Key measurements:
┌─────────────────────────────────────────────┐
│ DEPTH (d)    = longest path root → leaf     │
│              = determines SPACE complexity   │
│                                             │
│ BRANCHING (b) = max children per node       │
│               = how fast tree grows          │
│                                             │
│ TOTAL NODES  = sum of all calls             │
│              = determines TIME complexity    │
│                                             │
│ WORK PER NODE = time for non-recursive work │
│               = constant or depends on level│
└─────────────────────────────────────────────┘
```

---

## Reading Information from the Tree

```
Given a recursion tree, you can determine:

1. TIME COMPLEXITY  = (total nodes) × (work per node)
                    = Σ (nodes at each level × work per node)

2. SPACE COMPLEXITY = maximum depth + any auxiliary space
                    = O(depth)

3. NUMBER OF RESULTS = number of leaf nodes
                     = base case hits

4. DUPLICATE WORK   = repeated subtrees
                    = opportunity for memoization
```

---

## Visualization Tools

### Method 1: Indent-based (for tracing)
```
factorial(4)
  factorial(3)
    factorial(2)
      factorial(1) → 1
    → 2
  → 6
→ 24
```

### Method 2: Tree diagram (for analysis)
```
        f(4)
       /
     f(3)
    /
  f(2)
 /
f(1) → 1
```

### Method 3: Table trace
```
┌──────┬───────┬──────────┬────────┐
│ Call │ Input │ Subcalls │ Return │
├──────┼───────┼──────────┼────────┤
│  1   │ n=4   │ f(3)     │  24    │
│  2   │ n=3   │ f(2)     │   6    │
│  3   │ n=2   │ f(1)     │   2    │
│  4   │ n=1   │ none     │   1    │
└──────┴───────┴──────────┴────────┘
```

---

## Summary Table

| Tree Property | What It Tells You |
|--------------|-------------------|
| Total nodes | Time complexity |
| Max depth | Space complexity |
| Branching factor | Growth rate |
| Leaf count | Number of results/solutions |
| Repeated subtrees | Overlapping subproblems → DP |
| Level-by-level work | Master theorem analysis |

---

## Quick Revision Questions

1. **What does each node** in a recursion tree represent?
2. **How do you determine** space complexity from a recursion tree?
3. **What is the branching factor**, and how does it affect total nodes?
4. **Draw the recursion tree** for power(2, 8) using fast exponentiation.
5. **What do repeated subtrees** indicate about the algorithm?
6. **Why is the recursion tree** more useful than tracing individual calls?

---

[← Previous: String Compression](../04-Recursion-With-Strings/06-string-compression.md) | [Next: Fibonacci Recursion Tree →](02-fibonacci-recursion-tree.md)

[← Back to README](../README.md)
