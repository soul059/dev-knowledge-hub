# Chapter 1.5: Two's Complement

> **Unit 1 — Binary Number System**
> The universal method for representing negative numbers in modern computers.

---

## Overview

**Two's complement** is the standard way every modern CPU represents signed integers. It allows both positive and negative numbers to be stored and manipulated using the same addition/subtraction hardware. Mastering two's complement is essential for understanding how bit manipulation works with negative numbers.

---

## How to Compute Two's Complement

To find the two's complement (i.e., negate a number):

### Method 1: Invert and Add 1

```
  Step 1: Write the binary representation
  Step 2: Flip ALL bits (0→1, 1→0)  — this is the "one's complement"
  Step 3: Add 1

  ┌─────────────────────────────────────────────┐
  │  TWO'S COMPLEMENT = INVERT ALL BITS + 1     │
  └─────────────────────────────────────────────┘
```

### Example: Negate +5 (8-bit)

```
  +5  =  00000101

  Step 1: Start with           00000101
  Step 2: Invert all bits      11111010   (one's complement)
  Step 3: Add 1              + 00000001
                             ──────────
  -5  =                       11111011

  Verify:  00000101  (+5)
         + 11111011  (-5)
         ──────────
        1 00000000   (= 0, carry bit discarded) ✓
```

### Visual Step-by-Step

```
  Original:    0  0  0  0  0  1  0  1     (+5)
               ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓   FLIP
  Inverted:    1  1  1  1  1  0  1  0     (one's complement)
                                    + 1    ADD 1
               ─  ─  ─  ─  ─  ─  ─  ─
  Result:      1  1  1  1  1  0  1  1     (-5 in two's complement)
               ↑
            Sign bit = 1 (negative)
```

---

## Method 2: Shortcut — Find Rightmost 1, Flip Everything to Its Left

```
  +12 = 00001100
                ↑
         Rightmost 1 is here (position 2)
  
  Keep everything from position 2 rightward:  ...100
  Flip everything to the left:                11110...
  
  Result: 11110100 = -12
  
  ┌──────────────────────────────────────────────────┐
  │  Shortcut: Scan from RIGHT                       │
  │  - Keep all 0s and the first 1 unchanged         │
  │  - Flip all remaining bits to the left            │
  └──────────────────────────────────────────────────┘
```

### Trace: Negate +40 (8-bit)

```
  +40 = 00101000
             ↑
      Rightmost 1 at position 3
  
  Keep:  ...1000  (rightmost 1 and zeros below)
  Flip:  11011...  (everything left of first 1)
  
  Result: 11011000 = -40
  
  Verify: 00101000 + 11011000 = 1|00000000 ✓ (carry out)
```

---

## Reading Two's Complement Values

### Finding the Decimal Value of a Negative Binary Number

```
  Given: 11100110₂ (8-bit signed)

  Method A: MSB weight is NEGATIVE
  ─────────────────────────────────
  = -1×2⁷ + 1×2⁶ + 1×2⁵ + 0×2⁴ + 0×2³ + 1×2² + 1×2¹ + 0×2⁰
  = -128   + 64   + 32   + 0    + 0    + 4    + 2    + 0
  = -128 + 102
  = -26

  Method B: Negate first, then read magnitude
  ─────────────────────────────────────────────
  11100110  → invert → 00011001  → add 1 → 00011010
  00011010₂ = 16 + 8 + 2 = 26
  Original was negative, so value = -26 ✓
```

---

## Two's Complement Arithmetic

The beauty: **addition just works**, even with mixed signs!

### Example 1: 7 + (-3) = 4

```
    00000111     (+7)
  + 11111101     (-3 in two's complement)
  ──────────
  1 00000100     (= 4, carry discarded)
  
  Result: 00000100 = +4 ✓
```

### Example 2: (-5) + (-3) = -8

```
    11111011     (-5)
  + 11111101     (-3)
  ──────────
  1 11111000     (-8, carry discarded)
  
  Verify -8: 11111000 → invert → 00000111 → +1 → 00001000 = 8
  Sign bit = 1 → -8 ✓
```

### Example 3: 5 - 7 = -2 (Subtraction as Addition)

```
  5 - 7 = 5 + (-7)
  
    00000101     (+5)
  + 11111001     (-7)
  ──────────
    11111110     (-2)
    
  Verify: 11111110 → invert → 00000001 → +1 → 00000010 = 2
  Sign bit = 1 → -2 ✓
```

---

## Special Values in Two's Complement

```
  8-bit Two's Complement Special Values:
  
  ┌──────────────────┬──────────┬───────────────────────┐
  │ Pattern          │ Value    │ Note                  │
  ├──────────────────┼──────────┼───────────────────────┤
  │ 00000000         │    0     │ Zero                  │
  │ 00000001         │   +1     │ Positive one          │
  │ 01111111         │  +127    │ Maximum positive      │
  │ 10000000         │  -128    │ Minimum (most neg.)   │
  │ 11111111         │   -1     │ All ones = -1         │
  │ 11111110         │   -2     │                       │
  │ 10000001         │  -127    │                       │
  └──────────────────┴──────────┴───────────────────────┘

  KEY INSIGHT: -1 is always ALL ONES in two's complement
  
    8-bit:  11111111        = -1
   16-bit:  1111111111111111 = -1
   32-bit:  (all 32 ones)   = -1
```

---

## The Asymmetry: -128 Has No Positive Counterpart

```
  8-bit signed:  -128  to  +127
  
  Try to negate -128:
    10000000  → invert → 01111111 → +1 → 10000000
    
  Wait... that's -128 again!
  
  ┌─────────────────────────────────────────────┐
  │  -(-128) = -128  in 8-bit two's complement  │
  │                                             │
  │  This is because +128 cannot be represented │
  │  in 8 bits signed (max is +127).            │
  │                                             │
  │  This is a REAL BUG SOURCE in programs!     │
  │  abs(INT_MIN) causes overflow!              │
  └─────────────────────────────────────────────┘
```

---

## Sign Extension

When converting a signed number to a larger bit width, **copy the sign bit**:

```
  8-bit to 16-bit:
  
  +5:   00000101  →  00000000 00000101    (fill with 0s)
  -5:   11111011  →  11111111 11111011    (fill with 1s)
  
  ┌──────────────────────────────────────────────────┐
  │  Sign Extension: Copy the MSB to fill new bits   │
  │                                                  │
  │  Positive (MSB=0): pad with 0s on the left       │
  │  Negative (MSB=1): pad with 1s on the left       │
  └──────────────────────────────────────────────────┘
  
  This preserves the value when widening!
  
  Verify -5 in 16-bit:
  11111111 11111011 = -1×2¹⁵ + all the rest = -5 ✓
```

---

## Two's Complement in Programming

### Pseudocode: Negate a Number Using Bits

```
FUNCTION negate(n):
    RETURN ~n + 1      // bitwise NOT + 1

// Equivalent to:
FUNCTION negate(n):
    RETURN -n          // compiler does this automatically
```

### Pseudocode: Get Absolute Value

```
FUNCTION abs_value(n):
    IF n < 0:
        RETURN ~n + 1   // negate using two's complement
    RETURN n

// WARNING: abs(INT_MIN) overflows!
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Two's complement | Invert all bits + add 1 to negate |
| Shortcut | Keep rightmost 1 and trailing 0s, flip rest |
| -1 representation | All bits set to 1 |
| Addition | Same hardware circuit for both + and − |
| Sign extension | Copy sign bit when widening |
| MSB weight | In n-bit: MSB has weight −2ⁿ⁻¹ |
| Asymmetry | One extra negative value (e.g., -128 vs +127) |
| abs(MIN) overflow | Negating the minimum value causes overflow |

---

## Quick Revision Questions

1. **What is the two's complement of 01010110 (8-bit)?**
   <details><summary>Answer</summary>Invert: 10101001, Add 1: 10101010. So −86₁₀</details>

2. **Why is −1 always represented as all 1s?**
   <details><summary>Answer</summary>+1 = 00000001, invert → 11111110, add 1 → 11111111. All ones!</details>

3. **What is −1 + 1 in 8-bit two's complement?**
   <details><summary>Answer</summary>11111111 + 00000001 = 1|00000000. Carry discarded → 00000000 = 0 ✓</details>

4. **Can abs(−128) be stored in an 8-bit signed integer?**
   <details><summary>Answer</summary>No! +128 exceeds the 8-bit signed max of +127. This causes overflow.</details>

5. **How do you sign-extend -3 from 4-bit to 8-bit?**
   <details><summary>Answer</summary>-3 in 4-bit = 1101. Sign-extend: 11111101 (copy the 1 sign bit into all new positions)</details>

6. **Compute 6 − 10 using 8-bit two's complement addition.**
   <details><summary>Answer</summary>6 + (-10): 00000110 + 11110110 = 11111100. Negate: 00000100 = 4. Answer: −4 ✓</details>

---

[← Previous: Signed vs Unsigned](04-signed-vs-unsigned.md) | [Next: Bit Positions →](06-bit-positions.md)
