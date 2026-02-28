# Chapter 2.3: XOR Operation (^)

> **Unit 2 — Bitwise Operators**
> The XOR operator — the magician of bit manipulation that can detect differences, toggle bits, and solve elegant puzzles.

---

## Overview

The **bitwise XOR** (exclusive OR) operator returns **1 when the two input bits are different**, and **0 when they are the same**. XOR is arguably the most versatile and "magical" bitwise operator, powering everything from swapping variables to cryptography.

---

## Truth Table

```
  ┌─────┬─────┬─────────┐
  │  A  │  B  │ A XOR B │
  ├─────┼─────┼─────────┤
  │  0  │  0  │    0    │  Same → 0
  │  0  │  1  │    1    │  Different → 1
  │  1  │  0  │    1    │  Different → 1
  │  1  │  1  │    0    │  Same → 0
  └─────┴─────┴─────────┘

  INTUITION: XOR is a "difference detector"
  
  Returns 1 WHERE the two numbers DIFFER
  Returns 0 WHERE the two numbers are the SAME
```

### Logic Gate Diagram

```
       A ──────╮
               )=1──── A ^ B
       B ──────╯

  XOR gate: output is high when inputs are DIFFERENT.
```

---

## Bitwise XOR on Multi-Bit Numbers

```
  Example:  13 ^ 10

      13 = 1 1 0 1
      10 = 1 0 1 0
           ───────
   13^10 = 0 1 1 1  = 7

  Trace each column:
  ┌──────────┬─────┬─────┬────────┬──────────┐
  │ Position │ b₁  │ b₂  │ Result │ Reason   │
  ├──────────┼─────┼─────┼────────┼──────────┤
  │    3     │  1  │  1  │   0    │ Same     │
  │    2     │  1  │  0  │   1    │ Different│
  │    1     │  0  │  1  │   1    │ Different│
  │    0     │  1  │  0  │   1    │ Different│
  └──────────┴─────┴─────┴────────┴──────────┘
```

---

## XOR's Magical Properties

These properties make XOR the star of competitive programming:

```
  ╔════════════════════════════════════════════════════════╗
  ║  FUNDAMENTAL PROPERTIES OF XOR                        ║
  ╠════════════════════════════════════════════════════════╣
  ║                                                        ║
  ║  1. SELF-INVERSE:     a ^ a = 0                        ║
  ║     Any number XOR itself = 0                          ║
  ║                                                        ║
  ║  2. IDENTITY:         a ^ 0 = a                        ║
  ║     XOR with 0 changes nothing                         ║
  ║                                                        ║
  ║  3. COMMUTATIVE:      a ^ b = b ^ a                    ║
  ║     Order doesn't matter                               ║
  ║                                                        ║
  ║  4. ASSOCIATIVE:      (a^b)^c = a^(b^c)                ║
  ║     Grouping doesn't matter                            ║
  ║                                                        ║
  ║  5. TOGGLING:         a ^ 1 = ~a  (for single bit)     ║
  ║     XOR with 1 flips the bit                           ║
  ║                                                        ║
  ║  6. CANCELLATION:     a ^ b ^ b = a                    ║
  ║     XOR twice cancels out (from 1 and 4)               ║
  ╚════════════════════════════════════════════════════════╝
```

### Visualizing Self-Inverse

```
  a     = 1 0 1 1 0 1 0 0
  a     = 1 0 1 1 0 1 0 0
          ─────────────────
  a ^ a = 0 0 0 0 0 0 0 0  = 0

  Every bit matches itself → all differences = 0  → result = 0
```

### Visualizing Cancellation

```
  a     = 1 0 1 1 0 1 0 0    (original)
  b     = 1 1 0 0 1 0 1 1    (XOR with b)
          ─────────────────
  a ^ b = 0 1 1 1 1 1 1 1    (scrambled)
  
  Now XOR with b again:
  a^b   = 0 1 1 1 1 1 1 1
  b     = 1 1 0 0 1 0 1 1
          ─────────────────
  a^b^b = 1 0 1 1 0 1 0 0  = a !    (restored!)
  
  XOR is its own inverse — this enables encryption & swapping!
```

---

## XOR for Toggling Bits

```
  XOR with 1 → FLIPS the bit
  XOR with 0 → KEEPS the bit unchanged
  
  ┌───────────────────────────────────────────────────┐
  │  n    = 1 0 1 1 0 1 1 0     original              │
  │  mask = 0 0 0 0 1 1 1 1     toggle lower 4 bits   │
  │         ─────────────────                          │
  │  n^mask= 1 0 1 1 1 0 0 1   lower 4 toggled!       │
  │                                                    │
  │  Upper 4: XOR 0 → unchanged  (1011 stays 1011)    │
  │  Lower 4: XOR 1 → flipped   (0110 becomes 1001)   │
  └───────────────────────────────────────────────────┘
```

---

## Classic XOR Applications

### 1. Swap Two Variables Without Temp

```
  a = 5, b = 3

  Step 1: a = a ^ b    →  a = 5^3 = 6,  b = 3
  Step 2: b = a ^ b    →  a = 6,  b = 6^3 = 5
  Step 3: a = a ^ b    →  a = 6^5 = 3,  b = 5

  Result: a = 3, b = 5  (swapped!) ✓
  
  WHY IT WORKS:
  a = a^b
  b = (a^b)^b = a^(b^b) = a^0 = a     ← b gets original a
  a = (a^b)^a = (a^a)^b = 0^b = b     ← a gets original b
```

### 2. Find the Single Non-Duplicate

```
  Array: [2, 3, 5, 3, 2]
  
  XOR all elements:
  2 ^ 3 ^ 5 ^ 3 ^ 2
  = (2^2) ^ (3^3) ^ 5    (rearrange by associativity)
  = 0 ^ 0 ^ 5
  = 5  ← The single number! ✓
  
  Duplicates cancel, singleton survives.
```

### 3. Detect Changed Bits

```
  old = 10110100
  new = 10010110
  
  old ^ new = 00100010
                ↑    ↑
           Bits 1 and 5 changed
  
  XOR shows EXACTLY which bits are different.
```

---

## XOR vs AND vs OR Comparison

```
  ┌─────┬─────┬───────┬──────┬───────┐
  │  A  │  B  │ A & B │ A|B  │ A ^ B │
  ├─────┼─────┼───────┼──────┼───────┤
  │  0  │  0  │   0   │  0   │   0   │
  │  0  │  1  │   0   │  1   │   1   │
  │  1  │  0  │   0   │  1   │   1   │
  │  1  │  1  │   1   │  1   │   0   │
  └─────┴─────┴───────┴──────┴───────┘
  
  AND = both must be 1
  OR  = at least one must be 1
  XOR = exactly one must be 1 (they must differ)
```

---

## XOR Identity Proofs

```
  Proof: a ^ b ^ b = a
  
  Step 1: By associativity, a ^ (b ^ b)
  Step 2: By self-inverse,   b ^ b = 0
  Step 3: By identity,       a ^ 0 = a ∎
  
  ─────────────────────────────────────
  
  Proof: a ^ b = 0  iff  a = b
  
  If a = b:  a ^ a = 0  (self-inverse) ✓
  If a ≠ b:  at least one bit differs → at least one 1 in result → ≠ 0 ✓  ∎
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| a ^ b | O(1) | O(1) |
| XOR of n elements | O(n) | O(1) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| XOR rule | 1 when bits are different, 0 when same |
| Self-inverse | `a ^ a = 0` |
| Identity | `a ^ 0 = a` |
| Cancellation | `a ^ b ^ b = a` |
| Toggling | XOR with 1 flips, XOR with 0 preserves |
| Swap trick | `a^=b; b^=a; a^=b` swaps without temp |
| Find unique | XOR all elements; duplicates cancel |

---

## Quick Revision Questions

1. **What is 0b10110 ^ 0b01101?**
   <details><summary>Answer</summary>0b11011 = 27. XOR gives 1 where bits differ.</details>

2. **If a ^ b = 15, and a = 9, what is b?**
   <details><summary>Answer</summary>b = a ^ 15 = 9 ^ 15 = 0b1001 ^ 0b1111 = 0b0110 = 6</details>

3. **What happens when you XOR a number with itself?**
   <details><summary>Answer</summary>You get 0. Every bit matches, so all differences are 0.</details>

4. **Why can XOR find a single non-duplicate in an array?**
   <details><summary>Answer</summary>Because duplicates XOR to 0 (a^a=0), and 0^unique = unique. Only the non-duplicate survives.</details>

5. **How does XOR toggle bit 3 of a number n?**
   <details><summary>Answer</summary>`n ^ (1 << 3)` — XOR with 1 flips that bit, XOR with 0 keeps all others unchanged</details>

6. **Is XOR commutative and associative?**
   <details><summary>Answer</summary>Yes! `a^b = b^a` (commutative) and `(a^b)^c = a^(b^c)` (associative). Order and grouping don't matter.</details>

---

[← Previous: OR Operation](02-or-operation.md) | [Next: NOT Operation →](04-not-operation.md)
