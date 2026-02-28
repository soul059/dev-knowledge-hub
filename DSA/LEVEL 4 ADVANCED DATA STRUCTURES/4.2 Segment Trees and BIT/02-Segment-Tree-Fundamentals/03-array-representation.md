# Array Representation

## Chapter Overview

Segment trees are most commonly stored as **flat arrays** rather than linked node structures. This chapter explains how a binary tree maps to an array, with indexing schemes and their tradeoffs.

---

## The Mapping

A binary tree can be stored in an array where parent-child relationships are computed using arithmetic:

```
Tree view:                    Array view (1-indexed):

         1                    Index: [1] [2] [3] [4] [5] [6] [7] [8] [9] [10] [11]
        / \                   Value: [22] [8] [14] [3] [5] [7] [7] [2] [1] [5]  [3]
       2   3
      / \ / \                 Index 1 = root
     4  5 6  7                Index 2,3 = level 1
    /\ 
   8  9  10 11                Index 4-7 = level 2
                              Index 8-11 = level 3 (leaves in this case)
```

---

## 1-Indexed vs 0-Indexed

### 1-Indexed (Most Common)

```
Root at index 1

Node i:
  Left child:  2 * i
  Right child: 2 * i + 1
  Parent:      i / 2

Example:
  Node 3 → Left: 6, Right: 7, Parent: 1
  Node 5 → Left: 10, Right: 11, Parent: 2

          1
         / \
        2   3
       / \ / \
      4  5 6  7
     /\  /\
    8 9 10 11
```

### 0-Indexed (Alternative)

```
Root at index 0

Node i:
  Left child:  2 * i + 1
  Right child: 2 * i + 2
  Parent:      (i - 1) / 2

Example:
  Node 2 → Left: 5, Right: 6, Parent: 0
  Node 4 → Left: 9, Right: 10, Parent: 1

          0
         / \
        1   2
       / \ / \
      3  4 5  6
     /\  /\
    7 8 9 10
```

### Comparison

```
┌──────────────────┬─────────────────┬─────────────────┐
│                  │  1-Indexed      │  0-Indexed      │
├──────────────────┼─────────────────┼─────────────────┤
│  Root            │  tree[1]        │  tree[0]        │
│  Left child      │  2*i            │  2*i + 1        │
│  Right child     │  2*i + 1        │  2*i + 2        │
│  Parent          │  i/2            │  (i-1)/2        │
│  Bit trick       │  i<<1, i<<1|1   │  More complex   │
│  Wasted space    │  tree[0] unused │  None           │
│  Common in CP    │  YES ✓          │  Less common    │
└──────────────────┴─────────────────┴─────────────────┘

Recommendation: Use 1-indexed for simplicity and speed.
```

---

## Memory Layout

```
Array A = [2, 1, 5, 3] (n=4)

Segment tree (1-indexed, sum):

Tree:          [0-3]=11
              /        \
         [0-1]=3     [2-3]=8
         /    \      /    \
       [0]=2 [1]=1 [2]=5 [3]=3

Array: tree[0..8]
Index:   0    1     2     3     4    5    6    7
Value: [--] [11]  [3]   [8]   [2]  [1]  [5]  [3]
        ↑    ↑     ↑     ↑     ↑    ↑    ↑    ↑
      unused root  L1    L1    L2   L2   L2   L2
                   left  right             (leaves)

Leaves are at indices n to 2n-1 (for n = power of 2):
  Leaf for A[0] → tree[4]
  Leaf for A[1] → tree[5]
  Leaf for A[2] → tree[6]
  Leaf for A[3] → tree[7]
  
  General: Leaf for A[i] → tree[n + i]  (when n is power of 2)
```

---

## Building from Array

The array representation determines how we build the tree:

```
Given A = [2, 1, 5, 3]:

Step 1: Place leaves
  tree[4] = A[0] = 2
  tree[5] = A[1] = 1
  tree[6] = A[2] = 5
  tree[7] = A[3] = 3

Step 2: Build internal nodes (bottom-up)
  tree[3] = tree[6] + tree[7] = 5 + 3 = 8
  tree[2] = tree[4] + tree[5] = 2 + 1 = 3
  tree[1] = tree[2] + tree[3] = 3 + 8 = 11

Final: tree = [_, 11, 3, 8, 2, 1, 5, 3]
```

---

## Recursive Approach with Node Index

In the recursive approach, each function call carries:
- `node`: current position in the tree array
- `start, end`: the segment this node represents

```
function build(node, start, end):
    if start == end:
        tree[node] = A[start]
        return
    
    mid = (start + end) / 2
    build(2*node, start, mid)        // left child
    build(2*node+1, mid+1, end)      // right child
    tree[node] = tree[2*node] + tree[2*node+1]  // merge

// Initial call:
build(1, 0, n-1)
```

**Trace for A = [2, 1, 5, 3]**:
```
build(1, 0, 3)
├── build(2, 0, 1)
│   ├── build(4, 0, 0) → tree[4] = 2
│   ├── build(5, 1, 1) → tree[5] = 1
│   └── tree[2] = 2 + 1 = 3
├── build(3, 2, 3)
│   ├── build(6, 2, 2) → tree[6] = 5
│   ├── build(7, 3, 3) → tree[7] = 3
│   └── tree[3] = 5 + 3 = 8
└── tree[1] = 3 + 8 = 11
```

---

## How Segments Map to Array Indices

```
For n = 8, 1-indexed:

Array Index │ Segment │ Level │ Purpose
────────────┼─────────┼───────┼──────────────
     1      │ [0-7]   │   0   │ Root
     2      │ [0-3]   │   1   │ Left subtree
     3      │ [4-7]   │   1   │ Right subtree
     4      │ [0-1]   │   2   │ 
     5      │ [2-3]   │   2   │ 
     6      │ [4-5]   │   2   │ 
     7      │ [6-7]   │   2   │ 
     8      │ [0]     │   3   │ Leaf
     9      │ [1]     │   3   │ Leaf
    10      │ [2]     │   3   │ Leaf
    11      │ [3]     │   3   │ Leaf
    12      │ [4]     │   3   │ Leaf
    13      │ [5]     │   3   │ Leaf
    14      │ [6]     │   3   │ Leaf
    15      │ [7]     │   3   │ Leaf
```

---

## Memory Allocation

```
Safe allocation: tree = new array[4 * n]

Why 4n?
  If n is exactly a power of 2: need 2n - 1 nodes → 2n suffices
  If n is NOT a power of 2: nodes can go up to 2^(⌈log₂(n)⌉+1)
  
  Worst case: n = 2^k + 1
    Height = k + 1
    Max index = 2^(k+2) - 1 ≈ 4 × (2^k + 1) ≈ 4n
  
  So 4n is always safe.

Memory usage examples:
  n = 100,000 → 400,000 integers → ~1.6 MB (for 32-bit ints)
  n = 1,000,000 → 4,000,000 integers → ~16 MB
```

---

## Summary Table

| Concept | Details |
|---------|---------|
| Storage | Flat 1-indexed array |
| Root | tree[1] |
| Left child of i | tree[2*i] |
| Right child of i | tree[2*i + 1] |
| Parent of i | tree[i / 2] |
| Leaf for A[j] | tree[n + j] (when n is power of 2) |
| Allocation | 4 × n elements |
| Build direction | Bottom-up (leaves first, then internal nodes) |
| Advantage | No pointers, cache-friendly, simple indexing |

---

## Quick Revision Questions

1. In 1-indexed representation, what is the right child of node 6? *(Node 13)*
2. Why do we use 1-indexed arrays instead of 0-indexed? *(Simpler parent/child formulas: 2i, 2i+1, i/2)*
3. How much memory should we allocate for a segment tree of array size n? *(4 × n)*
4. Where are the leaves located in the array for n = power of 2? *(At indices n to 2n−1)*
5. What is stored at tree[1]? *(The answer for the entire array — root of the tree)*
6. In the recursive build, what parameters does each call carry? *(node index, start, end of the segment)*

---

[← Previous: Tree Structure and Properties](02-tree-structure-and-properties.md) | [Next: Building Segment Tree →](04-building-segment-tree.md)
