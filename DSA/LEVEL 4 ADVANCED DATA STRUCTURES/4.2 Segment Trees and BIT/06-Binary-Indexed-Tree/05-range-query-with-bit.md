# Range Query with BIT

## Chapter Overview

BIT natively computes prefix sums. Range queries [L, R] are derived using the prefix difference trick. This chapter also covers how to handle **range updates with point queries** using a difference BIT.

---

## Range Sum Query

```
range_sum(L, R) = prefix_sum(R) - prefix_sum(L - 1)

This decomposition works because sum is invertible:
  prefix_sum(R)   = A[1] + A[2] + ... + A[R]
  prefix_sum(L-1) = A[1] + A[2] + ... + A[L-1]
  Difference       = A[L] + A[L+1] + ... + A[R]  ✓

Time: O(log n) — two prefix queries, each O(log n)
```

---

## Range Update + Point Query

```
Problem: Add val to all A[L..R], then query A[i].

Trick: Use a DIFFERENCE ARRAY with BIT.

Store D[i] = A[i] - A[i-1]  in the BIT (with D[1] = A[1]).
Then: A[i] = D[1] + D[2] + ... + D[i] = prefix_sum(i)

Range add [L, R] by val:
  D[L] += val       → update(L, +val)
  D[R+1] -= val     → update(R+1, -val)

Point query A[i]:
  A[i] = prefix_sum(i)

         Original A:  [_, 3, 1, 4, 2, 5, 3]
         Difference D: [_, 3, -2, 3, -2, 3, -2]
         
         Range add [2,4] by +10:
           D[2] += 10 → D = [_, 3, 8, 3, -2, 3, -2]
           D[5] -= 10 → D = [_, 3, 8, 3, -2, -7, -2]
         
         Query A[3]:
           prefix_sum(3) = 3 + 8 + 3 = 14
           Check: original A[3]=4, plus 10 = 14  ✓
         
         Query A[5]:
           prefix_sum(5) = 3 + 8 + 3 + (-2) + (-7) = 5
           Check: original A[5]=5, not updated = 5  ✓

Both operations: O(log n)!
```

---

## Range Update + Range Query

```
Problem: Add val to all A[L..R], then query sum(L', R').

This needs TWO BITs!

Derivation:
  A[i] = original[i] + sum of updates affecting index i
  
  Let D[i] be the difference array of updates.
  A[i] = original[i] + prefix_sum_D(i)
  
  prefix_sum_A(p) = Σ(i=1 to p) A[i]
                   = Σ original[i] + Σ prefix_sum_D(i)
                   = Σ original[i] + Σ(i=1 to p) Σ(j=1 to i) D[j]
  
  The double sum simplifies:
  Σ(i=1 to p) Σ(j=1 to i) D[j] = Σ(j=1 to p) D[j] × (p - j + 1)
                                 = (p+1) × Σ D[j] - Σ (j × D[j])

  So: prefix_sum_A(p) = Σ original[i] + (p+1) × Σ D[j] - Σ (j × D[j])

  Use two BITs:
    BIT1 stores D[j]       → gives Σ D[j]
    BIT2 stores j × D[j]   → gives Σ (j × D[j])

Range add [L, R] by val:
  BIT1: update(L, val), update(R+1, -val)
  BIT2: update(L, L*val), update(R+1, -(R+1)*val)

Prefix sum query(p):
  prefix_sum_D(p) = (p+1) * BIT1.prefix(p) - BIT2.prefix(p)
  prefix_sum_A(p) = original_prefix[p] + prefix_sum_D(p)

Range sum(L, R):
  prefix_sum_A(R) - prefix_sum_A(L-1)

Time: O(log n) for both operations!
```

---

## Two-BIT Trace

```
A = [_, 1, 2, 3, 4, 5]   n=5
original_prefix = [0, 1, 3, 6, 10, 15]

BIT1 = [0, 0, 0, 0, 0, 0]   (tracks D[j])
BIT2 = [0, 0, 0, 0, 0, 0]   (tracks j×D[j])

Range add [2, 4] by +10:
  BIT1: update(2, 10), update(5, -10)
  BIT2: update(2, 2×10=20), update(5, -5×10=-50)

  BIT1 after: conceptually D = [0, 0, 10, 0, 0, -10]
  BIT2 after: conceptually jD = [0, 0, 20, 0, 0, -50]

Query: prefix_sum_A(3)
  BIT1.prefix(3) = 0 + 10 + 0 = 10   (sum of D[1..3])
  BIT2.prefix(3) = 0 + 20 + 0 = 20   (sum of j×D[1..3])
  prefix_sum_D(3) = (3+1)×10 - 20 = 40 - 20 = 20
  prefix_sum_A(3) = original_prefix[3] + 20 = 6 + 20 = 26

  Check: A = [_, 1, 12, 13, 14, 5]
  sum(1,3) = 1+12+13 = 26  ✓

Query: sum(2, 4)
  prefix_sum_A(4):
    BIT1.prefix(4) = 10
    BIT2.prefix(4) = 20
    prefix_sum_D(4) = 5×10 - 20 = 30
    prefix_sum_A(4) = 10 + 30 = 40
  
  prefix_sum_A(1):
    BIT1.prefix(1) = 0
    BIT2.prefix(1) = 0
    prefix_sum_D(1) = 2×0 - 0 = 0
    prefix_sum_A(1) = 1 + 0 = 1
  
  sum(2,4) = 40 - 1 = 39
  Check: 12+13+14 = 39  ✓
```

---

## Comparison of BIT Techniques

```
┌─────────────────────────┬─────────────┬────────────┬──────────┐
│       Technique         │  # of BITs  │   Update   │  Query   │
├─────────────────────────┼─────────────┼────────────┼──────────┤
│ Point update +          │     1       │ O(log n)   │ O(log n) │
│ Range query             │             │            │          │
├─────────────────────────┼─────────────┼────────────┼──────────┤
│ Range update +          │     1       │ O(log n)   │ O(log n) │
│ Point query             │  (diff BIT) │            │          │
├─────────────────────────┼─────────────┼────────────┼──────────┤
│ Range update +          │     2       │ O(log n)   │ O(log n) │
│ Range query             │             │            │          │
└─────────────────────────┴─────────────┴────────────┴──────────┘

All achieve O(log n) — the question is which to use:
  - Simplest case (point update, range query): 1 standard BIT
  - Range update, point query: 1 BIT on difference array
  - Both range: 2 BITs on D[j] and j×D[j]
```

---

## BIT vs Segment Tree for Range Operations

```
┌───────────────────────────┬──────────────┬───────────────┐
│        Capability         │     BIT      │  Seg Tree     │
├───────────────────────────┼──────────────┼───────────────┤
│ Range update + point      │ 1 BIT        │ Lazy          │
│ Range update + range      │ 2 BITs       │ Lazy          │
│ Range update + range min  │ Impossible   │ Lazy          │
│ Implementation effort     │ Minimal      │ Moderate      │
│ Speed                     │ Faster       │ Slower        │
│ Memory                    │ n or 2n      │ 4n or 8n      │
└───────────────────────────┴──────────────┴───────────────┘

For sum operations: BIT is almost always preferred.
For min/max/gcd: Must use segment tree.
```

---

## Summary Table

| Query Type | Method | Time | BITs Needed |
|-----------|--------|------|-------------|
| Prefix sum | Direct | O(log n) | 1 |
| Range sum [L,R] | prefix(R) - prefix(L-1) | O(log n) | 1 |
| Point query after range update | prefix on difference BIT | O(log n) | 1 |
| Range query after range update | Two-BIT formula | O(log n) | 2 |

---

## Quick Revision Questions

1. How do you compute range_sum(L, R) with BIT? *(prefix_sum(R) - prefix_sum(L-1))*
2. How do you support range updates with point queries? *(Store difference array D[i] in BIT; range add [L,R] → update(L, +val), update(R+1, -val))*
3. Why do you need two BITs for range update + range query? *(The formula decomposes into Σ D[j] and Σ j×D[j], each needing its own prefix sum)*
4. Why can't BIT handle range update + range min query? *(Min is not invertible — BIT fundamentally relies on prefix difference)*
5. What is the formula for prefix sum using two BITs? *(prefix_sum_D(p) = (p+1) × BIT1.prefix(p) - BIT2.prefix(p))*

---

[← Previous: Prefix Query](04-prefix-query.md) | [Next: Building a BIT →](06-building-a-bit.md)
