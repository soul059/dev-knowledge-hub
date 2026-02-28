# Chapter 1.6: Bit Positions

> **Unit 1 — Binary Number System**
> Understanding which bit is where — the foundation for all bit manipulation techniques.

---

## Overview

Every bit in an integer has a **position** (also called index). Understanding bit positions is crucial because all bit manipulation operations — set, clear, toggle, check — target specific positions. This chapter establishes the **mental model** you'll use throughout the rest of the course.

---

## Bit Position Numbering

Bits are numbered **from right to left**, starting at **position 0** (the LSB):

```
  32-bit integer example (value 42 = 00000000 00000000 00000000 00101010):

  Position: 31 30 29 28 27 26 25 24  23 22 21 20 19 18 17 16
  Bit:       0  0  0  0  0  0  0  0   0  0  0  0  0  0  0  0

  Position: 15 14 13 12 11 10  9  8   7  6  5  4  3  2  1  0
  Bit:       0  0  0  0  0  0  0  0   0  0  1  0  1  0  1  0
                                             ↑     ↑     ↑
                                          pos 5  pos 3  pos 1

  Bit positions with value 1:  {1, 3, 5}
  Value = 2¹ + 2³ + 2⁵ = 2 + 8 + 32 = 42
```

---

## Bit Position Anatomy

```
  ┌──────────────────────────────────────────────────────┐
  │                  8-BIT INTEGER                       │
  │                                                      │
  │   ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐ │
  │   │ b₇  │ b₆  │ b₅  │ b₄  │ b₃  │ b₂  │ b₁  │ b₀ │ │
  │   └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘ │
  │      │     │     │     │     │     │     │     │     │
  │    MSB   Higher           Middle           Lower  LSB│
  │   (Most                                   (Least    │
  │  Signif.)                                Signif.)    │
  │                                                      │
  │   Weight: 2⁷   2⁶   2⁵   2⁴   2³   2²   2¹   2⁰   │
  │          =128  =64  =32  =16   =8   =4   =2   =1   │
  └──────────────────────────────────────────────────────┘
  
  KEY RELATIONSHIPS:
  ┌─────────────────────────────────────────┐
  │ Position i  →  Weight = 2ⁱ             │
  │ Position i  →  Mask  = 1 << i          │
  │ Position 0  →  LSB (rightmost)         │
  │ Position n-1→  MSB (leftmost, n-bit)   │
  └─────────────────────────────────────────┘
```

---

## The Bit Mask: Targeting a Position

A **bitmask** isolates a specific position. To create a mask for position `i`:

```
  mask = 1 << i    (left-shift 1 by i positions)

  Position 0:  1 << 0  =  00000001  (targets bit 0)
  Position 1:  1 << 1  =  00000010  (targets bit 1)
  Position 2:  1 << 2  =  00000100  (targets bit 2)
  Position 3:  1 << 3  =  00001000  (targets bit 3)
  Position 4:  1 << 4  =  00010000  (targets bit 4)
  Position 5:  1 << 5  =  00100000  (targets bit 5)
  Position 6:  1 << 6  =  01000000  (targets bit 6)
  Position 7:  1 << 7  =  10000000  (targets bit 7)

  ┌────────────────────────────────────────────────┐
  │  Each mask has EXACTLY ONE bit set — the bit   │
  │  at the position you want to manipulate.       │
  └────────────────────────────────────────────────┘
```

---

## Operations at a Specific Position

All bit operations use the mask `(1 << pos)`:

```
  ┌──────────────────────────────────────────────────────────┐
  │  OPERATION          │  FORMULA            │  EXAMPLE     │
  │                     │                     │  (n=42,pos=3)│
  ├─────────────────────┼─────────────────────┼──────────────┤
  │  Check if bit set   │  (n >> pos) & 1     │  1 (set)     │
  │  Set bit            │  n | (1 << pos)     │  42 (same)   │
  │  Clear bit          │  n & ~(1 << pos)    │  34          │
  │  Toggle bit         │  n ^ (1 << pos)     │  34          │
  │  Get bit value      │  (n & (1<<pos))!=0  │  true        │
  └──────────────────────────────────────────────────────────┘
  
  42 = 00101010
              ↑ pos 3 is 1

  Clear pos 3:  00101010 & 11110111 = 00100010 = 34
  Toggle pos 3: 00101010 ^ 00001000 = 00100010 = 34
```

---

## Finding Bit Positions

### Position of the Rightmost Set Bit

```
  n = 40 = 00101000
                ↑
           Position 3

  Isolate rightmost 1:  n & (-n) = 00101000 & 11011000 = 00001000
  
  Then position = log₂(00001000) = log₂(8) = 3
  
  OR count trailing zeros: __builtin_ctz(40) = 3  (in C/C++)
```

### Position of the Leftmost (Highest) Set Bit

```
  n = 40 = 00101000
           ↑
       Position 5

  Position = ⌊log₂(40)⌋ = ⌊5.32⌋ = 5
  
  OR count leading zeros:  31 - __builtin_clz(40) = 31 - 26 = 5
```

---

## Multi-Bit Masks: Targeting Ranges

### Create mask for bits i through j (inclusive)

```
  Mask for bits 2 to 5:
  
  Method: ((1 << (j - i + 1)) - 1) << i
  
  j=5, i=2:
  (1 << 4) - 1 = 00010000 - 1 = 00001111
  00001111 << 2 = 00111100
  
  Result:  00111100
               ↑↑↑↑
           Bits 5432 are set

  Alternative: Create all 1s, then clear unwanted bits
  ~0           = 11111111
  ~0 << (j+1)  = 11000000   (clear bits 0 to j)
  ~0 << i      = 11111100   (clear bits 0 to i-1)
  XOR them: 11000000 ^ 11111100 = 00111100  ✓
```

---

## Byte and Nibble Positions

```
  32-bit integer:
  
  ┌────────────┬────────────┬────────────┬────────────┐
  │  Byte 3    │  Byte 2    │  Byte 1    │  Byte 0    │
  │  (bits     │  (bits     │  (bits     │  (bits     │
  │  31-24)    │  23-16)    │  15-8)     │  7-0)      │
  └────────────┴────────────┴────────────┴────────────┘
  
  Extract Byte k:  (n >> (k * 8)) & 0xFF
  
  Example: n = 0x1A2B3C4D
  Byte 0: (n >> 0)  & 0xFF = 0x4D = 77
  Byte 1: (n >> 8)  & 0xFF = 0x3C = 60
  Byte 2: (n >> 16) & 0xFF = 0x2B = 43
  Byte 3: (n >> 24) & 0xFF = 0x1A = 26
  
  ┌────────┬────────┬────────┬────────┐
  │  0x1A  │  0x2B  │  0x3C  │  0x4D  │
  │   26   │   43   │   60   │   77   │
  └────────┴────────┴────────┴────────┘
    Byte 3   Byte 2   Byte 1   Byte 0
```

---

## Common Bit Position Patterns

```
  Pattern        │ Binary        │ Hex    │ Use
  ───────────────┼───────────────┼────────┼─────────────────
  All zeros      │ 00000000      │ 0x00   │ Clear/Initialize
  All ones       │ 11111111      │ 0xFF   │ Mask all bits
  Alternating 01 │ 01010101      │ 0x55   │ Even positions
  Alternating 10 │ 10101010      │ 0xAA   │ Odd positions
  Low nibble     │ 00001111      │ 0x0F   │ Extract low 4
  High nibble    │ 11110000      │ 0xF0   │ Extract high 4
  Single bit     │ 00010000      │ 0x10   │ Test/Set one bit
  Low byte       │ 000000FF      │ 0xFF   │ Extract byte
```

---

## Real-World Applications of Bit Positions

```
  ╔══════════════════════════════════════════════════════╗
  ║  1. FLAGS & PERMISSIONS                              ║
  ║     Bit 0 = Read, Bit 1 = Write, Bit 2 = Execute    ║
  ║     rwx = 111₂ = 7                                  ║
  ║                                                      ║
  ║  2. ERROR CODES                                      ║
  ║     Each bit position = different error type          ║
  ║     Multiple errors stored in one integer             ║
  ║                                                      ║
  ║  3. HARDWARE REGISTERS                               ║
  ║     Specific bits control specific hardware features  ║
  ║                                                      ║
  ║  4. CHESS PROGRAMMING                                ║
  ║     64-bit integer = 64 squares on a board           ║
  ║     Bit 0 = a1, Bit 63 = h8                         ║
  ╚══════════════════════════════════════════════════════╝
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Position numbering | Right to left, starting at 0 |
| LSB | Position 0 (rightmost bit) |
| MSB | Position n−1 (leftmost bit, in n-bit number) |
| Bit mask | `1 << pos` creates a mask for position `pos` |
| Weight | Bit at position i contributes 2ⁱ to the value |
| Byte extraction | `(n >> (k*8)) & 0xFF` gets byte k |
| Range mask | `((1 << len) - 1) << start` targets a range |

---

## Quick Revision Questions

1. **What is the weight of bit position 10?**
   <details><summary>Answer</summary>2¹⁰ = 1024</details>

2. **Create a mask to isolate bit 5.**
   <details><summary>Answer</summary>1 << 5 = 00100000 = 32 = 0x20</details>

3. **In the number 0xAB, which bit positions are set?**
   <details><summary>Answer</summary>0xAB = 10101011₂. Positions: 0, 1, 3, 5, 7</details>

4. **How do you extract bits 4–7 from a number n?**
   <details><summary>Answer</summary>(n >> 4) & 0x0F — shift right by 4, then mask the lower 4 bits</details>

5. **What is the MSB position in a 32-bit integer?**
   <details><summary>Answer</summary>Position 31 (numbering starts at 0)</details>

6. **Create a mask with bits 2, 3, and 4 set.**
   <details><summary>Answer</summary>(1<<2) | (1<<3) | (1<<4) = 4+8+16 = 28 = 00011100₂</details>

---

[← Previous: Two's Complement](05-twos-complement.md) | [Next Unit: Bitwise Operators →](../02-Bitwise-Operators/01-and-operation.md)
