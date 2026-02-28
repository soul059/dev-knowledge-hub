# Chapter 6.1: Generate All Subsets Using Bitmasks

> **Unit 6 — Subsets with Bits**
> Use binary numbers as masks to enumerate every subset of an n-element set — the foundation of bitmask algorithms.

---

## Overview

A set with n elements has 2^n subsets. Each subset can be represented by an n-bit binary number (a **bitmask**), where bit i is 1 if element i is included and 0 if excluded.

---

## The Bitmask–Subset Mapping

```
  Set = {A, B, C}     (n = 3, indices: A=0, B=1, C=2)

  Mask  │ Binary │ Subset
  ──────┼────────┼────────────
    0   │  000   │  {}
    1   │  001   │  {A}
    2   │  010   │  {B}
    3   │  011   │  {A, B}
    4   │  100   │  {C}
    5   │  101   │  {A, C}
    6   │  110   │  {B, C}
    7   │  111   │  {A, B, C}

  Total: 2^3 = 8 subsets  ✓
```

---

## Visual Concept

```
  mask = 5 = 101 (binary)

  Bit position:    2    1    0
  Element:         C    B    A
  Bit value:       1    0    1
                   ↓         ↓
  Included:        C         A
  
  Subset = {A, C}
```

---

## Pseudocode

```
FUNCTION generateSubsets(set[], n):
    FOR mask = 0 TO (1 << n) - 1:
        subset = []
        FOR i = 0 TO n - 1:
            IF mask & (1 << i) != 0:
                subset.ADD(set[i])
        PRINT subset
```

---

## Step-by-Step Trace: set = [X, Y, Z]

```
  mask = 0 (000): no bits set                    → {}
  mask = 1 (001): bit 0 set                      → {X}
  mask = 2 (010): bit 1 set                      → {Y}
  mask = 3 (011): bits 0,1 set                   → {X, Y}
  mask = 4 (100): bit 2 set                      → {Z}
  mask = 5 (101): bits 0,2 set                   → {X, Z}
  mask = 6 (110): bits 1,2 set                   → {Y, Z}
  mask = 7 (111): bits 0,1,2 set                 → {X, Y, Z}
```

---

## Why Bitmasks Are Powerful

```
  ┌──────────────────────────────────────────────────────────┐
  │  Subset operations become BITWISE operations:            │
  │                                                          │
  │  Union:        A | B         (elements in A or B)        │
  │  Intersection: A & B         (elements in both)          │
  │  Difference:   A & ~B        (elements in A not in B)    │
  │  Complement:   ~A & FULL     (elements not in A)         │
  │  Is subset:    (A & B) == A  (all of A is in B)          │
  │  Is empty:     A == 0                                    │
  │  Size:         popcount(A)   (number of set bits)        │
  │  Add element:  A | (1 << i)                              │
  │  Remove:       A & ~(1 << i)                             │
  │  Toggle:       A ^ (1 << i)                              │
  │  Contains:     A & (1 << i) != 0                         │
  └──────────────────────────────────────────────────────────┘
```

---

## Complexity

| Aspect | Value |
|--------|-------|
| Number of subsets | 2^n |
| Time per subset | O(n) to extract elements |
| Total time | O(n × 2^n) |
| Space | O(n) for current subset |

> Practical for n ≤ 20 (2^20 ≈ 1 million). Beyond ~25, it's too slow.

---

## Ordering of Subsets

```
  The mask iteration 0, 1, 2, ..., 2^n-1 produces subsets
  in BINARY COUNTING ORDER, not by size.

  To enumerate by size:
  FOR size = 0 TO n:
      FOR mask = 0 TO (1<<n)-1:
          IF popcount(mask) == size:
              process(mask)

  Or use Gosper's hack to enumerate masks with exactly k bits.
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Bitmask representation | Bit i = 1 means element i is included |
| Total subsets | 2^n for n elements |
| Enumeration | Loop mask from 0 to 2^n - 1 |
| Subset operations | Map to AND, OR, XOR, NOT |
| Practical limit | n ≤ 20–25 |

---

## Quick Revision Questions

1. **How many subsets does a set of 5 elements have?**
   <details><summary>Answer</summary>2^5 = 32 subsets.</details>

2. **What subset does mask = 13 (1101) represent for set {a,b,c,d}?**
   <details><summary>Answer</summary>Bits 0,2,3 are set → {a, c, d}.</details>

3. **How do you compute the union of subsets represented by masks 5 and 6?**
   <details><summary>Answer</summary>5 | 6 = 101 | 110 = 111 = 7. Union = all elements.</details>

4. **Is mask 3 (011) a subset of mask 7 (111)?**
   <details><summary>Answer</summary>Check: 3 & 7 = 011 & 111 = 011 = 3. Since (A & B) == A, yes, 3 is a subset of 7.</details>

5. **Why can't we use bitmasks for n = 30?**
   <details><summary>Answer</summary>2^30 ≈ 10^9 subsets. Even O(1) per subset takes ~1 second. With O(n) per subset it's ~30 seconds — too slow for competitive programming limits.</details>

---

[← Previous: XOR of Range](../05-XOR-Applications/06-xor-of-range.md) | [Next: Count Subsets with Given Property →](02-count-subsets.md)
