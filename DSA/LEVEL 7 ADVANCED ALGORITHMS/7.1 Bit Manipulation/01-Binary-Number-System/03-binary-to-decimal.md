# Chapter 1.3: Converting Binary to Decimal

> **Unit 1 — Binary Number System**
> Translating machine-level binary back into human-readable decimal values.

---

## Overview

Converting binary to decimal is the reverse of decimal-to-binary conversion. We reconstruct the decimal value by summing up the **weighted contributions** of each bit position. This skill is critical for debugging, reading memory dumps, and understanding bitwise operation results.

---

## Method 1: Positional Notation (Weighted Sum)

Multiply each bit by its corresponding power of 2, then sum.

### Formula

$$
\text{Decimal} = \sum_{i=0}^{n-1} b_i \times 2^i
$$

Where $b_i$ is the bit at position $i$ (from right, starting at 0).

### Step-by-Step Trace: Convert 11010110₂ to Decimal

```
  Position:    7     6     5     4     3     2     1     0
  Bit:         1     1     0     1     0     1     1     0
  Weight:     2⁷    2⁶    2⁵    2⁴    2³    2²    2¹    2⁰
  Value:      128    64    32    16     8     4     2     1

  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │  1  │  1  │  0  │  1  │  0  │  1  │  1  │  0  │
  └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘
     │     │     │     │     │     │     │     │
   1×128 1×64  0×32 1×16  0×8  1×4  1×2   0×1
   = 128  = 64  = 0  = 16  = 0  = 4   = 2   = 0

  Sum = 128 + 64 + 0 + 16 + 0 + 4 + 2 + 0
      = 214₁₀

  11010110₂ = 214₁₀ ✓
```

---

## Method 2: Doubling Method (Horner's Method)

Process bits left to right. Start with 0, double the running total and add the next bit.

### Algorithm

```
FUNCTION binaryToDecimal(bits[]):
    result = 0
    FOR each bit b from LEFT to RIGHT:
        result = result × 2 + b
    RETURN result
```

### Step-by-Step Trace: Convert 101101₂

```
  Bits: 1  0  1  1  0  1

  Step │ Current Bit │ Calculation        │ Result
  ─────┼─────────────┼────────────────────┼────────
    1  │     1       │  0 × 2 + 1 = 1     │    1
    2  │     0       │  1 × 2 + 0 = 2     │    2
    3  │     1       │  2 × 2 + 1 = 5     │    5
    4  │     1       │  5 × 2 + 1 = 11    │   11
    5  │     0       │ 11 × 2 + 0 = 22    │   22
    6  │     1       │ 22 × 2 + 1 = 45    │   45

  101101₂ = 45₁₀ ✓
```

### Why Horner's Method Works

```
  101101₂ = 1×2⁵ + 0×2⁴ + 1×2³ + 1×2² + 0×2¹ + 1×2⁰

  Factor out 2 repeatedly:
  = ((((1×2 + 0)×2 + 1)×2 + 1)×2 + 0)×2 + 1
  
  This is exactly what the doubling method computes!
  No need to know powers of 2 — just double and add.
```

---

## Method 3: Using Bitwise Operations (Code)

### Pseudocode

```
FUNCTION binaryStringToDecimal(s):
    result = 0
    FOR i = 0 TO length(s) - 1:
        result = result << 1          // left shift = multiply by 2
        IF s[i] == '1':
            result = result | 1       // OR with 1 = add 1
    RETURN result
```

### Visual Trace: "1010"

```
  Start: result = 0          → 0000₂

  i=0, bit='1':
    result << 1  → 0000₂     (0 × 2 = 0)
    result | 1   → 0001₂     (0 + 1 = 1)

  i=1, bit='0':
    result << 1  → 0010₂     (1 × 2 = 2)
    (no OR)      → 0010₂     (result stays 2)

  i=2, bit='1':
    result << 1  → 0100₂     (2 × 2 = 4)
    result | 1   → 0101₂     (4 + 1 = 5)

  i=3, bit='0':
    result << 1  → 1010₂     (5 × 2 = 10)
    (no OR)      → 1010₂     (result stays 10)

  Final: 1010₂ = 10₁₀ ✓
```

---

## Method 4: Only Sum the 1-Bits

Instead of multiplying every bit, just add powers of 2 where bits are 1:

### Trace: Convert 10110₂

```
  ┌─────┬─────┬─────┬─────┬─────┐
  │  1  │  0  │  1  │  1  │  0  │
  └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘
     │     │     │     │     │
    2⁴    2³    2²    2¹    2⁰
   =16   skip   =4    =2   skip
   
  Sum of 1-bits only: 16 + 4 + 2 = 22₁₀
  
  10110₂ = 22₁₀ ✓
```

---

## Converting Binary Fractions to Decimal

For bits after the binary point, use **negative** powers of 2:

### Trace: Convert 101.011₂

```
  Integer part:  1 0 1
                 │ │ └── 1 × 2⁰ = 1
                 │ └──── 0 × 2¹ = 0
                 └────── 1 × 2² = 4
                 Integer = 5

  Fractional part: . 0  1  1
                     │  │  └── 1 × 2⁻³ = 1/8 = 0.125
                     │  └──── 1 × 2⁻² = 1/4 = 0.25
                     └────── 0 × 2⁻¹ = 0
                     Fraction = 0.375

  Total: 5 + 0.375 = 5.375₁₀
  
  101.011₂ = 5.375₁₀ ✓
```

---

## Comparison of Methods

```
  ┌──────────────────────────────────────────────────────────┐
  │                METHOD COMPARISON                         │
  ├────────────────────┬─────────────┬───────────────────────┤
  │ Method             │ Direction   │ Best For              │
  ├────────────────────┼─────────────┼───────────────────────┤
  │ Weighted Sum       │ Right→Left  │ Understanding theory  │
  │ Doubling (Horner)  │ Left→Right  │ Mental math           │
  │ Bitwise Operations │ Left→Right  │ Programming           │
  │ Sum 1-bits only    │ Any         │ Quick calculation     │
  └────────────────────┴─────────────┴───────────────────────┘
```

---

## Complexity Analysis

| Method | Time Complexity | Space Complexity |
|--------|----------------|------------------|
| Weighted Sum | O(n) | O(1) |
| Doubling Method | O(n) | O(1) |
| Bitwise Method | O(n) | O(1) |
| Sum 1-bits | O(k) | O(1) |

> Where **n** = number of bits, **k** = number of set bits (k ≤ n)

---

## Quick Mental Math Tricks

```
  ╔══════════════════════════════════════════════════════╗
  ║  MENTAL MATH SHORTCUTS                              ║
  ╠══════════════════════════════════════════════════════╣
  ║                                                      ║
  ║  1. All 1s pattern:                                  ║
  ║     1111₂ = 2⁴ - 1 = 15                             ║
  ║     11111111₂ = 2⁸ - 1 = 255                        ║
  ║                                                      ║
  ║  2. Single 1 followed by zeros:                      ║
  ║     10000₂ = 2⁴ = 16                                ║
  ║     100000000₂ = 2⁸ = 256                           ║
  ║                                                      ║
  ║  3. Recognize common patterns:                       ║
  ║     1010₂ = 10    (every other bit set)              ║
  ║     1100₂ = 12    (top half of nibble)               ║
  ║                                                      ║
  ║  4. Subtract from known value:                       ║
  ║     1111110₂ = 1111111₂ - 1 = 127 - 1 = 126        ║
  ╚══════════════════════════════════════════════════════╝
```

---

## Practice Conversions

Convert the following to decimal:

| Binary | Your Answer | Actual Decimal |
|--------|-------------|----------------|
| 1001 | ___ | 9 |
| 110110 | ___ | 54 |
| 10000000 | ___ | 128 |
| 11111111 | ___ | 255 |
| 1010101 | ___ | 85 |
| 11000011 | ___ | 195 |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Weighted sum | Multiply each bit by 2^position, add them |
| Horner's method | Double running total + next bit (left→right) |
| Bitwise code | Use left-shift and OR operations |
| 1-bits shortcut | Only sum powers of 2 where bit = 1 |
| All-1s pattern | n ones = 2ⁿ − 1 |
| Fractions | Use negative powers: 2⁻¹ = 0.5, 2⁻² = 0.25 |

---

## Quick Revision Questions

1. **Convert 1100100₂ to decimal using the doubling method.**
   <details><summary>Answer</summary>1→1, 1→3, 0→6, 0→12, 1→25, 0→50, 0→100. Answer: 100₁₀</details>

2. **What decimal value does 11111111₂ represent (unsigned)?**
   <details><summary>Answer</summary>2⁸ − 1 = 255</details>

3. **Convert 0.101₂ to decimal.**
   <details><summary>Answer</summary>1×(1/2) + 0×(1/4) + 1×(1/8) = 0.5 + 0 + 0.125 = 0.625</details>

4. **Which method avoids computing powers of 2?**
   <details><summary>Answer</summary>Horner's doubling method — just double and add</details>

5. **How do you quickly know that 10000₂ = 16?**
   <details><summary>Answer</summary>A single 1 at position 4 = 2⁴ = 16</details>

6. **What is the decimal value of 1111₂ without calculating?**
   <details><summary>Answer</summary>2⁴ − 1 = 15 (all-ones pattern shortcut)</details>

---

[← Previous: Decimal to Binary](02-decimal-to-binary.md) | [Next: Signed vs Unsigned →](04-signed-vs-unsigned.md)
