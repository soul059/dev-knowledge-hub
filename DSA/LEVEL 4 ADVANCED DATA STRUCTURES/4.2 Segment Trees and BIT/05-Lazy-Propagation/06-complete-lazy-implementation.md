# Complete Lazy Implementation

## Chapter Overview

This chapter brings together all the pieces — build, push_down, range_update, and range_query — into a complete, ready-to-use lazy segment tree template. We provide versions for range-add and range-set on a sum tree.

---

## Template: Range Add + Sum Query

```
// Global arrays
tree[1..4*N]    // segment tree (sum)
lazy[1..4*N]    // lazy array (pending add)
A[0..N-1]       // input array

function build(node, start, end):
    lazy[node] = 0
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = tree[2*node] + tree[2*node+1]

function push_down(node, start, end):
    if lazy[node] != 0:
        mid = (start + end) / 2
        
        // Push to left child
        tree[2*node]   += lazy[node] * (mid - start + 1)
        lazy[2*node]   += lazy[node]
        
        // Push to right child
        tree[2*node+1] += lazy[node] * (end - mid)
        lazy[2*node+1] += lazy[node]
        
        lazy[node] = 0

function range_add(node, start, end, L, R, val):
    if end < L or start > R:
        return
    if L <= start and end <= R:
        tree[node] += val * (end - start + 1)
        lazy[node] += val
        return
    push_down(node, start, end)
    mid = (start + end) / 2
    range_add(2*node, start, mid, L, R, val)
    range_add(2*node+1, mid+1, end, L, R, val)
    tree[node] = tree[2*node] + tree[2*node+1]

function range_sum(node, start, end, L, R):
    if end < L or start > R:
        return 0
    if L <= start and end <= R:
        return tree[node]
    push_down(node, start, end)
    mid = (start + end) / 2
    return range_sum(2*node, start, mid, L, R) +
           range_sum(2*node+1, mid+1, end, L, R)

// Usage:
//   build(1, 0, n-1)
//   range_add(1, 0, n-1, L, R, val)
//   result = range_sum(1, 0, n-1, L, R)
```

---

## Template: Range Set + Sum Query

```
tree[1..4*N]
lazy[1..4*N]
has_lazy[1..4*N]   // boolean flag (can't use 0 as sentinel)

function build(node, start, end):
    has_lazy[node] = false
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = tree[2*node] + tree[2*node+1]

function push_down(node, start, end):
    if has_lazy[node]:
        mid = (start + end) / 2
        val = lazy[node]
        
        tree[2*node]      = val * (mid - start + 1)
        lazy[2*node]      = val
        has_lazy[2*node]   = true
        
        tree[2*node+1]    = val * (end - mid)
        lazy[2*node+1]    = val
        has_lazy[2*node+1] = true
        
        has_lazy[node] = false

function range_set(node, start, end, L, R, val):
    if end < L or start > R:
        return
    if L <= start and end <= R:
        tree[node] = val * (end - start + 1)
        lazy[node] = val
        has_lazy[node] = true
        return
    push_down(node, start, end)
    mid = (start + end) / 2
    range_set(2*node, start, mid, L, R, val)
    range_set(2*node+1, mid+1, end, L, R, val)
    tree[node] = tree[2*node] + tree[2*node+1]

function range_sum(node, start, end, L, R):
    if end < L or start > R:
        return 0
    if L <= start and end <= R:
        return tree[node]
    push_down(node, start, end)
    mid = (start + end) / 2
    return range_sum(2*node, start, mid, L, R) +
           range_sum(2*node+1, mid+1, end, L, R)
```

---

## Template: Range Add + Min Query

```
tree[1..4*N]    // stores range minimum
lazy[1..4*N]    // pending add

function build(node, start, end):
    lazy[node] = 0
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = min(tree[2*node], tree[2*node+1])

function push_down(node, start, end):
    if lazy[node] != 0:
        tree[2*node]   += lazy[node]    // No length multiplication!
        lazy[2*node]   += lazy[node]
        
        tree[2*node+1] += lazy[node]
        lazy[2*node+1] += lazy[node]
        
        lazy[node] = 0

function range_add(node, start, end, L, R, val):
    if end < L or start > R:
        return
    if L <= start and end <= R:
        tree[node] += val              // Min shifts by val
        lazy[node] += val
        return
    push_down(node, start, end)
    mid = (start + end) / 2
    range_add(2*node, start, mid, L, R, val)
    range_add(2*node+1, mid+1, end, L, R, val)
    tree[node] = min(tree[2*node], tree[2*node+1])

function range_min(node, start, end, L, R):
    if end < L or start > R:
        return +∞
    if L <= start and end <= R:
        return tree[node]
    push_down(node, start, end)
    mid = (start + end) / 2
    return min(range_min(2*node, start, mid, L, R),
               range_min(2*node+1, mid+1, end, L, R))
```

---

## Complete Walkthrough

```
A = [3, 1, 4, 1, 5]   n = 5

1. build(1, 0, 4):
              1:[0-4]=14
              /          \
        2:[0-2]=8      3:[3-4]=6
        /     \         /     \
    4:[0-1]=4  5:[2]=4 6:[3]=1  7:[4]=5
    / \
   3   1

   All lazy = 0.

2. range_add(1, 0, 4, 1, 3, +10):  "Add 10 to A[1..3]"

   Node 1 [0-4]: PARTIAL, push_down(skip)
     Node 2 [0-2]: PARTIAL, push_down(skip)
       Node 4 [0-1]: PARTIAL, push_down(skip)
         Node 8 [0-0]: NO OVERLAP
         Node 9 [1-1]: FULL → tree=1+10=11, lazy=10
       tree[4] = 3+11 = 14
       Node 5 [2-2]: FULL → tree=4+10=14, lazy=10
     tree[2] = 14+14 = 28
     Node 3 [3-4]: PARTIAL, push_down(skip)
       Node 6 [3-3]: FULL → tree=1+10=11, lazy=10
       Node 7 [4-4]: NO OVERLAP
     tree[3] = 11+5 = 16
   tree[1] = 28+16 = 44

   State:
              1:[0-4]=44
              /          \
        2:[0-2]=28     3:[3-4]=16
        /     \         /      \
    4:[0-1]=14 5:[2]=14 6:[3]=11 7:[4]=5
    / \
   3   11
   A effectively = [3, 11, 14, 11, 5]  sum = 44  ✓

3. range_sum(1, 0, 4, 0, 2):  "Sum of A[0..2]"

   Node 1 [0-4]: PARTIAL, push_down(skip)
     Node 2 [0-2]: FULL → return tree[2] = 28
   
   Answer: 28  ✓  (3 + 11 + 14 = 28)

4. range_add(1, 0, 4, 0, 4, +1):  "Add 1 to entire array"

   Node 1 [0-4]: FULL → tree=44+1×5=49, lazy=1
   Done! One node touched.

5. range_sum(1, 0, 4, 2, 3):  "Sum of A[2..3]"

   Node 1 [0-4]: PARTIAL
     push_down(1): lazy=1
       tree[2] += 1×3 = 28+3 = 31,  lazy[2] = 1
       tree[3] += 1×2 = 16+2 = 18,  lazy[3] = 1
       lazy[1] = 0
     Node 2 [0-2]: PARTIAL
       push_down(2): lazy=1
         tree[4] += 1×2 = 14+2 = 16, lazy[4] = 1
         tree[5] += 1×1 = 14+1 = 15, lazy[5] = 1
         lazy[2] = 0
       Node 4: NO OVERLAP
       Node 5 [2-2]: FULL → return 15
     Node 3 [3-4]: PARTIAL
       push_down(3): lazy=1
         tree[6] += 1×1 = 11+1 = 12, lazy[6] = 1+10=11? 
         
   Wait — node 6 already had lazy=10 from step 2!
   push_down(3): lazy[3]=1
     tree[6] += 1×1 = 11+1 = 12,  lazy[6] += 1 = 10+1 = 11
     tree[7] += 1×1 = 5+1 = 6,    lazy[7] += 1 = 0+1 = 1
     lazy[3] = 0
   
   Node 6 [3-3]: FULL → return tree[6] = 12
   Node 7 [4-4]: NO OVERLAP
   tree[3] part = 12
   
   Total = 15 + 12 = 27?
   Check: A = [4, 12, 15, 12, 6]  (after +1 to all)
   sum(2,3) = 15 + 12 = 27  ✓
```

---

## Complexity Summary

```
┌──────────────────┬──────────┬──────────┐
│    Operation     │   Time   │   Space  │
├──────────────────┼──────────┼──────────┤
│  Build           │  O(n)    │  O(n)    │
│  Range Update    │  O(log n)│    —     │
│  Range Query     │  O(log n)│    —     │
│  Point Update    │  O(log n)│    —     │
│  Point Query     │  O(log n)│    —     │
│  Total Space     │    —     │  O(8n)   │
└──────────────────┴──────────┴──────────┘

8n = 4n for tree[] + 4n for lazy[]
```

---

## Common Implementation Checklist

```
□ Array size: 4*N for both tree[] and lazy[]
□ lazy[] initialized to identity (0 for add, 1 for multiply)
□ push_down checks lazy != identity before doing work
□ push_down called in both update AND query (partial overlap)
□ push_down NOT called at leaves (start == end guard)
□ Full overlap in update: modify tree AND set lazy, then return
□ After recursing in update: re-merge tree[node] from children
□ Identity returned for no-overlap in query
□ tree[node] correctly reflects its range at all times
```

---

## Summary Table

| Template | Update Type | Query Type | Lazy Identity | Push Down Key |
|----------|------------|------------|---------------|---------------|
| Add + Sum | Range add | Range sum | 0 | tree += lazy × len |
| Set + Sum | Range set | Range sum | NONE flag | tree = lazy × len |
| Add + Min | Range add | Range min | 0 | tree += lazy |
| Add + Max | Range add | Range max | 0 | tree += lazy |

---

## Quick Revision Questions

1. How much extra space does lazy propagation need? *(An additional array of size 4n, so total 8n)*
2. What's the difference between push_down for sum vs min tree? *(Sum: tree[child] += lazy × child_length. Min: tree[child] += lazy — no length multiplication)*
3. Why does range-set need a has_lazy flag but range-add doesn't? *(Because lazy=0 for range-set means "set all to 0" — a valid operation, not "no pending update")*
4. What happens at full overlap during a range update? *(Update tree[node], set lazy[node], and return WITHOUT recursing)*
5. After recursing into both children during an update, what must you do? *(Re-merge: tree[node] = merge(tree[left], tree[right]))*
6. Is the O(log n) bound per operation worst-case or amortized? *(Worst-case — each operation visits at most O(log n) nodes)*

---

[← Previous: Range Query with Lazy](05-range-query-with-lazy.md) | [Back to README →](../README.md)
