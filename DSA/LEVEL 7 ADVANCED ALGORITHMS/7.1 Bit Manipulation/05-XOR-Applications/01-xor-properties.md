# Chapter 5.1: XOR Properties Deep Dive

> **Unit 5 — XOR Applications**
> Master the algebraic properties of XOR that make it the most powerful single bitwise operator in competitive programming.

---

## Overview

XOR (exclusive OR) has a unique combination of mathematical properties that no other bitwise operator shares. Understanding these properties is the key to solving an entire class of problems elegantly.

---

## The Four Fundamental Properties

### 1. Self-Inverse: a ^ a = 0

```
  Any number XOR-ed with itself is 0.

    0101 1010
  ^ 0101 1010
  ──────────
    0000 0000

  Same bits always produce 0. Opposite → never happens (same number!)
```

### 2. Identity: a ^ 0 = a

```
  Any number XOR-ed with 0 is unchanged.

    0101 1010
  ^ 0000 0000
  ──────────
    0101 1010

  XOR with 0 preserves all bits.
```

### 3. Commutativity: a ^ b = b ^ a

```
  Order doesn't matter:
  5 ^ 3 = 3 ^ 5 = 6

    0101 ^ 0011 = 0110
    0011 ^ 0101 = 0110   ✓
```

### 4. Associativity: (a ^ b) ^ c = a ^ (b ^ c)

```
  Grouping doesn't matter:
  (5 ^ 3) ^ 7 = 5 ^ (3 ^ 7) = 1

    (0101 ^ 0011) ^ 0111 = 0110 ^ 0111 = 0001
    0101 ^ (0011 ^ 0111) = 0101 ^ 0100 = 0001   ✓
```

---

## Derived Properties

### Cancellation: a ^ b ^ a = b

```
  ┌──────────────────────────────────────────────────┐
  │  a ^ b ^ a                                      │
  │  = (a ^ a) ^ b     (commutativity + associativity)│
  │  = 0 ^ b            (self-inverse)               │
  │  = b                 (identity)                   │
  └──────────────────────────────────────────────────┘

  This is WHY XOR swap works, WHY find-unique works,
  and WHY XOR is used in cryptography.
```

### No Information Loss

```
  Given a ^ b = c:
    c ^ a = b    (can recover b)
    c ^ b = a    (can recover a)

  XOR is an INVERTIBLE operation — unlike AND or OR!

  Analogy: 
    AND loses info:  1 & 0 = 0  ← was the other bit 0 or 1?  
    OR  loses info:  1 | 1 = 1  ← were both 1 or just one?
    XOR keeps info:  1 ^ 0 = 1  ← XOR with 1 again → 0 (recovered!)
```

### XOR with All 1s = NOT

```
  a ^ 1111 = ~a

    0101 1010
  ^ 1111 1111
  ──────────
    1010 0101   = bitwise NOT

  Selective negation: XOR with a mask flips only masked bits.
```

---

## XOR Truth Table Patterns

```
  a  b  │ a^b │ Interpretation
  ──────┼─────┼──────────────────
  0  0  │  0  │ Same → 0
  0  1  │  1  │ Different → 1
  1  0  │  1  │ Different → 1
  1  1  │  0  │ Same → 0

  XOR answers: "Are these two bits DIFFERENT?"
```

---

## Property Comparison Table

```
  Property        │  AND  │  OR   │  XOR
  ────────────────┼───────┼───────┼──────
  Commutative     │  Yes  │  Yes  │  Yes
  Associative     │  Yes  │  Yes  │  Yes
  Self-inverse    │  No   │  No   │  YES! ★
  Identity elem   │  1    │  0    │  0
  Annihilator     │  0    │  1    │  None
  Invertible      │  No   │  No   │  YES! ★
```

> XOR's self-inverse and invertibility make it uniquely powerful.

---

## Applications Preview

| Property | Application |
|----------|-------------|
| Self-inverse | Find unique element in array |
| Cancellation | Swap without temp variable |
| Invertibility | Simple encryption (XOR cipher) |
| Difference detection | Compare two values bit-by-bit |
| Commutativity | Order-independent accumulation |
| a ^ 1s = NOT | Selective bit flipping |

---

## Summary Table

| Property | Formula | Key Use |
|----------|---------|---------|
| Self-inverse | a ^ a = 0 | Cancel pairs |
| Identity | a ^ 0 = a | Neutral element |
| Commutative | a ^ b = b ^ a | Reorder freely |
| Associative | (a^b)^c = a^(b^c) | Regroup freely |
| Cancellation | a^b^a = b | Recover values |
| XOR with 1s | a ^ ~0 = ~a | Flip all bits |

---

## Quick Revision Questions

1. **What is 42 ^ 42?**
   <details><summary>Answer</summary>0. Self-inverse property: any number XOR-ed with itself is 0.</details>

2. **If a ^ b = 15, what is a ^ b ^ b?**
   <details><summary>Answer</summary>a ^ b ^ b = a ^ (b^b) = a ^ 0 = a. The answer is a. (Also: 15 ^ b = a)</details>

3. **Why is XOR invertible but AND is not?**
   <details><summary>Answer</summary>Given c = a & b, you can't recover a from c and b alone. E.g., if c=0 and b=0, a could be anything. But c = a ^ b → a = c ^ b always works.</details>

4. **What is 0xFF ^ 0xFF ^ 0xFF?**
   <details><summary>Answer</summary>First pair: 0xFF ^ 0xFF = 0x00. Then 0x00 ^ 0xFF = 0xFF. Odd count → value survives.</details>

5. **How can XOR be used for simple encryption?**
   <details><summary>Answer</summary>plaintext ^ key = ciphertext, then ciphertext ^ key = plaintext. The cancellation property provides symmetric encryption/decryption.</details>

---

[← Previous: Turn Off Rightmost Set Bit](../04-Bit-Manipulation-Tricks/06-turn-off-rightmost-set-bit.md) | [Next: Single Number →](02-single-number.md)
