# Range Maximum Queries

## Chapter Overview

Range Maximum Query finds the largest element in a subarray. It mirrors RMQ (minimum) almost exactly — the merge function is `max` and the identity element is `-∞`. Understanding how max and min are structural twins strengthens intuition.

---

## Max Properties

```
1. ASSOCIATIVE:   max(max(a,b), c) = max(a, max(b,c))    ✓
2. COMMUTATIVE:   max(a, b) = max(b, a)                  ✓
3. IDENTITY:      max(a, -∞) = a                          ✓
4. IDEMPOTENT:    max(a, a) = a                           ✓
5. NOT INVERTIBLE: can't "subtract" a max                  ✗

Consequence: same as min
  ✓ Segment tree   ✓ Sparse table (O(1) static)
  ✗ BIT (not directly)   ✗ Prefix approach
```

---

## Segment Tree for Max

```
Merge function:     merge(a, b) = max(a, b)
Identity element:   IDENTITY = -∞  (or INT_MIN)

function build(node, start, end):
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = max(tree[2*node], tree[2*node+1])

function query(node, start, end, L, R):
    if end < L or start > R:
        return -∞                    // identity for max
    if L <= start and end <= R:
        return tree[node]
    mid = (start + end) / 2
    return max(query(2*node, start, mid, L, R),
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
    tree[node] = max(tree[2*node], tree[2*node+1])
```

---

## Worked Example

```
A = [3, 1, 7, 4, 9, 2, 6, 5]   n = 8

Max segment tree:

                1:[0-7] = 9
               /          \
         2:[0-3] = 7     3:[4-7] = 9
         /    \           /      \
     4:[0-1]=3 5:[2-3]=7  6:[4-5]=9 7:[6-7]=6
     / \       / \         / \       / \
    3   1     7   4       9   2     6   5

Query: max(1, 5)

query(1, 0, 7, 1, 5):
  PARTIAL
  query(2, 0, 3, 1, 5):
    PARTIAL
    query(4, 0, 1, 1, 5):
      PARTIAL
      query(8, 0, 0, 1, 5): NO OVERLAP → -∞
      query(9, 1, 1, 1, 5): COMPLETE → 1
      max(-∞, 1) = 1
    query(5, 2, 3, 1, 5): COMPLETE → 7
    max(1, 7) = 7
  query(3, 4, 7, 1, 5):
    PARTIAL
    query(6, 4, 5, 1, 5): COMPLETE → 9
    query(7, 6, 7, 1, 5): NO OVERLAP → -∞
    max(9, -∞) = 9
  max(7, 9) = 9

Answer: 9  ✓  (A[4] = 9 is the maximum in range [1,5])
```

---

## Maintaining Both Min and Max

Sometimes you need **both** min and max of a range (e.g., range spread = max - min):

```
struct Node:
    min_val: integer
    max_val: integer

function merge(a, b):
    return Node(
        min(a.min_val, b.min_val),
        max(a.max_val, b.max_val)
    )

Identity: Node(+∞, -∞)

Leaf: Node(A[i], A[i])

query(L, R) → Node with both min and max
Range spread = result.max_val - result.min_val
```

---

## Max with Second Maximum

A harder variant: track the **two largest distinct values**:

```
struct Node:
    max1: integer    // largest
    max2: integer    // second largest (< max1)

function merge(a, b):
    vals = sorted([a.max1, a.max2, b.max1, b.max2], descending)
    // pick two largest distinct values
    max1 = vals[0]
    max2 = first val in vals[1..] that is < max1
         = -∞ if no such value
    return Node(max1, max2)

Example:
  Merge Node(9, 5) and Node(7, 3):
    vals = [9, 7, 5, 3]
    max1 = 9, max2 = 7
    Result: Node(9, 7)

  Merge Node(5, 3) and Node(5, 2):
    vals = [5, 5, 3, 2]
    max1 = 5, max2 = 3  (skip duplicate 5)
    Result: Node(5, 3)
```

---

## Symmetry: Min and Max as Duals

```
Everything about min has a max twin:

┌──────────────────┬─────────────┬─────────────┐
│     Property     │     Min     │     Max     │
├──────────────────┼─────────────┼─────────────┤
│  Merge           │  min(a, b)  │  max(a, b)  │
│  Identity        │  +∞         │  -∞         │
│  Absorbing       │  -∞         │  +∞         │
│  Idempotent      │  Yes        │  Yes        │
│  Invertible      │  No         │  No         │
│  Sparse Table    │  O(1)       │  O(1)       │
│  BIT             │  No         │  No         │
│  Associative     │  Yes        │  Yes        │
└──────────────────┴─────────────┴─────────────┘

If you have code for min segment tree, just swap:
  - min → max
  - +∞ → -∞
```

---

## Applications

| Problem | How Max Helps |
|---------|--------------|
| Maximum temperature in period | Direct range max query |
| Peak finding | Max in range + index tracking |
| Range spread (volatility) | max - min in range |
| Maximum subarray product | Track max/min (sign flips) |
| Skyline problem variation | Range max height |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Merge function | max(a, b) |
| Identity element | -∞ (INT_MIN) |
| Invertible? | No |
| Works with BIT? | Not natively |
| Dual of | Range minimum |
| Multi-value variant | Store (max1, max2) for second maximum |
| Combined with min | Store (min, max) to compute range spread |

---

## Quick Revision Questions

1. What is the identity element for range max? *(-∞ or INT_MIN)*
2. Can overlapping ranges cause errors in max query? *(No — max is idempotent, so double-counting is harmless)*
3. How do you get the range spread (max - min) in O(log n)? *(Store both min and max in each node, merge separately)*
4. To convert a min segment tree to max, what changes? *(Replace min→max and +∞→-∞)*
5. How do you track the second maximum? *(Store (max1, max2) in each node; merge picks two largest distinct values)*

---

[← Previous: Range Minimum Queries](02-range-minimum-queries.md) | [Next: Range GCD Queries →](04-range-gcd-queries.md)
