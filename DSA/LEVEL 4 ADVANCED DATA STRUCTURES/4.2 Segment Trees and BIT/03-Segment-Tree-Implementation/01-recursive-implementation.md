# Recursive Implementation

## Chapter Overview

The recursive approach is the most intuitive and commonly taught implementation of a segment tree. This chapter presents the complete recursive implementation framework — the skeleton that all operations (build, query, update) share.

---

## The Recursive Pattern

Every segment tree operation follows the same skeleton:

```
function operation(node, start, end, ...params):
    // 1. BASE or TERMINAL condition
    // 2. Split into halves
    // 3. Recurse on relevant children
    // 4. Combine results
```

The three operations differ only in what happens at each step:

```
┌─────────────┬──────────────┬──────────────────┬──────────────┐
│   Step      │    Build     │     Query        │    Update    │
├─────────────┼──────────────┼──────────────────┼──────────────┤
│ Terminal    │ start==end   │ no overlap OR    │ start==end   │
│             │ (leaf)       │ full overlap     │ (leaf)       │
├─────────────┼──────────────┼──────────────────┼──────────────┤
│ Recurse     │ BOTH children│ BOTH children    │ ONE child    │
│             │              │ (if partial)     │              │
├─────────────┼──────────────┼──────────────────┼──────────────┤
│ Combine     │ merge(L, R)  │ merge(L, R)      │ merge(L, R)  │
└─────────────┴──────────────┴──────────────────┴──────────────┘
```

---

## Complete Implementation (Pseudocode)

```
// Global variables
MAXN = 100005
tree[4 * MAXN]           // segment tree array
A[MAXN]                  // original array
n                        // array size

// ========== MERGE FUNCTION ==========
function merge(a, b):
    return a + b          // change for min/max/etc.

IDENTITY = 0              // change for min(+∞), max(-∞)

// ========== BUILD ==========
function build(node, start, end):
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = merge(tree[2*node], tree[2*node+1])

// ========== QUERY ==========
function query(node, start, end, L, R):
    if R < start OR end < L:           // no overlap
        return IDENTITY
    if L <= start AND end <= R:        // complete overlap
        return tree[node]
    mid = (start + end) / 2            // partial overlap
    leftVal  = query(2*node, start, mid, L, R)
    rightVal = query(2*node+1, mid+1, end, L, R)
    return merge(leftVal, rightVal)

// ========== UPDATE ==========
function update(node, start, end, index, value):
    if start == end:
        tree[node] = value
        return
    mid = (start + end) / 2
    if index <= mid:
        update(2*node, start, mid, index, value)
    else:
        update(2*node+1, mid+1, end, index, value)
    tree[node] = merge(tree[2*node], tree[2*node+1])

// ========== USAGE ==========
build(1, 0, n-1)                       // build tree
result = query(1, 0, n-1, L, R)       // query [L, R]
update(1, 0, n-1, index, value)       // update A[index]
```

---

## Parameter Explanation

```
Every recursive call carries:

┌────────────────────────────────────────────────────┐
│  Parameter │  Meaning                              │
├────────────┼───────────────────────────────────────┤
│  node      │  Index in tree[] array (1-indexed)    │
│  start     │  Left boundary of node's segment      │
│  end       │  Right boundary of node's segment     │
│  L, R      │  Query range (for query only)         │
│  index     │  Target position (for update only)    │
│  value     │  New value (for update only)          │
└────────────┴───────────────────────────────────────┘

The (start, end) pair tells us WHICH SEGMENT this node covers.
The node index tells us WHERE in the array this node is stored.
```

---

## Recursion Tree Visualization

```
For n = 8, query(1, 0, 7, 2, 5):

                    (1, 0, 7)
                    PARTIAL
                   /        \
          (2, 0, 3)          (3, 4, 7)
          PARTIAL             PARTIAL
         /      \            /       \
    (4, 0, 1)  (5, 2, 3)  (6, 4, 5)  (7, 6, 7)
    NO OVERLAP  COMPLETE   COMPLETE   NO OVERLAP
    return 0    return 5   return 14  return 0

Merge path:
  merge(0, 5) = 5     [node 2]
  merge(14, 0) = 14   [node 3]
  merge(5, 14) = 19   [node 1]

Answer: 19
```

---

## Call Stack Analysis

```
For update(1, 0, 7, 5, 20):

Call Stack (grows down):
┌──────────────────────────┐
│ update(1, 0, 7, 5, 20)  │  → index 5 > mid=3, go right
├──────────────────────────┤
│ update(3, 4, 7, 5, 20)  │  → index 5 > mid=5? No, go left... 
│                          │    wait: mid=(4+7)/2=5, index 5 ≤ 5, go left
├──────────────────────────┤
│ update(6, 4, 5, 5, 20)  │  → index 5 > mid=4, go right
├──────────────────────────┤
│ update(13, 5, 5, 5, 20) │  → start==end, leaf! tree[13]=20
└──────────────────────────┘

Unwind (merge back up):
  tree[6]  = merge(tree[12], tree[13]) = merge(5, 20) = 25
  tree[3]  = merge(tree[6], tree[7])   = merge(25, 8) = 33
  tree[1]  = merge(tree[2], tree[3])   = merge(9, 33) = 42

Stack depth: 4 = log₂(8) + 1 = O(log n)
```

---

## Common Mistakes

### 1. Forgetting to re-merge after update
```
// WRONG:
function update(node, start, end, index, value):
    if start == end:
        tree[node] = value
        return
    mid = (start + end) / 2
    if index <= mid:
        update(2*node, start, mid, index, value)
    else:
        update(2*node+1, mid+1, end, index, value)
    // MISSING: tree[node] = merge(tree[2*node], tree[2*node+1])
```

### 2. Wrong overlap check in query
```
// WRONG ordering:
if start < L OR end > R:    // This is NOT "no overlap"!

// CORRECT:
if R < start OR end < L:    // No overlap: query is entirely outside
```

### 3. Using 0 as identity for min query
```
// WRONG for min query:
IDENTITY = 0     // min(5, 0) = 0, not 5!

// CORRECT:
IDENTITY = INT_MAX   // min(5, INT_MAX) = 5 ✓
```

### 4. Integer overflow in mid calculation
```
// Risky for large values:
mid = (start + end) / 2         // may overflow if start+end > INT_MAX

// Safe:
mid = start + (end - start) / 2  // no overflow
```

---

## Template: Plug-and-Play

To switch between different query types, only change two things:

```
// ===== For SUM =====
merge(a, b) = a + b
IDENTITY = 0

// ===== For MIN =====
merge(a, b) = min(a, b)
IDENTITY = INT_MAX

// ===== For MAX =====
merge(a, b) = max(a, b)
IDENTITY = INT_MIN

// ===== For GCD =====
merge(a, b) = gcd(a, b)
IDENTITY = 0        // gcd(x, 0) = x

// ===== For XOR =====
merge(a, b) = a ^ b
IDENTITY = 0        // x ^ 0 = x

Everything else stays EXACTLY the same!
```

---

## Summary Table

| Component | Description |
|-----------|-------------|
| Skeleton | All ops share: terminal check → split → recurse → merge |
| Parameters | (node, start, end) + operation-specific params |
| Build | Recurse both children, merge on return |
| Query | Three cases: no overlap, full overlap, partial overlap |
| Update | Recurse into ONE child, merge on return |
| Customization | Only merge function and identity change |
| Stack depth | O(log n) |
| Common bugs | Missing re-merge, wrong overlap check, wrong identity |

---

## Quick Revision Questions

1. What three parameters do all segment tree operations share? *(node, start, end)*
2. How does update differ from query in terms of recursion? *(Update goes to ONE child; query goes to BOTH children on partial overlap)*
3. What two things need to change to switch from sum to min? *(merge function: min(a,b) instead of a+b; identity: +∞ instead of 0)*
4. What happens if you forget to re-merge in the update function? *(Parent nodes will have stale values — queries will return wrong answers)*
5. What is the maximum recursion depth? *(O(log n) — the height of the tree)*

---

[← Previous Unit: Point Update](../02-Segment-Tree-Fundamentals/06-point-update.md) | [Next: Build Function →](02-build-function.md)
