# Time Complexity

## Chapter Overview

This chapter provides rigorous time complexity analysis for all segment tree operations, proving why each achieves O(log n) per operation and O(n) for build.

---

## Summary of Complexities

```
┌──────────────────┬──────────────┬───────────────────────┐
│    Operation     │   Time       │  Amortized per op     │
├──────────────────┼──────────────┼───────────────────────┤
│  Build           │  O(n)        │  N/A (one-time)       │
│  Point Query     │  O(log n)    │  O(log n)             │
│  Range Query     │  O(log n)    │  O(log n)             │
│  Point Update    │  O(log n)    │  O(log n)             │
│  Range Update*   │  O(log n)    │  O(log n)             │
│  Q operations    │  O(Q log n)  │  O(log n) per op      │
└──────────────────┴──────────────┴───────────────────────┘
* With lazy propagation
```

---

## Build: O(n)

### Approach 1: Counting nodes
```
Total nodes visited = total nodes in tree ≤ 4n
Work per node = O(1) (one merge operation)
Total work = O(4n) = O(n)
```

### Approach 2: Recurrence relation
```
T(n) = 2 × T(n/2) + O(1)

Apply Master Theorem:
  a = 2, b = 2, f(n) = O(1) = O(n^0)
  log_b(a) = log_2(2) = 1
  Since 0 < 1: Case 1 applies
  T(n) = O(n^1) = O(n)
```

### Approach 3: Level-by-level analysis
```
Level 0:  1 node    × O(1) work = O(1)
Level 1:  2 nodes   × O(1) work = O(2)
Level 2:  4 nodes   × O(1) work = O(4)
...
Level k:  2^k nodes × O(1) work = O(2^k)

Total = 1 + 2 + 4 + ... + 2^(log n)
      = 2^(log n + 1) - 1
      = 2n - 1
      = O(n)
```

---

## Query: O(log n)

### Why not O(n)?

It might seem like both children are visited at every partial overlap, leading to exponential branching. But the number of partial-overlap nodes is bounded.

### Proof: At most 4 nodes per level

```
Key observation: At any level of the tree, the query [L, R] 
can interact with nodes in exactly 3 ways:

  Nodes fully OUTSIDE [L,R]:  immediately return (O(1))
  Nodes fully INSIDE [L,R]:   return tree[node] (O(1), no recursion)
  Nodes PARTIALLY in [L,R]:   recurse into children

At any level, at most 2 nodes can be partially overlapping:
  - The leftmost partially-overlapping node (contains L)
  - The rightmost partially-overlapping node (contains R)

All nodes between them are fully inside [L,R] → no recursion.
All nodes outside them are fully outside [L,R] → no recursion.

So at each level:
  ≤ 2 partial nodes that spawn recursive calls
  ≤ O(1) full/empty nodes encountered before stopping

Total active nodes per level: ≤ 4
Levels: ≤ log₂(n) + 1

Total: O(4 × log n) = O(log n)
```

### Visual proof
```
Level 0:  [───────────────── root ──────────────────]   1 partial
Level 1:  [──── left ────]  [──── right ────]           ≤2 partial
Level 2:  [a] [b] [c] [d]  [e] [f] [g] [h]            ≤2 partial
Level 3:  ...                                           ≤2 partial

Query [L ────────────── R]:

Level 2:  [a] [b] [c] [d]  [e] [f] [g] [h]
           ○   ●   ●   ●    ●   ●   ○   ○
           ↑    ↑               ↑   ↑
          OUT  PARTIAL     PARTIAL  OUT
               (L here)   (R here)
          ↑ these return identity
                    ↑ these return tree[node] directly
               ↑                   ↑
          only THESE two recurse further
```

---

## Update: O(log n)

### Analysis
```
Update traverses from root to ONE leaf:
  At each level: visit exactly 1 node
  Make exactly 1 recursive call (left OR right)
  Perform O(1) merge on return

Levels: log₂(n) + 1
Work per level: O(1)
Total: O(log n)

This is trivially O(log n) — much simpler than query analysis.
```

---

## Total Cost for Q Operations

```
Scenario: Build once, then Q mixed queries and updates.

Total time = Build + Q × (Query or Update)
           = O(n) + Q × O(log n)
           = O(n + Q log n)

Practical examples:

n = 10^5, Q = 10^5:
  Build: 10^5
  Queries: 10^5 × 17 ≈ 1.7 × 10^6
  Total: ~2 × 10^6 operations  → well within 1 second

n = 10^6, Q = 10^6:
  Build: 10^6
  Queries: 10^6 × 20 = 2 × 10^7
  Total: ~2 × 10^7 operations → comfortable

n = 10^6, Q = 10^6 (brute force comparison):
  Total: 10^6 × 10^6 = 10^12 → TLE!
```

---

## Constant Factors

```
Segment tree has higher constant factors than simpler structures:

┌────────────────────┬───────────────────────┐
│  Structure         │  Constant Factor      │
├────────────────────┼───────────────────────┤
│  Array scan        │  ~1× (cache-friendly) │
│  Prefix sum query  │  ~1× (2 lookups)      │
│  BIT query/update  │  ~2-3× per log n op   │
│  Seg tree query    │  ~4-6× per log n op   │
│  Seg tree + lazy   │  ~8-10× per log n op  │
└────────────────────┴───────────────────────┘

The recursive calls, cache misses, and function overhead
make segment trees ~3-5× slower than BIT for the same task.

Rule of thumb: Segment tree handles ~10^7 operations/second.
```

---

## Comparison with Alternatives

```
For Q queries + U updates on array of size n:

┌──────────────────┬────────────────────┬───────────────────┐
│  Approach        │  Total Time        │  For n=Q=10^5     │
├──────────────────┼────────────────────┼───────────────────┤
│  Brute force     │  O(n × (Q+U))      │  10^10 → TLE     │
│  Prefix rebuild  │  O(n × U + Q)      │  10^10 → TLE     │
│  Sqrt decomp     │  O((Q+U) × √n)    │  3 × 10^7 → OK   │
│  BIT             │  O((Q+U) × log n)  │  3 × 10^6 → FAST │
│  Segment tree    │  O((Q+U) × log n)  │  3 × 10^6 → FAST │
│  Seg tree + lazy │  O((Q+U) × log n)  │  3 × 10^6 → FAST │
└──────────────────┴────────────────────┴───────────────────┘
```

---

## Summary Table

| Operation | Time | Why |
|-----------|:----:|-----|
| Build | O(n) | Visit each of ~2n nodes once |
| Query | O(log n) | ≤ 4 nodes per level × O(log n) levels |
| Point Update | O(log n) | Single root-to-leaf path |
| Q operations | O(Q log n) | Each operation is independent |
| Build + Q ops | O(n + Q log n) | Build once, then O(log n) per op |

---

## Quick Revision Questions

1. Why is building O(n) and not O(n log n)? *(Each of the ~2n nodes is visited exactly once, doing O(1) work each)*
2. Why is range query O(log n) despite recursing into both children? *(At each level, at most 2 nodes create further recursion; most nodes terminate immediately)*
3. What is the total time for 10⁵ builds followed by 10⁵ queries? *(O(n + Q log n) ≈ 10⁵ + 10⁵ × 17 ≈ 2 × 10⁶)*
4. Why is segment tree slower than BIT in practice for the same O(log n) complexity? *(Higher constant factor due to recursion overhead, cache misses, and more comparisons)*
5. What is the recurrence for build time? *(T(n) = 2T(n/2) + O(1) → O(n))*

---

[← Previous: Update Function](04-update-function.md) | [Next: Space Complexity →](06-space-complexity.md)
