# Range Updates with Queries

## Chapter Overview

Many problems combine range updates with range or point queries. This chapter categorizes the patterns, shows which data structure to use for each combination, and provides solution templates.

---

## The Four Combinations

```
┌────────────────────┬──────────────────┬───────────────────┐
│                    │   Point Query    │   Range Query      │
├────────────────────┼──────────────────┼───────────────────┤
│  Point Update      │  Direct array    │  BIT / Seg Tree   │
│                    │  access O(1)     │  O(log n)          │
├────────────────────┼──────────────────┼───────────────────┤
│  Range Update      │  Difference BIT  │  Lazy Seg Tree    │
│                    │  O(log n)        │  OR 2 BITs O(log n)│
└────────────────────┴──────────────────┴───────────────────┘
```

---

## Pattern 1: Point Update + Point Query

```
Simplest case — just use the array directly.

update(i, val): A[i] = val      // O(1)
query(i):       return A[i]     // O(1)

No data structure needed!
(Unless the query is aggregate like "sum of all" — then use BIT)
```

---

## Pattern 2: Point Update + Range Query

```
"Update A[i], then query sum/min/max of A[L..R]"

For sum: BIT
  update(i, delta):  BIT.add(i, delta)
  query(L, R):       BIT.prefix(R) - BIT.prefix(L-1)

For min/max: Segment Tree
  update(i, val):    seg.update(i, val)
  query(L, R):       seg.query(L, R)

Both: O(log n) per operation
```

---

## Pattern 3: Range Update + Point Query

```
"Add val to all elements in [L, R], then query A[i]"

Use DIFFERENCE ARRAY with BIT:

Difference array: D[i] = A[i] - A[i-1]
Then A[i] = D[1] + D[2] + ... + D[i] = prefix_sum(D, i)

range_add(L, R, val):
    D[L] += val     →  BIT.add(L, +val)
    D[R+1] -= val   →  BIT.add(R+1, -val)

point_query(i):
    return BIT.prefix(i)    // = sum of D[1..i] = A[i]

Time: O(log n) per operation
```

### Trace

```
A = [0, 0, 0, 0, 0]   (5 elements, 1-indexed)

range_add(2, 4, 3):   Add 3 to A[2..4]
  BIT.add(2, +3)
  BIT.add(5, -3)
  
  Logical D: [0, 3, 0, 0, -3]
  Logical A: [0, 3, 3, 3, 0]

range_add(3, 5, 2):   Add 2 to A[3..5]
  BIT.add(3, +2)
  BIT.add(6, -2)    // index 6 > n, but harmless
  
  Logical D: [0, 3, 2, 0, -3]
  Logical A: [0, 3, 5, 5, 2]

point_query(3):
  BIT.prefix(3) = D[1] + D[2] + D[3] = 0 + 3 + 2 = 5 ✓

point_query(5):
  BIT.prefix(5) = 0 + 3 + 2 + 0 + (-3) = 2 ✓
```

---

## Pattern 4: Range Update + Range Query

```
"Add val to all elements in [L, R], then query sum of A[L'..R']"

Method A: Lazy Segment Tree
  Standard approach, O(log n) per operation.
  Handles any merge function.

Method B: Two BITs (for sum only)
  Uses the identity:
    A[i] = prefix_sum of differences
    sum(A[1..i]) = Σ_{j=1}^{i} Σ_{k=1}^{j} D[k]
                 = Σ_{k=1}^{i} D[k] × (i - k + 1)
                 = (i+1) × Σ D[k] - Σ k × D[k]

  Maintain two BITs:
    BIT1: stores D[k]
    BIT2: stores k × D[k]

  prefix_sum(i) = (i + 1) × BIT1.prefix(i) - BIT2.prefix(i)
  range_sum(L, R) = prefix_sum(R) - prefix_sum(L - 1)

  range_add(L, R, val):
    BIT1.add(L, val)
    BIT1.add(R+1, -val)
    BIT2.add(L, val × L)
    BIT2.add(R+1, -val × (R+1))
```

### Derivation of Two-BIT Formula

```
After range_add(L, R, val), the difference array:
  D[L] += val, D[R+1] -= val

A[i] = Σ_{k=1}^{i} D[k]

sum(1, i) = Σ_{j=1}^{i} A[j]
          = Σ_{j=1}^{i} Σ_{k=1}^{j} D[k]

Swap summation order (k goes from 1 to j, j from 1 to i):
  = Σ_{k=1}^{i} D[k] × (i - k + 1)
  = (i + 1) × Σ_{k=1}^{i} D[k] - Σ_{k=1}^{i} k × D[k]
  = (i + 1) × BIT1.prefix(i) - BIT2.prefix(i)
```

---

## Interleaved Operations

```
Most problems INTERLEAVE updates and queries:

"Perform operations in order:
  1. update(2, 4, +3)
  2. query(1, 5) → ?
  3. update(1, 3, +2)
  4. query(2, 4) → ?
  5. update(3, 5, -1)
  6. query(1, 5) → ?"

Choose structure based on:
  - Update type (point/range) + Query type (point/range)
  - Operation type (sum/min/max)
  - Use the table from "Four Combinations" above
```

---

## Range Set + Range Query

```
"Set all elements in [L, R] to val, query sum of [L', R']"

This needs LAZY SEGMENT TREE (not BIT).

Why? "Set" overwrites previous values — it's not additive.
  BIT's difference trick only works for additive updates.

Lazy approach:
  lazy[node] = value to set (or -1 / sentinel for "no pending set")
  
  push_down: set children's values to lazy value
  
  Important: "set" lazy REPLACES any pending "add" lazy.
  If combining set + add: set takes priority, cancels pending add.
```

---

## Range Multiply + Range Sum

```
"Multiply all elements in [L, R] by val, query sum of [L', R']"

Lazy segment tree with multiply lazy:

push_down(node):
    for each child c:
        tree[c] *= lazy_mul[node]
        lazy_mul[c] *= lazy_mul[node]
    lazy_mul[node] = 1

Combined add + multiply:
  Must track both. Order matters!
  Convention: apply multiply FIRST, then add.
  
  push_down(node):
      for child c:
          lazy_mul[c] *= lazy_mul[node]
          lazy_add[c] = lazy_add[c] * lazy_mul[node] + lazy_add[node]
          tree[c] = tree[c] * lazy_mul[node] + lazy_add[node] * length[c]
      lazy_mul[node] = 1
      lazy_add[node] = 0
```

---

## Pattern Recognition Guide

```
Signal                      → Structure
────────────────────────    ──────────
"add X to range [L,R]"     → Lazy seg tree or 2 BITs
"set range to X"            → Lazy seg tree only
"multiply range by X"       → Lazy seg tree only
"XOR range with X"          → Lazy seg tree (XOR lazy)
"query sum after updates"   → BIT (if add only) or lazy seg tree
"query min after updates"   → Lazy seg tree only
"increment + query"         → Difference BIT for point query
"offline queries"           → Consider sorting + sweep
```

---

## Summary Table

| Update | Query | Best Structure | Complexity |
|--------|-------|---------------|-----------|
| Point | Point | Array | O(1) |
| Point | Range Sum | BIT | O(log n) |
| Point | Range Min/Max | Segment Tree | O(log n) |
| Range Add | Point | Difference BIT | O(log n) |
| Range Add | Range Sum | 2 BITs or Lazy SegTree | O(log n) |
| Range Add | Range Min | Lazy Segment Tree | O(log n) |
| Range Set | Range Sum | Lazy Segment Tree | O(log n) |
| Range Multiply | Range Sum | Lazy Segment Tree | O(log n) |

---

## Quick Revision Questions

1. How does the difference array trick work for range add + point query? *(D[L] += val, D[R+1] -= val; point query = prefix sum of D)*
2. Why can't BIT handle range set operations? *(Set overwrites values non-additively; BIT difference trick only works for additive operations)*
3. What are the two BITs for range add + range sum? *(BIT1 stores D[k], BIT2 stores k×D[k]; prefix_sum(i) = (i+1)×BIT1.prefix(i) - BIT2.prefix(i))*
4. How do you combine multiply and add lazy values? *(Convention: apply multiply first, then add; when pushing down, transform add by multiply)*
5. Which pattern requires lazy segment tree and cannot use BIT? *(Range set, range multiply, range XOR, or any query involving min/max with range updates)*

---

[← Previous: Inversion Count](01-inversion-count.md) | [Next: Number of Elements in Range →](03-number-of-elements-in-range.md)
