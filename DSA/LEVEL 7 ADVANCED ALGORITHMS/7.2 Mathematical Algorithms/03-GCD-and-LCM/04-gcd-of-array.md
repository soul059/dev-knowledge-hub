# Chapter 4: GCD of an Array

[← Previous: LCM Computation](03-lcm-computation.md) | [Back to README](../README.md) | [Next: Properties and Theorems →](05-properties-and-theorems.md)

---

## Overview

Finding GCD of more than two numbers is a core pattern in competitive programming. This chapter covers iterative reduction, segment trees for range GCD queries, and advanced GCD problems.

---

## 4.1 Basic Reduction

The GCD of an array can be computed by repeatedly applying the two-element GCD:

$$\gcd(a_1, a_2, \ldots, a_n) = \gcd(\gcd(\ldots\gcd(a_1, a_2)\ldots), a_n)$$

```
FUNCTION gcdArray(arr):
    result = arr[0]
    FOR i = 1 TO len(arr) - 1:
        result = gcd(result, arr[i])
        IF result == 1:     // early exit optimization
            RETURN 1
    RETURN result

Time: O(n × log(max_element))
Space: O(1)
```

### Why Does Reduction Work?

```
  GCD is ASSOCIATIVE:
  gcd(a, b, c) = gcd(gcd(a, b), c) = gcd(a, gcd(b, c))

  GCD is COMMUTATIVE:
  gcd(a, b) = gcd(b, a)

  Therefore, the order of reduction doesn't matter!
```

---

## 4.2 Trace: GCD of Array

```
  arr = [48, 36, 60, 84]

  Step 1: g = gcd(48, 36)
          48 = 36×1 + 12
          36 = 12×3 + 0
          g = 12

  Step 2: g = gcd(12, 60)
          60 = 12×5 + 0
          g = 12

  Step 3: g = gcd(12, 84)
          84 = 12×7 + 0
          g = 12

  Result: GCD(48, 36, 60, 84) = 12

  ┌───────────────────────────────┐
  │  Verify: 48/12=4, 36/12=3,   │
  │          60/12=5, 84/12=7     │
  │  All integers, and 4,3,5,7   │
  │  have GCD=1 → 12 is correct  │
  └───────────────────────────────┘
```

---

## 4.3 Early Termination Optimization

```
  When g reaches 1, it STAYS 1 forever.
  gcd(1, x) = 1 for ANY x.
  
  So we can exit early!

  arr = [7, 13, 21, 35]

  Step 1: g = gcd(7, 13) = 1    ← g is 1!
  RETURN 1                       ← Skip rest of array

  Without early exit: would compute gcd(1,21) and gcd(1,35) needlessly.
```

---

## 4.4 GCD of Differences Property

```
  ┌──────────────────────────────────────────────────────────┐
  │  KEY THEOREM:                                            │
  │  gcd(a₁, a₂, ..., aₙ)                                  │
  │    = gcd(a₁, a₂ - a₁, a₃ - a₂, ..., aₙ - aₙ₋₁)       │
  │                                                          │
  │  More generally:                                         │
  │  gcd(a₁, a₂, ..., aₙ) = gcd(a₁, gcd of all pairwise   │
  │                              differences)                │
  └──────────────────────────────────────────────────────────┘

  Example: arr = [12, 18, 24, 30]
  Differences: 6, 6, 6
  gcd(12, 6) = 6 ✓  (matches gcd(12,18,24,30) = 6)

  USEFUL FOR: "Make all elements equal" problems.
  The GCD of all pairwise differences tells you the step size.
```

---

## 4.5 Segment Tree for Range GCD Queries

When you need GCD of a subarray repeatedly (range queries):

```
  ┌─────────────────────────────────────────────┐
  │  Segment Tree (GCD)                          │
  │                                              │
  │  Array: [12, 18, 24, 36, 48]                 │
  │                                              │
  │              [6]              (GCD of all)    │
  │           /      \                           │
  │        [6]       [12]                        │
  │       /   \     /    \                       │
  │     [6]  [24] [36]  [48]                     │
  │    /   \                                     │
  │  [12] [18]                                   │
  │                                              │
  │  Query GCD(1..3) = GCD of node[6], node[24]  │
  │                  = GCD(6, 24) = 6            │
  └─────────────────────────────────────────────┘
```

### Build & Query

```
FUNCTION build(arr, node, start, end):
    IF start == end:
        tree[node] = arr[start]
        RETURN
    mid = (start + end) / 2
    build(arr, 2*node, start, mid)
    build(arr, 2*node+1, mid+1, end)
    tree[node] = gcd(tree[2*node], tree[2*node+1])

FUNCTION query(node, start, end, l, r):
    IF r < start OR end < l:
        RETURN 0       // identity for GCD
    IF l <= start AND end <= r:
        RETURN tree[node]
    mid = (start + end) / 2
    leftGcd  = query(2*node, start, mid, l, r)
    rightGcd = query(2*node+1, mid+1, end, l, r)
    RETURN gcd(leftGcd, rightGcd)

Build: O(n log n)
Query: O(log n)
Update: O(log n)
Space: O(n)
```

---

## 4.6 Point Update for GCD Segment Tree

```
FUNCTION update(node, start, end, idx, val):
    IF start == end:
        tree[node] = val
        RETURN
    mid = (start + end) / 2
    IF idx <= mid:
        update(2*node, start, mid, idx, val)
    ELSE:
        update(2*node+1, mid+1, end, idx, val)
    tree[node] = gcd(tree[2*node], tree[2*node+1])
```

---

## 4.7 Sparse Table for Range GCD (No Updates)

If no updates needed, sparse table gives O(1) per query:

```
  Sparse Table: O(n log n) build, O(1) query

  Since GCD is an OVERLAP-FRIENDLY function:
  gcd(a, a) = a  (idempotent)

  So we can overlap ranges:
  gcd(l..r) = gcd(sparse[l][k], sparse[r-2^k+1][k])
  where k = floor(log₂(r - l + 1))

FUNCTION buildSparseTable(arr):
    n = len(arr)
    LOG = floor(log₂(n)) + 1
    sparse[i][0] = arr[i]  for all i
    FOR j = 1 TO LOG:
        FOR i = 0 TO n - 2^j:
            sparse[i][j] = gcd(sparse[i][j-1], sparse[i + 2^(j-1)][j-1])

FUNCTION queryGCD(l, r):
    k = floor(log₂(r - l + 1))
    RETURN gcd(sparse[l][k], sparse[r - 2^k + 1][k])
```

---

## 4.8 Problem-Solving Patterns

### Pattern 1: Count Subarrays with GCD = k

```
  Key insight: As you extend a subarray, GCD can only
  DECREASE or STAY the same, never increase!

  arr = [6, 3, 9, 12]:

  Ending at index 0: GCD values = {6}
  Ending at index 1: GCD values = {3, 3} → {3}
  Ending at index 2: GCD values = {9, 3} → {9, 3}
  Ending at index 3: GCD values = {12, 3} → {12, 3}

  At each position, there are at most O(log(max)) distinct
  GCD values. Total across array: O(n log(max)).
```

### Pattern 2: Minimum Operations to Make GCD = 1

```
  If GCD of entire array > 1, impossible!
  Otherwise, find shortest subarray with GCD = 1.
  Answer = n - 1 + (length of shortest GCD-1 subarray) - 1
```

### Pattern 3: GCD Queries with Updates

```
  Use Segment Tree (Section 4.5-4.6)
  Build: O(n)
  Each query: O(log n)
  Each update: O(log n)
```

---

## 4.9 GCD and Divisibility

```
  ┌──────────────────────────────────────────────────────────┐
  │  KEY FACTS:                                               │
  │                                                          │
  │  1. gcd(a₁,...,aₙ) divides every aᵢ                     │
  │  2. gcd(a₁,...,aₙ) divides every linear combination     │
  │     c₁a₁ + c₂a₂ + ... + cₙaₙ                           │
  │  3. There EXIST integers c₁,...,cₙ such that             │
  │     c₁a₁ + ... + cₙaₙ = gcd(a₁,...,aₙ)                 │
  │  4. gcd(a, b, c) = 1 does NOT mean pairwise coprime     │
  │     Example: gcd(6, 10, 15) = 1 but                     │
  │     gcd(6,10)=2, gcd(6,15)=3, gcd(10,15)=5              │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Method | Build | Query | Update | Space |
|--------|-------|-------|--------|-------|
| Linear scan | - | O(n log max) | - | O(1) |
| Segment Tree | O(n) | O(log n) | O(log n) | O(n) |
| Sparse Table | O(n log n) | O(1) | Not supported | O(n log n) |

---

## Quick Revision Questions

1. **Why can we compute GCD of an array** by reducing pairwise?
2. **What is the early termination** condition when computing array GCD?
3. **Subarrays ending at index i** can have at most how many distinct GCD values?
4. **Build a segment tree** for array [8, 12, 16, 24]. Draw it.
5. **When would you choose sparse table** over segment tree for GCD queries?
6. **Prove or disprove**: GCD(6, 10, 15) = 1 implies all pairs are coprime.

---

[← Previous: LCM Computation](03-lcm-computation.md) | [Back to README](../README.md) | [Next: Properties and Theorems →](05-properties-and-theorems.md)
