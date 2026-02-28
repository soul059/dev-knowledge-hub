# Chapter 1.4: Signed vs Unsigned Integers

> **Unit 1 — Binary Number System**
> Understanding how computers represent both positive and negative numbers.

---

## Overview

Computers store numbers as sequences of bits. But a fundamental question arises: **how do we represent negative numbers** if we only have 0s and 1s? The answer lies in the distinction between **signed** and **unsigned** integer representations.

---

## Unsigned Integers

All bits contribute to the magnitude. Only non-negative values (≥ 0).

### Range Formula
$$\text{Range} = 0 \text{ to } 2^n - 1$$

### 8-bit Unsigned Range

```
  ┌──────────────────────────────────────────────────┐
  │              8-BIT UNSIGNED                      │
  │                                                  │
  │   00000000 = 0     (minimum)                     │
  │   00000001 = 1                                   │
  │   01111111 = 127                                 │
  │   10000000 = 128                                 │
  │   11111110 = 254                                 │
  │   11111111 = 255   (maximum = 2⁸ - 1)            │
  │                                                  │
  │   ALL 256 values represent non-negative numbers  │
  └──────────────────────────────────────────────────┘
  
  Number Line:
  0                              128                            255
  ├────────────────────────────────┼────────────────────────────────┤
  00000000                    10000000                       11111111
```

---

## Signed Integers (Two's Complement)

The **most significant bit (MSB)** serves as the **sign bit**:
- **0** → positive (or zero)
- **1** → negative

### Range Formula
$$\text{Range} = -2^{n-1} \text{ to } +2^{n-1} - 1$$

### 8-bit Signed (Two's Complement) Range

```
  ┌──────────────────────────────────────────────────┐
  │          8-BIT SIGNED (Two's Complement)         │
  │                                                  │
  │   01111111 = +127   (maximum positive)           │
  │   01111110 = +126                                │
  │   00000001 = +1                                  │
  │   00000000 =  0                                  │
  │   11111111 = -1     (all ones = -1!)             │
  │   11111110 = -2                                  │
  │   10000001 = -127                                │
  │   10000000 = -128   (minimum negative)           │
  │                                                  │
  │   Sign Bit: MSB = 0 → positive, MSB = 1 → neg   │
  └──────────────────────────────────────────────────┘

  Number Line:
  -128            -1  0  1              127
  ├────────────────┼──┼──┼────────────────┤
  10000000    11111111  00000001     01111111
```

---

## Visual Comparison: Signed vs Unsigned (4-bit)

```
  Binary │ Unsigned │  Signed
  ───────┼──────────┼─────────
   0000  │    0     │    0
   0001  │    1     │   +1
   0010  │    2     │   +2
   0011  │    3     │   +3
   0100  │    4     │   +4
   0101  │    5     │   +5
   0110  │    6     │   +6
   0111  │    7     │   +7      ← max positive
   ------│----------│---------   SPLIT POINT
   1000  │    8     │   -8      ← min negative  
   1001  │    9     │   -7
   1010  │   10     │   -6
   1011  │   11     │   -5
   1100  │   12     │   -4
   1101  │   13     │   -3
   1110  │   14     │   -2
   1111  │   15     │   -1
```

### The "Circular" Nature of Signed Numbers

```
         Unsigned 4-bit circle          Signed 4-bit circle
         
              0 (0000)                       0 (0000)
           15/    \1                      -1/    \1
         14 |      | 2                  -2 |      | 2  
        13  |      | 3                -3   |      | 3
        12  |      | 4               -4    |      | 4
         11 |      | 5                -5   |      | 5
           10\    /6                    -6\    /6
              9  7                       -7  7
               8                          -8
```

---

## Why Two's Complement Won

There were other methods to represent signed numbers, but two's complement **won** because:

### Comparison of Methods

```
  ┌──────────────────┬───────────┬────────────────────────────┐
  │ Method           │ Zeros     │ Problem                    │
  ├──────────────────┼───────────┼────────────────────────────┤
  │ Sign-Magnitude   │ Two (+0,  │ Hardware needs separate    │
  │ (MSB = sign)     │  -0)      │ adder/subtractor circuits  │
  ├──────────────────┼───────────┼────────────────────────────┤
  │ One's Complement │ Two (+0,  │ Need end-around carry      │
  │ (flip all bits)  │  -0)      │ for addition               │
  ├──────────────────┼───────────┼────────────────────────────┤
  │ Two's Complement │ ONE zero  │ ✓ No problems!             │
  │ (flip + add 1)   │  only     │ ✓ Same adder for +/-       │
  └──────────────────┴───────────┴────────────────────────────┘
```

### Two's Complement Advantages

```
  ╔══════════════════════════════════════════════════════╗
  ║  WHY TWO'S COMPLEMENT IS SUPERIOR                   ║
  ╠══════════════════════════════════════════════════════╣
  ║                                                      ║
  ║  1. SINGLE ZERO                                      ║
  ║     Only one representation of 0 (all zeros)         ║
  ║                                                      ║
  ║  2. SIMPLE ADDITION                                  ║
  ║     Same hardware circuit adds both positive          ║
  ║     and negative numbers — no special logic!          ║
  ║                                                      ║
  ║  3. EXTRA NEGATIVE VALUE                              ║
  ║     Can represent one more negative value             ║
  ║     (e.g., -128 to +127 in 8 bits)                   ║
  ║                                                      ║
  ║  4. HARDWARE EFFICIENT                                ║
  ║     Subtraction = Addition of two's complement        ║
  ╚══════════════════════════════════════════════════════╝
```

---

## Common Integer Sizes in Programming

```
  ┌──────────────┬───────┬──────────────────────────────────────┐
  │ Type         │ Bits  │ Range                                │
  ├──────────────┼───────┼──────────────────────────────────────┤
  │ int8 / byte  │   8   │ Signed: -128 to 127                 │
  │ uint8        │   8   │ Unsigned: 0 to 255                  │
  ├──────────────┼───────┼──────────────────────────────────────┤
  │ int16/short  │  16   │ Signed: -32,768 to 32,767           │
  │ uint16       │  16   │ Unsigned: 0 to 65,535               │
  ├──────────────┼───────┼──────────────────────────────────────┤
  │ int32 / int  │  32   │ Signed: -2.1B to 2.1B               │
  │ uint32       │  32   │ Unsigned: 0 to 4.3B                 │
  ├──────────────┼───────┼──────────────────────────────────────┤
  │ int64 / long │  64   │ Signed: -9.2×10¹⁸ to 9.2×10¹⁸      │
  │ uint64       │  64   │ Unsigned: 0 to 1.8×10¹⁹             │
  └──────────────┴───────┴──────────────────────────────────────┘
```

---

## Overflow: The Dangerous Edge Case

### Unsigned Overflow

```
  8-bit unsigned:
  
  255 + 1 = ???
  
    11111111     (255)
  +        1     (1)
  ──────────
  1 00000000     (256... but only 8 bits!)
  ↑
  Carry bit is LOST  →  Result = 00000000 = 0
  
  255 + 1 = 0  (wraps around!)
```

### Signed Overflow

```
  8-bit signed:
  
  127 + 1 = ???
  
    01111111     (127, maximum positive)
  +        1     (1)
  ──────────
    10000000     (-128 in two's complement!)
  
  127 + 1 = -128  (wraps to most negative!)
```

```
  ┌────────────────────────────────────────────┐
  │        OVERFLOW VISUALIZATION              │
  │                                            │
  │  Unsigned:  ... 254 → 255 → 0 → 1 ...     │
  │                         ↑ wraps            │
  │                                            │
  │  Signed:  ... 126 → 127 → -128 → -127 ... │
  │                         ↑ wraps            │
  └────────────────────────────────────────────┘
```

---

## Real-World Implications

1. **Array indices** — Always unsigned (can't have negative index in most languages)
2. **Loop counters** — Be careful with unsigned when decrementing past 0
3. **Network protocols** — Port numbers are unsigned 16-bit (0–65535)
4. **Pixel values** — Colors are unsigned 8-bit per channel (0–255)
5. **Bug source** — Comparing signed and unsigned can cause subtle bugs

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Unsigned | All bits for magnitude, range: 0 to 2ⁿ − 1 |
| Signed | MSB is sign bit, range: −2ⁿ⁻¹ to 2ⁿ⁻¹ − 1 |
| Two's complement | Standard for signed integers in all modern CPUs |
| Single zero | Two's complement has only one zero representation |
| Overflow | Values wrap around at the boundaries |
| Sign bit | 0 = positive/zero, 1 = negative (in signed) |

---

## Quick Revision Questions

1. **What is the range of a 16-bit unsigned integer?**
   <details><summary>Answer</summary>0 to 65,535 (2¹⁶ − 1)</details>

2. **What is the range of a 16-bit signed integer?**
   <details><summary>Answer</summary>−32,768 to 32,767 (−2¹⁵ to 2¹⁵ − 1)</details>

3. **In 8-bit signed, what does 11111111₂ represent?**
   <details><summary>Answer</summary>−1 (not 255; that's the unsigned interpretation)</details>

4. **Why does two's complement have one more negative value than positive?**
   <details><summary>Answer</summary>Because zero uses one pattern from the "positive side" (0 has MSB=0), leaving one extra pattern on the negative side</details>

5. **What happens when you add 1 to the maximum unsigned 8-bit value?**
   <details><summary>Answer</summary>It overflows and wraps to 0: 255 + 1 = 0</details>

6. **Why is two's complement preferred over sign-magnitude?**
   <details><summary>Answer</summary>Single zero representation, same addition hardware for positive and negative numbers, no special circuit needed for subtraction</details>

---

[← Previous: Binary to Decimal](03-binary-to-decimal.md) | [Next: Two's Complement →](05-twos-complement.md)
