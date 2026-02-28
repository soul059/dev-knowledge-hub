# Range Update with Lazy

## Chapter Overview

This chapter presents the complete range update algorithm with lazy propagation. We cover range-add and range-set updates with detailed pseudocode and step-by-step traces.

---

## Range Add Update — Pseudocode

```
function range_add(node, start, end, L, R, val):
    // Case 1: No overlap
    if end < L or start > R:
        return
    
    // Case 2: Complete overlap — LAZY!
    if L <= start and end <= R:
        tree[node] += val * (end - start + 1)
        lazy[node] += val
        return
    
    // Case 3: Partial overlap — must recurse
    push_down(node, start, end)      // ← KEY: push lazy first!
    mid = (start + end) / 2
    range_add(2*node, start, mid, L, R, val)
    range_add(2*node+1, mid+1, end, L, R, val)
    tree[node] = tree[2*node] + tree[2*node+1]    // merge children

// Called as: range_add(1, 0, n-1, L, R, val)
```

---

## The Three Cases Visualized

```
Update range [L, R]:

Case 1: NO OVERLAP       Case 2: FULL OVERLAP     Case 3: PARTIAL OVERLAP
  [start ... end]           [start ... end]           [start ... end]
               [L .. R]     [L .......... R]            [L ..... R]
  → return                  → update node              → push_down
                              set lazy                   → recurse both
                              DON'T recurse!              → merge children

The magic of lazy: Case 2 stops recursion!
Without lazy: Case 2 would still recurse to leaves → O(n)
With lazy:    Case 2 does O(1) work → total O(log n)
```

---

## Complete Trace: Range Add

```
Array A = [2, 4, 1, 5, 3, 6]   n = 6

Initial sum segment tree:
              1:[0-5]=21
              /          \
        2:[0-2]=7      3:[3-5]=14
        /     \         /      \
    4:[0-1]=6  5:[2]=1 6:[3-4]=8  7:[5]=6
    / \                 / \
   2   4               5   3

Operation: range_add(1, 0, 5, 1, 4, +3)
  "Add 3 to all elements in A[1..4]"

Step 1: Node 1 [0-5], L=1,R=4 → PARTIAL
  push_down(1): lazy[1]=0, skip
  Recurse both children

Step 2: Node 2 [0-2], L=1,R=4 → PARTIAL
  push_down(2): lazy[2]=0, skip
  Recurse both children

Step 3: Node 4 [0-1], L=1,R=4 → PARTIAL
  push_down(4): lazy[4]=0, skip
  - Node 8 [0-0]: L=1,R=4 → NO OVERLAP, return
  - Node 9 [1-1]: L=1,R=4 → FULL OVERLAP
    tree[9] += 3×1 = 4+3 = 7
    lazy[9] = 3 (leaf, doesn't matter)
  tree[4] = tree[8]+tree[9] = 2+7 = 9

Step 4: Node 5 [2-2], L=1,R=4 → FULL OVERLAP
  tree[5] += 3×1 = 1+3 = 4
  lazy[5] = 3

Step 5: tree[2] = tree[4]+tree[5] = 9+4 = 13

Step 6: Node 3 [3-5], L=1,R=4 → PARTIAL
  push_down(3): lazy[3]=0, skip
  - Node 6 [3-4]: L=1,R=4 → FULL OVERLAP!  ← LAZY stops here!
    tree[6] += 3×2 = 8+6 = 14
    lazy[6] = 3                    ← stored, not pushed to children
  - Node 7 [5-5]: L=1,R=4 → NO OVERLAP, return
  tree[3] = tree[6]+tree[7] = 14+6 = 20

Step 7: tree[1] = tree[2]+tree[3] = 13+20 = 33

Final tree:
              1:[0-5]=33
              /          \
        2:[0-2]=13     3:[3-5]=20
        /     \         /       \
    4:[0-1]=9  5:[2]=4 6:[3-4]=14  7:[5]=6
    / \         lazy=3  lazy=3
   2   7

Effective array: [2, 7, 4, 8, 6, 6]
  Original:      [2, 4, 1, 5, 3, 6]
  After +3:      [2, 7, 4, 8, 6, 6]  ✓

Node 6 has lazy=3: its children (nodes 12,13) still show 5,3
but will be corrected when accessed via push_down.
```

---

## Stacking Updates (No Push Down Needed)

```
Continuing from above, add another update:
  range_add(1, 0, 5, 3, 5, +2)  → "Add 2 to A[3..5]"

At Node 3 [3-5]: FULL OVERLAP
  tree[3] += 2×3 = 20+6 = 26
  lazy[3] += 2 = 0+2 = 2

At Node 6 [3-4]: NOT VISITED (parent had full overlap)
  lazy[6] still = 3  (from previous update)

Now tree[1] = tree[2]+tree[3] = 13+26 = 39

Later, if a query needs node 6's children:
  push_down(3): pushes lazy=2 to nodes 6 and 7
    tree[6] += 2×2 = 14+4 = 18     lazy[6] += 2 = 3+2 = 5
    tree[7] += 2×1 = 6+2 = 8       lazy[7] += 2 = 0+2 = 2
    lazy[3] = 0
  
  push_down(6): pushes lazy=5 to nodes 12 and 13
    tree[12] += 5 = 5+5 = 10       lazy[12] = 5
    tree[13] += 5 = 3+5 = 8        lazy[13] = 5
    lazy[6] = 0

  Effective: A[3]=5+5=10, A[4]=3+5=8  ✓  (original 5+3+2=10, 3+3+2=8)
```

---

## Range Set Update — Pseudocode

```
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

function push_down_set(node, start, end):
    if has_lazy[node]:
        mid = (start + end) / 2
        val = lazy[node]
        
        tree[2*node]   = val * (mid - start + 1)
        lazy[2*node]   = val
        has_lazy[2*node] = true
        
        tree[2*node+1] = val * (end - mid)
        lazy[2*node+1] = val
        has_lazy[2*node+1] = true
        
        has_lazy[node] = false
```

---

## Range Set Trace

```
A = [2, 4, 1, 5, 3, 6]
Operation: range_set(1, 0, 5, 1, 4, 10)  → "Set A[1..4] = 10"

Step 1: Node 1 [0-5] → PARTIAL
  Recurse both children

Step 2: Node 2 [0-2] → PARTIAL
  - Node 4 [0-1] → PARTIAL
    - Node 8 [0-0]: NO OVERLAP
    - Node 9 [1-1]: FULL → tree[9]=10, lazy=10
    tree[4] = 2+10 = 12
  - Node 5 [2-2]: FULL → tree[5]=10, lazy=10
  tree[2] = 12+10 = 22

Step 3: Node 3 [3-5] → PARTIAL
  - Node 6 [3-4]: FULL → tree[6]=10×2=20, lazy=10
  - Node 7 [5-5]: NO OVERLAP
  tree[3] = 20+6 = 26

tree[1] = 22+26 = 48

Effective array: [2, 10, 10, 10, 10, 6]
Sum = 2+10+10+10+10+6 = 48  ✓
```

---

## Nodes Visited Analysis

```
For any range update [L, R]:

  Nodes visited = O(log n)

Why? Same argument as range query:
  At each level of the tree, at most 4 nodes are visited
  (at most 2 "boundary" nodes per side)
  
  Depth = O(log n) → Total = O(4 × log n) = O(log n)

  Specifically:
  - Some nodes have FULL overlap → O(1) work each (lazy tag)
  - Some nodes have NO overlap → O(1) work each (return)  
  - Some nodes have PARTIAL overlap → O(1) + recurse
  - Partial nodes form two paths from root to boundary leaves

  Total work: O(log n)
```

---

## Summary Table

| Operation | Time | What Stops Recursion | Lazy Composition |
|-----------|------|---------------------|------------------|
| Range add | O(log n) | Full overlap → apply + lazy tag | lazy += val |
| Range set | O(log n) | Full overlap → overwrite + lazy tag | lazy = val |
| Push down cost | O(1) per node | Applied before partial-overlap recurse | — |

---

## Quick Revision Questions

1. In which case does lazy propagation stop recursion early? *(Full overlap — the update covers the node's entire range)*
2. What must happen before recursing into children during a partial overlap? *(push_down must be called to clear pending lazy updates)*
3. After recursing into both children during an update, what's the last step? *(Merge: tree[node] = tree[2*node] + tree[2*node+1])*
4. How many nodes are visited during a range update? *(O(log n) — at most 4 per level)*
5. What's the difference in lazy composition between add and set? *(Add: lazy += val (accumulate). Set: lazy = val (replace))*

---

[← Previous: Push Down Logic](03-push-down-logic.md) | [Next: Range Query with Lazy →](05-range-query-with-lazy.md)
