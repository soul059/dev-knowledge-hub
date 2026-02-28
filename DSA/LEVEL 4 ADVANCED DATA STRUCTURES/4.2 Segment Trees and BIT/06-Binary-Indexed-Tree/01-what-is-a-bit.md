# What is a BIT?

## Chapter Overview

A Binary Indexed Tree (BIT), also called a **Fenwick Tree**, is a data structure that efficiently supports prefix sum queries and point updates. It's simpler, faster, and uses less memory than a segment tree — but only works for invertible operations.

---

## The Core Idea

```
BIT exploits BINARY REPRESENTATION of indices to create
a tree-like structure using a flat array.

Each index i is "responsible" for a range of elements
determined by the LOWEST SET BIT of i.

Key function: lowbit(i) = i & (-i)
  Returns the value of the lowest set bit.

Examples:
  i = 12 = 1100₂  → lowbit(12) = 4  (100₂)
  i = 10 = 1010₂  → lowbit(10) = 2  (10₂)
  i =  6 = 0110₂  → lowbit(6)  = 2  (10₂)
  i =  8 = 1000₂  → lowbit(8)  = 8  (1000₂)
  i =  7 = 0111₂  → lowbit(7)  = 1  (1₂)
```

---

## BIT Array Structure

```
BIT uses 1-indexed array B[1..n]

B[i] stores the SUM of elements from index (i - lowbit(i) + 1) to i

Index  Binary  lowbit  Range covered     B[i] stores
  1    0001      1     [1, 1]            A[1]
  2    0010      2     [1, 2]            A[1] + A[2]
  3    0011      1     [3, 3]            A[3]
  4    0100      4     [1, 4]            A[1] + A[2] + A[3] + A[4]
  5    0101      1     [5, 5]            A[5]
  6    0110      2     [5, 6]            A[5] + A[6]
  7    0111      1     [7, 7]            A[7]
  8    1000      8     [1, 8]            A[1]+...+A[8]

Pattern: B[i] covers the last lowbit(i) elements ending at i
```

---

## Visualizing the BIT

```
Index:  1    2    3    4    5    6    7    8

B[8]: ├────────────────────────────────────┤  covers 8 elements
B[4]: ├──────────────────┤                     covers 4 elements
B[6]:                          ├─────────┤     covers 2 elements
B[2]: ├─────────┤                              covers 2 elements
B[1]: ├────┤                                   covers 1 element
B[3]:                ├────┤                    covers 1 element
B[5]:                          ├────┤          covers 1 element
B[7]:                                    ├──┤  covers 1 element

Tree view (parent = i + lowbit(i)):

            B[8]
           / |  \  \
        B[4] B[6] B[7]
        /|    |
     B[2] B[3] B[5]
      |
     B[1]

Parent of i = i + lowbit(i):
  parent(1) = 1 + 1 = 2
  parent(2) = 2 + 2 = 4
  parent(3) = 3 + 1 = 4
  parent(5) = 5 + 1 = 6
  parent(6) = 6 + 2 = 8
  parent(7) = 7 + 1 = 8
```

---

## BIT vs Segment Tree

```
┌────────────────────┬─────────────┬───────────────┐
│    Feature         │     BIT     │ Segment Tree  │
├────────────────────┼─────────────┼───────────────┤
│ Point update       │  O(log n)   │  O(log n)     │
│ Prefix query       │  O(log n)   │  O(log n)     │
│ Range query        │  O(log n)   │  O(log n)     │
│ Range update       │  O(log n)*  │  O(log n)     │
│ Code complexity    │  ~10 lines  │  ~50 lines    │
│ Constant factor    │  Very small │  Larger       │
│ Memory             │  n+1        │  4n           │
│ Operations         │  Invertible │  Any assoc.   │
│                    │  only       │  operation    │
│ Built-in lazy      │  No         │  Yes          │
│ 2D extension       │  Easy       │  Complex      │
└────────────────────┴─────────────┴───────────────┘

* Range update with BIT uses a clever trick (next unit)

RULE OF THUMB:
  If the operation is invertible (sum, xor) → prefer BIT
  If not (min, max, gcd) → must use segment tree
```

---

## Why "Binary Indexed"?

```
The name comes from how BINARY REPRESENTATION determines structure:

Index 6 = 110₂:
  lowbit = 10₂ = 2
  B[6] covers indices [5, 6] (2 elements)

To compute prefix_sum(6):
  Start at 6 = 110₂
  Add B[6]  (covers [5,6])
  Remove lowbit: 110 - 10 = 100 = 4
  Add B[4]  (covers [1,4])
  Remove lowbit: 100 - 100 = 000 = 0
  STOP.
  
  prefix_sum(6) = B[6] + B[4]
                = (A[5]+A[6]) + (A[1]+A[2]+A[3]+A[4])
                = A[1]+A[2]+A[3]+A[4]+A[5]+A[6]  ✓

The index DECREASES by removing lowest set bits:
  6 → 4 → 0  (only 2 steps for prefix sum of 6 elements!)
  
  In general: at most log₂(n) steps  → O(log n)
```

---

## Why BIT is Faster in Practice

```
BIT wins over segment tree in practice for several reasons:

1. FEWER OPERATIONS per query/update
   BIT: 1 loop, 1 addition, 1 bitwise operation per step
   SegTree: function calls, branching, two children

2. CACHE FRIENDLY
   BIT: sequential array access with good locality
   SegTree: tree traversal with pointer chasing

3. SMALLER MEMORY
   BIT: exactly n+1 integers
   SegTree: 4n integers (+ 4n for lazy)

4. SIMPLE CODE
   BIT: 5-10 lines total
   SegTree: 50+ lines with helper functions

Typical speedup: BIT is 2-5× faster than segment tree for sum queries.
```

---

## Limitations

```
BIT CANNOT efficiently handle:
  ✗ Range minimum/maximum queries (not invertible)
  ✗ Range GCD queries (not invertible)
  ✗ Arbitrary range queries without prefix decomposition
  ✗ Lazy propagation (needs segment tree)

BIT CAN handle:
  ✓ Prefix sums
  ✓ Point updates
  ✓ Range sum queries (via prefix difference)
  ✓ Range XOR queries (XOR is its own inverse)
  ✓ Range updates with point queries (via difference array)
  ✓ 2D prefix sums with updates
  ✓ Counting inversions
  ✓ Order statistics (with coordinate compression)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Other name | Fenwick Tree |
| Key function | lowbit(i) = i & (-i) |
| Array type | 1-indexed, size n+1 |
| B[i] covers | Elements from (i - lowbit(i) + 1) to i |
| Query mechanism | Remove lowest set bits: i -= lowbit(i) |
| Update mechanism | Add lowest set bit: i += lowbit(i) |
| Operations supported | Invertible operations (sum, XOR) |
| Memory | O(n) — much less than segment tree |
| Practical speed | 2-5× faster than segment tree |

---

## Quick Revision Questions

1. What is another name for BIT? *(Fenwick Tree)*
2. What does lowbit(i) compute? *(The value of the lowest set bit: i & (-i))*
3. How many elements does B[i] cover? *(lowbit(i) elements, from i-lowbit(i)+1 to i)*
4. Why can't BIT handle range minimum queries? *(Minimum is not invertible — can't compute range min from prefix mins)*
5. How much memory does a BIT use vs a segment tree? *(BIT: n+1, Segment tree: 4n — roughly 4× less)*
6. What types of operations work with BIT? *(Invertible operations like sum and XOR)*

---

[← Previous Unit: Complete Lazy Implementation](../05-Lazy-Propagation/06-complete-lazy-implementation.md) | [Next: Binary Representation →](02-binary-representation.md)
