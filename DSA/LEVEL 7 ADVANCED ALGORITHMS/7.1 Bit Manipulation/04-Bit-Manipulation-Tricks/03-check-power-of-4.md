# Chapter 4.3: Check If a Number Is a Power of 4

> **Unit 4 — Bit Manipulation Tricks**
> Combine the power-of-2 check with a bit-position constraint to detect powers of 4.

---

## Overview

A number is a power of 4 if it's of the form 4^k = 2^(2k). That means:
1. It must be a power of 2 (exactly one set bit)
2. That set bit must be at an **even** position (0, 2, 4, 6, …)

---

## Visual Pattern

```
  Power of 4    Binary           Position of set bit
  ─────────────────────────────────────────────────
  4^0 = 1       0000 0001        bit 0 (even ✓)
  4^1 = 4       0000 0100        bit 2 (even ✓)
  4^2 = 16      0001 0000        bit 4 (even ✓)
  4^3 = 64      0100 0000        bit 6 (even ✓)

  NOT power of 4:
  2 =           0000 0010        bit 1 (odd ✗)
  8 =           0000 1000        bit 3 (odd ✗)
  32 =          0010 0000        bit 5 (odd ✗)
```

---

## The Magic Mask: 0x55555555

```
  Hex:  5    5    5    5    5    5    5    5
  Bin:  0101 0101 0101 0101 0101 0101 0101 0101

  This mask has 1s at ALL even positions (0, 2, 4, 6, …)
```

---

## Three-Condition Check

```
  ┌───────────────────────────────────────────────────────┐
  │  isPowerOf4(n):                                       │
  │    1) n > 0                  (exclude 0 and negatives)│
  │    2) n & (n - 1) == 0       (power of 2 check)       │
  │    3) n & 0x55555555 != 0    (set bit at even pos)    │
  └───────────────────────────────────────────────────────┘
```

### Trace: n = 16

```
  n = 16 = 0001 0000

  Step 1: 16 > 0  → true ✓

  Step 2: n & (n-1)
          0001 0000    (16)
        & 0000 1111    (15)
        ──────────
          0000 0000 = 0 → true ✓   (it's a power of 2)

  Step 3: n & 0x55555555
          0001 0000    (16)
        & 0101 0101    (0x55 lower byte)
        ──────────
          0001 0000 ≠ 0 → true ✓   (bit 4 is even position)

  Result: Power of 4 ✓
```

### Trace: n = 8

```
  n = 8 = 0000 1000

  Step 1: 8 > 0  → true ✓

  Step 2: n & (n-1)
          0000 1000    (8)
        & 0000 0111    (7)
        ──────────
          0000 0000 = 0 → true ✓   (it's a power of 2)

  Step 3: n & 0x55555555
          0000 1000    (8)
        & 0101 0101    (0x55 lower byte)
        ──────────
          0000 0000 = 0 → false ✗  (bit 3 is odd position!)

  Result: NOT a power of 4 ✗
```

---

## Pseudocode

```
FUNCTION isPowerOf4(n):
    RETURN n > 0
       AND n & (n - 1) == 0
       AND n & 0x55555555 != 0
```

---

## Alternative: Modulo Trick

```
  Powers of 4 mod 3:
    1 % 3 = 1
    4 % 3 = 1
    16 % 3 = 1
    64 % 3 = 1
  
  Powers of 2 (not 4) mod 3:
    2 % 3 = 2
    8 % 3 = 2
    32 % 3 = 2

  So: isPowerOf4 = isPowerOf2(n) AND n % 3 == 1
```

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| Bitmask (0x55555555) | O(1) | O(1) |
| Modulo 3 | O(1) | O(1) |
| Loop (divide by 4) | O(log n) | O(1) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Power of 4 | 2^(2k) — single set bit at even position |
| Magic mask | 0x55555555 = bits at even positions |
| Three checks | n > 0, power-of-2, even position |
| Alt method | n % 3 == 1 for powers of 4 |

---

## Quick Revision Questions

1. **Why does 0x55555555 work?**
   <details><summary>Answer</summary>In binary it's 0101...0101 — it has 1s at all even bit positions (0,2,4,...). AND-ing with it checks if the set bit of a power-of-2 is at an even position.</details>

2. **Is 0 a power of 4?**
   <details><summary>Answer</summary>No. The n > 0 check excludes it. Also, 0 has no set bits.</details>

3. **Can we check power of 8 using a similar mask?**
   <details><summary>Answer</summary>Power of 8 = 2^(3k), so set bit at positions 0,3,6,9,... Mask = 0x49249249 (binary: ...001001001). Check power-of-2 AND mask hit.</details>

4. **Why does n % 3 == 1 work for powers of 4?**
   <details><summary>Answer</summary>4 ≡ 1 mod 3, so 4^k ≡ 1^k = 1 mod 3. But 2 ≡ 2 mod 3, and 2·4^k = 2 mod 3. This separates powers of 4 from other powers of 2.</details>

5. **What's the largest power of 4 in a 32-bit signed integer?**
   <details><summary>Answer</summary>4^15 = 2^30 = 1,073,741,824. (4^16 = 2^32 overflows)</details>

---

[← Previous: Find Odd Occurring Element](02-find-odd-occurring-element.md) | [Next: Reverse Bits →](04-reverse-bits.md)
