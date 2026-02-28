# Tree Structure and Properties

## Chapter Overview

Understanding the mathematical structure of a segment tree — its shape, node relationships, and properties — is essential for implementing it correctly and reasoning about its complexity.

---

## Tree Shape

A segment tree for an array of size `n` is a **nearly complete binary tree**.

```
For n = 8 (power of 2) → PERFECT binary tree:

Level 0 (root):         [0-7]                           1 node
Level 1:           [0-3]     [4-7]                      2 nodes
Level 2:        [0-1] [2-3] [4-5] [6-7]                4 nodes  
Level 3 (leaves): [0][1] [2][3] [4][5] [6][7]          8 nodes
                                                    Total: 15

Height = log₂(8) = 3
Total nodes = 2×8 - 1 = 15
```

```
For n = 6 (NOT power of 2) → not perfectly balanced:

Level 0:              [0-5]
Level 1:         [0-2]       [3-5]
Level 2:      [0-1]  [2]   [3-4]  [5]
Level 3:     [0] [1]       [3] [4]

Some leaves at level 2, some at level 3.
Height = ⌈log₂(6)⌉ = 3
Total nodes = 11
```

---

## Node Relationships

For a node at index `i` (1-indexed array representation):

```
┌──────────────────────────────────────────────┐
│           NODE INDEX FORMULAS                │
│                                              │
│  Parent of node i:        i / 2  (i >> 1)   │
│  Left child of node i:    2 * i  (i << 1)   │
│  Right child of node i:   2*i+1  (i<<1 | 1) │
│  Is leaf?:                2*i > total nodes  │
│  Sibling of node i:       i ^ 1             │
└──────────────────────────────────────────────┘
```

```
Tree indexed 1-based:

              1:[0-7]
             /       \
         2:[0-3]    3:[4-7]
         /    \      /    \
      4:[0-1] 5:[2-3] 6:[4-5] 7:[6-7]
      /   \    /   \    /   \    /   \
     8    9   10   11  12   13  14   15

Node 5: left child = 10, right child = 11, parent = 2
Node 12: parent = 6, sibling = 13
```

---

## Segment Splitting

Each internal node's segment `[start, end]` is split at `mid = (start + end) / 2`:

```
Node segment: [start, end]
  Left child:  [start, mid]
  Right child: [mid+1, end]

Example: Node [2, 7]
  mid = (2+7)/2 = 4
  Left:  [2, 4]
  Right: [5, 7]

Example: Node [3, 5]
  mid = (3+5)/2 = 4
  Left:  [3, 4]
  Right: [5, 5]   ← single element (leaf-like)
```

---

## Properties Matrix

```
┌─────────────────────────────────────────────────────────┐
│  Property              │  Value                         │
├────────────────────────┼────────────────────────────────┤
│  Type                  │  Binary tree                   │
│  Height                │  ⌈log₂(n)⌉                    │
│  # Leaves              │  n                             │
│  # Internal nodes      │  n - 1 (for complete tree)     │
│  Total nodes           │  ≤ 4n (safe array size)        │
│  Exact nodes (n=2^k)   │  2n - 1                        │
│  Each leaf             │  Single array element          │
│  Each internal node    │  merge(left_child, right_child)│
│  Root answer           │  Query over entire array       │
│  Guaranteed balanced   │  YES (height always O(log n))  │
└─────────────────────────────────────────────────────────┘
```

---

## Depth Analysis

The depth of a node tells us how much work is needed to reach it.

```
n = 8:
Depth 0:  1 node   (root)           → covers 8 elements
Depth 1:  2 nodes                   → covers 4 elements each
Depth 2:  4 nodes                   → covers 2 elements each
Depth 3:  8 nodes  (leaves)         → covers 1 element each

General:
Depth d:  up to 2^d nodes           → covers n/2^d elements each

Total depths: ⌈log₂(n)⌉ + 1 levels
```

For n = 10:
```
Depth 0:  [0-9]                                    → 10 elements
Depth 1:  [0-4], [5-9]                             → 5 each
Depth 2:  [0-2], [3-4], [5-7], [8-9]              → 2-3 each
Depth 3:  [0-1], [2], [3], [4], [5-6], [7], [8], [9]  → 1-2 each
Depth 4:  [0], [1], [5], [6]                       → 1 each

Height = 4 = ⌈log₂(10)⌉
```

---

## Why 4×n Array Size?

When storing the segment tree in an array, we need to allocate enough space:

```
For n elements:
  Height h = ⌈log₂(n)⌉
  Max nodes in a complete binary tree of height h:
    = 2^(h+1) - 1

  Worst case: n = 2^k + 1 (just over a power of 2)
    h = k + 1
    Max nodes = 2^(k+2) - 1 ≈ 4 × 2^k ≈ 4 × n

  So we allocate: tree[4 * n]  ← safe for all cases

Examples:
  n = 8  → h = 3  → max = 15  → 4×8 = 32  (wastes 17)
  n = 5  → h = 3  → max = 15  → 4×5 = 20  (wastes 5)
  n = 9  → h = 4  → max = 31  → 4×9 = 36  (wastes 5)
  n = 10 → h = 4  → max = 31  → 4×10 = 40 (wastes 9)
```

---

## Segment Coverage Theorem

**Theorem**: Any range [L, R] can be represented as the union of at most `2 × ⌈log₂(n)⌉` non-overlapping segments from the tree.

```
Intuition:
  At each level, at most 2 partial segments may be needed
  (one at the left boundary, one at the right boundary).
  All segments between them are fully covered by larger nodes.

  [L........................R]
  [partial][full][full][partial]
      ↑                    ↑
  Left boundary        Right boundary
  splits here          splits here
```

This is why query time is O(log n) — we visit at most O(log n) nodes.

---

## Invariant: Node Value

The fundamental invariant of a segment tree:

```
For every node representing segment [s, e]:
  tree[node] = merge(A[s], A[s+1], ..., A[e])

For sum tree:
  tree[node] = A[s] + A[s+1] + ... + A[e]

For min tree:
  tree[node] = min(A[s], A[s+1], ..., A[e])

This invariant is maintained after every update.
```

---

## Summary Table

| Property | Value |
|----------|-------|
| Tree type | Nearly complete binary tree |
| Height | ⌈log₂(n)⌉ |
| Leaf count | n |
| Total nodes | ≤ 4n |
| Left child of node i | 2i |
| Right child of node i | 2i + 1 |
| Parent of node i | i / 2 |
| Segment split | [start, mid] and [mid+1, end] |
| Max nodes visited per query | O(log n) |
| Node invariant | tree[node] = merge of all elements in its segment |

---

## Quick Revision Questions

1. For a node representing segment [3, 8], what are its children's segments? *([3, 5] and [6, 8])*
2. In 1-indexed array representation, what is the left child of node 5? *(Node 10)*
3. Why do we allocate 4×n space for the array? *(Worst case when n is just above a power of 2, the complete tree has up to 4n nodes)*
4. What is the height of a segment tree for n = 100,000? *(⌈log₂(100000)⌉ = 17)*
5. How many nodes exist at depth d in a perfect segment tree? *(2^d)*

---

[← Previous: What is a Segment Tree?](01-what-is-a-segment-tree.md) | [Next: Array Representation →](03-array-representation.md)
