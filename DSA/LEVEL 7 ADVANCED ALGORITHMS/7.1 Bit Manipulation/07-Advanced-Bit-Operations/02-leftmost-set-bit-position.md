# Chapter 7.2: Leftmost (Highest) Set Bit Position

> **Unit 7 — Advanced Bit Operations**
> Find the position of the most significant bit (MSB) — determines the number's magnitude and bit-width.

---

## Overview

The leftmost set bit (Most Significant Bit or MSB) tells you the **floor of log2(n)** — the bit-width needed to represent n. This is essential for binary search bounds, power-of-2 rounding, and segment tree sizing.

---

## Methods

### Method 1: __builtin_clz (Count Leading Zeros)

```
  pos = 31 - __builtin_clz(n)     // 32-bit
  pos = 63 - __builtin_clzll(n)   // 64-bit

  Java:   31 - Integer.numberOfLeadingZeros(n)
          or Integer.highestOneBit(n)
  Python: n.bit_length() - 1
```

### Method 2: Bit Smearing + Count

```
FUNCTION msbPosition(n):
    // Smear all bits below MSB
    n = n | (n >> 1)
    n = n | (n >> 2)
    n = n | (n >> 4)
    n = n | (n >> 8)
    n = n | (n >> 16)
    // Now n is all 1s from MSB downward
    RETURN popcount(n) - 1
```

### Method 3: Simple Loop

```
FUNCTION msbPosition(n):
    IF n == 0: RETURN -1
    pos = 0
    WHILE n > 1:
        n = n >> 1
        pos += 1
    RETURN pos
```

---

## Step-by-Step Trace

### Example: n = 100

```
  100 = 0110 0100

  Method 1: __builtin_clz(100)
    Binary: 0000 0000 0000 0000 0000 0000 0110 0100
    Leading zeros: 25
    Position = 31 - 25 = 6  ✓ (bit 6 is the highest set bit)
    
  Method 3: Loop
    100 >> 1 = 50, pos=1
    50 >> 1 = 25, pos=2
    25 >> 1 = 12, pos=3
    12 >> 1 = 6,  pos=4
    6 >> 1 = 3,   pos=5
    3 >> 1 = 1,   pos=6
    1 is not > 1  → STOP, pos=6  ✓

  Verification: 2^6 = 64 ≤ 100 < 128 = 2^7  ✓
```

### Example: n = 1

```
  1 = 0000 0001
  MSB position = 0 (only bit 0 is set)
  floor(log2(1)) = 0  ✓
```

---

## Isolating the Highest Set Bit

```
  Java: Integer.highestOneBit(n)
  
  Manual:
    n = n | (n >> 1)     // smear
    n = n | (n >> 2)
    n = n | (n >> 4)
    n = n | (n >> 8)
    n = n | (n >> 16)
    highestBit = n - (n >> 1)   // or (n+1) >> 1

  Example: n = 100 (0110 0100)
    After smearing: 0111 1111
    0111 1111 - 0011 1111 = 0100 0000 = 64  ✓
```

---

## floor(log2(n)) Table

```
  n     │ Binary    │ MSB Position │ floor(log2(n))
  ──────┼───────────┼──────────────┼────────────────
  1     │ 1         │ 0            │ 0
  2     │ 10        │ 1            │ 1
  3     │ 11        │ 1            │ 1
  4     │ 100       │ 2            │ 2
  7     │ 111       │ 2            │ 2
  8     │ 1000      │ 3            │ 3
  15    │ 1111      │ 3            │ 3
  16    │ 10000     │ 4            │ 4
  255   │ 11111111  │ 7            │ 7
```

> MSB position = floor(log2(n)) for n > 0.

---

## Applications

| Use Case | How MSB Helps |
|----------|--------------|
| Binary search on answer | Determine search range: [0, 2^(msb+1)] |
| Segment tree size | Need 2^(msb+1) nodes, or next power of 2 |
| Bit-length of number | msb + 1 = number of bits needed |
| Most significant difference | Compare two numbers from MSB down |
| Priority in scheduling | Higher bits = higher priority |

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| __builtin_clz | O(1) — hardware | O(1) |
| Bit smearing | O(1) — 5 OR + 5 shifts | O(1) |
| Loop | O(log n) | O(1) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| MSB position | = floor(log2(n)) for n > 0 |
| clz method | 31 - __builtin_clz(n) |
| Smearing method | Fill bits below MSB, then count |
| Isolate MSB | Smear then subtract (n >> 1) |
| Edge case | n = 0 is undefined (clz(0) = undefined behavior) |

---

## Quick Revision Questions

1. **What is the MSB position of 200?**
   <details><summary>Answer</summary>200 = 1100 1000. MSB at bit 7. floor(log2(200)) = 7.</details>

2. **How many bits are needed to represent 1000?**
   <details><summary>Answer</summary>MSB position of 1000 = floor(log2(1000)) = 9. Bits needed = 9 + 1 = 10.</details>

3. **What is Integer.highestOneBit(50)?**
   <details><summary>Answer</summary>50 = 110010. Highest set bit = 100000 = 32.</details>

4. **What's the difference between n.bit_length() and MSB position?**
   <details><summary>Answer</summary>bit_length = MSB position + 1. E.g., 8 = 1000 has bit_length = 4, MSB position = 3.</details>

5. **How do you compute ceil(log2(n))?**
   <details><summary>Answer</summary>If n is a power of 2: ceil(log2(n)) = floor(log2(n)) = msb position. Otherwise: ceil(log2(n)) = floor(log2(n)) + 1 = msb position + 1. Or: ceil(log2(n)) = 32 - clz(n - 1) for n > 1.</details>

---

[← Previous: Rightmost Set Bit Position](01-rightmost-set-bit-position.md) | [Next: Count Bits in a Range →](03-count-bits-in-range.md)
