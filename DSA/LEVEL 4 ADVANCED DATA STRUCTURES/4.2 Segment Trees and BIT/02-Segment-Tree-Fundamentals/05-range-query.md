# Range Query

## Chapter Overview

The range query is the primary reason segment trees exist. This chapter dissects how a query decomposes a range into precomputed segments and combines them efficiently.

---

## Query Algorithm

```
ALGORITHM: Range Query

Input:  Query range [L, R]
Output: merge of A[L..R]

function query(node, start, end, L, R):
    // Case 1: No overlap → return identity
    if R < start OR end < L:
        return IDENTITY
    
    // Case 2: Complete overlap → return this node's value
    if L <= start AND end <= R:
        return tree[node]
    
    // Case 3: Partial overlap → query both children and merge
    mid = (start + end) / 2
    leftResult  = query(2*node, start, mid, L, R)
    rightResult = query(2*node+1, mid+1, end, L, R)
    return merge(leftResult, rightResult)

// Call: query(1, 0, n-1, L, R)
```

Identity values:
- Sum: 0
- Min: +∞
- Max: -∞
- GCD: 0
- XOR: 0

---

## The Three Cases Visualized

```
Node's segment: [start ────────── end]

Case 1: NO OVERLAP
  Query: [L ── R]                           [L ── R]
  Node:              [start ── end]
  → Return identity (this node is irrelevant)

Case 2: COMPLETE OVERLAP
  Query: [L ──────────────────────── R]
  Node:       [start ────── end]
  → Return tree[node] directly (no need to go deeper)

Case 3: PARTIAL OVERLAP
  Query:           [L ────────── R]
  Node:   [start ────────── end]
  → Must recurse into BOTH children
```

---

## Complete Trace: Sum Query

```
Array A = [3, 1, 4, 1, 5, 9, 2, 6]    n = 8

Segment Tree (sum):
                  1:[0-7]=31
                 /          \
           2:[0-3]=9      3:[4-7]=22
           /    \          /      \
       4:[0-1]=4 5:[2-3]=5 6:[4-5]=14 7:[6-7]=8
       / \       / \       / \        / \
      8:3 9:1  10:4 11:1  12:5 13:9  14:2 15:6

Query: sum(2, 6)  — sum of A[2] through A[6]
```

```
query(1, 0, 7, 2, 6)
  [0-7] partially overlaps [2-6] → PARTIAL → recurse

  ├── query(2, 0, 3, 2, 6)
  │   [0-3] partially overlaps [2-6] → PARTIAL → recurse
  │
  │   ├── query(4, 0, 1, 2, 6)
  │   │   [0-1] does NOT overlap [2-6] → NO OVERLAP → return 0
  │   │
  │   ├── query(5, 2, 3, 2, 6)
  │   │   [2-3] is completely inside [2-6] → COMPLETE → return 5
  │   │
  │   └── merge(0, 5) = 5
  │
  ├── query(3, 4, 7, 2, 6)
  │   [4-7] partially overlaps [2-6] → PARTIAL → recurse
  │
  │   ├── query(6, 4, 5, 2, 6)
  │   │   [4-5] is completely inside [2-6] → COMPLETE → return 14
  │   │
  │   ├── query(7, 6, 7, 2, 6)
  │   │   [6-7] partially overlaps [2-6] → PARTIAL → recurse
  │   │
  │   │   ├── query(14, 6, 6, 2, 6)
  │   │   │   [6-6] is completely inside [2-6] → COMPLETE → return 2
  │   │   │
  │   │   ├── query(15, 7, 7, 2, 6)
  │   │   │   [7-7] does NOT overlap [2-6] → NO OVERLAP → return 0
  │   │   │
  │   │   └── merge(2, 0) = 2
  │   │
  │   └── merge(14, 2) = 16
  │
  └── merge(5, 16) = 21

Answer: 21
Verify: A[2]+A[3]+A[4]+A[5]+A[6] = 4+1+5+9+2 = 21  ✓
```

---

## Nodes Visited (Highlighted)

```
                  1:[0-7]=31    ← PARTIAL (visited)
                 /          \
           2:[0-3]=9      3:[4-7]=22     ← both PARTIAL (visited)
           /    \          /      \
       4:[0-1]  5:[2-3]=5 6:[4-5]=14 7:[6-7]    
        ↑ NO     ↑ FULL    ↑ FULL    ↑ PARTIAL (visited)
       OVERLAP  OVERLAP   OVERLAP    
                                     / \
                                  14:2  15:6
                                  ↑ FULL  ↑ NO OVERLAP

Nodes visited: 1, 2, 4, 5, 3, 6, 7, 14, 15 = 9 nodes
Nodes contributing: 5, 6, 14 → answer = 5 + 14 + 2 = 21

For n = 8, log₂(8) = 3 → visited O(4 × log n) ≈ 12 nodes max
```

---

## Why O(log n) Nodes?

**Key insight**: At each level of the tree, at most **4 nodes** are visited.

```
At each level, the query range [L, R] can:

Level k:  ... [entirely outside] [partial] [full] [full] ... [full] [partial] [outside] ...
                                    ↑                                   ↑
                              left boundary                       right boundary

At most 2 partial nodes per level (left and right boundaries)
All nodes between them are either fully inside or fully outside

Partially-overlapping nodes: at most 2 per level
Fully-overlapping nodes: their values are used directly (no recursion)
No-overlap nodes: return immediately

Total nodes visited at each level: ≤ 4
Levels: O(log n)
Total: O(4 × log n) = O(log n)
```

---

## Edge Cases

### Query for a Single Element
```
query(1, 0, 7, 3, 3)  — query A[3] only

Goes straight down to leaf node for index 3.
Path: root → [0-3] → [2-3] → [3]
Nodes visited: O(log n)
```

### Query for Entire Array
```
query(1, 0, 7, 0, 7)  — query entire array

Root [0-7] is COMPLETELY inside [0-7] → return tree[1] immediately!
Nodes visited: 1  (best case)
```

### Query for Left Half
```
query(1, 0, 7, 0, 3)

Root [0-7] partial → recurse
  Left [0-3] COMPLETE → return tree[2]
  Right [4-7] NO OVERLAP → return identity

Nodes visited: 3
```

---

## Query Visualization for Different Ranges

```
Array indices: 0  1  2  3  4  5  6  7

query(0, 7): [=============================]  → 1 node (root)
query(0, 3): [==============]                 → 3 nodes
query(4, 7):                  [==============] → 3 nodes
query(2, 5):       [=================]        → ~8 nodes
query(3, 3):          [==]                    → 4 nodes (path to leaf)
query(0, 0): [==]                             → 4 nodes (path to leaf)

Worst case: ranges that cross many segment boundaries
Best case: ranges that align with segment boundaries
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Three cases | No overlap, complete overlap, partial overlap |
| No overlap | Return identity element |
| Complete overlap | Return stored node value |
| Partial overlap | Recurse into both children, merge results |
| Time complexity | O(log n) — at most 4 nodes per level |
| Best case | O(1) — query aligns with a single node |
| Worst case | O(4 × log n) — crosses many boundaries |
| Identity for sum | 0 |
| Identity for min | +∞ |
| Identity for max | -∞ |

---

## Quick Revision Questions

1. What are the three cases in a segment tree range query? *(No overlap → identity; Complete overlap → return node value; Partial overlap → recurse both children)*
2. Why do we need an identity element? *(To handle no-overlap cases without affecting the merge result)*
3. What is the identity element for range minimum queries? *(+∞ — min(x, +∞) = x)*
4. How many nodes are visited per level in the worst case? *(At most 4)*
5. When does a query visit only 1 node? *(When the query range exactly matches a node's segment, like querying the entire array)*

---

[← Previous: Building Segment Tree](04-building-segment-tree.md) | [Next: Point Update →](06-point-update.md)
