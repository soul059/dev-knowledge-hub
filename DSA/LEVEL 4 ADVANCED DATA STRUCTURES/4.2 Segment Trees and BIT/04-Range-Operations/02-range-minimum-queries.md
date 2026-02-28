# Range Minimum Queries

## Chapter Overview

Range Minimum Query (RMQ) finds the smallest element in a subarray. Unlike sum, minimum is not invertible, making segment trees (or sparse tables) the go-to solution for dynamic RMQ.

---

## Problem Statement

```
Given array A[0..n-1]:
  - Query(L, R): Return min(A[L], A[L+1], ..., A[R])
  - Update(i, val): Set A[i] = val
```

---

## Min Properties

```
1. ASSOCIATIVE:   min(min(a,b), c) = min(a, min(b,c))    ✓
2. COMMUTATIVE:   min(a, b) = min(b, a)                  ✓
3. IDENTITY:      min(a, +∞) = a                          ✓
4. IDEMPOTENT:    min(a, a) = a                           ✓
5. NOT INVERTIBLE: can't "subtract" a min                  ✗

Key consequence:
  Prefix sums don't work (can't undo a min)
  Sparse table works (idempotent → overlapping ranges OK)
  Segment tree works (associative → split ranges OK)
  BIT does NOT work natively (needs invertibility)
```

---

## Segment Tree for RMQ

```
Merge function:    merge(a, b) = min(a, b)
Identity element:  IDENTITY = +∞  (or INT_MAX)

function build(node, start, end):
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = min(tree[2*node], tree[2*node+1])

function query(node, start, end, L, R):
    if end < L or start > R:
        return +∞                    // identity for min
    if L <= start and end <= R:
        return tree[node]
    mid = (start + end) / 2
    return min(query(2*node, start, mid, L, R),
               query(2*node+1, mid+1, end, L, R))

function update(node, start, end, idx, val):
    if start == end:
        tree[node] = val
        return
    mid = (start + end) / 2
    if idx <= mid:
        update(2*node, start, mid, idx, val)
    else:
        update(2*node+1, mid+1, end, idx, val)
    tree[node] = min(tree[2*node], tree[2*node+1])
```

---

## Worked Example

```
A = [5, 2, 8, 3, 1, 7, 4, 6]   n = 8

Min segment tree:

                1:[0-7] = 1
               /          \
         2:[0-3] = 2     3:[4-7] = 1
         /    \           /      \
     4:[0-1]=2 5:[2-3]=3 6:[4-5]=1 7:[6-7]=4
     / \       / \        / \       / \
    5   2     8   3      1   7     4   6

Query: min(2, 6)

query(1, 0, 7, 2, 6):
  PARTIAL → recurse both
  
  query(2, 0, 3, 2, 6):
    PARTIAL
    query(4, 0, 1, 2, 6): NO OVERLAP → +∞
    query(5, 2, 3, 2, 6): COMPLETE → 3
    min(+∞, 3) = 3
  
  query(3, 4, 7, 2, 6):
    PARTIAL
    query(6, 4, 5, 2, 6): COMPLETE → 1
    query(7, 6, 7, 2, 6):
      PARTIAL
      query(14, 6, 6, 2, 6): COMPLETE → 4
      query(15, 7, 7, 2, 6): NO OVERLAP → +∞
      min(4, +∞) = 4
    min(1, 4) = 1
  
  min(3, 1) = 1

Answer: 1  ✓  (A[4] = 1 is the minimum in range [2,6])
```

---

## Update Effect on Min Tree

```
Update A[4] = 10 (was 1):

Before:                          After:
        1:[0-7]=1                       1:[0-7]=2
       /         \                     /         \
  2:[0-3]=2    3:[4-7]=1         2:[0-3]=2    3:[4-7]=4
  /    \        /      \         /    \        /      \
4:=2  5:=3   6:[4-5]=1 7:=4   4:=2  5:=3   6:[4-5]=7 7:=4
              / \                            / \
             1   7                          10  7

Changed nodes: leaf(4) → [4-5] → [4-7] → [0-7]
  tree[12]=10, tree[6]=min(10,7)=7, tree[3]=min(7,4)=4, tree[1]=min(2,4)=2

Note: Global minimum changed from 1 to 2!
The new minimum comes from A[1]=2, a completely different element.
```

---

## RMQ with Index Tracking

Often you need not just the minimum value but also its **position**:

```
// Store both value and index
struct Node:
    value: integer
    index: integer

function merge(a, b):
    if a.value <= b.value:
        return a
    else:
        return b

// Build: leaf nodes store Node(A[i], i)
// Query returns Node with min value and original index

Example:
  A = [5, 2, 8, 3]
  
  Leaves: Node(5,0), Node(2,1), Node(8,2), Node(3,3)
  
  merge(Node(5,0), Node(2,1)) = Node(2,1)
  merge(Node(8,2), Node(3,3)) = Node(3,3)
  merge(Node(2,1), Node(3,3)) = Node(2,1)
  
  query(0, 3) → Node(2, 1)  → min is 2 at index 1
```

---

## RMQ vs Sparse Table (Static Case)

```
For STATIC data (no updates), Sparse Table is better:

┌─────────────────┬──────────────┬──────────────┐
│                 │ Segment Tree │ Sparse Table │
├─────────────────┼──────────────┼──────────────┤
│  Build          │  O(n)        │  O(n log n)  │
│  Query          │  O(log n)    │  O(1) !      │
│  Update         │  O(log n)    │  Impossible  │
│  Space          │  O(4n)       │  O(n log n)  │
│  Use when       │  Dynamic     │  Static only │
└─────────────────┴──────────────┴──────────────┘

Sparse Table can answer RMQ in O(1) because min is idempotent:
  min(a, a) = a → overlapping ranges are OK
```

---

## Applications

| Problem | How RMQ Helps |
|---------|--------------|
| Lowest Common Ancestor | Reduce LCA to RMQ on Euler tour |
| Stock price analysis | Find lowest price in date range |
| Flood fill bounds | Minimum elevation in region |
| Cartesian tree construction | Find minimum to split ranges |
| Histogram largest rectangle | Find min height in range |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Merge function | min(a, b) |
| Identity element | +∞ (INT_MAX) |
| Invertible? | No — can't subtract a minimum |
| Works with BIT? | Not natively |
| Works with Sparse Table? | Yes — O(1) query for static data |
| Index tracking | Store (value, index) pairs and merge by value |
| Lazy propagation | Possible for range assignment (set all to val) |

---

## Quick Revision Questions

1. What is the identity element for range minimum? *(+∞ or INT_MAX)*
2. Why can't BIT handle range minimum queries natively? *(Min is not invertible — can't compute range min from prefix mins)*
3. Why does Sparse Table give O(1) RMQ but segment tree gives O(log n)? *(Min is idempotent so overlapping ranges are fine in Sparse Table; Segment Tree doesn't exploit this)*
4. How do you track the index of the minimum element? *(Store (value, index) pairs; merge picks the pair with smaller value)*
5. How does updating one element affect the min tree? *(The minimum of ancestor nodes may change — must re-merge all the way up)*

---

[← Previous: Range Sum Queries](01-range-sum-queries.md) | [Next: Range Maximum Queries →](03-range-maximum-queries.md)
