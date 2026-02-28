# Chapter 4.4: Reverse Bits of an Integer

> **Unit 4 — Bit Manipulation Tricks**
> Flip the entire bit sequence of a 32-bit integer — the mirror image in binary.

---

## Overview

Reversing bits means taking the binary representation and reading it backwards. This is different from flipping bits (NOT). It's a mirror/reflection operation.

```
  Original:  1101 0010 0000 0000 0000 0000 0000 0000
  Reversed:  0000 0000 0000 0000 0100 1011

  n = 3523215360    →    reversed = 75
```

---

## Method 1: One-Bit-At-a-Time

### Concept

```
  ┌─────────────────────────────────────────────────────┐
  │  Extract the rightmost bit of n.                    │
  │  Push it into result by shifting result left.       │
  │  Shift n right to process the next bit.             │
  │  Repeat 32 times.                                   │
  └─────────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION reverseBits(n):
    result = 0
    FOR i = 0 TO 31:
        result = result << 1         // make room
        result = result | (n & 1)    // copy rightmost bit of n
        n = n >> 1                   // move to next bit of n
    RETURN result
```

### Trace: n = 13 (8-bit simplified)

```
  n = 0000 1101 (13)

  i=0: result = 0<<1 = 0, n&1 = 1  → result = 1    n = 0000 0110
  i=1: result = 1<<1 = 10, n&1 = 0 → result = 10   n = 0000 0011
  i=2: result = 10<<1 = 100, n&1 = 1 → result = 101 n = 0000 0001
  i=3: result = 101<<1 = 1010, n&1 = 1 → result = 1011 n = 0000 0000
  i=4..7: n is all 0, result keeps shifting left

  Final (8-bit): result = 1011 0000 = 176

  Verify: reverse of 0000 1101 → 1011 0000 ✓
```

---

## Method 2: Divide and Conquer (Parallel Swap)

### Concept

```
  Swap adjacent 1-bit blocks
  Then swap adjacent 2-bit blocks
  Then swap adjacent 4-bit blocks
  Then swap adjacent 8-bit blocks
  Then swap adjacent 16-bit blocks

  ┌────────────────────────────────────────────────────┐
  │  abcd efgh   (original 8-bit for illustration)     │
  │                                                    │
  │  Step 1: Swap 1-bit   → badc fehg                 │
  │  Step 2: Swap 2-bit   → dcba hgfe                 │
  │  Step 3: Swap 4-bit   → hgfe dcba                 │
  │                                                    │
  │  Done! Fully reversed!                              │
  └────────────────────────────────────────────────────┘
```

### Pseudocode (32-bit)

```
FUNCTION reverseBits(n):
    // Swap adjacent bits
    n = ((n >> 1) & 0x55555555) | ((n & 0x55555555) << 1)
    // Swap adjacent 2-bit pairs
    n = ((n >> 2) & 0x33333333) | ((n & 0x33333333) << 2)
    // Swap adjacent 4-bit nibbles
    n = ((n >> 4) & 0x0F0F0F0F) | ((n & 0x0F0F0F0F) << 4)
    // Swap adjacent bytes
    n = ((n >> 8) & 0x00FF00FF) | ((n & 0x00FF00FF) << 8)
    // Swap adjacent 2-byte halves
    n = (n >> 16) | (n << 16)
    RETURN n
```

### Masks Explained

```
  0x55555555 = 0101 0101 ... 0101   (alternating bits)
  0x33333333 = 0011 0011 ... 0011   (alternating 2-bit pairs)
  0x0F0F0F0F = 0000 1111 ... 0000 1111  (alternating nibbles)
  0x00FF00FF = 0000 0000 1111 1111 ...  (alternating bytes)
```

---

## Method 3: Lookup Table

```
  Pre-compute reversed value of every byte (0-255).
  Split 32-bit integer into 4 bytes.
  Look up each reversed byte.
  Reassemble in reverse order.

  ┌──────────────────────────────────────────────┐
  │  n = [byte3][byte2][byte1][byte0]            │
  │                                              │
  │  result = [rev(byte0)][rev(byte1)]           │
  │           [rev(byte2)][rev(byte3)]           │
  └──────────────────────────────────────────────┘
```

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| One-bit-at-a-time | O(32) = O(1) | O(1) |
| Divide & Conquer | O(1) — 5 operations | O(1) |
| Lookup table | O(1) — 4 lookups | O(256) for table |

---

## Real-World Applications

- **FFT (Fast Fourier Transform)** — bit-reversal permutation
- **Network protocols** — bit order conversion (LSB ↔ MSB)
- **Graphics** — image mirroring at bit level
- **Cryptography** — permutation operations

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Reverse vs NOT | Reverse changes order; NOT flips values |
| Simple loop | Extract LSB of n, push into result, 32 iterations |
| Divide & conquer | Parallel swaps: 1→2→4→8→16 bit chunks |
| Lookup table | Pre-compute 256-byte table, 4 lookups |

---

## Quick Revision Questions

1. **Reverse the 8-bit representation of 5 (0000 0101).**
   <details><summary>Answer</summary>1010 0000 = 160</details>

2. **What happens if you reverse bits of a palindrome binary number?**
   <details><summary>Answer</summary>You get the same number. E.g., 0110 0110 reversed (8-bit) = 0110 0110.</details>

3. **How many operations does the divide-and-conquer method use?**
   <details><summary>Answer</summary>5 swap steps (each with 2 shifts, 2 ANDs, 1 OR). Total ~15 bitwise operations. All O(1).</details>

4. **Why do we need exactly 32 iterations in the loop method?**
   <details><summary>Answer</summary>We're reversing all 32 bit positions of a 32-bit integer. Even if the number is small, we must place bits at correct positions from the other end.</details>

5. **Is reverseBits(reverseBits(n)) == n?**
   <details><summary>Answer</summary>Yes, it's a self-inverse operation. Reversing twice restores the original.</details>

---

[← Previous: Check Power of 4](03-check-power-of-4.md) | [Next: Next Power of 2 →](05-next-power-of-2.md)
