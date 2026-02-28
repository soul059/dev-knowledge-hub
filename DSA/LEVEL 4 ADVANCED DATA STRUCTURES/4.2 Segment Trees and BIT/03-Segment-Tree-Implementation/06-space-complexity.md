# Space Complexity

## Chapter Overview

Segment trees use more memory than the original array. This chapter analyzes exactly how much space is needed, why, and how to minimize it.

---

## Space Requirements

```
┌──────────────────────────────────────────────────┐
│           SEGMENT TREE SPACE USAGE               │
├──────────────────────────────────────────────────┤
│  Array representation: O(4n)                     │
│  Recursive stack:      O(log n)                  │
│  Total:                O(n)                      │
│                                                  │
│  In practice: allocate tree[4 * n + 5]           │
│  (the +5 is safety margin)                       │
└──────────────────────────────────────────────────┘
```

---

## Why 4n?

### Case 1: n is a power of 2
```
n = 8 (2³):

Level 0:  1 node      (root)
Level 1:  2 nodes
Level 2:  4 nodes
Level 3:  8 nodes     (leaves)
Total:    15 nodes = 2×8 - 1 = 2n - 1

Allocation: 2n is enough (1-indexed, so tree[1..15])
```

### Case 2: n is NOT a power of 2
```
n = 5:

         [0-4]
        /      \
     [0-2]    [3-4]
    /   \     /   \
  [0-1] [2] [3]  [4]
  / \
[0] [1]

Nodes: 9

But in 1-indexed array, the deepest leaf may have index
up to 2^(⌈log₂ n⌉ + 1).

For n = 5: ⌈log₂ 5⌉ = 3, so max index up to 2^4 = 16

We need tree[1..16], but only 9 positions are used.
```

### Worst case: n = 2^k + 1
```
n = 9 (2³ + 1):

Height = ⌈log₂ 9⌉ = 4
Max array index = 2^5 - 1 = 31

But 4 × 9 = 36 > 31, so 4n is always safe.

n = 2^k + 1:
  Height = k + 1
  Max index = 2^(k+2) - 1
  4n = 4(2^k + 1) = 2^(k+2) + 4 > 2^(k+2) - 1  ✓

So 4n always works regardless of n.
```

---

## Exact vs Safe Allocation

```
┌────────────┬─────────────────────┬──────────────────┐
│  n         │  Exact nodes needed │  4n allocation   │
├────────────┼─────────────────────┼──────────────────┤
│  1         │  1                  │  4               │
│  2         │  3                  │  8               │
│  4         │  7                  │  16              │
│  5         │  9                  │  20              │
│  7         │  13                 │  28              │
│  8         │  15                 │  32              │
│  9         │  17                 │  36              │
│ 16         │  31                 │  64              │
│ 100        │  ~199               │  400             │
│ 100,000    │  ~199,999           │  400,000         │
└────────────┴─────────────────────┴──────────────────┘

Wasted space: up to ~2n (about 50% overhead)
But O(n) space overall — acceptable.
```

---

## Memory Usage in Practice

```
For 32-bit integers (4 bytes each):

n = 100,000:
  Original array:   100,000 × 4 = 400 KB
  Segment tree:     400,000 × 4 = 1.6 MB
  Total:            ~2 MB

n = 1,000,000:
  Original array:   1,000,000 × 4 = 4 MB
  Segment tree:     4,000,000 × 4 = 16 MB
  Total:            ~20 MB

n = 10,000,000:
  Segment tree:     40,000,000 × 4 = 160 MB
  → May exceed memory limits in competitive programming!

Typical CP limit: 256 MB
  Max n for int seg tree: ~15,000,000
  Max n for long long:    ~7,500,000
```

---

## Recursive Stack Space

```
Each recursive call uses:
  - Function arguments: ~5 integers (node, start, end, L, R)
  - Return address
  - Local variable mid

Typical: ~40-60 bytes per stack frame

Stack depth: O(log n) = ~20 for n = 10^6

Total stack: 20 × 50 bytes = ~1 KB  (negligible)
```

---

## Space Optimization Techniques

### 1. Exact Size Calculation
```
Instead of 4n, compute exact size:

function treeSize(n):
    height = ceil(log2(n))
    return 2^(height + 1)  // or (1 << (height + 1))

// Saves up to 50% compared to 4n
// But 4n is simpler and universally safe
```

### 2. Iterative Segment Tree (Power-of-2 Padding)
```
Pad n to next power of 2, use exactly 2n space:

sz = 1
while sz < n: sz *= 2

tree = new array[2 * sz]  // exactly 2 × (next power of 2)

// For n = 100,000: sz = 131072, tree size = 262144
// vs 4n = 400,000 → saves ~35%
```

### 3. Dynamic Segment Tree
```
Only create nodes when needed (for very large ranges):

struct Node:
    value
    left_child_ptr
    right_child_ptr

// Instead of allocating 4n upfront,
// create nodes only when accessed.
// Useful when n can be up to 10^9 but only ~10^5 updates.
```

---

## Comparison with Other Structures

```
┌──────────────────┬──────────────────────────┐
│  Structure       │  Space Complexity         │
├──────────────────┼──────────────────────────┤
│  Original array  │  O(n)                    │
│  Prefix sum      │  O(n)                    │
│  BIT             │  O(n)     ← 1×n          │
│  Segment tree    │  O(4n)    ← 4×n          │
│  Sparse table    │  O(n log n) ← n×log(n)   │
│  Seg tree + lazy │  O(4n) + O(4n) = O(8n)   │
│  2D Segment tree │  O(16n²)                  │
│  Persistent ST   │  O(n + Q log n)           │
└──────────────────┴──────────────────────────┘

BIT uses 4× less memory than segment tree!
```

---

## When Space is a Concern

```
Decision factors:

1. n > 5×10^6 and using 64-bit values?
   → Consider BIT (uses n instead of 4n)
   
2. Range is huge (up to 10^9) but few updates?
   → Dynamic segment tree (pointer-based)
   
3. Need multiple versions of the tree?
   → Persistent segment tree (O(n + Q log n) total)
   
4. 2D problems?
   → Space is O(4n × 4m) — can be prohibitive
   → Consider 2D BIT instead
```

---

## Summary Table

| Aspect | Value |
|--------|-------|
| Tree array size | 4n (safe allocation) |
| Exact nodes (n = 2^k) | 2n - 1 |
| Worst-case ratio | ~4:1 (tree to array) |
| Stack space | O(log n) ≈ ~1 KB |
| Per-element overhead | 4 bytes × 4 = 16 bytes for int tree |
| Max n for 256 MB (int) | ~15 million |
| Max n for 256 MB (long) | ~7.5 million |
| BIT space comparison | BIT uses n vs 4n → 4× less |

---

## Quick Revision Questions

1. Why do we allocate 4n instead of 2n? *(When n isn't a power of 2, the 1-indexed tree array can have gaps; 4n guarantees enough space)*
2. How much memory does a segment tree use for n = 10⁶ with 32-bit integers? *(4 × 10⁶ × 4 bytes = 16 MB)*
3. How much space does the recursion stack use? *(O(log n) frames ≈ 1 KB — negligible)*
4. How does BIT compare to segment tree in space usage? *(BIT uses n elements vs 4n — about 4× less)*
5. When would you use a dynamic segment tree? *(When the range is huge like 0 to 10⁹ but only a few thousand updates/queries)*

---

[← Previous: Time Complexity](05-time-complexity.md) | [Next Unit: Range Operations →](../04-Range-Operations/01-range-sum-queries.md)
