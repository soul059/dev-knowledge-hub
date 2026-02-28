# BIT vs Segment Tree

## Chapter Overview

This chapter provides a definitive comparison between BIT and Segment Tree — when to use which, their trade-offs, and how to make the right choice for each problem.

---

## Side-by-Side Comparison

```
┌─────────────────────────┬────────────────┬──────────────────┐
│       Feature           │      BIT       │  Segment Tree    │
├─────────────────────────┼────────────────┼──────────────────┤
│ Point update            │ O(log n)       │ O(log n)         │
│ Prefix query            │ O(log n)       │ O(log n)         │
│ Range query             │ O(log n)       │ O(log n)         │
│ Range update (lazy)     │ O(log n)*      │ O(log n)         │
│ Arbitrary range ops     │ Limited        │ Full support     │
│ Memory                  │ n + 1          │ 4n (+ 4n lazy)   │
│ Code length             │ ~10 lines      │ ~50+ lines       │
│ Constant factor         │ Very small     │ 2-5× larger      │
│ Supported operations    │ Invertible     │ Any associative   │
│ 2D extension            │ Simple         │ Complex           │
│ Persistence             │ Hard           │ Natural           │
│ Lazy propagation        │ Via tricks     │ Built-in          │
│ Min/Max queries         │ No             │ Yes               │
│ GCD queries             │ No             │ Yes               │
│ Custom merge            │ No             │ Yes               │
└─────────────────────────┴────────────────┴──────────────────┘

* BIT range update uses difference array trick
```

---

## Decision Flowchart

```
                    What operation?
                    /           \
            Invertible?        Not invertible
           (sum, XOR)        (min, max, GCD)
              /                     \
     Need range update?      → USE SEGMENT TREE
        /        \
      No          Yes
      |            |
      |      Need range query too?
      |        /        \
      |      No          Yes
      |      |            |
      |   1 BIT         2 BITs (or Segment Tree)
      |   (diff array)     
      |
   USE BIT
   (fastest, simplest)
```

---

## Performance Benchmarks (Typical)

```
Operation counts: n = 10^6, q = 10^6 queries

                    BIT          Segment Tree
                    ────         ────────────
Point update:       ~0.3s        ~0.8s
Prefix query:       ~0.3s        ~0.7s
Range query:        ~0.5s        ~0.8s
Range update+query: ~0.8s        ~1.2s (with lazy)

BIT is typically 2-3× faster due to:
  - No recursion overhead
  - No function call stack
  - Better cache locality
  - Fewer arithmetic operations per step
  - Smaller memory footprint → fewer cache misses
```

---

## When BIT Wins

```
✓ Problem only needs sum or XOR
✓ Time limit is tight
✓ 2D problems (2D BIT is much simpler than 2D SegTree)
✓ You need quick implementation (competitive programming)
✓ Memory limit is tight
✓ Counting/frequency problems

Examples:
  - Inversion counting → BIT
  - Range sum + point update → BIT
  - 2D rectangle sum queries → 2D BIT
  - Online rank queries → BIT
```

---

## When Segment Tree Wins

```
✓ Non-invertible operations (min, max, GCD)
✓ Complex merge functions (max subarray sum, matrix multiplication)
✓ Need lazy propagation for range updates
✓ Need to store multiple values per node
✓ Persistence (persistent segment tree)
✓ Dynamic/implicit segment tree (sparse indices)

Examples:
  - Range minimum query with updates → Segment tree
  - Range set + range sum → Segment tree (lazy)
  - Max subarray sum queries → Segment tree
  - Persistent queries (time-travel) → Persistent segment tree
```

---

## Code Length Comparison

```
BIT (complete implementation):

  B[N+1]  // global array
  
  update(i, val):                    // 3 lines
      while i <= n: B[i] += val; i += i & (-i)
  
  prefix(i):                         // 3 lines
      s = 0; while i > 0: s += B[i]; i -= i & (-i); return s
  
  range_sum(L, R):                   // 1 line
      return prefix(R) - prefix(L-1)
  
  Total: ~10 lines

Segment Tree (sum, no lazy):         // ~30 lines
Segment Tree (with lazy):            // ~50 lines
Segment Tree (multiple operations):  // ~80+ lines

In competitive programming, BIT saves 5-10 minutes of coding time.
```

---

## Can BIT Do Min/Max? (Mostly No)

```
Special case: BIT can answer PREFIX minimum/maximum:
  "What is the minimum of A[1..i]?"

This works because:
  prefix_min(i) = min(prefix_min(i-lowbit(i)), B[i])
  
  The query path naturally visits ALL prefix ranges.
  Since min is associative, this computes the correct answer.

But ARBITRARY RANGE min/max [L, R] with L > 1:
  range_min(L, R) ≠ prefix_min(R) "minus" prefix_min(L-1)
  
  Min has no inverse → can't compute range from prefixes.
  → Must use segment tree (or sparse table for static data).

Summary:
  BIT prefix_min: ✓ (but updates are tricky — can only decrease values)
  BIT range_min:  ✗ (impossible in general)
```

---

## Hybrid Approaches

```
Sometimes both are used together:

1. BIT for sums + Segment tree for min
   When you need both types of queries on the same data

2. BIT for offline + Segment tree for online
   Process easy parts with BIT, complex parts with SegTree

3. BIT for 2D + Segment tree for 1D
   2D BIT handles grid, 1D segment tree handles rows/columns
```

---

## Summary Table

| Criterion | Choose BIT | Choose Segment Tree |
|-----------|-----------|-------------------|
| Operation type | Sum, XOR (invertible) | Min, max, GCD, custom |
| Speed priority | Faster (2-3×) | Adequate |
| Memory constraint | Uses n+1 | Uses 4n to 8n |
| Code simplicity | ~10 lines | ~50+ lines |
| Dimensional extension | 2D easy | 2D hard |
| Lazy propagation | Via tricks | Built-in |
| Persistence | Not practical | Natural |

---

## Quick Revision Questions

1. When should you prefer BIT over segment tree? *(When the operation is invertible (sum, XOR), speed matters, and range update tricks suffice)*
2. Why is BIT faster in practice? *(No recursion, better cache locality, fewer operations per step, smaller memory)*
3. Can BIT handle range minimum queries? *(Not for arbitrary ranges [L,R]; only prefix minimums with restrictions)*
4. How much less memory does BIT use? *(About 4-8× less: n+1 vs 4n to 8n)*
5. When is segment tree the only choice? *(Non-invertible operations, complex merges, lazy propagation, persistence)*

---

[← Previous: 2D BIT](03-2d-bit.md) | [Next: Space Efficiency →](05-space-efficiency.md)
