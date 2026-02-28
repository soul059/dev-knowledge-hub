# Building a BIT

## Chapter Overview

Building a BIT from an initial array can be done in O(n log n) naively or O(n) optimally. This chapter covers both methods, the initialization strategy, and the difference between "empty" and "pre-filled" BIT construction.

---

## Method 1: O(n log n) — Repeated Updates

```
function build_slow(A, n):
    B = array of size n+1, initialized to 0
    for i = 1 to n:
        update(i, A[i])      // Each update is O(log n)
    return B

// Total: n × O(log n) = O(n log n)

This is simple but sub-optimal.
In competitive programming, this is usually fast enough (n ≤ 10^5).
For n ≈ 10^6 or tighter time limits, use Method 2.
```

---

## Method 2: O(n) — Parent Propagation

```
function build_fast(A, n):
    B = array of size n+1
    B[0] = 0
    for i = 1 to n:
        B[i] = A[i]         // Step 1: Copy values
    
    for i = 1 to n:
        parent = i + (i & (-i))
        if parent <= n:
            B[parent] += B[i]   // Step 2: Propagate to parent
    return B

Why this works:
  After Step 1: B[i] = A[i] (just the single element)
  Step 2 processes indices in order 1, 2, 3, ..., n
  
  When we process i:
    B[i] already contains the sum of all elements in its range
    (because all children of i have smaller indices and were
    processed first, and they propagated their sums up)
  
  B[parent] needs to include B[i]'s range, so we add B[i] to B[parent]

Time: O(n) — each index processed once, O(1) per index
```

---

## O(n) Build Trace

```
A = [_, 5, 3, 7, 2, 8, 1, 4, 6]   n = 8

Step 1: B = [0, 5, 3, 7, 2, 8, 1, 4, 6]

Step 2: Process each index:

  i=1: parent = 1+(1&(-1)) = 1+1 = 2
    B[2] += B[1] → B[2] = 3+5 = 8
    B = [0, 5, 8, 7, 2, 8, 1, 4, 6]

  i=2: parent = 2+(2&(-2)) = 2+2 = 4
    B[4] += B[2] → B[4] = 2+8 = 10
    B = [0, 5, 8, 7, 10, 8, 1, 4, 6]

  i=3: parent = 3+(3&(-3)) = 3+1 = 4
    B[4] += B[3] → B[4] = 10+7 = 17
    B = [0, 5, 8, 7, 17, 8, 1, 4, 6]

  i=4: parent = 4+(4&(-4)) = 4+4 = 8
    B[8] += B[4] → B[8] = 6+17 = 23
    B = [0, 5, 8, 7, 17, 8, 1, 4, 23]

  i=5: parent = 5+(5&(-5)) = 5+1 = 6
    B[6] += B[5] → B[6] = 1+8 = 9
    B = [0, 5, 8, 7, 17, 8, 9, 4, 23]

  i=6: parent = 6+(6&(-6)) = 6+2 = 8
    B[8] += B[6] → B[8] = 23+9 = 32
    B = [0, 5, 8, 7, 17, 8, 9, 4, 32]

  i=7: parent = 7+(7&(-7)) = 7+1 = 8
    B[8] += B[7] → B[8] = 32+4 = 36
    B = [0, 5, 8, 7, 17, 8, 9, 4, 36]

  i=8: parent = 8+(8&(-8)) = 8+8 = 16 > 8 → skip


Final: B = [0, 5, 8, 7, 17, 8, 9, 4, 36]

Verify:
  B[1] = A[1] = 5                    ✓ range [1,1]
  B[2] = A[1]+A[2] = 5+3 = 8        ✓ range [1,2]
  B[3] = A[3] = 7                    ✓ range [3,3]
  B[4] = A[1..4] = 5+3+7+2 = 17     ✓ range [1,4]
  B[5] = A[5] = 8                    ✓ range [5,5]
  B[6] = A[5]+A[6] = 8+1 = 9        ✓ range [5,6]
  B[7] = A[7] = 4                    ✓ range [7,7]
  B[8] = A[1..8] = 36               ✓ range [1,8]
```

---

## Building from Empty (All Zeros)

```
If you don't have initial values, just initialize:
  B = array of size n+1, all zeros

No build step needed! 
  - An all-zero BIT correctly represents an all-zero array
  - Elements are added later via update() calls
```

---

## Building Difference BIT

```
For range-update + point-query, build a BIT on the difference array:

function build_difference_bit(A, n):
    D = array of size n+1
    D[0] = 0
    D[1] = A[1]
    for i = 2 to n:
        D[i] = A[i] - A[i-1]
    
    // Now build BIT on D
    return build_fast(D, n)

After building:
  - prefix_sum(i) gives A[i]
  - update(L, +val) and update(R+1, -val) does range add [L,R]
```

---

## Comparison of Build Methods

```
┌──────────────────────┬───────────┬──────────────┬─────────┐
│       Method         │   Time    │  Complexity  │  When   │
├──────────────────────┼───────────┼──────────────┼─────────┤
│ Repeated updates     │ O(n log n)│    Simple    │ Default │
│ Parent propagation   │ O(n)      │    Simple    │ Large n │
│ All-zeros init       │ O(n)      │   Trivial    │ Dynamic │
│ From difference arr  │ O(n)      │   Moderate   │ RU+PQ   │
└──────────────────────┴───────────┴──────────────┴─────────┘

RU+PQ = Range Update + Point Query
```

---

## Segment Tree Build vs BIT Build

```
Segment tree O(n) build:
  - Recursively bottom-up
  - Builds leaves first, then merges up
  - Result: 4n array with proper structure

BIT O(n) build:
  - Linear scan with parent propagation
  - Each node sends its value to parent
  - Result: n+1 array with proper structure

Both are O(n), but BIT build is simpler:
  - No recursion
  - Single pass
  - Fewer memory accesses
```

---

## Verify Build Correctness

```
After building, you can verify by checking prefix sums:

function verify_build(A, B, n):
    for i = 1 to n:
        expected = A[1] + A[2] + ... + A[i]
        actual = prefix_sum(i)
        assert expected == actual

For debugging:
  - Print B[] after build
  - Check B[i] covers exactly [i - lowbit(i) + 1, i]
  - Check prefix_sum(n) equals total sum
```

---

## Summary Table

| Aspect | O(n log n) Build | O(n) Build |
|--------|-----------------|------------|
| Algorithm | n × update(i, A[i]) | Copy + parent propagation |
| Lines of code | 3 (uses existing update) | 8 |
| Practical speed | Acceptable for n ≤ 10^5 | Needed for n ≥ 10^6 |
| Cache behavior | Random access pattern | Sequential scan |
| Extra space | None | None (in-place) |

---

## Quick Revision Questions

1. What are the two methods for building a BIT? *(O(n log n) via repeated updates, and O(n) via parent propagation)*
2. In the O(n) build, what does each step do? *(B[parent] += B[i], where parent = i + lowbit(i))*
3. Why must indices be processed in order 1, 2, ..., n? *(So that each node has already accumulated its full range before it propagates to its parent)*
4. How do you build a BIT for range updates? *(Build a BIT on the difference array D[i] = A[i] - A[i-1])*
5. What's an all-zero BIT used for? *(When elements are added dynamically via update calls, no pre-build needed)*

---

[← Previous: Range Query with BIT](05-range-query-with-bit.md) | [Back to README →](../README.md)
