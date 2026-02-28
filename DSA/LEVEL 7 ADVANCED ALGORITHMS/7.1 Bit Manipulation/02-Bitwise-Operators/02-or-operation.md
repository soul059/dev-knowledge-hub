# Chapter 2.2: OR Operation (|)

> **Unit 2 — Bitwise Operators**
> The OR operator — the inclusive combiner that merges bits from two sources.

---

## Overview

The **bitwise OR** operator compares each pair of corresponding bits. A result bit is **1 if at least one of the input bits is 1**. OR is primarily used for **setting bits** and **combining flags**.

---

## Truth Table

```
  ┌─────┬─────┬────────┐
  │  A  │  B  │ A OR B │
  ├─────┼─────┼────────┤
  │  0  │  0  │   0    │  ← Only case that gives 0
  │  0  │  1  │   1    │
  │  1  │  0  │   1    │
  │  1  │  1  │   1    │
  └─────┴─────┴────────┘

  INTUITION: OR is like "either one is enough"
  
  OR is the OPPOSITE pattern of AND:
  AND → all must be 1 for result to be 1
  OR  → any one being 1 is enough for result to be 1
```

### Logic Gate Diagram

```
       A ──────╮
               )≥1──── A | B
       B ──────╯

  OR gate lets signal through if ANY input is high.
```

---

## Bitwise OR on Multi-Bit Numbers

```
  Example:  12 | 10

      12 = 1 1 0 0
      10 = 1 0 1 0
           ───────
   12|10 = 1 1 1 0  = 14

  Trace each column:
  ┌──────────┬─────┬─────┬────────┐
  │ Position │ b₁  │ b₂  │ Result │
  ├──────────┼─────┼─────┼────────┤
  │    3     │  1  │  1  │   1    │  At least one 1
  │    2     │  1  │  0  │   1    │  At least one 1
  │    1     │  0  │  1  │   1    │  At least one 1
  │    0     │  0  │  0  │   0    │  Neither is 1
  └──────────┴─────┴─────┴────────┘
```

---

## OR for Setting Bits

The primary use of OR: **turning ON specific bits** without affecting others.

```
  SETTING CONCEPT:
  
  OR with 1 → Forces bit to 1 (SET)
  OR with 0 → Leaves bit unchanged (PASS-THROUGH)
  
  ┌─────────────────────────────────────────────────┐
  │  n    = 1 0 1 0 0 0 1 0     original (= 162)   │
  │  mask = 0 0 0 0 1 1 0 0     set bits 2 and 3   │
  │         ─────────────────                       │
  │  n|mask= 1 0 1 0 1 1 1 0   bits 2,3 now ON     │
  └─────────────────────────────────────────────────┘

  What happened:
  - Mask bits = 0: original bits pass through unchanged
  - Mask bits = 1: result bits forced to 1
```

### Visual: OR as a Setter

```
  Data bits:    1  0  1  0  0  0  1  0
                ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓
  Mask:         0  0  0  0  1  1  0  0
                ↓  ↓  ↓  ↓  ↑  ↑  ↓  ↓    ← ↑ means forced ON
  Result:       1  0  1  0  1  1  1  0

  The mask FORCES ON bits where mask = 1
  The mask PRESERVES bits where mask = 0
```

---

## Common OR Patterns

### 1. Set a Specific Bit

```
  n = n | (1 << pos)
  
  Example: Set bit 4 of n = 35
  
  35   = 00100011
  1<<4 = 00010000
         ────────
  OR   = 00110011 = 51
  
  Bit 4 is now ON. All other bits unchanged.
```

### 2. Combine Multiple Flags

```
  // Define flags
  READ    = 0b001  = 1
  WRITE   = 0b010  = 2
  EXECUTE = 0b100  = 4
  
  // Give read and execute permissions
  perms = READ | EXECUTE
        = 001 | 100
        = 101 = 5
  
  // Later: add write permission
  perms = perms | WRITE
        = 101 | 010
        = 111 = 7   (all permissions granted)
```

### 3. Set Lower n Bits to 1

```
  (1 << n) - 1
  
  Example: Set lower 5 bits
  (1 << 5) - 1 = 100000 - 1 = 011111 = 31
```

### 4. Ensure a Bit is Set (Idempotent)

```
  OR is safe to apply multiple times:
  
  n | (1 << 3) | (1 << 3)  = same as  n | (1 << 3)
  
  Setting an already-set bit does nothing.
  This makes OR IDEMPOTENT for setting.
```

---

## Step-by-Step Trace: Combining RGB into Packed Color

```
  Red   = 0x3F = 63    (0011 1111)
  Green = 0xA8 = 168   (1010 1000)
  Blue  = 0x4C = 76    (0100 1100)
  
  Pack into 24-bit color: 0xRRGGBB
  
  Step 1: Shift Red to bits 16-23
    0x3F << 16 = 0x3F0000
    = 00111111 00000000 00000000
  
  Step 2: Shift Green to bits 8-15
    0xA8 << 8  = 0x00A800
    = 00000000 10101000 00000000
  
  Step 3: Blue stays in bits 0-7
    0x4C       = 0x00004C
    = 00000000 00000000 01001100
  
  Step 4: OR them all together
    0x3F0000 | 0x00A800 | 0x00004C = 0x3FA84C
    
    00111111 00000000 00000000
  | 00000000 10101000 00000000
  | 00000000 00000000 01001100
  ─────────────────────────────
    00111111 10101000 01001100  = 0x3FA84C ✓
  
  Combined: color = (r << 16) | (g << 8) | b
```

---

## OR Properties

```
  ┌────────────────────────────────────────────────┐
  │  MATHEMATICAL PROPERTIES OF OR                │
  ├────────────────────────────────────────────────┤
  │                                                │
  │  Identity:     a | 0   = a    (OR with 0)      │
  │  Annihilation: a | ~0  = ~0   (OR with all 1s) │
  │  Idempotent:   a | a   = a    (OR with self)   │
  │  Commutative:  a | b   = b | a                │
  │  Associative:  (a|b)|c = a|(b|c)              │
  │  Absorption:   a | (a&b) = a                   │
  │                                                │
  │  De Morgan's: ~(a | b) = ~a & ~b              │
  │  Distributive: a|(b&c) = (a|b)&(a|c)          │
  └────────────────────────────────────────────────┘
```

---

## AND vs OR Comparison

```
  ┌─────────────────────────────────────────────────┐
  │             AND vs OR                           │
  ├───────────────────┬─────────────────────────────┤
  │       AND (&)     │         OR (|)              │
  ├───────────────────┼─────────────────────────────┤
  │ Requires BOTH = 1 │ Requires ANY = 1            │
  │ Used to CLEAR bits│ Used to SET bits             │
  │ Used to TEST bits │ Used to COMBINE flags        │
  │ Mask 1 = keep     │ Mask 0 = keep               │
  │ Mask 0 = clear    │ Mask 1 = force ON            │
  │ x & 0 = 0         │ x | 0 = x                   │
  │ x & ~0 = x        │ x | ~0 = ~0                  │
  └───────────────────┴─────────────────────────────┘
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| a \| b | O(1) | O(1) |

> Single CPU instruction — O(1) constant time and space.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| OR rule | 1 if at least one bit is 1 |
| Setting bits | OR with 1 forces bit ON |
| Preserving bits | OR with 0 leaves bit unchanged |
| Flag combining | `READ \| WRITE \| EXECUTE` |
| Idempotent | Setting twice = setting once |
| De Morgan's | `~(a \| b) = ~a & ~b` |

---

## Quick Revision Questions

1. **What is 0b11000011 | 0b00111100?**
   <details><summary>Answer</summary>0b11111111 = 255. All positions have at least one 1.</details>

2. **How do you set bit 7 in a variable x?**
   <details><summary>Answer</summary>`x = x | (1 << 7)` or `x |= 128`</details>

3. **What does `n | 0` always return?**
   <details><summary>Answer</summary>n — OR with zero is the identity operation</details>

4. **Pack R=200, G=100, B=50 into a 24-bit color.**
   <details><summary>Answer</summary>`(200 << 16) | (100 << 8) | 50 = 0xC86432`</details>

5. **What is the result of `n | n`?**
   <details><summary>Answer</summary>n (idempotent property)</details>

6. **State De Morgan's law for OR.**
   <details><summary>Answer</summary>`~(a | b) = ~a & ~b` — NOT of OR equals AND of NOTs</details>

---

[← Previous: AND Operation](01-and-operation.md) | [Next: XOR Operation →](03-xor-operation.md)
