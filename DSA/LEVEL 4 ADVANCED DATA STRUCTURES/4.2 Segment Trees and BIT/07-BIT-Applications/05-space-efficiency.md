# Space Efficiency

## Chapter Overview

BIT's space efficiency is one of its greatest advantages. This chapter explores the memory layout, cache behavior, and practical optimizations that make BIT the most memory-efficient dynamic range query structure.

---

## Memory Comparison

```
For an array of n = 10^6 elements (integers):

Structure            Arrays        Memory (int32)     Ratio
─────────────────    ──────        ──────────────     ─────
BIT                  1 array       n + 1 = ~4 MB      1×
Segment Tree         1 array       4n = ~16 MB         4×
Seg Tree + Lazy      2 arrays      8n = ~32 MB         8×
Sparse Table         log(n) arrays n·log n = ~80 MB   20×

BIT uses 4-8× less memory than a segment tree!
```

---

## Why BIT Uses Less Memory

```
BIT stores n + 1 values (1-indexed):
  B[1], B[2], ..., B[n]

Each B[i] stores the sum of a specific range determined
by the binary representation of i.

There are NO extra nodes:
  - No internal nodes (segment tree has ~n internal nodes)
  - No lazy array
  - No sentinel values
  
The original array is NOT stored separately:
  B[i] for odd i stores exactly A[i] (leaf-like behavior)
  B[i] for even i stores the cumulative sum of a range

Total: just 1 array of size n + 1.
```

---

## Cache Performance

```
Cache Line (typically 64 bytes = 16 int32 values):

BIT query for prefix_sum(15):
  Visit: B[15] → B[14] → B[12] → B[8]
  Indices: 15, 14, 12, 8
  
  These indices trend DOWNWARD → likely in same or nearby cache lines.
  
  Best case: All indices in 1-2 cache lines.
  Worst case: O(log n) cache misses.

Segment Tree query for range_sum(1, 15):
  Visit: nodes at various levels
  Indices might be: 1, 2, 5, 11, 23, ...
  
  These indices SPREAD across the tree.
  Higher chance of cache misses.

Result: BIT has better cache locality on average.

              Memory Access Pattern
              ─────────────────────
  BIT:    [▓▓▓▓░░░░░░░░░░░░░░]  ← compact, localized
  SegTree: [▓░░▓░░░░▓░░░░░░▓░]  ← scattered
```

---

## Constant Factor Analysis

```
Operations per query/update step:

BIT:
  while i > 0:          // 1 comparison
      s += B[i]         // 1 array access + 1 addition
      i -= i & (-i)     // 1 AND + 1 negation + 1 subtraction
  
  Per step: ~5 operations + 1 memory access
  Total: ~5 · log(n) operations

Segment Tree (recursive):
  query(node, L, R, qL, qR):    // function call overhead
      if qL > R or qR < L:      // 2 comparisons
          return identity
      if qL <= L and R <= qR:   // 2 comparisons
          return tree[node]      // 1 memory access
      mid = (L + R) / 2         // 1 addition + 1 division
      return merge(             // function call
          query(2*node, ...),   // recursive call
          query(2*node+1, ...)) // recursive call
  
  Per step: ~8 operations + 2 recursive calls + memory access
  Total: much higher constant factor

Aspect               BIT        Segment Tree
──────               ───        ────────────
Function calls       0          O(log n) recursive calls
Stack frames         0          O(log n) frames
Comparisons/step     1          4+
Arithmetic/step      3          3+
Memory accesses      1          1-2
```

---

## Memory Layout Optimization

```
BIT is naturally array-based → no pointer overhead.

Comparison with pointer-based segment tree:

Array-based segment tree (4n):
  ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
  │ 0│ 1│ 2│ 3│ 4│ 5│ 6│ 7│ 8│ 9│10│
  └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
  Contiguous → moderate cache performance
  But 4× the data of BIT

Pointer-based segment tree:
  Each node: value + left_ptr + right_ptr = 3 fields
  Plus memory allocation overhead
  MUCH worse cache performance (nodes scattered in heap)

BIT array:
  ┌──┬──┬──┬──┬──┬──┬──┬──┐
  │ 1│ 2│ 3│ 4│ 5│ 6│ 7│ 8│
  └──┴──┴──┴──┴──┴──┴──┴──┘
  Minimal, contiguous → best cache performance
```

---

## Practical Optimizations for BIT

### 1. Use 1-Indexed Array

```
B[0..n]   // index 0 unused, simplifies lowbit logic

// DON'T use 0-indexing — it breaks the lowbit trick:
//   0 & (-0) = 0  → infinite loop!
```

### 2. Build in O(n) Instead of O(n log n)

```
// Naive build: n updates × O(log n) each = O(n log n)
for i from 1 to n:
    update(i, A[i])

// Optimized O(n) build:
copy A[1..n] into B[1..n]
for i from 1 to n:
    parent = i + (i & (-i))
    if parent <= n:
        B[parent] += B[i]
```

### 3. Binary-Lifting k-th Query

```
// Find the smallest prefix sum ≥ target in O(log n)
// Works when all values are non-negative

find_kth(target):
    pos = 0
    for bit from log2(n) down to 0:
        next_pos = pos + (1 << bit)
        if next_pos <= n AND B[next_pos] < target:
            target -= B[next_pos]
            pos = next_pos
    return pos + 1
```

### 4. Batch Updates

```
// If you have many updates before any query:
// Don't update BIT one by one

// Instead: store updates in original array, then build BIT once
// O(n) build instead of O(k log n) for k updates
```

---

## Space Efficiency in 2D

```
2D BIT:  n × m array → n × m storage
2D SegTree (array-based): 4n × 4m = 16nm storage
2D SegTree (pointer): even worse with pointer overhead

For n = m = 1000:
  2D BIT:     ~4 MB
  2D SegTree: ~64 MB    ← may exceed memory limit!

This is why 2D BIT is heavily preferred in competitive programming.
```

---

## When Memory Matters

```
Problem constraints that favor BIT:

1. Large n (>10^6): Segment tree may TLE or MLE
2. 2D problems: 2D segment tree often exceeds memory
3. Multiple BITs needed: e.g., two BITs for range update+query
   Still uses 2n+2 ≪ 8n for lazy segment tree
4. Online judge memory limits: 256 MB common limit
   For n = 5×10^7: BIT fits, segment tree doesn't
```

---

## Summary Table

| Aspect | BIT | Segment Tree |
|--------|-----|-------------|
| Array size | n + 1 | 4n (+ 4n lazy) |
| Memory ratio | 1× | 4-8× |
| Cache locality | Excellent | Moderate |
| Function call overhead | None | O(log n) |
| 2D memory | n × m | 16 × n × m |
| Build optimization | O(n) possible | Always O(n) |
| Pointer overhead | None | None (array) / High (pointer) |

---

## Quick Revision Questions

1. How much less memory does BIT use vs a lazy segment tree? *(About 8× less: n+1 vs 8n)*
2. Why does BIT have better cache performance? *(Query indices trend close together; array is compact and contiguous)*
3. Why can't BIT use 0-indexing? *(0 & (-0) = 0, causing an infinite loop in the lowbit operation)*
4. What is the optimized BIT build time? *(O(n) using parent propagation, instead of O(n log n) individual updates)*
5. Why is 2D BIT preferred over 2D segment tree? *(16× less memory; 2D segment tree often exceeds memory limits)*

---

[← Previous: BIT vs Segment Tree](04-bit-vs-segment-tree.md) | [Next: When to Use BIT →](06-when-to-use-bit.md)
