# When to Use BIT

## Chapter Overview

This chapter serves as a practical decision guide. Given a problem, should you reach for a BIT, a segment tree, or something else entirely? We provide a classification framework, pattern recognition tips, and competitive programming heuristics.

---

## The Decision Framework

```
Step 1: What type of query?
  ┌──────────────────────────┐
  │ Is the query a PREFIX    │
  │ operation? (1..i)        │──Yes──→ BIT is ideal
  └──────────────────────────┘
              │ No
              ▼
  ┌──────────────────────────┐
  │ Can it be expressed as   │
  │ prefix(R) - prefix(L-1)?│──Yes──→ BIT works (invertible op)
  └──────────────────────────┘
              │ No
              ▼
  ┌──────────────────────────┐
  │ Operation is non-        │
  │ invertible (min,max,GCD) │──→ Use Segment Tree
  └──────────────────────────┘

Step 2: What type of update?
  ┌──────────────────────────┐
  │ Point update only?       │──Yes──→ BIT is perfect
  └──────────────────────────┘
              │ No
              ▼
  ┌──────────────────────────┐
  │ Range update +           │
  │ point query?             │──Yes──→ 1 BIT (difference array)
  └──────────────────────────┘
              │ No
              ▼
  ┌──────────────────────────┐
  │ Range update +           │
  │ range query (invertible)?│──Yes──→ 2 BITs (or Segment Tree)
  └──────────────────────────┘
              │ No
              ▼
      Use Segment Tree with Lazy Propagation
```

---

## Problem Classification

### Category 1: BIT is Best

```
These problems scream "use BIT":

1. Dynamic prefix/range sums
   "After updates, find sum of A[L..R]"
   → Standard BIT

2. Counting inversions
   "How many pairs (i,j) with i < j and A[i] > A[j]?"
   → BIT as frequency counter

3. Coordinate compression + counting
   "How many elements less than X seen so far?"
   → BIT on compressed values

4. 2D rectangle sum queries
   "Sum of elements in sub-rectangle with updates"
   → 2D BIT

5. Range increment + point query
   "Add val to all elements in [L,R], then query A[i]"
   → Difference BIT

6. Online rank queries
   "What is the rank of X among elements seen so far?"
   → BIT on frequency array
```

### Category 2: Segment Tree Required

```
These need segment tree — BIT won't work:

1. Range min/max queries with updates
   → Segment tree (min is not invertible)

2. Range set (assign) operations
   → Lazy segment tree (set overwrites, not additive)

3. Max subarray sum
   → Segment tree with 4-field merge

4. Historical queries / persistence
   → Persistent segment tree

5. Distinct elements in range
   → Merge sort tree or persistent segment tree

6. Custom complex merges
   → Segment tree with custom node type
```

### Category 3: Neither (Better alternatives)

```
1. Static RMQ → Sparse Table (O(1) query after O(n log n) build)
2. Static range sum → Prefix sums (O(1) query, no update)
3. Offline queries → Mo's algorithm or sort-based approaches
4. Very simple updates → Sqrt decomposition (easy to write)
```

---

## Competitive Programming Heuristics

```
Quick rules of thumb:

1. "sum" or "count" in the problem → Try BIT first
2. "min" or "max" with updates → Segment tree
3. n ≤ 10^5 and lazy needed → Segment tree
4. n ≥ 10^6 and sum only → BIT (safer on time)
5. 2D grid queries → BIT (memory-safe)
6. Need to code fast → BIT (10 lines vs 50+)
7. Not sure if BIT works → Use segment tree (safe default)

Time investment:
  BIT: 2-3 minutes to code
  Segment tree: 5-10 minutes to code
  Segment tree + lazy: 10-15 minutes to code
```

---

## Pattern Recognition

### Signal Words for BIT

```
"prefix sum"          → direct BIT application
"cumulative"          → prefix query
"number of elements"  → frequency BIT
"inversions"          → BIT counting
"rank", "order"       → BIT on sorted values
"XOR of range"        → BIT (XOR is invertible, XOR is its own inverse)
"toggle"              → XOR BIT
"2D", "grid", "matrix"→ consider 2D BIT
"coordinate compress" → BIT after compression
```

### Signal Words for Segment Tree

```
"minimum", "maximum"  → segment tree (non-invertible)
"GCD", "LCM"          → segment tree
"set all to X"        → lazy segment tree
"multiply range"      → lazy segment tree
"max subarray"        → segment tree (complex merge)
"persistent"          → persistent segment tree
"k-th smallest"       → merge sort tree
```

---

## Common Mistakes

```
Mistake 1: Using segment tree when BIT suffices
  Impact: 3× slower, 4× more memory, more bugs
  Fix: Check if operation is invertible first

Mistake 2: Using BIT for min/max
  Impact: Wrong answers for range queries with L > 1
  Fix: Min/max are not invertible → use segment tree

Mistake 3: Forgetting coordinate compression
  Impact: Array too large (values up to 10^9)
  Fix: Compress to [1..n] before using BIT

Mistake 4: 0-indexed BIT
  Impact: Infinite loop (0 & -0 = 0)
  Fix: Always use 1-indexed BIT

Mistake 5: Building BIT in O(n log n) when O(n) is possible
  Impact: Slower build when n is large
  Fix: Use parent propagation build
```

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────┐
│           WHEN TO USE BIT — CHEAT SHEET         │
├─────────────────────────────────────────────────┤
│                                                 │
│  ✓ Sum / XOR / addition-based queries           │
│  ✓ Point update + range query                   │
│  ✓ Range update + point query (diff array)      │
│  ✓ Counting / frequency problems                │
│  ✓ 2D grid queries                              │
│  ✓ Speed or memory critical                     │
│  ✓ Quick to implement                           │
│                                                 │
│  ✗ Min / Max / GCD queries                      │
│  ✗ Range set (assign) operations                │
│  ✗ Complex merges (max subarray, matrix)        │
│  ✗ Persistence / versioning                     │
│  ✗ Non-associative or non-invertible ops        │
│                                                 │
│  When unsure → segment tree (always safe)       │
│  When sum/count + speed → BIT                   │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## Summary Table

| Problem Type | Best Structure | Why |
|-------------|---------------|-----|
| Range sum + point update | BIT | Fastest, simplest |
| Range sum + range update | 2 BITs or Lazy SegTree | BIT if speed needed |
| Range min/max | Segment Tree | Non-invertible |
| Inversion count | BIT | Frequency counting |
| 2D range sum | 2D BIT | Memory efficient |
| Max subarray sum | Segment Tree | Complex merge |
| Static RMQ | Sparse Table | O(1) query |
| Range set + query | Lazy Segment Tree | Overwrite semantics |

---

## Quick Revision Questions

1. What is the first question to ask when deciding BIT vs segment tree? *(Is the operation invertible? If yes, BIT can work; if no, use segment tree)*
2. Can BIT handle range update + range query? *(Yes, using 2 BITs for invertible operations like sum)*
3. What problem type most strongly indicates BIT? *(Counting/frequency problems and prefix sums)*
4. When should you always choose segment tree over BIT? *(Non-invertible operations: min, max, GCD, or when lazy propagation with complex updates is needed)*
5. What is the safe default if unsure? *(Segment tree — it handles all associative operations)*
6. Why is BIT preferred in competitive programming when applicable? *(2-3 minutes to code vs 10-15 minutes; 2-3× faster; fewer bugs)*

---

[← Previous: Space Efficiency](05-space-efficiency.md) | [Next: 2D Segment Tree →](../08-Advanced-Segment-Tree/01-2d-segment-tree.md)
