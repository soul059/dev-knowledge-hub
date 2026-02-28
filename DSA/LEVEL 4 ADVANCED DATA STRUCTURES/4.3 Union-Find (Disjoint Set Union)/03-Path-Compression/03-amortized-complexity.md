# Chapter 3: Amortized Complexity

## Overview

Path compression doesn't make individual operations O(1), but it makes the **average** (amortized) cost per operation nearly constant. This chapter explains the amortized analysis.

---

## What is Amortized Analysis?

```
Amortized analysis looks at the TOTAL cost of a SEQUENCE of operations,
not individual operations.

Individual operation cost: Can vary widely (O(1) to O(n))
Amortized cost per operation: Total cost / Number of operations

Analogy: 
  Some months you spend $500 on car maintenance, most months $0.
  Amortized monthly cost: ~$42/month (spread over the year)
```

---

## Path Compression Alone: O(log* n)

With **only** path compression (no union by rank), the amortized cost is O(log* n).

### What is log* n? (Iterated Logarithm)

```
log*(n) = number of times you need to take log₂ until you reach ≤ 1

log*(2)     = 1        (log₂(2) = 1)
log*(4)     = 2        (log₂(4) = 2, log₂(2) = 1)
log*(16)    = 3        (16 → 4 → 2 → 1)
log*(65536) = 4        (65536 → 16 → 4 → 2 → 1)
log*(2^65536) = 5      (2^65536 → 65536 → 16 → 4 → 2 → 1)

  ┌───────────────────────────────────┐
  │  n              │  log*(n)        │
  ├──────────────────┼────────────────┤
  │  1               │  0             │
  │  2               │  1             │
  │  4               │  2             │
  │  16              │  3             │
  │  65,536          │  4             │
  │  2^65536         │  5             │
  │  (atoms in       │  ≤ 5           │
  │   universe)      │                │
  └──────────────────┴────────────────┘

  log*(n) ≤ 5 for ALL practical values of n!
  Practically: log*(n) = O(1)
```

---

## Path Compression + Union by Rank: O(α(n))

With **both** optimizations, the amortized cost becomes O(α(n)) — the **inverse Ackermann function**.

```
α(n) grows even SLOWER than log*(n):

  ┌──────────────────┬────────┬─────────┐
  │  n               │ log*(n)│  α(n)   │
  ├──────────────────┼────────┼─────────┤
  │  1               │  0     │  0      │
  │  4               │  2     │  2      │
  │  16              │  3     │  3      │
  │  65,536          │  4     │  3      │
  │  2^65536         │  5     │  4      │
  │  10^80 (atoms!)  │  5     │  4      │
  └──────────────────┴────────┴─────────┘

  α(n) ≤ 4 for any conceivable input size!
```

---

## Intuitive Explanation

```
Why is path compression so effective?

Round 1: Find traverses a long path, but FLATTENS it
  Before:  0─1─2─3─4─5─6─7─8─9
  After:   0←1, 0←2, 0←3, ..., 0←9

Round 2: Same elements? Find is now O(1)
  Find(9): 9→0 (direct!)

The KEY insight:
  Each node can only be "compressed" log*(n) times before
  it points directly to the root.
  
  After that, every Find involving that node is O(1).
  
  Total work for ALL compressions: O(n · log*(n))
  Spread over m operations: O(log*(n)) amortized each
```

---

## The Accounting Method

```
Think of each node having "credits" (potential energy):

Initial state: Each non-root node has log*(n) credits

When Find traverses node x:
  - If x is compressed (parent changes): spend 1 credit
  - x can only be compressed log*(n) times total
  - After credits exhausted: x points directly to root

Total credits: n × log*(n)
Total operations: m
Amortized cost: O((n × log*(n) + m) / m) = O(log*(n) + n/m)

For m ≥ n (typical): O(log*(n)) amortized per operation
```

---

## Practical Impact

```
For n = 1,000,000 elements and m = 5,000,000 operations:

  ┌────────────────────┬─────────────────┬───────────┐
  │ Method             │ Total Operations│ Time      │
  ├────────────────────┼─────────────────┼───────────┤
  │ Naive              │ 5×10¹²          │ ~hours    │
  │ Path Compression   │ 2.5×10⁷         │ ~0.1s     │
  │ PC + Union by Rank │ 2×10⁷           │ ~0.08s    │
  └────────────────────┴─────────────────┴───────────┘

  Path compression alone provides 99.99% of the speedup!
  Union by rank adds a small constant factor improvement.
```

---

## Why Not O(1) Per Operation?

```
Individual operations CAN be expensive:

  Even with path compression, the FIRST Find on a deep node
  must traverse the entire path:

  First Find(deep_node):
       0
       |
       1               Path length = k
       |               First Find = O(k)
       ...              
       |               But this is the LAST time!
       k               Future Find(deep_node) = O(1)

  The expensive operations "pay for" all future cheap operations.
  This is the essence of amortized analysis.
```

---

## Comparison of Complexities

```
Per-operation amortized complexity:

┌────────────────────────────────┬───────────┬─────────────┐
│ Optimization                   │ Amortized │ Practical   │
├────────────────────────────────┼───────────┼─────────────┤
│ None (naive)                   │ O(n)      │ Slow        │
│ Path compression only          │ O(log* n) │ Very fast   │
│ Union by rank only             │ O(log n)  │ Fast        │
│ Path compression + rank        │ O(α(n))   │ Fastest     │
├────────────────────────────────┼───────────┼─────────────┤
│ Theoretical lower bound        │ Ω(α(n))   │ Optimal!    │
└────────────────────────────────┴───────────┴─────────────┘

  O(α(n)) is OPTIMAL — proven by Fredman & Saks (1989)
  You can't do better than this with pointer-based DSU!
```

---

## Total Complexity for m Operations on n Elements

```
┌────────────────────────────────┬────────────────────┐
│ Version                        │ Total for m ops    │
├────────────────────────────────┼────────────────────┤
│ Naive                          │ O(m · n)           │
│ Path compression only          │ O(m · log* n)      │
│ Union by rank only             │ O(m · log n)       │
│ Both optimizations             │ O(m · α(n))        │
│ Both (since α(n) ≤ 4)         │ ≈ O(m)             │
└────────────────────────────────┴────────────────────┘
```

---

## Summary Table

| Concept | Value |
|---------|-------|
| log*(n) for n = 10^80 | ≤ 5 |
| α(n) for n = 10^80 | ≤ 4 |
| Amortized cost (PC only) | O(log* n) |
| Amortized cost (PC + Rank) | O(α(n)) |
| Is O(α(n)) optimal? | Yes, proven lower bound |
| Practical implication | ≈ O(1) per operation |

---

## Quick Revision Questions

1. **What is amortized analysis and how does it differ from worst-case analysis?**
2. **What is log*(n) and why is it effectively constant?**
3. **What is the amortized complexity with only path compression?**
4. **What is the amortized complexity with both path compression and union by rank?**
5. **Why can an individual Find still be expensive even with path compression?**
6. **Is O(α(n)) the theoretical lower bound for DSU? Who proved it?**

---

[← Previous: Implementation Techniques](02-implementation-techniques.md) | [Next: Visual Demonstration →](04-visual-demonstration.md)
