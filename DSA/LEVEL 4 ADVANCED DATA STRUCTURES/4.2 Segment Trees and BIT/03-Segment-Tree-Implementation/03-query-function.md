# Query Function

## Chapter Overview

The query function answers questions about any range [L, R] of the array in O(log n) time. This chapter provides the complete implementation with multiple examples and edge case handling.

---

## Complete Query Implementation

```
function query(node, start, end, L, R):
    // Case 1: Complete miss — node's segment is outside [L, R]
    if end < L or start > R:
        return IDENTITY
    
    // Case 2: Complete hit — node's segment is inside [L, R]
    if L <= start and end <= R:
        return tree[node]
    
    // Case 3: Partial overlap — split and merge
    mid = start + (end - start) / 2
    leftResult  = query(2 * node,     start,   mid, L, R)
    rightResult = query(2 * node + 1, mid + 1, end, L, R)
    return merge(leftResult, rightResult)
```

---

## Decision Tree at Each Node

```
                  ┌──────────────────────┐
                  │  Node segment [s, e]  │
                  │  Query range  [L, R]  │
                  └──────────┬───────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
        e < L or s > R    L≤s and e≤R    otherwise
              │              │              │
              ▼              ▼              ▼
         ┌─────────┐  ┌──────────┐  ┌──────────────┐
         │ IDENTITY │  │ tree[node]│  │  Recurse     │
         │ (skip)   │  │ (use as  │  │  BOTH        │
         └─────────┘  │  is)     │  │  children    │
                       └──────────┘  └──────────────┘
```

---

## Detailed Trace: query(2, 5)

```
Array A = [5, 8, 6, 3, 2, 7, 1, 4]  (sum tree)

                  1:[0-7]=36
                 /          \
           2:[0-3]=22     3:[4-7]=14
           /    \          /      \
       4:[0-1]=13 5:[2-3]=9  6:[4-5]=9  7:[6-7]=5
       / \       / \         / \        / \
      5   8     6   3       2   7      1   4

query(1, 0, 7, 2, 5):

Step 1: node=1, seg=[0,7], query=[2,5]
  0≤2? No. 7≤5? No. → PARTIAL
  mid = 3

Step 2a: node=2, seg=[0,3], query=[2,5]
  0≤2? No. 3≤5? Yes. → PARTIAL
  mid = 1
  
  Step 3a: node=4, seg=[0,1], query=[2,5]
    1 < 2? YES → NO OVERLAP → return 0
  
  Step 3b: node=5, seg=[2,3], query=[2,5]
    2≤2? Yes. 3≤5? Yes. → COMPLETE HIT → return 9

  Merge: merge(0, 9) = 9

Step 2b: node=3, seg=[4,7], query=[2,5]
  2≤4? Yes. 7≤5? No. → PARTIAL
  mid = 5
  
  Step 3c: node=6, seg=[4,5], query=[2,5]
    2≤4? Yes. 5≤5? Yes. → COMPLETE HIT → return 9
  
  Step 3d: node=7, seg=[6,7], query=[2,5]
    6 > 5? YES → NO OVERLAP → return 0
  
  Merge: merge(9, 0) = 9

Final merge: merge(9, 9) = 18

Answer: 18
Verify: A[2]+A[3]+A[4]+A[5] = 6+3+2+7 = 18  ✓
```

---

## Nodes Visited Map

```
For query(2, 5):

                  1:[0-7]          PARTIAL    (visited)
                 /          \
           2:[0-3]        3:[4-7]    PARTIAL  (visited)
           /    \          /      \
       4:[0-1]  5:[2-3]  6:[4-5]  7:[6-7]
        NO      FULL      FULL     NO
       OVERLAP  OVERLAP   OVERLAP  OVERLAP
       (visited) (visited) (visited) (visited)

Total: 7 nodes visited
Contributing: nodes 5 and 6 → answer = 9 + 9 = 18

Legend:
  VISITED + FULL overlap:  answer used directly
  VISITED + NO overlap:    returned identity, skipped
  VISITED + PARTIAL:       recursed into children
  NOT VISITED:             subtrees skipped entirely (nodes 8,9,10,11,12,13,14,15)
```

---

## More Query Examples

### Query for single element: query(3, 3)
```
query(1, 0, 7, 3, 3):
  [0-7] PARTIAL → mid=3
    [0-3] PARTIAL → mid=1
      [0-1] NO OVERLAP → 0
      [2-3] PARTIAL → mid=2
        [2-2] NO OVERLAP → 0
        [3-3] COMPLETE → 3
        merge(0, 3) = 3
    merge(0, 3) = 3
    [4-7] NO OVERLAP → 0
  merge(3, 0) = 3

Answer: 3 = A[3]  ✓
Path: root → left → right → right (depth = log n)
```

### Query for full array: query(0, 7)
```
query(1, 0, 7, 0, 7):
  [0-7] → L≤0 and 7≤R → COMPLETE → return tree[1] = 36

Answer: 36 (returned immediately!)
Only 1 node visited.
```

### Query for left half: query(0, 3)
```
query(1, 0, 7, 0, 3):
  [0-7] PARTIAL → mid=3
    [0-3] COMPLETE → return 22
    [4-7] NO OVERLAP → return 0
  merge(22, 0) = 22

Answer: 22
Only 3 nodes visited.
```

---

## Counting Visited Nodes

```
query(L, R) visits at most 4 × height nodes.

Proof sketch:
  At each level, there can be at most:
  - 2 "partial overlap" nodes (left boundary, right boundary)
  - Some "complete overlap" nodes (don't recurse further)
  - Some "no overlap" nodes (return immediately)
  
  Only partial-overlap nodes generate recursive calls.
  At most 2 partial nodes per level → 2 branches per level.
  But each branch may kill one side → net ≤ 4 per level.

Height = O(log n)
Total visited = O(4 × log n) = O(log n)
```

---

## Common Query Patterns

```
// Find sum in range
sumResult = query(1, 0, n-1, L, R)

// Find minimum in range  (change merge to min, identity to INF)
minResult = query(1, 0, n-1, L, R)

// Count elements in range (node stores count=1 for leaves)
countResult = query(1, 0, n-1, L, R)

// Find k-th element (needs augmented query — walk the tree)
function kthElement(node, start, end, k):
    if start == end:
        return start
    mid = (start + end) / 2
    leftCount = tree[2*node]
    if k <= leftCount:
        return kthElement(2*node, start, mid, k)
    else:
        return kthElement(2*node+1, mid+1, end, k - leftCount)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Three cases | No overlap → identity; Complete → tree[node]; Partial → recurse both |
| Time complexity | O(log n) per query |
| Nodes visited | ≤ 4 × height |
| Best case | O(1) — query aligns with one node |
| Worst case | O(4 log n) — crosses many boundaries |
| Identity for sum | 0 |
| Identity for min | +∞ |
| Identity for max | -∞ |
| Partial → recurse | Into BOTH children, merge results |

---

## Quick Revision Questions

1. What does the query return when there is no overlap? *(The identity element)*
2. When does a query visit only 1 node? *(When [L, R] exactly matches a node's segment)*
3. Why must we recurse into both children on partial overlap? *(Part of the answer is in the left child, part in the right)*
4. What is the worst-case number of nodes visited? *(O(4 × log n))*
5. How do you adapt the query for range minimum instead of range sum? *(Change merge to min() and identity to +∞)*

---

[← Previous: Build Function](02-build-function.md) | [Next: Update Function →](04-update-function.md)
