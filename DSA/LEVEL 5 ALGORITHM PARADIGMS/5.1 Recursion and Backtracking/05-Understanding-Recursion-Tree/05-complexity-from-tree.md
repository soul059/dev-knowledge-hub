# Chapter 5: Deriving Complexity From Recursion Trees

## Overview
The recursion tree method is one of the most **visual and intuitive** ways to derive time complexity. Instead of solving recurrences algebraically, you **draw the tree, sum the work at each level**, and find the total.

---

## The 4-Step Method

```
┌────────────────────────────────────────────────────┐
│  Step 1: DRAW the tree (at least 3-4 levels)       │
│  Step 2: COMPUTE work at each level                │
│  Step 3: SUM across all levels                     │
│  Step 4: IDENTIFY the dominant term                │
└────────────────────────────────────────────────────┘
```

---

## Example 1: T(n) = 2T(n/2) + n (Merge Sort)

### Step 1: Draw
```
Level 0:            cn                    ← 1 node, work = cn
                   /    \
Level 1:       cn/2      cn/2             ← 2 nodes, work = cn/2 each
               / \        / \
Level 2:    cn/4  cn/4  cn/4  cn/4        ← 4 nodes, work = cn/4 each
              ...
Level k:    cn/2^k  (repeated 2^k times)  ← 2^k nodes
              ...
Level log n: c c c c c ... c              ← n nodes, work = c each
```

### Step 2: Work per Level
```
Level 0: 1 × cn      = cn
Level 1: 2 × cn/2    = cn
Level 2: 4 × cn/4    = cn
Level 3: 8 × cn/8    = cn
...
Level i: 2^i × cn/2^i = cn    ← SAME at every level!
...
Level log n:  n × c   = cn
```

### Step 3: Sum
```
Total = cn × (log₂n + 1) = cn·log₂n + cn
```

### Step 4: Result
```
T(n) = O(n log n)  ✓

This is the UNIFORM case: same work at every level.
```

---

## Example 2: T(n) = 2T(n/2) + n² (Root-Heavy)

### Tree
```
Level 0:        cn²                  work = cn²
               /    \
Level 1:   c(n/2)²  c(n/2)²         work = 2·c·n²/4 = cn²/2
            / \       / \
Level 2: c(n/4)²×4               work = 4·c·n²/16 = cn²/4
```

### Work per Level
```
Level 0: cn²
Level 1: cn²/2
Level 2: cn²/4
Level 3: cn²/8
...
Level i: cn²/2^i

Total = cn²(1 + 1/2 + 1/4 + 1/8 + ...)
      = cn² × 2    (geometric series, ratio 1/2)
      = O(n²)

ROOT-HEAVY: The root level dominates!
```

---

## Example 3: T(n) = 2T(n-1) + 1 (Leaf-Heavy / Exponential)

### Tree
```
Level 0:        1                    work = 1
               / \
Level 1:      1   1                  work = 2
             / \ / \
Level 2:    1  1 1  1                work = 4
           ...
Level n:   1 1 1 1 ... 1 1           work = 2^n
```

### Work per Level
```
Level 0: 1
Level 1: 2
Level 2: 4
...
Level i: 2^i
...
Level n: 2^n

Total = 1 + 2 + 4 + ... + 2^n = 2^(n+1) - 1 = O(2^n)

LEAF-HEAVY: Bottom level dominates!
```

---

## Example 4: T(n) = T(n/3) + T(2n/3) + n (Unbalanced)

### Tree
```
Level 0:           cn                         work = cn
                 /      \
Level 1:      cn/3     2cn/3                  work = cn
              /  \      /    \
Level 2:  cn/9  2cn/9  2cn/9  4cn/9           work = cn
           ...
```

### Key Insight: Uneven Splitting
```
Shortest path: n → n/3 → n/9 → ... → 1   depth = log₃(n)
Longest path:  n → 2n/3 → 4n/9 → ... → 1  depth = log₃⸝₂(n)

Work at each FULL level = cn  (same as merge sort!)
Number of full levels ≥ log₃(n)
Number of levels ≤ log₃⸝₂(n)

Total = cn × Θ(log n) = O(n log n)  ✓
```

---

## The Three Cases Pattern

```
For T(n) = aT(n/b) + f(n):

Compare f(n) with n^(log_b(a)):

┌─────────────────────────────────────────────────────┐
│                                                     │
│  CASE 1: f(n) < n^(log_b a)   → LEAF-HEAVY         │
│          Leaves dominate                            │
│          T(n) = Θ(n^(log_b a))                      │
│                                                     │
│  CASE 2: f(n) = n^(log_b a)   → UNIFORM            │
│          All levels equal                           │
│          T(n) = Θ(n^(log_b a) · log n)              │
│                                                     │
│  CASE 3: f(n) > n^(log_b a)   → ROOT-HEAVY         │
│          Root dominates                             │
│          T(n) = Θ(f(n))                             │
│                                                     │
│  This is the Master Theorem!                        │
└─────────────────────────────────────────────────────┘
```

---

## Quick Reference: Common Recurrences

```
┌────────────────────────────────┬──────────┬────────────┐
│ Recurrence                     │ Tree Type│ Complexity  │
├────────────────────────────────┼──────────┼────────────┤
│ T(n) = T(n-1) + O(1)          │ Linear   │ O(n)       │
│ T(n) = T(n-1) + O(n)          │ Linear   │ O(n²)      │
│ T(n) = 2T(n-1) + O(1)         │ Leaf-    │ O(2^n)     │
│ T(n) = T(n/2) + O(1)          │ Linear   │ O(log n)   │
│ T(n) = T(n/2) + O(n)          │ Root-    │ O(n)       │
│ T(n) = 2T(n/2) + O(n)         │ Uniform  │ O(n log n) │
│ T(n) = 2T(n/2) + O(1)         │ Leaf-    │ O(n)       │
│ T(n) = 2T(n/2) + O(n²)        │ Root-    │ O(n²)      │
│ T(n) = 3T(n/2) + O(n)         │ Leaf-    │ O(n^1.58)  │
│ T(n) = T(n/2) + T(n/2) + O(1) │ Leaf-    │ O(n)       │
│ T(n) = nT(n-1)                │ Factorial│ O(n!)      │
└────────────────────────────────┴──────────┴────────────┘
```

---

## Worked Example: Deriving O(1.618^n) for Fibonacci

```
T(n) = T(n-1) + T(n-2) + O(1)

The tree is NOT balanced:
- Left branch: depth n (subtracting 1)
- Right branch: depth n/2 (subtracting 2, roughly)

Let T(n) = φ^n where φ is the golden ratio:
φ^n = φ^(n-1) + φ^(n-2)
φ² = φ + 1
φ = (1 + √5) / 2 ≈ 1.618

So T(n) = O(φ^n) = O(1.618...^n)

This is LESS than O(2^n) because the tree is unbalanced:
- Right subtree is always 1 level shorter
- Many branches terminate early
```

---

## Summary Table

| Tree Shape | Work Distribution | Total Complexity | Dominated By |
|-----------|-------------------|-----------------|--------------|
| Balanced + O(n) merge | Uniform across levels | O(n log n) | All levels equally |
| Balanced + O(n²) merge | Decreasing (root heavy) | O(n²) | Root level |
| Balanced + O(1) merge | Increasing (leaf heavy) | O(n) | Leaf level |
| Linear + O(1) | Uniform | O(n) | All levels |
| Binary + O(1) per node | Exponential growth | O(2^n) | Leaves |

---

## Quick Revision Questions

1. **What are the 4 steps** of the recursion tree method?
2. **For T(n) = 4T(n/2) + n**, which case does it fall into?
3. **Why is the Fibonacci tree** NOT perfectly balanced?
4. **What geometric series** appears in root-heavy trees?
5. **How does the Master Theorem** relate to the three cases?
6. **Derive the complexity** of T(n) = 3T(n/3) + n using a tree.

---

[← Previous: Tree Structure Analysis](04-tree-structure-analysis.md) | [Next: Leading to DP →](06-leading-to-dp.md)

[← Back to README](../README.md)
