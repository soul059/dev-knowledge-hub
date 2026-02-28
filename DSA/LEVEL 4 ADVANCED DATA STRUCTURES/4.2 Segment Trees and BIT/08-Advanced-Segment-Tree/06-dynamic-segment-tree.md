# Dynamic Segment Tree (Implicit Segment Tree)

## Chapter Overview

A standard segment tree pre-allocates 4n nodes. But what if the value range is huge (e.g., [1, 10⁹]) but only a few positions are actually used? A dynamic (implicit) segment tree creates nodes ON DEMAND, using pointers instead of fixed array positions.

---

## The Problem with Large Ranges

```
Standard segment tree on range [1, 10^9]:
  Array size = 4 × 10^9 = 16 GB of memory → impossible!

But if only q = 10^5 updates/queries happen:
  Only O(q × log(10^9)) ≈ q × 30 nodes are ever needed
  = 3 × 10^6 nodes ≈ ~72 MB → feasible!

Solution: Don't create nodes until they're needed.
```

---

## Core Idea

```
Standard segment tree: all 4n nodes pre-allocated

      [0..7]
     /      \
  [0..3]   [4..7]       ← all exist from the start
  / \       / \
[0..1][2..3][4..5][6..7]
/ \  / \  / \  / \
0 1  2 3  4 5  6 7

Dynamic segment tree: nodes created on demand

After update(3, 10) and update(7, 5) on range [0, 10^9]:

      [0..10^9]          ← root (created)
     /          \
  [0..5×10^8]   ...      ← created (path to 3)
  /          
[0..2.5×10^8]   ...      ← path continues...
  ...
    [3]                  ← leaf created

Only ~30 nodes created per update (one per level)
All other nodes are implicit (treated as identity/zero)
```

---

## Node Structure

```
Node:
    value: int           // aggregate for this range
    lazy: int            // lazy tag (if needed)
    left_child: int      // index into node pool (0 = null/not created)
    right_child: int     // index into node pool (0 = null/not created)

// Null children are treated as containing identity values:
//   sum → 0
//   min → +∞
//   max → -∞

Node pool:
  nodes[MAX_NODES]       // pre-allocate enough
  cnt = 0

  MAX_NODES = q × 30 + padding  // ~30 nodes per operation

new_node():
    cnt += 1
    nodes[cnt] = {value: 0, lazy: 0, left: 0, right: 0}
    return cnt
```

---

## Point Update

```
update(node, l, r, pos, val):
    if node == 0:
        node = new_node()     // create node on demand!
    if l == r:
        nodes[node].value += val
        return node
    mid = (l + r) / 2
    if pos <= mid:
        nodes[node].left = update(nodes[node].left, l, mid, pos, val)
    else:
        nodes[node].right = update(nodes[node].right, mid+1, r, pos, val)
    
    // Pull up: handle null children as 0
    left_val = nodes[nodes[node].left].value if nodes[node].left else 0
    right_val = nodes[nodes[node].right].value if nodes[node].right else 0
    nodes[node].value = left_val + right_val
    return node

// Usage:
root = 0
root = update(root, 0, MAX_RANGE, pos, val)
```

---

## Range Query

```
query(node, l, r, ql, qr):
    if node == 0:               // null node → identity
        return 0
    if ql > r or qr < l:
        return 0
    if ql <= l and r <= qr:
        return nodes[node].value
    mid = (l + r) / 2
    return query(nodes[node].left, l, mid, ql, qr) +
           query(nodes[node].right, mid+1, r, ql, qr)

// Null children automatically return 0 (identity for sum)
```

---

## Step-by-Step Trace

```
Range: [0, 7], operations:
  update(2, 5)
  update(6, 3)
  query(0, 7) → expect 8
  query(2, 4) → expect 5

After update(2, 5):

     node1 [0..7] val=5
     /
   node2 [0..3] val=5
        \
      node3 [2..3] val=5
       /
     node4 [2] val=5

Nodes created: 4 (root → left → right → left = path to leaf 2)

After update(6, 3):

     node1 [0..7] val=8
     /              \
   node2 [0..3]    node5 [4..7] val=3
   val=5                \
        \            node6 [6..7] val=3
      node3 [2..3]    /
      val=5        node7 [6] val=3
       /
     node4 [2]
     val=5

Total nodes: 7 (vs 15 for full segment tree on [0..7])

query(0, 7): node1.value = 8 ✓
query(2, 4): 
  Visit node1 → not fully covered
  Visit node2 → not fully covered  
  Visit node3 → fully in [2,4] → return 5
  Visit node5 → not fully covered
  Visit node6 → out of range [2,4] → return 0
  (node5's left child is null → return 0)
  Total: 5 + 0 = 5 ✓
```

---

## With Lazy Propagation

```
push_down(node, l, r):
    if nodes[node].lazy == 0:
        return
    mid = (l + r) / 2
    
    // Create children if they don't exist
    if nodes[node].left == 0:
        nodes[node].left = new_node()
    if nodes[node].right == 0:
        nodes[node].right = new_node()
    
    lc = nodes[node].left
    rc = nodes[node].right
    
    // Push lazy to children
    nodes[lc].value += nodes[node].lazy × (mid - l + 1)
    nodes[lc].lazy += nodes[node].lazy
    nodes[rc].value += nodes[node].lazy × (r - mid)
    nodes[rc].lazy += nodes[node].lazy
    
    nodes[node].lazy = 0

range_update(node, l, r, ql, qr, val):
    if node == 0:
        node = new_node()
    if ql > r or qr < l:
        return node
    if ql <= l and r <= qr:
        nodes[node].value += val × (r - l + 1)
        nodes[node].lazy += val
        return node
    push_down(node, l, r)
    mid = (l + r) / 2
    nodes[node].left = range_update(nodes[node].left, l, mid, ql, qr, val)
    nodes[node].right = range_update(nodes[node].right, mid+1, r, ql, qr, val)
    // pull up
    left_val = nodes[nodes[node].left].value if nodes[node].left else 0
    right_val = nodes[nodes[node].right].value if nodes[node].right else 0
    nodes[node].value = left_val + right_val
    return node
```

---

## Memory Estimation

```
Each update creates at most O(log R) nodes
  where R is the range size (e.g., 10^9 → log ≈ 30)

Each range update with lazy: O(log R) nodes per operation

For q operations:
  Total nodes ≈ q × 2 × log R  (factor of 2 for safety)
  
  q = 10^5, R = 10^9:
    nodes ≈ 10^5 × 60 = 6 × 10^6
    Each node ≈ 16-24 bytes
    Total ≈ 96-144 MB

Pre-allocate: MAX_NODES = 60 × q
```

---

## Applications

```
1. Coordinate-free operations
   Range [1, 10^9] with sparse updates
   → No coordinate compression needed!

2. Persistent + Dynamic
   Combine persistence with dynamic allocation
   → Each version creates O(log R) new nodes

3. Online problems
   When you don't know all positions in advance
   → Can't compress coordinates offline

4. Interval scheduling on timeline
   Range [0, 10^18] for timestamps
   → Dynamic segment tree on time axis

5. Balanced BST simulation
   Dynamic segment tree on value range
   → Insert/delete/rank queries
```

---

## Comparison

```
                    Standard        Dynamic         With Coord
                    Seg Tree        Seg Tree        Compression
                    ────────        ────────        ───────────
Range size limit    ~10^7           ~10^18          any (map to n)
Space               4n              ~q·log R        4n
Speed               Fastest         Moderate        Fast
Lazy support        Standard        Yes (complex)   Standard
Implementation      Array-based     Pointer-based   Array-based
Offline required?   No              No              Yes

When to use dynamic:
  - Value range too large for array
  - Online (can't preprocess all positions)
  - Combined with persistence
  
When to use coordinate compression instead:
  - Offline (know all positions upfront)
  - Simpler code preferred
  - Speed matters
```

---

## Summary Table

| Aspect | Standard Seg Tree | Dynamic Seg Tree |
|--------|------------------|-----------------|
| Pre-allocated space | 4n | - |
| Nodes per update | 0 (in-place) | O(log R) new |
| Max range | ~10^7 | ~10^18 |
| Null handling | Not needed | Return identity |
| Implementation | Array [2i, 2i+1] | Pointer-based |
| Cache performance | Excellent | Poor (scattered) |
| Total nodes (q ops) | Fixed 4n | O(q · log R) |

---

## Quick Revision Questions

1. When do you need a dynamic segment tree? *(When the value range is too large to allocate (e.g., 10^9 or 10^18) but only a few positions are used)*
2. How are non-existent nodes handled? *(Treated as containing identity values — 0 for sum, +∞ for min, -∞ for max)*
3. How many nodes does each update create? *(O(log R) where R is the range size — one per level)*
4. What is the total memory for q operations on range [0, R]? *(O(q × log R) nodes)*
5. When should you prefer coordinate compression over dynamic segment tree? *(When all positions are known upfront (offline), and simpler/faster code is preferred)*

---

[← Previous: Iterative Segment Tree](05-iterative-segment-tree.md) | [Next: Inversion Count →](../09-Problem-Patterns/01-inversion-count.md)
