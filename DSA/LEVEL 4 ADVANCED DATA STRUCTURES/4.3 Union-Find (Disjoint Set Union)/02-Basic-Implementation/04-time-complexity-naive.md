# Chapter 4: Time Complexity (Naive)

## Overview

Without optimizations, Union-Find operations can be **surprisingly slow**. Understanding why motivates the need for path compression and union by rank.

---

## Per-Operation Complexity

| Operation | Time | Depends On |
|-----------|------|-----------|
| MakeSet(x) | O(1) | Nothing |
| Find(x) | O(h) | Height of tree |
| Union(x, y) | O(h) | Height (due to Find) |
| Connected(x, y) | O(h) | Height (due to Find) |

Where **h** = height of the tallest tree involved.

---

## Best Case: O(1)

```
When the tree is perfectly flat:

        0
      / | \ \
     1  2  3  4

Find(any child) = O(1)  — only 1 hop to root
Find(root) = O(1)       — immediate

This happens when all unions go to the same root:
  Union(0,1): parent[1]=0
  Union(0,2): parent[2]=0
  Union(0,3): parent[3]=0
  Union(0,4): parent[4]=0

  All children directly under root → height = 1
```

---

## Worst Case: O(n)

```
When the tree degenerates into a chain:

  0 ─ 1 ─ 2 ─ 3 ─ 4 ─ 5 ─ 6 ─ 7

  (drawn as a tree)
  7
  |
  6
  |
  5
  |
  4
  |
  3
  |
  2
  |
  1
  |
  0

Height = n-1 = 7
Find(0) requires traversing 7 edges → O(n)
```

### How to Create the Worst Case

```
Sequence that creates a chain of height n:
  
  // If Union always attaches rootX under rootY:
  function Union(x, y):
      rootX = Find(x)
      rootY = Find(y)
      parent[rootX] = rootY     // ← rootX becomes child

  Union(0, 1):     1─0
  Union(1, 2):     2─1─0
  Union(2, 3):     3─2─1─0
  Union(3, 4):     4─3─2─1─0
  ...
  Union(n-2, n-1): (n-1)─(n-2)─...─2─1─0

  Each Union makes previous root a child of new root.
  Result: chain of length n.
```

---

## Amortized Analysis (Naive)

For a sequence of **m** operations on **n** elements:

```
Worst case for m operations:

  Each Find can take O(n) in the worst case.
  m operations → O(m · n) total.

  ┌──────────────────────────────────────┐
  │  Total time for m operations:        │
  │                                      │
  │  Worst case:  O(m · n)               │
  │  Average case: O(m · log n)          │
  │  Best case:    O(m)                  │
  └──────────────────────────────────────┘
```

### Concrete Example

```
n = 1000 elements, m = 100,000 operations

Naive worst case: 100,000 × 1,000 = 100,000,000 operations
                  → 10^8 → might TLE (Time Limit Exceeded)

With optimizations: 100,000 × α(1000) ≈ 100,000 × 4 = 400,000
                    → 4 × 10^5 → very fast!
```

---

## Height Growth Analysis

```
How fast does tree height grow?

Naive Union (arbitrary direction):

  n unions can create height up to n-1:
  
  Unions:  0   1   2   3   4   5
  
  U(0,1):  1         height = 1
           |
           0
           
  U(1,2):  2         height = 2
           |
           1
           |
           0
  
  U(2,3):  3         height = 3
           |
           2
           |
           1
           |
           0
  
  After n-1 Unions: height = n-1 = O(n)
  
  Compare with Union by Rank (coming later):
  After n-1 Unions: height ≤ log₂(n) = O(log n)
```

---

## Experimental Comparison

```
Number of elements: n = 10,000
Number of operations: m = 1,000,000

                    │ Naive  │ With Rank │ Rank + PC │
  ──────────────────┼────────┼──────────┼───────────┤
  Worst Find depth  │ 9,999  │    13    │     2     │
  Avg Find depth    │ ~5,000 │    ~7    │    ~1.5   │
  Total operations  │ ~5×10⁹ │  ~7×10⁶  │  ~1.5×10⁶│
  Time (approx)     │  50s   │  0.07s   │   0.015s  │
  
  Naive is 3000x slower than optimized!
```

---

## Space Complexity

```
Naive Union-Find space is always O(n):

  ┌────────────────────────────────────┐
  │  parent[] array: n integers        │
  │  numComponents:  1 integer         │
  │                                    │
  │  Total: O(n) space                 │
  │                                    │
  │  This is OPTIMAL — you need at     │
  │  least n entries to track n items  │
  └────────────────────────────────────┘

  Note: Optimized versions add rank[] or size[]
  arrays, but space remains O(n).
```

---

## Complexity Summary Table

```
┌──────────────┬──────────┬──────────┬─────────────┐
│              │   Best   │ Average  │   Worst     │
├──────────────┼──────────┼──────────┼─────────────┤
│ MakeSet      │   O(1)   │   O(1)   │    O(1)     │
│ Find         │   O(1)   │ O(log n) │    O(n)     │
│ Union        │   O(1)   │ O(log n) │    O(n)     │
│ Connected    │   O(1)   │ O(log n) │    O(n)     │
│ Space        │   O(n)   │   O(n)   │    O(n)     │
├──────────────┼──────────┴──────────┴─────────────┤
│ m operations │  O(m) to O(m·n) total             │
└──────────────┴───────────────────────────────────┘
```

---

## Why We Need Optimizations

```
Problem sizes in competitive programming:
  n ≤ 10^5 to 10^6

Naive:     O(n²) for n Unions → 10^10 to 10^12 → TLE!
Optimized: O(n·α(n)) ≈ O(n)  → 10^5 to 10^6   → AC!

The two key optimizations:
  1. Path Compression → flattens trees during Find
  2. Union by Rank   → keeps trees balanced during Union
  
  Combined: O(α(n)) amortized per operation
            α(n) ≤ 4 for any practical n
```

---

## Summary Table

| Metric | Naive Value | Notes |
|--------|------------|-------|
| Find worst case | O(n) | Degenerate chain |
| Union worst case | O(n) | Dominated by Find |
| m operations worst | O(m·n) | Can be very slow |
| Space | O(n) | Optimal |
| Practical usability | Limited | Fails for n > 10⁴ |
| Solution | Optimize! | Path comp. + rank |

---

## Quick Revision Questions

1. **What determines the time complexity of Find in the naive version?**
2. **What sequence of Unions creates the worst-case tree shape?**
3. **What is the total worst-case time for m operations on n elements?**
4. **Why is the average case O(log n) even without optimizations?**
5. **At what problem size does naive Union-Find become impractical?**

---

[← Previous: Naive Implementation](03-naive-implementation.md) | [Next: Example Walkthrough →](05-example-walkthrough.md)
