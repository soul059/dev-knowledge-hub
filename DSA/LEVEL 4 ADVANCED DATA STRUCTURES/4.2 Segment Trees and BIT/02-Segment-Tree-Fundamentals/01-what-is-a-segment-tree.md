# What is a Segment Tree?

## Chapter Overview

A Segment Tree is a binary tree data structure that allows efficient range queries and updates on an array. Each node stores the result of a query (sum, min, max, etc.) for a **segment** (contiguous subarray) of the original array.

---

## Core Idea

**Divide the array recursively into halves. Each node represents a segment and stores the precomputed answer for that segment.**

```
Array A = [2, 1, 5, 3, 4, 7]

Divide into segments:

                    [0-5]           ← entire array
                   /     \
              [0-2]       [3-5]     ← halves
             /    \       /    \
          [0-1]  [2-2] [3-4]  [5-5] ← quarters
          /   \        /   \
       [0-0] [1-1]  [3-3] [4-4]    ← individual elements
```

For a **sum** segment tree:
```
                    [0-5]=22
                   /        \
              [0-2]=8      [3-5]=14
             /    \        /      \
          [0-1]=3 [2]=5  [3-4]=7  [5]=7
          /   \          /    \
        [0]=2 [1]=1    [3]=3  [4]=4
```

Each node stores: `sum of elements in its range`

---

## Why "Segment" Tree?

```
Each node represents a SEGMENT of the array:

Node [2-4] → represents the segment A[2], A[3], A[4]
Node [0-0] → represents just A[0] (a leaf)
Node [0-5] → represents the entire array (root)

The tree structure lets us decompose ANY query range [L, R]
into O(log n) precomputed segments.
```

---

## Key Properties

```
┌──────────────────────────────────────────────┐
│         SEGMENT TREE PROPERTIES              │
├──────────────────────────────────────────────┤
│  1. Binary tree (each node has 0 or 2 kids)  │
│  2. Leaves = individual array elements       │
│  3. Internal node = merge of its children    │
│  4. Root = answer for the entire array       │
│  5. Height = ⌈log₂(n)⌉                      │
│  6. Total nodes ≤ 4n (for array size n)      │
│  7. Each level halves the segment size       │
└──────────────────────────────────────────────┘
```

---

## How Queries Work (Intuition)

To answer `query(L, R)`, we traverse the tree and collect nodes whose segments are **completely inside** [L, R]:

```
Query sum(1, 4) on array [2, 1, 5, 3, 4, 7]:

                    [0-5]=22
                   /        \
              [0-2]=8      [3-5]=14
             /    \        /      \
          [0-1]=3 [2]=5  [3-4]=7  [5]=7
          /   \          /    \
        [0]=2 [1]=1    [3]=3  [4]=4

Step 1: Visit root [0-5]. Range [1,4] partially overlaps → go to both children

Step 2: Visit [0-2]. Range [1,4] partially overlaps → go to both children
   Visit [0-1]. Range [1,4] partially overlaps → go to both children
     Visit [0]. Range [1,4] does NOT contain 0 → return 0 (identity)
     Visit [1]. Range [1,4] contains 1 → return 1 ✓
   Visit [2]. Range [1,4] contains 2 → return 5 ✓

Step 3: Visit [3-5]. Range [1,4] partially overlaps → go to both children
   Visit [3-4]. Range [1,4] completely contains [3-4] → return 7 ✓
   Visit [5]. Range [1,4] does NOT contain 5 → return 0 (identity)

Result: 1 + 5 + 7 = 13

Verify: A[1]+A[2]+A[3]+A[4] = 1+5+3+4 = 13  ✓
```

---

## How Updates Work (Intuition)

To update `A[i]`, change the leaf and update all ancestors:

```
Update A[1] = 9 (was 1):

Before:                          After:
        [0-5]=22                         [0-5]=30
       /        \                       /        \
  [0-2]=8      [3-5]=14           [0-2]=16     [3-5]=14
  /    \        /      \          /    \        /      \
[0-1]=3 [2]=5 [3-4]=7 [5]=7   [0-1]=11 [2]=5 [3-4]=7 [5]=7
/   \                          /    \
[0]=2 [1]=1                  [0]=2  [1]=9  ← updated
                                     ↑
Only the path from leaf to root changes!
4 nodes updated out of 11 = O(log n)
```

---

## Segment Tree vs Other Structures

```
┌────────────────┬──────────┬───────────┬───────────┐
│                │ Seg Tree │  Array    │ Prefix S  │
├────────────────┼──────────┼───────────┼───────────┤
│  Build         │  O(n)    │  O(1)     │  O(n)     │
│  Point Query   │  O(log n)│  O(1)     │  O(1)     │
│  Range Query   │  O(log n)│  O(n)     │  O(1)*    │
│  Point Update  │  O(log n)│  O(1)     │  O(n)     │
│  Range Update  │  O(log n)│  O(n)     │  O(n)     │
│  Space         │  O(n)    │  O(n)     │  O(n)     │
│  Flexibility   │  HIGH    │  LOW      │  LOW      │
└────────────────┴──────────┴───────────┴───────────┘
* O(1) only for sum; not for min/max
```

---

## When to Use a Segment Tree

```
USE Segment Tree when:
  ✓ You need BOTH range queries AND updates
  ✓ Query operation is associative (sum, min, max, gcd, xor)
  ✓ n and Q are large (≥ 10⁴)
  ✓ You need range updates (with lazy propagation)
  ✓ You need to handle multiple query types

DON'T use Segment Tree when:
  ✗ Data is static → use prefix sums or sparse table
  ✗ Only sum queries → BIT is simpler
  ✗ n is tiny (≤ 1000) → brute force is fine
  ✗ Offline queries → Mo's algorithm might be simpler
```

---

## Real-World Applications

| Application | Query Type | Update Type |
|------------|-----------|-------------|
| Database range queries | Sum/Count | Insert/Delete |
| Image processing | 2D sum/min | Pixel modification |
| Computational geometry | Sweep line | Interval add/remove |
| Gaming leaderboards | k-th rank | Score changes |
| Network flow analysis | Max bottleneck | Capacity changes |
| Scheduling | Interval overlap | Add/cancel events |

---

## Summary Table

| Aspect | Description |
|--------|-------------|
| Structure | Binary tree where each node represents a segment |
| Leaves | Individual array elements |
| Internal Nodes | Merge of children (sum, min, max, etc.) |
| Root | Answer for the entire array |
| Height | O(log n) |
| Build | O(n) — bottom-up, each node from its children |
| Query | O(log n) — decompose [L,R] into O(log n) segments |
| Update | O(log n) — update leaf, propagate to root |

---

## Quick Revision Questions

1. What does each node in a segment tree represent? *(The precomputed answer for a contiguous segment of the array)*
2. What are the leaves of a segment tree? *(Individual array elements)*
3. How is an internal node's value computed? *(By merging its two children's values)*
4. What is the height of a segment tree for an array of size n? *(⌈log₂(n)⌉)*
5. How many nodes does a query visit in the worst case? *(O(log n) — at most ~4 × log₂(n) nodes)*
6. Why is a segment tree better than prefix sums for dynamic data? *(Updates take O(log n) vs O(n) for prefix sums)*

---

[← Previous Unit: Update Types](../01-Range-Query-Problems/06-update-types.md) | [Next: Tree Structure and Properties →](02-tree-structure-and-properties.md)
