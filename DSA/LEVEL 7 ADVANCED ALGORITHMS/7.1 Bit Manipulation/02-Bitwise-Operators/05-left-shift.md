# Chapter 2.5: Left Shift (<<)

> **Unit 2 — Bitwise Operators**
> The left shift operator — multiplying by powers of 2 at hardware speed.

---

## Overview

The **left shift** operator (`<<`) moves all bits to the left by a specified number of positions. Bits shifted out from the left are **discarded**, and **zeros fill in** from the right. Left shifting by `k` positions is equivalent to **multiplying by 2ᵏ**.

---

## How Left Shift Works

```
  n << k  →  shift all bits of n to the LEFT by k positions

  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  Before (n = 13, 8-bit):   0  0  0  0  1  1  0  1       │
  │                                                          │
  │  n << 1:                                                 │
  │  ┌───────────────────────────────────────────────────┐   │
  │  │  0  0  0  0  1  1  0  1                           │   │
  │  │  ↙  ↙  ↙  ↙  ↙  ↙  ↙  ↙                         │   │
  │  │  0  0  0  1  1  0  1  0  ← 0 fills from right    │   │
  │  └───────────────────────────────────────────────────┘   │
  │  ↑ bit shifted out (discarded)                           │
  │                                                          │
  │  Result: 00011010 = 26  (13 × 2 = 26) ✓                 │
  │                                                          │
  │  n << 2:                                                 │
  │  0  0  0  0  1  1  0  1                                  │
  │     ↙ ↙  ↙  ↙  ↙  ↙  ↙  ↙                              │
  │  0  0  1  1  0  1  0  0  ← 00 fills from right          │
  │                                                          │
  │  Result: 00110100 = 52  (13 × 4 = 13 × 2² = 52) ✓       │
  └──────────────────────────────────────────────────────────┘
```

---

## Left Shift = Multiply by 2ᵏ

```
  ┌──────────────────────────────────────────────┐
  │  FUNDAMENTAL IDENTITY:                       │
  │                                              │
  │     n << k  =  n × 2ᵏ                       │
  │                                              │
  │  Examples (8-bit unsigned):                  │
  │                                              │
  │   3 << 0 =  3 × 1 =   3   (no shift)        │
  │   3 << 1 =  3 × 2 =   6                     │
  │   3 << 2 =  3 × 4 =  12                     │
  │   3 << 3 =  3 × 8 =  24                     │
  │   3 << 4 =  3 × 16 = 48                     │
  │   3 << 5 =  3 × 32 = 96                     │
  │   1 << k =  2ᵏ          (generating powers) │
  └──────────────────────────────────────────────┘
```

### Why Multiplication by 2?

```
  Decimal analogy — shifting LEFT in base 10:
  
  42 × 10  = 420    (append a 0 on the right)
  42 × 100 = 4200   (append two 0s)
  
  Binary — shifting LEFT in base 2:
  
  101 × 2  = 1010   (append a 0 on the right)
  101 × 4  = 10100  (append two 0s)
  
  Left shift APPENDS zeros → multiplies by base!
```

---

## Step-by-Step Trace: Creating Bit Masks

Left shift is how we create masks to target specific bit positions:

```
  1 << 0  =  00000001  =   1   (position 0 mask)
  1 << 1  =  00000010  =   2   (position 1 mask)
  1 << 2  =  00000100  =   4   (position 2 mask)
  1 << 3  =  00001000  =   8   (position 3 mask)
  1 << 4  =  00010000  =  16   (position 4 mask)
  1 << 5  =  00100000  =  32   (position 5 mask)
  1 << 6  =  01000000  =  64   (position 6 mask)  
  1 << 7  =  10000000  = 128   (position 7 mask)

  ┌──────────────────────────────────────┐
  │  1 << k  creates a number with       │
  │  ONLY bit k set to 1.               │
  │  This is the MASK for position k.    │
  └──────────────────────────────────────┘
```

---

## Overflow Behavior

When bits are shifted past the leftmost position, they are **lost**:

```
  8-bit unsigned:
  
  200 << 1 = ???
  
  200 = 11001000
        ↙↙↙↙↙↙↙↙
       10010000 ← leftmost 1 is LOST
       = 144  (NOT 400!)
  
  ┌──────────────────────────────────────────────┐
  │  WARNING: Left shift can OVERFLOW            │
  │                                              │
  │  n << k overflows when high bits are lost    │
  │  The mathematical result n × 2ᵏ only holds   │
  │  if the result fits in the bit width.        │
  └──────────────────────────────────────────────┘
```

### Overflow Detection

```
  Before shifting n << k, check:
  
  Safe if:  n < 2^(width - k)
  
  Example (32-bit):
  n << 3 is safe if n < 2^29 = 536,870,912
  
  Or check: (n << k) >> k == n  (if equal, no overflow occurred)
```

---

## Common Left Shift Patterns

### 1. Generate Power of 2

```
  1 << n  =  2ⁿ

  1 << 10 = 1024
  1 << 20 = 1,048,576  (1 MB)
  1 << 30 = 1,073,741,824  (1 GB)
```

### 2. Fast Multiply by Power of 2

```
  n * 8  =  n << 3    (8 = 2³)
  n * 16 =  n << 4    (16 = 2⁴)
  n * 64 =  n << 6    (64 = 2⁶)
  
  Much faster than multiplication on simple processors!
```

### 3. Multiply by Non-Power Using Shifts

```
  n × 10 = n × (8 + 2) = (n << 3) + (n << 1)
  n × 12 = n × (8 + 4) = (n << 3) + (n << 2)
  n × 7  = n × (8 - 1) = (n << 3) - n
  
  Compilers often use these optimizations!
```

### 4. Pack Values

```
  Pack two 4-bit values into one byte:
  
  high = 0xA  (1010)
  low  = 0x5  (0101)
  
  packed = (high << 4) | low
         = 10100000 | 00000101
         = 10100101 = 0xA5
```

---

## Left Shift in Algorithms

### Building a Number Bit by Bit

```
  Pseudocode: Build number from MSB to LSB

  FUNCTION buildFromBits(bits[]):
      result = 0
      FOR each bit b in bits (left to right):
          result = (result << 1) | b
      RETURN result
  
  Trace: bits = [1, 0, 1, 1]
  
  Step 1: result = (0 << 1) | 1 = 1       = 0001
  Step 2: result = (1 << 1) | 0 = 2       = 0010
  Step 3: result = (2 << 1) | 1 = 5       = 0101
  Step 4: result = (5 << 1) | 1 = 11      = 1011
  
  Result: 1011₂ = 11₁₀ ✓
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| n << k | O(1) | O(1) |

> Single CPU instruction — barrel shifter handles any shift amount in constant time.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Left shift | Moves all bits left, fills 0s from right |
| Multiplication | `n << k = n × 2ᵏ` |
| Mask creation | `1 << k` creates mask for position k |
| Overflow | High bits lost if shifted too far |
| Packing data | Shift values to their byte/nibble position |
| Zero fill | Right side always gets zeros |

---

## Quick Revision Questions

1. **What is 7 << 3?**
   <details><summary>Answer</summary>7 × 2³ = 7 × 8 = 56</details>

2. **How do you compute 2¹⁰ using left shift?**
   <details><summary>Answer</summary>`1 << 10` = 1024</details>

3. **What bits are filled in from the right during left shift?**
   <details><summary>Answer</summary>Always zeros</details>

4. **Can left shifting cause overflow?**
   <details><summary>Answer</summary>Yes. If high bits are shifted out of the integer width, they are lost and the value wraps/overflows.</details>

5. **Express n × 6 using only left shifts and addition.**
   <details><summary>Answer</summary>`n × 6 = n × (4 + 2) = (n << 2) + (n << 1)`</details>

6. **What does (1 << n) - 1 compute?**
   <details><summary>Answer</summary>A number with the lower n bits all set to 1. Example: (1 << 4) - 1 = 15 = 01111₂</details>

---

[← Previous: NOT Operation](04-not-operation.md) | [Next: Right Shift →](06-right-shift.md)
