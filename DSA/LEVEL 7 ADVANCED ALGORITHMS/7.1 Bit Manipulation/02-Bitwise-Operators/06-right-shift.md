# Chapter 2.6: Right Shift (Arithmetic & Logical)

> **Unit 2 — Bitwise Operators**
> The right shift operator — dividing by powers of 2, with two important flavors.

---

## Overview

The **right shift** operator (`>>`) moves all bits to the right by a specified number of positions. Bits shifted out from the right are **discarded**. The critical question is: **what fills in from the left?** This determines whether it's a **logical** or **arithmetic** right shift.

---

## Two Types of Right Shift

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  LOGICAL RIGHT SHIFT  (>>>)                              │
  │  ─────────────────────────                               │
  │  Fills with 0s from the left                             │
  │  Used for unsigned values                                │
  │  Available as >>> in Java, >> for unsigned in C           │
  │                                                          │
  │  ARITHMETIC RIGHT SHIFT  (>>)                            │
  │  ────────────────────────────                            │
  │  Fills with the SIGN BIT from the left                   │
  │  Used for signed values (preserves sign)                 │
  │  Default >> for signed types in C/C++/Java               │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## Logical Right Shift (>>>)

Always fills zeros from the left, regardless of sign:

```
  n >>> k  →  shift right by k, fill 0s from left

  Example: 200 >>> 2  (8-bit unsigned)
  
  200 = 1 1 0 0 1 0 0 0
                        ↘ ↘   discarded
  0 0 1 1 0 0 1 0 ← 00 fills from left
  
  Result: 00110010 = 50  (200 ÷ 4 = 50) ✓
  
  
  Negative example: -8 >>> 2  (8-bit signed)
  
  -8 = 1 1 1 1 1 0 0 0
                     ↘ ↘   discarded
  0 0 1 1 1 1 1 0 ← 00 fills from left (ignores sign!)
  
  Result: 00111110 = 62  (NOT -2, because sign bit cleared)
```

---

## Arithmetic Right Shift (>>)

Fills with copies of the **sign bit** (MSB), preserving the sign:

```
  n >> k  →  shift right by k, fill with SIGN BIT from left

  POSITIVE number (sign bit = 0):
  
  40 >> 2  (8-bit signed)
  
  40 = 0 0 1 0 1 0 0 0
       ↑ sign=0
  0 0 0 0 1 0 1 0 ← copies of sign bit (0) fill in
  
  Result: 00001010 = 10  (40 ÷ 4 = 10) ✓


  NEGATIVE number (sign bit = 1):
  
  -8 >> 2  (8-bit signed)
  
  -8 = 1 1 1 1 1 0 0 0
       ↑ sign=1
  1 1 1 1 1 1 1 0 ← copies of sign bit (1) fill in
  
  Result: 11111110 = -2  (rounds toward negative infinity)
  
  Note: -8 ÷ 4 = -2 ✓ (sign is preserved!)
```

---

## Visual Comparison

```
  Value: -24 = 11101000 (8-bit signed)

  ARITHMETIC RIGHT SHIFT (>> 2):
  ┌─────────────────────────────┐
  │  1  1  1  0  1  0  0  0    │  original (-24)
  │  ↓  ↓                      │  sign bit copies
  │  1  1  1  1  1  0  1  0    │  result = -6
  └─────────────────────────────┘
  Sign preserved! -24 ÷ 4 = -6 ✓

  LOGICAL RIGHT SHIFT (>>> 2):
  ┌─────────────────────────────┐
  │  1  1  1  0  1  0  0  0    │  original (-24)
  │  0  0                      │  zeros fill in
  │  0  0  1  1  1  0  1  0    │  result = 58
  └─────────────────────────────┘
  Sign LOST! Result is positive 58 (not -6)
```

---

## Right Shift = Division by 2ᵏ

```
  ┌──────────────────────────────────────────────────┐
  │  FUNDAMENTAL IDENTITY:                           │
  │                                                  │
  │  n >> k  =  ⌊n ÷ 2ᵏ⌋   (floor division)        │
  │                                                  │
  │  Examples:                                       │
  │                                                  │
  │   100 >> 1 = 50    (100 ÷ 2)                     │
  │   100 >> 2 = 25    (100 ÷ 4)                     │
  │   100 >> 3 = 12    (100 ÷ 8, truncated from 12.5)│
  │    15 >> 1 = 7     (15 ÷ 2, truncated from 7.5)  │
  │    15 >> 2 = 3     (15 ÷ 4, truncated from 3.75) │
  │     1 >> 1 = 0     (1 ÷ 2, truncated from 0.5)   │
  └──────────────────────────────────────────────────┘
  
  WARNING for negative numbers:
  -7 >> 1 = -4  (not -3!)
  
  Arithmetic shift rounds toward NEGATIVE INFINITY:
  ⌊-7/2⌋ = ⌊-3.5⌋ = -4  (not -3)
  
  Regular integer division rounds toward ZERO:
  -7 / 2 = -3  (truncation toward zero)
```

---

## Common Right Shift Patterns

### 1. Fast Division by Power of 2

```
  n / 2  →  n >> 1
  n / 4  →  n >> 2
  n / 8  →  n >> 3
  n / 16 →  n >> 4

  Much faster than division on simple processors!
```

### 2. Extract Bits from Specific Positions

```
  Get bits 4-7 of a number:
  
  (n >> 4) & 0x0F
  
  Step 1: >> 4  shifts bits 4-7 into positions 0-3
  Step 2: & 0x0F masks off everything above position 3
  
  Example: n = 0xAB = 10101011
  n >> 4   = 00001010
  & 0x0F   = 00001010 = 0x0A = 10 ✓
```

### 3. Check Individual Bit at Position k

```
  (n >> k) & 1  →  returns 0 or 1
  
  Example: Check bit 5 of n = 42 (101010₂)
  42 >> 5 = 000001
  & 1     = 1  → bit 5 IS set ✓
```

### 4. Count Bits by Shifting

```
  FUNCTION countBits(n):
      count = 0
      WHILE n > 0:
          count += n & 1    // check LSB
          n = n >> 1        // shift out the LSB
      RETURN count
  
  Trace: n = 13 (1101₂)
  
  Step  │   n    │ n & 1 │ count │ n >> 1
  ──────┼────────┼───────┼───────┼────────
    1   │ 1101   │   1   │   1   │  110
    2   │  110   │   0   │   1   │   11
    3   │   11   │   1   │   2   │    1
    4   │    1   │   1   │   3   │    0
  
  Result: 3 set bits ✓
```

---

## Language Differences

```
  ┌──────────────────────────────────────────────────────────┐
  │  LANGUAGE      │ >> BEHAVIOR          │ >>> AVAILABLE?   │
  ├────────────────┼──────────────────────┼──────────────────┤
  │ Java           │ Arithmetic (signed)  │ Yes (logical)    │
  │ C/C++          │ Implementation-      │ No (use unsigned │
  │                │ defined for signed   │ types instead)   │
  │ Python         │ Arithmetic           │ No (integers are │
  │                │                      │ arbitrary prec.) │
  │ JavaScript     │ Arithmetic (32-bit)  │ Yes (logical)    │
  │ C# / Go        │ Arithmetic for signed│ No explicit >>>  │
  │                │ Logical for unsigned │                  │
  └──────────────────────────────────────────────────────────┘
  
  In C/C++: Use unsigned types if you want logical right shift:
  unsigned int x = -8;
  x >> 2;  // logical shift (zeros fill), because x is unsigned
```

---

## Shift Amount Constraints

```
  ╔══════════════════════════════════════════════════════╗
  ║  IMPORTANT RULES FOR SHIFT AMOUNTS                  ║
  ╠══════════════════════════════════════════════════════╣
  ║                                                      ║
  ║  1. Shift by 0     → no change  (n >> 0 = n)        ║
  ║                                                      ║
  ║  2. Shift by width → UNDEFINED BEHAVIOR in C/C++    ║
  ║     (e.g., int x; x >> 32 is UB for 32-bit int)     ║
  ║                                                      ║
  ║  3. Shift by negative → UNDEFINED BEHAVIOR           ║
  ║                                                      ║
  ║  4. Valid range: 0 to (width - 1)                    ║
  ║     For 32-bit int: 0 to 31                          ║
  ║     For 64-bit long: 0 to 63                         ║
  ╚══════════════════════════════════════════════════════╝
```

---

## Complete Shift Operator Summary

```
  ┌─────────────┬──────────────┬──────────────┬──────────────┐
  │ Operator    │ Direction    │ Fill Bits    │ Equivalent   │
  ├─────────────┼──────────────┼──────────────┼──────────────┤
  │ n << k      │ Left         │ Zeros (right)│ n × 2ᵏ       │
  │ n >> k      │ Right        │ Sign bit     │ ⌊n ÷ 2ᵏ⌋     │
  │             │ (arithmetic) │ (left)       │ (signed)     │
  │ n >>> k     │ Right        │ Zeros (left) │ Unsigned     │
  │             │ (logical)    │              │ division     │
  └─────────────┴──────────────┴──────────────┴──────────────┘
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| n >> k | O(1) | O(1) |
| n >>> k | O(1) | O(1) |

> Hardware barrel shifter — constant time for any shift amount within valid range.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Arithmetic >> | Fills with sign bit (preserves sign) |
| Logical >>> | Always fills with zeros |
| Division | `n >> k = ⌊n/2ᵏ⌋` (floor division) |
| Negative rounding | >> rounds toward -∞, not toward 0 |
| Bit extraction | `(n >> k) & 1` gets bit at position k |
| Valid shift | 0 to (bit_width − 1) only |

---

## Quick Revision Questions

1. **What is 100 >> 3?**
   <details><summary>Answer</summary>⌊100/8⌋ = 12 (truncated from 12.5)</details>

2. **What fills from the left in arithmetic right shift of a negative number?**
   <details><summary>Answer</summary>1s (copies of the sign bit, which is 1 for negative numbers)</details>

3. **What is -16 >> 2 in arithmetic shift?**
   <details><summary>Answer</summary>-4. ⌊-16/4⌋ = -4. The sign is preserved.</details>

4. **What is the difference between >> and >>> in Java?**
   <details><summary>Answer</summary>>> is arithmetic (sign-preserving), >>> is logical (zero-filling). For negative numbers, they give different results.</details>

5. **Why is shifting by the bit width undefined behavior in C?**
   <details><summary>Answer</summary>Hardware behavior varies — some CPUs shift by (k mod width), others produce 0. The C standard leaves it undefined to allow optimization.</details>

6. **Express extracting byte 2 (bits 16-23) from a 32-bit integer n.**
   <details><summary>Answer</summary>`(n >> 16) & 0xFF` — shift bits 16-23 into positions 0-7, then mask</details>

---

[← Previous: Left Shift](05-left-shift.md) | [Next Unit: Basic Bit Techniques →](../03-Basic-Bit-Techniques/01-check-if-bit-is-set.md)
