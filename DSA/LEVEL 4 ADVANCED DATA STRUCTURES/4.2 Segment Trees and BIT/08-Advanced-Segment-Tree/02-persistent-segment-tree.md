# Persistent Segment Tree

## Chapter Overview

A persistent segment tree preserves every version of the tree after each update. Instead of modifying nodes in place, it creates new nodes for the changed path while sharing unchanged subtrees. This enables "time-travel" queries — answering questions about any historical version.

---

## Why Persistence?

```
Normal segment tree: update DESTROYS previous version

  Before update(3, 10):    After update(3, 10):
       [15]                     [22]         ← old root gone!
      /    \                   /    \
    [6]    [9]              [13]    [9]
    / \    / \              / \    / \
  [1] [5] [4] [5]        [1] [12] [4] [5]
  
  Can't query "what was the sum before the update?" anymore.

Persistent segment tree: BOTH versions exist simultaneously

  Version 0 root: [15] ─→ query(v0, 1, 4) = 15
  Version 1 root: [22] ─→ query(v1, 1, 4) = 22
```

---

## Core Idea: Path Copying

```
When we update index 3, only the PATH from root to leaf 3 changes.
We create NEW nodes for this path, and SHARE the rest.

Version 0:              Version 1 (update index 3 to 12):

     [15]  ← root_v0         [22]  ← root_v1 (NEW)
    /    \                   /    \
  [6]    [9]              [13]    [9]  ← SHARED!
  / \    / \              / \      (points to same node)
[1] [5] [4] [5]        [1] [12]
 ↑   ↑       ↑          ↑    ↑
 |   |       |          |   NEW
 SHARED  SHARED      SHARED

Only 3 new nodes created (one per level) = O(log n) new nodes
```

---

## Node Structure

```
Unlike array-based segment tree, we use POINTER/INDEX-based nodes:

Node:
    value        // aggregate value for this node's range
    left_child   // index/pointer to left child
    right_child  // index/pointer to right child

// No parent pointer needed
// Ranges are implicit (passed as parameters during recursion)

Storage: use a node pool (array of nodes)

nodes[MAX_NODES]   // typically n·log(n) + q·log(n) nodes total
node_count = 0     // current allocation pointer

new_node():
    node_count += 1
    return node_count
```

---

## Build (Same as Normal)

```
build(l, r):
    node = new_node()
    if l == r:
        nodes[node].value = A[l]
        nodes[node].left = nodes[node].right = 0  // null
        return node
    mid = (l + r) / 2
    nodes[node].left = build(l, mid)
    nodes[node].right = build(mid + 1, r)
    nodes[node].value = merge(
        nodes[nodes[node].left].value,
        nodes[nodes[node].right].value)
    return node

root[0] = build(1, n)     // version 0
```

---

## Persistent Update

```
update(prev, l, r, pos, val):
    node = new_node()     // create NEW node (don't modify prev)
    if l == r:
        nodes[node].value = val
        nodes[node].left = nodes[node].right = 0
        return node
    mid = (l + r) / 2
    if pos <= mid:
        // Go left: create new left child, SHARE right child
        nodes[node].left = update(nodes[prev].left, l, mid, pos, val)
        nodes[node].right = nodes[prev].right    // ← SHARE!
    else:
        // Go right: SHARE left child, create new right child
        nodes[node].left = nodes[prev].left       // ← SHARE!
        nodes[node].right = update(nodes[prev].right, mid+1, r, pos, val)
    nodes[node].value = merge(
        nodes[nodes[node].left].value,
        nodes[nodes[node].right].value)
    return node

// Usage:
root[1] = update(root[0], 1, n, pos, val)   // version 1
root[2] = update(root[1], 1, n, pos2, val2) // version 2
// root[0] still accessible!
```

---

## Query Any Version

```
query(node, l, r, ql, qr):
    if ql > r or qr < l:
        return identity
    if ql <= l and r <= qr:
        return nodes[node].value
    mid = (l + r) / 2
    return merge(
        query(nodes[node].left, l, mid, ql, qr),
        query(nodes[node].right, mid+1, r, ql, qr))

// Query version k:
result = query(root[k], 1, n, ql, qr)
```

---

## Step-by-Step Trace

```
Array: [3, 1, 4, 2]     Build version 0

      [10]  (node 7)
     /    \
   [4]     [6]  (nodes 5, 6)
   / \     / \
 [3] [1] [4] [2]  (nodes 1,2,3,4)

root[0] = 7              Nodes used: 7

Update index 2 to 5 → version 1:

New path: root → left → right (index 2 is in left subtree, right child)

      [10]              [14]  (node 10) ← NEW root
     /    \            /    \
   [4]     [6]      [8]     [6]  ← SHARED right subtree
   / \     / \      / \
 [3] [1] [4] [2] [3] [5]  
                   ↑    ↑
                SHARED  NEW (node 8)

root[1] = 10             New nodes: 3 (nodes 8, 9, 10)

Query version 0: sum(1,4) → query(root[0], ...) → 10  ✓
Query version 1: sum(1,4) → query(root[1], ...) → 14  ✓
```

---

## Complexity Analysis

```
Operation          Time           Space (new nodes)
─────────          ────           ─────────────────
Build              O(n)           O(n) nodes (2n-1 for full tree)
Update (1 version) O(log n)       O(log n) new nodes
Query              O(log n)       0
Total for q ops    O(n + q·log n) O(n + q·log n) nodes

Memory estimate:
  n = 10^5, q = 10^5 operations
  Nodes ≈ 2×10^5 + 10^5 × 17 ≈ 1.9 × 10^6
  Each node: ~12-16 bytes (value + 2 pointers)
  Total: ~30 MB — feasible with 256 MB limit
```

---

## Classic Application: K-th Smallest in Range

```
Problem: Given array A[1..n], answer queries:
  "What is the k-th smallest element in A[L..R]?"

Approach: Persistent segment tree on VALUES (not indices)

1. Sort + coordinate compress values to [1..m]
2. Process elements left to right:
   For each i, create version i by inserting A[i] into
   the frequency segment tree
   
   root[i] = update(root[i-1], 1, m, compressed_A[i], +1)

3. For query (L, R, k):
   The "difference" root[R] - root[L-1] gives the
   frequency tree for elements in positions [L..R]
   
   Walk down this implicit difference tree to find k-th element

find_kth(vR, vL, l, r, k):
    if l == r:
        return l    // found the k-th value (compressed)
    mid = (l + r) / 2
    left_count = nodes[nodes[vR].left].value 
               - nodes[nodes[vL].left].value
    if k <= left_count:
        return find_kth(nodes[vR].left, nodes[vL].left, l, mid, k)
    else:
        return find_kth(nodes[vR].right, nodes[vL].right, 
                        mid+1, r, k - left_count)

// Query: kth_smallest(L, R, k) = find_kth(root[R], root[L-1], 1, m, k)
// Time: O(log m) per query
```

---

## Memory Pool Pattern

```
// Static allocation to avoid dynamic memory overhead

MAX_NODES = N * 20 + extra     // n * ~20 for n updates

struct {
    value: int
    left: int      // index into nodes array
    right: int     // index into nodes array
} nodes[MAX_NODES]

cnt = 0

new_node():
    cnt += 1
    nodes[cnt].value = 0
    nodes[cnt].left = 0
    nodes[cnt].right = 0
    return cnt

// Index 0 serves as "null" node with value 0
```

---

## Summary Table

| Aspect | Normal Segment Tree | Persistent Segment Tree |
|--------|-------------------|------------------------|
| Update | Modifies in place | Creates new path |
| Previous versions | Lost | All preserved |
| Space per update | O(1) | O(log n) new nodes |
| Total space | O(n) | O(n + q·log n) |
| Query any version | No | Yes |
| Implementation | Array-based | Pointer/index-based |
| Constant factor | Small | Larger (pointer chasing) |

---

## Quick Revision Questions

1. What is the core technique of persistent segment trees? *(Path copying — create new nodes only along the updated path, share unchanged subtrees)*
2. How many new nodes does each update create? *(O(log n) — one per level of the tree)*
3. How do you query a historical version? *(Use the root pointer for that version: query(root[k], ...))*
4. What is the classic application? *(K-th smallest element in a range [L, R] using prefix frequency trees)*
5. Why can't you use array-based indexing (2i, 2i+1) for persistent trees? *(Because new nodes don't follow the 2i/2i+1 pattern — they're allocated sequentially in a pool)*

---

[← Previous: 2D Segment Tree](01-2d-segment-tree.md) | [Next: Segment Tree with Sets →](03-segment-tree-with-sets.md)
