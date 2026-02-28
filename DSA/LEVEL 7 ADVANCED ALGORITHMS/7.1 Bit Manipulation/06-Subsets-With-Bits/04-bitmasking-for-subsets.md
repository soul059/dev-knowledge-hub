# Chapter 6.4: Bitmasking for Subsets — Operations & Idioms

> **Unit 6 — Subsets with Bits**
> A reference guide to all the subset operations you can perform with bitmask arithmetic.

---

## Overview

When a subset is stored as a bitmask (an integer), standard set operations become single bitwise instructions. This chapter catalogs every operation you'll need.

---

## Core Operations

### Element Operations

```
  Operation               │ Expression            │ Example (mask=0b1010, i=0)
  ────────────────────────┼───────────────────────┼───────────────────────────
  Add element i           │ mask | (1 << i)       │ 1010 | 0001 = 1011
  Remove element i        │ mask & ~(1 << i)      │ 1010 & 1110 = 1010
  Toggle element i        │ mask ^ (1 << i)       │ 1010 ^ 0001 = 1011
  Contains element i?     │ mask & (1 << i) != 0  │ 1010 & 0001 = 0 → No
  Contains element 1?     │ mask & (1 << 1) != 0  │ 1010 & 0010 = 0010 → Yes
```

### Set Operations

```
  Operation               │ Expression            │ Example
  ────────────────────────┼───────────────────────┼────────────────────
  Union A ∪ B             │ A | B                 │ 1010 | 0110 = 1110
  Intersection A ∩ B      │ A & B                 │ 1010 & 0110 = 0010
  Difference A \ B        │ A & ~B                │ 1010 & ~0110 = 1000
  Symmetric diff A △ B    │ A ^ B                 │ 1010 ^ 0110 = 1100
  Complement              │ ~A & ((1<<n)-1)       │ ~1010 & 1111 = 0101
```

### Subset Relations

```
  Operation               │ Expression            │ Meaning
  ────────────────────────┼───────────────────────┼────────────────────
  A ⊆ B ?                 │ (A & B) == A          │ A is subset of B
  A ⊇ B ?                 │ (A & B) == B          │ A is superset of B
  A ∩ B = ∅ ?             │ (A & B) == 0          │ Disjoint
  A == B ?                │ A == B                │ Equal sets
  A ≠ ∅ ?                 │ A != 0                │ Non-empty
```

### Size & Extremes

```
  Operation               │ Expression            │ Notes
  ────────────────────────┼───────────────────────┼────────────────────
  Size (cardinality)      │ popcount(mask)        │ __builtin_popcount
  Is empty?               │ mask == 0             │
  Full set (all n bits)   │ (1 << n) - 1          │
  Is singleton?           │ mask & (mask-1) == 0  │ Exactly 1 element
                          │ && mask != 0          │
  Lowest element          │ __builtin_ctz(mask)   │ Count trailing zeros
  Highest element         │ 31 - __builtin_clz(mask)│ Count leading zeros
  Isolate lowest element  │ mask & (-mask)        │ Lowest set bit
  Remove lowest element   │ mask & (mask - 1)     │ Kernighan's trick
```

---

## Iterating Over Set Bits

```
FUNCTION forEachElement(mask):
    WHILE mask != 0:
        i = lowestBit(mask)          // __builtin_ctz(mask)
        process(i)
        mask = mask & (mask - 1)     // remove lowest set bit
```

### Trace: mask = 13 (1101)

```
  Iteration 1: mask=1101, lowest=0, process(0), mask=1100
  Iteration 2: mask=1100, lowest=2, process(2), mask=1000
  Iteration 3: mask=1000, lowest=3, process(3), mask=0000
  Done. Processed: {0, 2, 3}
```

> This runs in O(k) where k = number of set bits, not O(n).

---

## Common Patterns

### Pattern 1: Process All Pairs from a Subset

```
FOR mask = 0 TO (1<<n)-1:
    temp = mask
    WHILE temp:
        i = ctz(temp)
        temp2 = temp & (temp - 1)    // remove i
        WHILE temp2:
            j = ctz(temp2)
            process(i, j)
            temp2 = temp2 & (temp2 - 1)
        temp = temp & (temp - 1)
```

### Pattern 2: Check if Subset Forms Valid Configuration

```
// Precompute: conflict[i] = bitmask of elements incompatible with i
FUNCTION isValid(mask):
    temp = mask
    WHILE temp:
        i = ctz(temp)
        IF mask & conflict[i]:     // any conflicting element is in mask?
            RETURN FALSE
        temp = temp & (temp - 1)
    RETURN TRUE
```

---

## Complexity of Operations

| Operation | Time |
|-----------|------|
| Any single bitwise op | O(1) |
| popcount | O(1) with hardware support |
| Iterate set bits | O(k), k = set bits |
| Check subset relation | O(1) |

---

## Summary Table

| Category | Key Operations |
|----------|---------------|
| Element | Add `\|`, Remove `& ~`, Toggle `^`, Check `&` |
| Set | Union `\|`, Intersection `&`, Difference `& ~`, Sym.Diff `^` |
| Query | Subset `(A&B)==A`, Disjoint `(A&B)==0`, Size `popcount` |
| Traverse | Lowest bit `& -mask`, Remove lowest `& (mask-1)` |

---

## Quick Revision Questions

1. **How do you compute A \ B (elements in A but not B)?**
   <details><summary>Answer</summary>A & ~B. This clears all bits in A that are set in B.</details>

2. **Given mask = 0b11010, what is mask & (mask - 1)?**
   <details><summary>Answer</summary>11010 & 11001 = 11000. The lowest set bit (bit 1) was removed.</details>

3. **How do you check if mask has exactly 2 elements?**
   <details><summary>Answer</summary>popcount(mask) == 2. Or: mask != 0 && (mask & (mask-1)) != 0 && ((mask & (mask-1)) & ((mask & (mask-1))-1)) == 0.</details>

4. **What is the complement of mask = 0b0110 with n = 4?**
   <details><summary>Answer</summary>~0110 & 1111 = 1001 & 1111 = 1001. Elements {0, 3}.</details>

5. **How do you iterate set bits in reverse (highest first)?**
   <details><summary>Answer</summary>Use the highest set bit: `i = 31 - clz(mask)`, then clear it: `mask ^= (1 << i)`. Or flip the traversal order.</details>

---

[← Previous: Subset Sum](03-subset-sum.md) | [Next: DP with Bitmasks Preview →](05-dp-with-bitmasks-preview.md)
