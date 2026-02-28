# Chapter 4: Tree Structure Analysis

## Overview
Learning to **read** a recursion tree — its shape, depth, branching factor, and node distribution — gives you a powerful tool for understanding algorithm behavior and deriving complexity.

---

## Key Properties of Recursion Trees

```
                        root ← Level 0 (1 node)
                       / | \
                      /  |  \
                     ●   ●   ● ← Level 1 (b nodes, b = branching factor)
                    /|\ /|\ /|\
                   ●●● ●●● ●●● ← Level 2 (b² nodes)
                   ...............
                   ← Level d (b^d nodes, d = depth)

Properties to analyze:
┌─────────────────────────────────────────────────┐
│  1. Branching Factor (b): children per node     │
│  2. Depth (d): longest root-to-leaf path        │
│  3. Nodes per Level: b^level (if uniform)       │
│  4. Total Nodes: Σ(b^i) for i=0..d             │
│  5. Shape: balanced? skewed? irregular?         │
│  6. Work per Node: constant? varies by level?   │
└─────────────────────────────────────────────────┘
```

---

## Common Tree Shapes

### Shape 1: Linear Chain (b=1)
```
f(n) → f(n-1) → f(n-2) → ... → f(0)

Depth: n
Nodes per level: 1
Total nodes: n+1
Shape: ● → ● → ● → ● → ● → ●

Examples: Factorial, Linear Search, Array Sum
Time: O(n)
```

### Shape 2: Binary Tree (b=2)
```
               f(n)
              /    \
          f(n-1)  f(n-2)
          /   \    /   \
        ...   ... ...  ...

Depth: n (for Fibonacci)
Nodes per level: up to 2^level
Total nodes: up to 2^(n+1) - 1
Shape: Full/almost full binary tree

Examples: Fibonacci, Subsets
Time: O(2^n)
```

### Shape 3: Balanced Binary (b=2, halving)
```
                f(n)
               /    \
          f(n/2)    f(n/2)
          /   \      /   \
      f(n/4) f(n/4) f(n/4) f(n/4)

Depth: log₂(n)
Nodes per level: 2^level
Total nodes: 2^(log n + 1) - 1 = 2n - 1
Shape: Perfect binary tree

Examples: Merge Sort
Time: O(n log n) [with O(n) merge work per level]
```

### Shape 4: k-ary Tree
```
                     f(n)
                 /  |  |  \
              f(?) f(?) f(?) f(?)
             /||\  /||\  ...

Depth: varies
Nodes per level: k^level
Total nodes: (k^(d+1) - 1) / (k - 1)

Examples: Permutations (n-ary), Combination Sum
Time: O(k^d) without pruning
```

### Shape 5: Decreasing-Width Tree
```
Level 0:  n choices     → n nodes
Level 1:  n-1 choices   → n(n-1) nodes
Level 2:  n-2 choices   → n(n-1)(n-2) nodes
...
Level n:  1 choice      → n! leaf nodes

Examples: Permutations (generating all n! arrangements)
Time: O(n × n!)
```

---

## Analyzing Work Distribution

```
The TOTAL WORK depends on where work is concentrated:

Case 1: LEAF-HEAVY (most work at bottom)
┌─────────────┐
│ ○            │  Little work
│ ○ ○          │
│ ○ ○ ○ ○      │
│ ●●●●●●●●    │  Most work at leaves
└─────────────┘
Total ≈ Work at leaves
Example: Subset enumeration, T(n) = 2T(n-1) + O(1) → O(2^n)

Case 2: ROOT-HEAVY (most work at top)
┌─────────────┐
│ ●●●●●●●●    │  Most work at root
│ ○ ○ ○ ○      │
│ ○ ○          │
│ ○            │  Little work at leaves
└─────────────┘
Total ≈ Work at root
Example: T(n) = 2T(n/2) + O(n²) → O(n²)

Case 3: UNIFORM (equal work per level)
┌─────────────┐
│ ●●●●●●●●    │
│ ●●●●●●●●    │  Same work each level
│ ●●●●●●●●    │
│ ●●●●●●●●    │
└─────────────┘
Total = Work per level × Number of levels
Example: Merge Sort, T(n) = 2T(n/2) + O(n) → O(n log n)
```

---

## Reading a Recursion Tree: Checklist

```
Given a recursive function, analyze:

Step 1: BRANCHING FACTOR
  □ How many recursive calls per invocation?
  □ Is branching constant or variable?

Step 2: DEPTH
  □ How does the parameter shrink? (n-1? n/2? n-k?)
  □ What is max depth before base case?

Step 3: WORK PER NODE
  □ What non-recursive work happens at each call?
  □ Does it vary by level? By input size?

Step 4: TOTAL WORK FORMULA
  □ Total = Σ (nodes at level i × work per node at level i)
  □ Identify dominant term (leaves? root? uniform?)

Step 5: SHAPE ANALYSIS
  □ Balanced or unbalanced?
  □ Can overlapping subproblems compress it?
```

---

## Worked Example: Analyzing mergeSort

```
FUNCTION mergeSort(arr, lo, hi):
    IF lo >= hi: RETURN
    mid = (lo + hi) / 2
    mergeSort(arr, lo, mid)       ← 2 recursive calls
    mergeSort(arr, mid+1, hi)     ← splitting in half
    merge(arr, lo, mid, hi)       ← O(n) work per call

Analysis:
┌──────────────────────────────────────────────┐
│ Branching factor: 2                          │
│ Size reduction: n → n/2 (halving)            │
│ Depth: log₂(n)                               │
│ Nodes at level i: 2^i                        │
│ Subproblem size at level i: n/2^i            │
│ Work per node at level i: O(n/2^i)           │
│                                              │
│ Work at level i = 2^i × (n/2^i) = n          │
│ → UNIFORM: O(n) work per level               │
│ → Total = n × log₂(n) = O(n log n)           │
└──────────────────────────────────────────────┘

Level 0: [●●●●●●●●] merge n items → work = n
Level 1: [●●●●][●●●●] merge n/2 each → work = n
Level 2: [●●][●●][●●][●●] merge n/4 each → work = n
Level 3: [●][●][●][●][●][●][●][●] → work = n
          ↑ base cases
Total: n × 4 levels = n × log₂(n)
```

---

## Summary Table

| Tree Shape | Branch | Depth | Total Nodes | Example | Time |
|-----------|--------|-------|-------------|---------|------|
| Linear | 1 | n | n | Factorial | O(n) |
| Binary (fixed) | 2 | n | 2^n | Subsets | O(2^n) |
| Binary (halving) | 2 | log n | ~2n | Merge Sort | O(n log n) |
| k-ary (halving) | k | log n | ~kn | k-way merge | O(kn log n) |
| Decreasing | n→1 | n | n! | Permutations | O(n!) |

---

## Quick Revision Questions

1. **What is the branching factor** of a Fibonacci recursion tree?
2. **Why is Merge Sort** O(n log n) despite having O(n) nodes?
3. **What makes a tree leaf-heavy** vs root-heavy?
4. **If branching factor is 3** and depth is n, what's the max total nodes?
5. **How does halving the input** change tree depth compared to decrementing?
6. **Draw and analyze** the recursion tree for T(n) = 3T(n/3) + O(n).

---

[← Previous: Identifying Overlapping Subproblems](03-identifying-overlapping-subproblems.md) | [Next: Complexity From Tree →](05-complexity-from-tree.md)

[← Back to README](../README.md)
