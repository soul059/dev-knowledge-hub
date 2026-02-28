# Prefix Sum Limitations

## Chapter Overview

Prefix sums are an elegant O(1) query solution for range sum problems on **static arrays**. However, they fall apart when updates are needed. Understanding these limitations motivates the need for Segment Trees and BITs.

---

## Prefix Sum Recap

The prefix sum array `P[i]` stores the sum of elements from index 0 to i.

```
Original:  A = [ 3,  1,  4,  1,  5,  9,  2,  6 ]
Index:           0   1   2   3   4   5   6   7

Prefix:    P = [ 3,  4,  8,  9, 14, 23, 25, 31 ]

P[i] = A[0] + A[1] + ... + A[i]
```

**Range Sum Query**:
```
sum(L, R) = P[R] - P[L-1]    (when L > 0)
sum(L, R) = P[R]              (when L = 0)

Example: sum(2, 5) = P[5] - P[1] = 23 - 4 = 19
Verify:  A[2]+A[3]+A[4]+A[5] = 4+1+5+9 = 19  ✓
```

---

## Where Prefix Sums Excel

```
┌────────────────────────────────────────────┐
│        STATIC ARRAY + RANGE SUM            │
│                                            │
│  Build:  O(n)   ← one-time cost            │
│  Query:  O(1)   ← instant!                 │
│  Update: ✗      ← not supported well       │
│                                            │
│  Perfect for: read-only data               │
└────────────────────────────────────────────┘
```

---

## Limitation 1: Updates are Expensive

When we update `A[i]`, all prefix sums from index `i` onwards become invalid.

```
Before Update:
  A = [ 3,  1,  4,  1,  5,  9,  2,  6 ]
  P = [ 3,  4,  8,  9, 14, 23, 25, 31 ]
                  ↑
Update A[2] = 10 (was 4, diff = +6)

After Update:
  A = [ 3,  1, 10,  1,  5,  9,  2,  6 ]
  P = [ 3,  4, 14, 15, 20, 29, 31, 37 ]
                ↑   ↑   ↑   ↑   ↑   ↑
          All these values must change!
```

**Cost of one update: O(n)** — must recompute all prefix sums from index `i` to `n-1`.

With Q updates and Q queries:
```
Prefix Sum:  O(n) build + O(n × Q) updates + O(Q) queries
           = O(n × Q) total  ← BAD for large Q

Example: n = 100,000 and Q = 100,000
         → 10^10 operations → TLE!
```

---

## Limitation 2: Only Works for Sum

Prefix sums exploit the property: `sum(L, R) = sum(0, R) - sum(0, L-1)`.

This **subtraction trick** does NOT work for other operations:

```
┌──────────────────────────────────────────────────────┐
│  Operation   │  Can subtract prefixes?  │  Works?    │
├──────────────┼──────────────────────────┼────────────┤
│  Sum         │  sum(L,R) = P[R]-P[L-1]  │  YES ✓    │
│  XOR         │  xor(L,R) = P[R]^P[L-1]  │  YES ✓    │
│  Min         │  min(L,R) ≠ P[R]-P[L-1]  │  NO  ✗    │
│  Max         │  max(L,R) ≠ P[R]-P[L-1]  │  NO  ✗    │
│  GCD         │  gcd(L,R) ≠ P[R]-P[L-1]  │  NO  ✗    │
│  Product     │  Needs division (risky)   │  MAYBE    │
└──────────────┴──────────────────────────┴────────────┘
```

**Why Min fails**:
```
A = [3, 1, 4, 1, 5]
Prefix Min = [3, 1, 1, 1, 1]

Query min(2, 4):
  PrefixMin[4] = 1, PrefixMin[1] = 1
  PrefixMin[4] - PrefixMin[1] = 0  ← WRONG!
  Actual min(4, 1, 5) = 1

There's no way to "undo" or "subtract" a min operation.
```

---

## Limitation 3: Range Updates

What if we need to add a value to all elements in a range?

```
Range Update: Add 5 to A[2..5]

Before: A = [ 3,  1,  4,  1,  5,  9,  2,  6 ]
After:  A = [ 3,  1,  9,  6, 10, 14,  2,  6 ]
                    +5  +5  +5  +5

With prefix sums: must update O(n) prefix values
                  AND the array itself → O(n) per update
```

---

## Limitation 4: Multi-Dimensional Queries

2D prefix sums exist but become complex and rigid:

```
2D Prefix Sum for matrix M:

P[i][j] = sum of all M[x][y] where x ≤ i, y ≤ j

Query sum(r1, c1, r2, c2):
  = P[r2][c2] - P[r1-1][c2] - P[r2][c1-1] + P[r1-1][c1-1]

Problem: Any update to M[i][j] requires O(n×m) rebuilding
```

---

## The Performance Gap

```
Scenario: n = 100,000 elements, Q = 100,000 mixed queries and updates

┌──────────────┬─────────────┬───────────────┬────────────────┐
│  Approach    │  Build      │  Per Query    │  Per Update    │
├──────────────┼─────────────┼───────────────┼────────────────┤
│  Brute Force │  O(1)       │  O(n)         │  O(1)          │
│  Prefix Sum  │  O(n)       │  O(1)         │  O(n)          │
│  Seg Tree    │  O(n)       │  O(log n)     │  O(log n)      │
│  BIT         │  O(n)       │  O(log n)     │  O(log n)      │
└──────────────┴─────────────┴───────────────┴────────────────┘

For Q mixed operations:
  Brute Force: O(Q × n)       ≈ 10^10  → TLE
  Prefix Sum:  O(Q × n)       ≈ 10^10  → TLE
  Seg Tree:    O(Q × log n)   ≈ 10^5 × 17 ≈ 2×10^6  → AC ✓
  BIT:         O(Q × log n)   ≈ 10^5 × 17 ≈ 2×10^6  → AC ✓
```

---

## When Prefix Sums ARE Sufficient

Use prefix sums when:
- Array is **static** (no updates after building)
- Query type is **sum** or **XOR**
- You need **O(1) per query**
- Space is a concern (only O(n) extra)

```
Decision Flow:
                    ┌─────────────────┐
                    │  Need updates?  │
                    └────────┬────────┘
                    NO ╱          ╲ YES
                      ╱            ╲
              ┌──────────┐   ┌────────────────┐
              │  Prefix  │   │  Seg Tree /    │
              │   Sums   │   │     BIT        │
              └──────────┘   └────────────────┘
```

---

## Summary Table

| Limitation | Description | Impact |
|-----------|-------------|--------|
| Update Cost | O(n) to rebuild after single update | Unusable for dynamic data |
| Operation Type | Only works for sum/XOR (invertible ops) | Can't handle min/max/GCD |
| Range Updates | No efficient way to update a range | O(n) per range update |
| Flexibility | Locked to one operation type | Can't mix queries |
| 2D Updates | Entire 2D prefix must be rebuilt | O(n×m) update cost |

---

## Quick Revision Questions

1. What is the time complexity of a range sum query using prefix sums? *(O(1))*
2. Why is update expensive with prefix sums? *(All prefix values from index i onward must be recomputed → O(n))*
3. Can prefix sums answer range minimum queries? Why not? *(No — min is not invertible; you can't subtract min from a prefix)*
4. For what type of data are prefix sums the best solution? *(Static arrays with only sum queries)*
5. What complexity do Segment Trees achieve for both query and update? *(O(log n) each)*

---

[← Previous: Types of Range Queries](01-types-of-range-queries.md) | [Next: Need for Advanced Structures →](03-need-for-advanced-structures.md)
