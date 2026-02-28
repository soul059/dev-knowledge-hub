# Number of Elements in Range

## Chapter Overview

A common query pattern asks: "How many elements in A[L..R] have values in [lo, hi]?" This chapter explores multiple approaches using BIT and segment tree variants, covering both offline and online scenarios.

---

## Problem Statement

```
Given array A[1..n], answer queries:
  count(L, R, lo, hi) = |{ i : L ≤ i ≤ R AND lo ≤ A[i] ≤ hi }|

Example: A = [3, 1, 4, 1, 5, 9, 2, 6]
  count(2, 6, 2, 5) = elements in A[2..6]={1,4,1,5,9} with value in [2,5]
                     = {4, 5} → answer = 2
```

---

## Approach 1: Merge Sort Tree (O(log² n) per query)

```
Build merge sort tree (each node = sorted subarray).

count(L, R, lo, hi):
    return count_leq(L, R, hi) - count_leq(L, R, lo - 1)

count_leq(node, l, r, ql, qr, X):
    if ql > r or qr < l: return 0
    if ql <= l and r <= qr:
        return upper_bound(tree[node], X)  // binary search
    mid = (l + r) / 2
    return count_leq(left, l, mid, ql, qr, X) +
           count_leq(right, mid+1, r, ql, qr, X)

Time: O(log² n) per query (O(log n) nodes × O(log n) binary search)
Space: O(n log n)
No updates supported (static).
```

---

## Approach 2: Persistent Segment Tree (O(log n) per query)

```
Build persistent segment tree on VALUES.

Process elements left to right:
  root[i] = insert A[i] into root[i-1]
  
  Each version i represents the frequency tree for A[1..i].

count(L, R, lo, hi):
    // "Subtract" version L-1 from version R
    return count_range(root[R], root[L-1], 1, MAX_VAL, lo, hi)

count_range(vR, vL, l, r, lo, hi):
    if lo > r or hi < l: return 0
    if lo <= l and r <= hi:
        return nodes[vR].value - nodes[vL].value
    mid = (l + r) / 2
    return count_range(left(vR), left(vL), l, mid, lo, hi) +
           count_range(right(vR), right(vL), mid+1, r, lo, hi)

Time: O(log n) per query
Space: O(n log n)
```

---

## Approach 3: Offline with BIT (O(n√q log n) or sweep)

```
If queries are known in advance, sort and sweep.

Sort-based approach:
  1. Sort queries by right endpoint R
  2. Process elements left to right
  3. For each element, add it to a BIT on values
  4. When reaching a query's R, answer it

But this only gives prefix counts [1..R] — need [L..R].

Mo's algorithm can handle arbitrary [L, R]:
  Sort queries by (L/√n, R)
  Maintain a frequency BIT
  Add/remove elements as L, R change
  
  Time: O((n + q) × √n × log n)
```

---

## Approach 4: Wavelet Tree (O(log V) per query)

```
A Wavelet Tree partitions values recursively by their bits.

For each node covering value range [lo, hi]:
  Partition elements: those ≤ mid go left, > mid go right
  Store a bitstring indicating which elements went where

count(L, R, lo, hi):
    Walk the wavelet tree, narrowing the position range [L,R]
    and value range [lo,hi] simultaneously.

Time: O(log V) per query (V = value range)
Space: O(n log V)

Very powerful but complex to implement.
```

---

## Special Case: Count Elements = X

```
"How many times does value X appear in A[L..R]?"

Method 1: Merge sort tree
  count(L, R, X, X) = count_leq(X) - count_leq(X-1)
  Time: O(log² n)

Method 2: Store positions for each value
  positions[X] = sorted list of indices where A[i] = X
  count = upper_bound(positions[X], R) - lower_bound(positions[X], L)
  Time: O(log n) with binary search
  Space: O(n)
  
  This is the simplest approach for exact value queries!
```

---

## Special Case: Count Elements < X (Value Threshold)

```
"How many elements in A[L..R] are less than X?"

This is directly count_leq(L, R, X-1).

Applications:
  - Rank of X in subarray
  - Percentile calculations
  - Count of elements exceeding threshold
    (= (R-L+1) - count_leq(L, R, threshold))
```

---

## With Updates

```
When elements can be modified:

Approach: BIT of BITs (2D BIT)
  Treat as 2D problem: position × value
  
  update(pos, old_val, new_val):
      BIT2D.update(pos, old_val, -1)
      BIT2D.update(pos, new_val, +1)
  
  count(L, R, lo, hi):
      BIT2D.query(R, hi) - BIT2D.query(L-1, hi) 
    - BIT2D.query(R, lo-1) + BIT2D.query(L-1, lo-1)

  But 2D BIT needs O(n × V) space → coordinate compress values

Alternative: Sqrt decomposition
  Divide array into blocks of size √n
  Each block stores a sorted copy of its elements
  
  Update: O(√n) to re-sort the block
  Query: O(√n × log(√n)) using binary search per block
```

---

## Worked Example: Merge Sort Tree Approach

```
A = [5, 2, 8, 1, 4, 7, 3, 6]

Merge sort tree:
  [1..8]: [1,2,3,4,5,6,7,8]
  [1..4]: [1,2,5,8]    [5..8]: [3,4,6,7]
  [1..2]: [2,5]  [3..4]: [1,8]  [5..6]: [4,7]  [7..8]: [3,6]

Query: count(2, 7, 3, 6)
  = count_leq(2,7,6) - count_leq(2,7,2)

count_leq(2, 7, 6):
  Decompose [2,7] → nodes [2..2], [3..4], [5..6], [7..7]
  [2..2] = [2]:   upper_bound(6) = 1
  [3..4] = [1,8]: upper_bound(6) = 1  (only 1 ≤ 6)
  [5..6] = [4,7]: upper_bound(6) = 1  (only 4 ≤ 6)
  [7..7] = [3]:   upper_bound(6) = 1
  Total = 4

count_leq(2, 7, 2):
  [2..2] = [2]:   upper_bound(2) = 1
  [3..4] = [1,8]: upper_bound(2) = 1
  [5..6] = [4,7]: upper_bound(2) = 0
  [7..7] = [3]:   upper_bound(2) = 0
  Total = 2

Answer = 4 - 2 = 2

Verify: A[2..7] = {2,8,1,4,7,3}, values in [3,6] = {4,3} → 2 ✓
```

---

## Comparison of Approaches

```
                    Merge Sort    Persistent    Wavelet    BIT+
                    Tree          Seg Tree      Tree       Offline
──────────────────  ──────────    ──────────    ───────    ────────
Build               O(n log n)   O(n log n)    O(n log V) O(n log n)
Query               O(log² n)    O(log n)      O(log V)   O(√n log n)
Update              Hard          Hard          Hard       O(√n)
Space               O(n log n)   O(n log n)    O(n log V) O(n)
Code complexity     Moderate     Moderate      Complex    Simple
Best for            Static,      Static, fast  All types  With updates
                    simple code  queries                   or offline
```

---

## Summary Table

| Approach | Query Time | Space | Updates | Complexity |
|----------|-----------|-------|---------|-----------|
| Merge Sort Tree | O(log² n) | O(n log n) | Rebuild | Moderate |
| Persistent Seg Tree | O(log n) | O(n log n) | O(n log n) per | Moderate |
| Wavelet Tree | O(log V) | O(n log V) | Hard | Complex |
| Sorted positions | O(log n) | O(n) | Hard | Simple |
| Sqrt Decomposition | O(√n log √n) | O(n) | O(√n) | Simple |

---

## Quick Revision Questions

1. How does merge sort tree answer "count in value range" queries? *(count_leq(R, hi) - count_leq(R, lo-1), where count_leq uses binary search at each decomposed node — O(log² n))*
2. How does the persistent segment tree approach work? *(Build version i for each prefix A[1..i]; count in [L,R] = version[R] - version[L-1])*
3. What is the simplest approach for counting exact value X in range? *(Store sorted positions for each value; binary search for L and R boundaries — O(log n))*
4. How can updates be supported? *(Sqrt decomposition with sorted blocks, or BIT of BITs with coordinate compression)*
5. What is the trade-off between merge sort tree and persistent segment tree? *(Merge sort tree: simpler code, O(log² n). Persistent: faster O(log n), but harder to implement)*

---

[← Previous: Range Updates with Queries](02-range-updates-with-queries.md) | [Next: Longest Increasing Subsequence →](04-longest-increasing-subsequence.md)
