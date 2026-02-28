# Longest Increasing Subsequence with Segment Tree / BIT

## Chapter Overview

The classic LIS problem has an O(n log n) patience sorting solution. But segment trees and BITs offer an alternative approach that naturally extends to harder variants: LIS with constraints, weighted LIS, and online LIS. This chapter covers these approaches.

---

## Standard LIS Recap

```
Problem: Find the length of the longest strictly increasing
subsequence in A[1..n].

Example: A = [3, 1, 4, 1, 5, 9, 2, 6]
  LIS: [1, 4, 5, 9] or [1, 4, 5, 6] or [1, 2, 6, 9] → length 4

Classic O(n log n) solution: patience sorting with binary search
  (not covered here — focus is on segment tree / BIT approach)
```

---

## LIS Using Segment Tree

```
Idea: dp[i] = length of LIS ending at index i
  dp[i] = 1 + max(dp[j]) for all j < i where A[j] < A[i]

Naive: O(n²) — check all j < i.

Optimization: Use segment tree on VALUES to find max dp for values < A[i].

Algorithm:
  1. Coordinate compress A to [1..n]
  2. Build segment tree of size n (stores max values)
  3. Process left to right:
     
     for i from 1 to n:
         // Find max dp among values strictly less than A[i]
         best = seg.query_max(1, A[i] - 1)
         dp[i] = best + 1
         // Update segment tree: at position A[i], store dp[i]
         seg.update_max(A[i], dp[i])
     
     answer = max(dp[1..n])

Time: O(n log n)
Space: O(n)
```

---

## Step-by-Step Trace

```
A = [3, 1, 4, 1, 5]
Compressed: [2, 1, 3, 1, 4]   (3→2, 1→1, 4→3, 5→4)

Segment tree on values [1..4], stores max dp value.
Initially all 0.

i=1: A[i]=2
  query_max(1, 1) = 0   (max dp for values < 2)
  dp[1] = 0 + 1 = 1
  update(2, 1)
  SegTree values: [0, 1, 0, 0]

i=2: A[i]=1
  query_max(1, 0) = 0   (empty range, no values < 1)
  dp[2] = 0 + 1 = 1
  update(1, 1)
  SegTree values: [1, 1, 0, 0]

i=3: A[i]=3
  query_max(1, 2) = max(1, 1) = 1  (best dp for values 1 or 2)
  dp[3] = 1 + 1 = 2
  update(3, 2)
  SegTree values: [1, 1, 2, 0]

i=4: A[i]=1
  query_max(1, 0) = 0
  dp[4] = 0 + 1 = 1
  update(1, max(1, 1)) = update(1, 1)   // no change
  SegTree values: [1, 1, 2, 0]

i=5: A[i]=4
  query_max(1, 3) = max(1, 1, 2) = 2
  dp[5] = 2 + 1 = 3
  update(4, 3)
  SegTree values: [1, 1, 2, 3]

Answer: max(1, 1, 2, 1, 3) = 3
LIS = [1, 4, 5] or [1, 3, 5] → length 3 ✓
```

---

## LIS Using BIT

```
BIT can find PREFIX MAXIMUM (values in [1..X]).

For LIS, we need max dp among values [1..A[i]-1] = prefix max up to A[i]-1.

BIT approach:
  BIT stores max values (using max instead of sum)
  
  query_prefix_max(i):
      result = 0
      while i > 0:
          result = max(result, B[i])
          i -= i & (-i)
      return result
  
  update_max(i, val):
      while i <= n:
          B[i] = max(B[i], val)
          i += i & (-i)

  Algorithm:
    for i from 1 to n:
        best = query_prefix_max(A[i] - 1)
        dp_i = best + 1
        update_max(A[i], dp_i)
        answer = max(answer, dp_i)

Note: BIT prefix max works here because:
  - We only do PREFIX maximum (from 1 to X)
  - Values only INCREASE (dp values are non-decreasing per position)
  - max is associative
```

---

## Why BIT Works for Prefix Max Here

```
Normally BIT can't handle max (not invertible).
But PREFIX max is special:

prefix_max(X) = max(B[1], B[2], ..., B[X])

The BIT query naturally visits a covering set of ranges:
  B[X], B[X - lowbit(X)], ..., B[some earlier position]
  
  These ranges COVER [1..X] without gaps.
  Taking max over them gives the correct prefix max.

CAVEAT: This only works if we NEVER DECREASE a value.
  In LIS, dp values at each position only increase → safe.
  
  If values can decrease, BIT prefix max breaks → use segment tree.
```

---

## Variants

### Weighted LIS

```
"Find the subsequence with max total weight (not max length)"

dp[i] = max weight of increasing subsequence ending with A[i]
dp[i] = W[i] + max(dp[j]) for j < i, A[j] < A[i]

Same segment tree approach:
  best = seg.query_max(1, A[i] - 1)
  dp[i] = W[i] + best
  seg.update_max(A[i], dp[i])

Time: O(n log n)
```

### Non-Decreasing Subsequence (LIS with ties)

```
"Longest non-decreasing subsequence (≤ instead of <)"

Change: query max dp for values ≤ A[i] (not < A[i])
  best = seg.query_max(1, A[i])    // include A[i] itself
  dp[i] = best + 1

But we must process duplicates carefully — if A[i] = A[j] for j < i,
we don't want to chain them incorrectly.

Solution: Process elements LEFT to RIGHT, and for equal values,
process them in DECREASING index order (or separated).
```

### LIS with Additional Constraints

```
"LIS where A[j] < A[i] AND B[j] < B[i]" (2D LIS)

This is the problem of longest chain with two keys.
Approach: Sort by A, then find LIS on B values using BIT.

Time: O(n log n) after sorting.
```

---

## Comparison with Classic O(n log n) Approach

```
                    Patience Sort    Segment Tree / BIT
                    ─────────────    ──────────────────
Time                O(n log n)       O(n log n)
Space               O(n)             O(n)
Code complexity     Short            Medium
Extensions          Hard             Easy (weighted, 2D, etc.)
Actual sequence     Recoverable      Recoverable (via dp)
Online              Yes              Yes
Constant factor     Smaller          Larger

For standard LIS: patience sort is simpler.
For LIS variants: segment tree / BIT approach is more flexible.
```

---

## Recovering the Actual LIS

```
To find the actual subsequence (not just length):

1. Compute dp[i] for all i
2. Find the position with max dp value
3. Trace back:

last_val = +∞, last_dp = max_dp
result = []
for i from n down to 1:
    if dp[i] == last_dp AND A[i] < last_val:
        result.prepend(A[i])
        last_val = A[i]
        last_dp -= 1
    if last_dp == 0: break
```

---

## Practice Applications

```
1. Standard LIS → BIT / segment tree
2. Longest non-decreasing subsequence → segment tree with careful ordering
3. Weighted LIS (max weight sum) → segment tree
4. 2D LIS (pairs) → sort + BIT
5. Number of LIS → segment tree storing (length, count) pairs
6. LIS in each prefix → online BIT processing
```

---

## Summary Table

| Variant | Structure | Query | Time |
|---------|----------|-------|------|
| Standard LIS | BIT prefix max | max in [1, A[i]-1] | O(n log n) |
| Standard LIS | Segment tree | max in [1, A[i]-1] | O(n log n) |
| Weighted LIS | Segment tree | max in [1, A[i]-1] | O(n log n) |
| 2D LIS | Sort + BIT | max in [1, B[i]-1] | O(n log n) |
| Count of LIS | Segment tree | (max, count) merge | O(n log n) |

---

## Quick Revision Questions

1. How does the segment tree approach for LIS work? *(Process left to right; for each A[i], query max dp among values < A[i] in the segment tree, set dp[i] = best + 1, update segment tree at position A[i])*
2. Why can BIT compute prefix maximum for LIS? *(Prefix max queries visit a covering set of ranges; values only increase so the property holds)*
3. What makes the segment tree approach better than patience sorting? *(Easier to extend: weighted LIS, 2D LIS, counting LIS, additional constraints)*
4. How do you handle values up to 10^9? *(Coordinate compression to [1..n] — relative order is all that matters)*
5. How do you find the number of distinct LIS? *(Store (length, count) pairs in segment tree; merge: take max length, sum counts if equal)*

---

[← Previous: Number of Elements in Range](03-number-of-elements-in-range.md) | [Next: Count Smaller After Self →](05-count-smaller-after-self.md)
