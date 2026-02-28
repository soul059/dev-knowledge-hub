# Range Query with Lazy

## Chapter Overview

Queries in a lazy segment tree are nearly identical to the standard version, with one critical addition: **push down before recursing**. This ensures children have correct values before we read them.

---

## Query with Lazy — Pseudocode

```
function query(node, start, end, L, R):
    // Case 1: No overlap
    if end < L or start > R:
        return IDENTITY
    
    // Case 2: Complete overlap
    if L <= start and end <= R:
        return tree[node]          // already correct!
    
    // Case 3: Partial overlap — push down first!
    push_down(node, start, end)    // ← This is the only change!
    mid = (start + end) / 2
    left_result  = query(2*node, start, mid, L, R)
    right_result = query(2*node+1, mid+1, end, L, R)
    return merge(left_result, right_result)
```

---

## Why Push Down is Needed for Queries

```
Without push_down:

Suppose node 3 has lazy=5, tree[3]=25 (range [4-7], sum tree)
  tree[6]=8 (range [4-5])   — STALE, should be 8+5×2=18
  tree[7]=7 (range [6-7])   — STALE, should be 7+5×2=17

Query(4, 5):
  Reaches node 6 → returns tree[6] = 8  ← WRONG! Should be 18

With push_down:
  At node 3 (partial overlap), push_down first:
    tree[6] = 8 + 5×2 = 18,   lazy[6] = 5
    tree[7] = 7 + 5×2 = 17,   lazy[7] = 5
    lazy[3] = 0
  
  Now query reaches node 6 (full overlap) → returns 18  ✓
```

---

## When Push Down is NOT Needed

```
Case 2 (full overlap): tree[node] is ALREADY correct!

This is the key invariant:
  tree[node] always reflects the correct aggregate for its range.
  lazy[node] only affects CHILDREN.

So if our query fully covers the node's range:
  → Just return tree[node]
  → No need to look at children
  → No need to push down

This is why lazy propagation is efficient:
  We only push down on the O(log n) "boundary" path.
```

---

## Complete Trace: Query After Updates

```
Starting state (from previous chapter's example):

              1:[0-5]=33
              /          \
        2:[0-2]=13     3:[3-5]=20
        /     \         /       \
    4:[0-1]=9  5:[2]=4 6:[3-4]=14  7:[5]=6
    / \         lazy=3  lazy=3
   2   7

Effective array: [2, 7, 4, 8, 6, 6]

Query: sum(2, 4)  → should be 4 + 8 + 6 = 18

query(1, 0, 5, 2, 4):
  PARTIAL overlap
  push_down(1): lazy[1]=0, skip
  
  query(2, 0, 2, 2, 4):
    PARTIAL
    push_down(2): lazy[2]=0, skip
    
    query(4, 0, 1, 2, 4): NO OVERLAP → return 0
    query(5, 2, 2, 2, 4): FULL → return tree[5]=4
    merge: 0 + 4 = 4
  
  query(3, 3, 5, 2, 4):
    PARTIAL
    push_down(3): lazy[3]=0, skip    ← no pending lazy here
    
    query(6, 3, 4, 2, 4): FULL → return tree[6]=14
      BUT WAIT: we need the CORRECT value for [3,4]
      tree[6]=14 with lazy[6]=3
      Since this is FULL overlap, we return tree[6]=14
      This IS correct! (tree[6] was updated to 14 = 8+3×2)

    query(7, 5, 5, 2, 4): NO OVERLAP → return 0
    merge: 14 + 0 = 14
  
  merge: 4 + 14 = 18

Answer: 18  ✓  (4 + 8 + 6 = 18)

Notice: Node 6 had lazy=3 but we never pushed it down!
Because the query fully covered node 6's range,
tree[6]=14 was already the correct sum.
```

---

## Query That Triggers Push Down

```
Query: sum(3, 3)  → should be 8 (A[3] = 5 + 3 = 8)

query(1, 0, 5, 3, 3):
  PARTIAL → push_down(1): clean
  
  query(2, 0, 2, 3, 3): NO OVERLAP → 0
  
  query(3, 3, 5, 3, 3):
    PARTIAL → push_down(3): clean
    
    query(6, 3, 4, 3, 3):
      PARTIAL → push_down(6)!  ← LAZY TAG = 3!
      
      Push down:
        tree[12] += 3×1 = 5+3 = 8    lazy[12] = 3
        tree[13] += 3×1 = 3+3 = 6    lazy[13] = 3
        lazy[6] = 0
      
      query(12, 3, 3, 3, 3): FULL → return 8
      query(13, 4, 4, 3, 3): NO OVERLAP → 0
      merge: 8
    
    query(7, 5, 5, 3, 3): NO OVERLAP → 0
    merge: 8
  
  merge: 0 + 8 = 8

Answer: 8  ✓

The push down at node 6 was NECESSARY because we needed
to go into node 6's children, and they were stale.
```

---

## Interleaving Updates and Queries

```
The beauty of lazy propagation: updates and queries interleave freely.

Sequence:
  1. range_add(0, 3, +5)     → O(log n)
  2. query(1, 2)              → O(log n), pushes some lazy
  3. range_add(2, 5, +10)    → O(log n), may push some lazy
  4. query(0, 5)              → O(log n)

Each operation is O(log n) regardless of history.
The lazy tags accumulate and push down as needed.

Amortized analysis is NOT needed:
  Each operation is worst-case O(log n), not amortized.
```

---

## Query vs Update: The Symmetry

```
Side-by-side comparison:

                QUERY                        UPDATE
    ┌─────────────────────────┬─────────────────────────────┐
    │ No overlap:             │ No overlap:                 │
    │   return IDENTITY       │   return (do nothing)       │
    │                         │                             │
    │ Full overlap:           │ Full overlap:               │
    │   return tree[node]     │   modify tree[node]         │
    │                         │   set lazy[node]            │
    │                         │   DON'T recurse             │
    │                         │                             │
    │ Partial overlap:        │ Partial overlap:            │
    │   push_down             │   push_down                 │
    │   recurse both          │   recurse both              │
    │   merge results         │   merge children into node  │
    └─────────────────────────┴─────────────────────────────┘

Almost identical! The only difference is what happens at full overlap.
```

---

## Point Query Shortcut

```
A point query is query(i, i). With lazy:

function point_query(node, start, end, idx):
    if start == end:
        return tree[node]
    push_down(node, start, end)
    mid = (start + end) / 2
    if idx <= mid:
        return point_query(2*node, start, mid, idx)
    else:
        return point_query(2*node+1, mid+1, end, idx)

Alternative: Accumulate lazy on the path down:

function point_query_no_push(node, start, end, idx):
    if start == end:
        return tree[node]
    // Don't push — just accumulate
    mid = (start + end) / 2
    add = lazy[node]
    if idx <= mid:
        return add + point_query_no_push(2*node, start, mid, idx)
    else:
        return add + point_query_no_push(2*node+1, mid+1, end, idx)

The second version is read-only (doesn't modify the tree).
Works for range-add. For range-set, must use push_down version.
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Query change vs standard | Add push_down before partial-overlap recursion |
| Full overlap behavior | Return tree[node] directly (already correct) |
| When push_down triggers | Only at partial overlaps on the query path |
| Nodes visited | O(log n) — same as standard query |
| Point query optimization | Can accumulate lazy on path without modifying tree |
| Correctness guarantee | tree[node] is always correct for its range |

---

## Quick Revision Questions

1. What's the only change to the query function for lazy propagation? *(Add push_down(node) before recursing on partial overlap)*
2. Why doesn't a full-overlap query need push_down? *(tree[node] already has the correct aggregate — lazy only affects children)*
3. Is the query time still O(log n) with lazy? *(Yes — push_down is O(1) per node, and we visit O(log n) nodes)*
4. Can a query modify the tree? *(Yes — push_down modifies children's tree[] and lazy[] values)*
5. What's the advantage of the "accumulate lazy" point query? *(It's read-only — doesn't modify the tree, useful for concurrent reads)*

---

[← Previous: Range Update with Lazy](04-range-update-with-lazy.md) | [Next: Complete Lazy Implementation →](06-complete-lazy-implementation.md)
