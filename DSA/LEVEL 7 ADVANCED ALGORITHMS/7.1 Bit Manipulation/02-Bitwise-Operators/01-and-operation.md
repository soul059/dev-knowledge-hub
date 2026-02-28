# Chapter 2.1: AND Operation (&)

> **Unit 2 — Bitwise Operators**
> The AND operator — the fundamental gatekeeper that lets bits through only when both agree.

---

## Overview

The **bitwise AND** operator compares each pair of corresponding bits from two numbers. A result bit is **1 only if both input bits are 1**. AND is the most commonly used operator for **masking** — extracting or testing specific bits.

---

## Truth Table

```
  ┌─────┬─────┬─────────┐
  │  A  │  B  │ A AND B │
  ├─────┼─────┼─────────┤
  │  0  │  0  │    0    │
  │  0  │  1  │    0    │
  │  1  │  0  │    0    │
  │  1  │  1  │    1    │  ← Only case that gives 1
  └─────┴─────┴─────────┘

  INTUITION: AND is like a "both must agree" gate
  
  Think of it as multiplication: 0×0=0, 0×1=0, 1×0=0, 1×1=1
```

### Logic Gate Diagram

```
       A ──────┐
               │ AND ├──── A & B
       B ──────┘
  
  Symbol:
       A ───╮
             )───── Output
       B ───╯
```

---

## Bitwise AND on Multi-Bit Numbers

AND operates on **each bit position independently**:

```
  Example:  13 & 10

      13 = 1 1 0 1
      10 = 1 0 1 0
           ───────
   13&10 = 1 0 0 0  = 8

  Trace each column:
  ┌──────────┬─────┬─────┬────────┐
  │ Position │ b₁  │ b₂  │ Result │
  ├──────────┼─────┼─────┼────────┤
  │    3     │  1  │  1  │   1    │  Both 1 → 1
  │    2     │  1  │  0  │   0    │  Not both → 0
  │    1     │  0  │  1  │   0    │  Not both → 0
  │    0     │  1  │  0  │   0    │  Not both → 0
  └──────────┴─────┴─────┴────────┘
```

---

## AND as a Bit Mask

The primary use of AND: **extracting specific bits** while clearing others.

```
  MASKING CONCEPT:
  
  A mask has 1s where you want to KEEP bits
              0s where you want to CLEAR bits
  
  ┌─────────────────────────────────────────────────┐
  │  n    = 1 0 1 1 0 1 1 0     original number    │
  │  mask = 0 0 0 0 1 1 1 1     keep lower 4 bits  │
  │         ─────────────────                       │
  │  n&mask= 0 0 0 0 0 1 1 0   lower nibble only   │
  └─────────────────────────────────────────────────┘

  What happened:
  - Upper 4 bits: AND with 0 → cleared to 0
  - Lower 4 bits: AND with 1 → preserved as-is
```

### Visual: AND as a Filter

```
  Data bits:    1  0  1  1  0  1  1  0
                ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓
  Mask:         0  0  0  0  1  1  1  1
                ×  ×  ×  ×  ↓  ↓  ↓  ↓    ← × means blocked
  Result:       0  0  0  0  0  1  1  0
  
  The mask BLOCKS (zeros out) bits where mask = 0
  The mask PASSES (keeps) bits where mask = 1
```

---

## Common AND Patterns

### 1. Check if a Number is Even or Odd

```
  n & 1  →  Extracts the last bit (LSB)
  
  If LSB = 0 → Even
  If LSB = 1 → Odd
  
  Example:
     7 = 0111  →  7 & 1 = 0111 & 0001 = 1  → Odd ✓
    12 = 1100  → 12 & 1 = 1100 & 0001 = 0  → Even ✓
```

### 2. Check if Bit at Position k is Set

```
  (n >> k) & 1   OR   n & (1 << k)
  
  Example: Is bit 2 of 13 set?
  13 = 1101
  (13 >> 2) & 1 = (0011) & 1 = 1  → Yes, bit 2 is set ✓
```

### 3. Clear Lower n Bits

```
  x & (~0 << n)
  
  Example: Clear lower 3 bits of 29
  29 = 11101
  ~0 << 3 = ...11111000
  29 & (~0 << 3) = 11101 & 11000 = 11000 = 24
```

### 4. Extract Lower n Bits

```
  x & ((1 << n) - 1)
  
  Example: Extract lower 4 bits of 0xAB
  0xAB = 10101011
  (1 << 4) - 1 = 00001111
  0xAB & 0x0F = 00001011 = 0x0B = 11
```

---

## Step-by-Step Trace: Practical Example

### Problem: Extract the green channel from RGB color

```
  RGB color = 0x3FA84C  (R=0x3F, G=0xA8, B=0x4C)
  
  Binary: 00111111  10101000  01001100
          ────────  ────────  ────────
             Red     Green      Blue

  To extract Green (bits 8-15):
  
  Step 1: Shift right by 8
    0x3FA84C >> 8 = 0x003FA8
    = 00000000 00111111 10101000
  
  Step 2: AND with 0xFF (mask lower 8 bits)
    0x003FA8 & 0xFF = 0xA8
    = 10101000
  
  Green = 0xA8 = 168 ✓
  
  Combined: green = (color >> 8) & 0xFF
```

---

## AND Properties

```
  ┌────────────────────────────────────────────────┐
  │  MATHEMATICAL PROPERTIES OF AND               │
  ├────────────────────────────────────────────────┤
  │                                                │
  │  Identity:     a & ~0  = a    (AND with all 1s)│
  │  Annihilation: a & 0   = 0    (AND with all 0s)│
  │  Idempotent:   a & a   = a    (AND with self)  │
  │  Commutative:  a & b   = b & a                │
  │  Associative:  (a&b)&c = a&(b&c)              │
  │  Absorption:   a & (a|b) = a                   │
  │                                                │
  │  KEY: n & (n-1)  turns off the RIGHTMOST       │
  │       set bit (used in Brian Kernighan's alg)  │
  └────────────────────────────────────────────────┘
```

---

## Complexity Analysis

| Operation | Time | Space | Notes |
|-----------|------|-------|-------|
| a & b | O(1) | O(1) | Single CPU instruction |
| AND on n-bit numbers | O(1) | O(1) | Hardware parallelism |

> AND is a **single CPU cycle** operation — it doesn't get faster than this!

---

## Real-World Applications

1. **Bit masking** — Extract specific bits from data
2. **Permission checking** — `if (perms & READ_FLAG)` checks read access
3. **IP subnet masking** — `IP & subnet_mask` gives network address
4. **Color channel extraction** — Extract R, G, B from packed color
5. **Alignment checking** — `if (addr & 0x3 == 0)` checks 4-byte alignment

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| AND rule | 1 only when BOTH bits are 1 |
| Masking | AND with 1 preserves, AND with 0 clears |
| Even/odd check | `n & 1` returns 0 (even) or 1 (odd) |
| Extract bits | AND with mask of 1s in desired positions |
| Clear bits | AND with mask of 0s in positions to clear |
| n & (n-1) | Turns off rightmost set bit |

---

## Quick Revision Questions

1. **What is 0b11001100 & 0b10101010?**
   <details><summary>Answer</summary>0b10001000 = 136. Only positions where both have 1: positions 3 and 7.</details>

2. **How do you check if bit 5 is set in variable x?**
   <details><summary>Answer</summary>`(x >> 5) & 1` or `x & (1 << 5)` — if non-zero, bit 5 is set</details>

3. **What does `n & 0` always return?**
   <details><summary>Answer</summary>0 — AND with all zeros clears everything (annihilation property)</details>

4. **Extract the lowest byte from 0x12345678.**
   <details><summary>Answer</summary>0x12345678 & 0xFF = 0x78 = 120</details>

5. **What is the result of `n & n`?**
   <details><summary>Answer</summary>n (idempotent property — AND with itself returns itself)</details>

6. **Why is AND used for subnet masking in networking?**
   <details><summary>Answer</summary>AND with the subnet mask zeros out the host portion, leaving only the network address. It "filters" the network bits.</details>

---

[← Previous Unit: Bit Positions](../01-Binary-Number-System/06-bit-positions.md) | [Next: OR Operation →](02-or-operation.md)
