# Range GCD Queries

## Chapter Overview

Range GCD (Greatest Common Divisor) queries compute gcd over a subarray. GCD is associative and idempotent but not invertible — making segment tree a natural fit and BIT unusable.

---

## GCD Properties

```
1. ASSOCIATIVE:   gcd(gcd(a,b), c) = gcd(a, gcd(b,c))    ✓
2. COMMUTATIVE:   gcd(a, b) = gcd(b, a)                   ✓
3. IDENTITY:      gcd(a, 0) = a                            ✓  (0 is identity!)
4. IDEMPOTENT:    gcd(a, a) = a                            ✓
5. NOT INVERTIBLE: can't "undo" a gcd                       ✗

Key insight: identity = 0 because gcd(x, 0) = x for all x ≥ 0
This is mathematically correct since every integer divides 0.
```

---

## Segment Tree for GCD

```
Merge function:     merge(a, b) = gcd(a, b)
Identity element:   IDENTITY = 0

function gcd(a, b):
    while b ≠ 0:
        a, b = b, a mod b
    return a

function build(node, start, end):
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = gcd(tree[2*node], tree[2*node+1])

function query(node, start, end, L, R):
    if end < L or start > R:
        return 0                    // identity for gcd
    if L <= start and end <= R:
        return tree[node]
    mid = (start + end) / 2
    return gcd(query(2*node, start, mid, L, R),
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
    tree[node] = gcd(tree[2*node], tree[2*node+1])
```

---

## Worked Example

```
A = [12, 8, 6, 18, 15, 10]   n = 6

Build GCD segment tree:

Step 1: Leaves
  [12] [8] [6] [18] [15] [10]

Step 2: Internal nodes (bottom-up)
  gcd(12, 8) = 4       gcd(6, 18) = 6      gcd(15, 10) = 5
  gcd(4, 6) = 2        gcd(5, ?) → handle odd size

Full tree (with padding — n=6, array size 4×6=24):

              1:[0-5] = 1
              /          \
        2:[0-2] = 2     3:[3-5] = 1
        /    \           /      \
    4:[0-1]=4 5:[2-2]=6 6:[3-4]=3 7:[5-5]=10
    / \                  / \
   12  8               18  15

Wait — let me trace more carefully:

  gcd(18, 15) = 3
  gcd(3, 10)  = 1
  gcd(4, 6)   = 2
  gcd(2, 1)   = 1

Tree:
              1:[0-5] = 1
              /          \
        2:[0-2] = 2     3:[3-5] = 1
        /    \           /      \
    4:[0-1]=4 5:[2]=6  6:[3-4]=3 7:[5]=10
    / \                 / \
   12  8              18  15

Query: gcd(1, 4) = gcd(8, 6, 18, 15) = ?

gcd(8, 6) = 2, gcd(2, 18) = 2, gcd(2, 15) = 1

Let's trace the tree:
  query(1, 0, 5, 1, 4):
    PARTIAL
    query(2, 0, 2, 1, 4):
      PARTIAL
      query(4, 0, 1, 1, 4):
        PARTIAL
        query(8, 0, 0, 1, 4): NO OVERLAP → 0
        query(9, 1, 1, 1, 4): COMPLETE → 8
        gcd(0, 8) = 8
      query(5, 2, 2, 1, 4): COMPLETE → 6
      gcd(8, 6) = 2
    query(3, 3, 5, 1, 4):
      PARTIAL
      query(6, 3, 4, 1, 4): COMPLETE → 3
      query(7, 5, 5, 1, 4): NO OVERLAP → 0
      gcd(3, 0) = 3
    gcd(2, 3) = 1

Answer: 1  ✓
```

---

## GCD Complexity Nuance

```
GCD itself is not O(1) — Euclidean algorithm takes O(log(min(a,b))):

Total query cost: O(log n × log V)
  where V = max value in array

For most competitive programming:
  - Values ≤ 10^9 → log V ≈ 30
  - This constant is small enough to treat as O(1)
  - So effective query: O(log n)

Build: O(n × log V) ≈ O(n)
```

---

## GCD with Range Updates

Range GCD with point updates is straightforward. But what about:

**"Add x to all elements in range [L,R], then query GCD of range [L',R']"?**

```
Key identity:
  gcd(a, b) = gcd(a, b - a)   for b ≥ a

Generalized:
  gcd(a₁, a₂, ..., aₙ) = gcd(a₁, a₂-a₁, a₃-a₂, ..., aₙ-aₙ₋₁)

Technique: Maintain a difference array D[i] = A[i] - A[i-1]:
  - Range add [L,R] by x:  D[L] += x, D[R+1] -= x   (two point updates!)
  - Query gcd(L, R) = gcd(A[L], gcd(D[L+1], ..., D[R]))

Store two segment trees:
  1. BIT/SegTree for prefix sums (to recover A[L])
  2. GCD SegTree on |D[i]| values for i > L

This gives O(log n) for both operations!
```

---

## LCM Queries (Bonus)

```
LCM (Least Common Multiple) is also associative:
  lcm(a, b) = a × b / gcd(a, b)

Identity for LCM: lcm(a, 1) = a  → identity = 1

Caution: LCM values grow FAST
  lcm(1,2,3,...,20) = 232792560 (small)
  lcm(1,2,...,40) = 5342931457063200 (large!)
  
  Use modular LCM or check for overflow.

Segment tree for LCM:
  merge(a, b) = lcm(a, b)
  identity = 1
```

---

## Applications

| Problem | How GCD Helps |
|---------|--------------|
| Find subarray with GCD = 1 | Shrinking window with GCD updates |
| Count subarrays with GCD = k | Segment tree + observation: GCD decreases in O(log V) distinct steps |
| GCD of range with updates | Difference array + GCD segment tree |
| Coprime range check | gcd(L, R) = 1 means range has no common factor |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Merge function | gcd(a, b) |
| Identity element | 0 (because gcd(x, 0) = x) |
| Invertible? | No |
| Idempotent? | Yes |
| Works with BIT? | No |
| Sparse Table? | Yes — O(1) for static |
| GCD cost per merge | O(log V) where V = max value |
| Effective query cost | O(log n · log V) ≈ O(log n) |
| Range update trick | Use difference array: gcd(a₁,...,aₙ) = gcd(a₁, D₂,...,Dₙ) |

---

## Quick Revision Questions

1. What is the identity element for GCD? *(0, because gcd(x, 0) = x)*
2. Why does GCD work with Sparse Table? *(GCD is idempotent — overlapping ranges give correct result)*
3. What is the time complexity of a single merge? *(O(log V) for Euclidean algorithm, but treated as constant in practice)*
4. How do you handle range updates with GCD queries? *(Use difference array: gcd(a₁, a₂,...,aₙ) = gcd(a₁, |a₂-a₁|,...,|aₙ-aₙ₋₁|))*
5. Why can't BIT handle GCD queries? *(GCD is not invertible — can't compute range GCD from prefix GCDs)*
6. What is the identity for LCM? *(1, because lcm(x, 1) = x)*

---

[← Previous: Range Maximum Queries](03-range-maximum-queries.md) | [Next: Range XOR Queries →](05-range-xor-queries.md)
