# Chapter 4.5: Find the Next Power of 2

> **Unit 4 — Bit Manipulation Tricks**
> Given n, find the smallest power of 2 that is greater than or equal to n.

---

## Overview

Finding the next (or equal) power of 2 is a common need in hash tables (bucket sizing), memory allocation, and buffer management. Bit manipulation gives us an elegant O(1) solution.

---

## Key Insight

```
  ┌─────────────────────────────────────────────────────┐
  │  We want to "fill" all bits below the highest       │
  │  set bit with 1s, then add 1.                       │
  │                                                      │
  │  n = 0001 xxxx   (some number with MSB at bit 4)    │
  │  fill → 0001 1111                                   │
  │  +1   → 0010 0000  ← next power of 2!              │
  └─────────────────────────────────────────────────────┘
```

But first, we subtract 1 (to handle the case where n is already a power of 2):

```
  n = 16 = 0001 0000
  n-1     = 0000 1111
  fill    = 0000 1111
  +1      = 0001 0000 = 16 ✓ (already a power)

  n = 17 = 0001 0001
  n-1     = 0001 0000
  fill    = 0001 1111
  +1      = 0010 0000 = 32 ✓ (next power)
```

---

## The Algorithm: Bit Smearing

```
FUNCTION nextPowerOf2(n):
    IF n <= 0: RETURN 1
    n = n - 1
    n = n | (n >> 1)    // copy bit 1 position right
    n = n | (n >> 2)    // copy bits 2 positions right
    n = n | (n >> 4)    // copy bits 4 positions right
    n = n | (n >> 8)    // copy bits 8 positions right
    n = n | (n >> 16)   // copy bits 16 positions right
    RETURN n + 1
```

---

## Step-by-Step Trace: n = 100

```
  n = 100 = 0110 0100

  Step 0: n = n - 1 = 99 = 0110 0011

  Step 1: n | (n >> 1)
          0110 0011
        | 0011 0001
        ──────────
          0111 0011

  Step 2: n | (n >> 2)
          0111 0011
        | 0001 1100
        ──────────
          0111 1111

  Step 3: n | (n >> 4)
          0111 1111
        | 0000 0111
        ──────────
          0111 1111   (already filled)

  Steps 4-5: No change (already all 1s below MSB)

  Step 6: n + 1
          0111 1111 + 1 = 1000 0000 = 128

  Answer: 128 ✓ (next power of 2 ≥ 100)
```

---

## Why Each Shift Width?

```
  After >> 1:  MSB copied 1 position  → 2 bits set
  After >> 2:  Those 2 copied 2 more  → 4 bits set
  After >> 4:  Those 4 copied 4 more  → 8 bits set
  After >> 8:  Those 8 copied 8 more  → 16 bits set
  After >> 16: Those 16 copied 16     → 32 bits set (all below MSB)

  Total coverage: 1 + 2 + 4 + 8 + 16 = 31 bits
  This handles any 32-bit number!
```

---

## Edge Cases

| Input | n - 1 | After Smearing | +1 | Result |
|-------|-------|----------------|-----|--------|
| 0 | return early | — | — | 1 |
| 1 | 0 | 0 | 1 | 1 |
| 2 | 1 | 1 | 2 | 2 |
| 3 | 2 (10) | 11 | 100 | 4 |
| 8 | 7 (0111) | 0111 | 1000 | 8 |
| 9 | 8 (1000) | 1111 | 10000 | 16 |

---

## Alternative: Using __builtin_clz

```
  // Count leading zeros → find highest bit position
  nextPow2(n) = 1 << (32 - __builtin_clz(n - 1))
  
  // Available in GCC/Clang. Java has Integer.highestOneBit().
```

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| Bit smearing | O(1) — 5 shifts, 5 ORs | O(1) |
| CLZ builtin | O(1) | O(1) |
| Loop (multiply by 2) | O(log n) | O(1) |

---

## Real-World Applications

- **Hash tables** — bucket count must be power of 2 for fast modulo
- **Memory allocators** — alignment to power-of-2 boundaries
- **Graphics buffers** — texture sizes often power of 2
- **Ring buffers** — size must be power of 2 for wrap-around with & mask

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Bit smearing | OR with shifted self: fills all bits below MSB |
| Why n - 1 first | Handles the case when n is already a power of 2 |
| Shift amounts | 1, 2, 4, 8, 16 — covers all 32 bits |
| Final step | Add 1 to 0111...1 → get 1000...0 |

---

## Quick Revision Questions

1. **What is the next power of 2 for n = 65?**
   <details><summary>Answer</summary>128. 64 < 65 ≤ 128, so the answer is 128.</details>

2. **Why do we subtract 1 before smearing?**
   <details><summary>Answer</summary>If n is already a power of 2 (e.g., 64 = 1000000), subtracting 1 gives 0111111. Smearing + adding 1 gives back 1000000 = 64. Without n-1, we'd get 128 incorrectly.</details>

3. **Would this work for 64-bit integers?**
   <details><summary>Answer</summary>Yes, add one more step: n = n | (n >> 32). This extends coverage to 64 bits.</details>

4. **What does bit smearing produce from 0001 0000?**
   <details><summary>Answer</summary>After n-1: 0000 1111. Smearing keeps it 0000 1111. Adding 1: 0001 0000 = 16 (same, it was already a power of 2).</details>

5. **How is this used in hash tables?**
   <details><summary>Answer</summary>If capacity is a power of 2, then `index = hash & (capacity - 1)` replaces expensive modulo `hash % capacity`.</details>

---

[← Previous: Reverse Bits](04-reverse-bits.md) | [Next: Turn Off Rightmost Set Bit →](06-turn-off-rightmost-set-bit.md)
