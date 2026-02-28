# Chapter 3.5: Check Power of 2

> **Unit 3 — Basic Bit Techniques**
> One of the most elegant bit tricks — detect if a number is a power of 2 in O(1).

---

## Overview

A number is a power of 2 if and only if it has **exactly one bit set** in its binary representation. This property leads to one of the most famous bit manipulation tricks.

---

## The Trick

```
  ┌───────────────────────────────────────────────────────┐
  │  n is a power of 2   ⟺   n > 0 AND n & (n-1) == 0  │
  └───────────────────────────────────────────────────────┘
```

## Why n & (n - 1) Works

When n is a power of 2, it has exactly one 1-bit. Subtracting 1 flips that bit and all bits below it:

```
  Powers of 2:         n          n-1        n & (n-1)
  ────────────────────────────────────────────────────
  1  = 00000001    00000001    00000000    00000000 = 0 ✓
  2  = 00000010    00000010    00000001    00000000 = 0 ✓
  4  = 00000100    00000100    00000011    00000000 = 0 ✓
  8  = 00001000    00001000    00000111    00000000 = 0 ✓
  16 = 00010000    00010000    00001111    00000000 = 0 ✓
  
  NOT powers of 2:
  ────────────────────────────────────────────────────
  3  = 00000011    00000011    00000010    00000010 ≠ 0 ✗
  6  = 00000110    00000110    00000101    00000100 ≠ 0 ✗
  12 = 00001100    00001100    00001011    00001000 ≠ 0 ✗

  Pattern:
  ┌──────────────────────────────────────────────────┐
  │  n   = ...0001000...0    (single 1 bit)          │
  │  n-1 = ...0000111...1    (that 1 becomes 0,      │
  │                           all below become 1)    │
  │  n & (n-1) = 0           (no overlapping 1s!)    │
  └──────────────────────────────────────────────────┘
```

---

## Visual Proof

```
  n = 16:    0 0 0 1 0 0 0 0
  n-1 = 15:  0 0 0 0 1 1 1 1
                     ↑ ↑ ↑ ↑ ↑
              No positions share a 1
  n & (n-1): 0 0 0 0 0 0 0 0 = 0 ✓ (power of 2!)

  n = 12:    0 0 0 0 1 1 0 0
  n-1 = 11:  0 0 0 0 1 0 1 1
                     ↑
              This position shares a 1
  n & (n-1): 0 0 0 0 1 0 0 0 = 8 ≠ 0 ✗ (NOT power of 2)
```

---

## Edge Case: n = 0

```
  0 is NOT a power of 2, but 0 & (0-1) = 0 & (-1) = 0
  
  This would be a false positive!
  
  Solution: Add n > 0 check.
  
  CORRECT formula:  n > 0  AND  n & (n-1) == 0
```

---

## Pseudocode

```
FUNCTION isPowerOf2(n):
    RETURN n > 0 AND (n & (n - 1)) == 0
```

---

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Bit trick (n & (n-1)) | O(1) | O(1) |
| Loop (divide by 2) | O(log n) | O(1) |
| Math (log₂) | O(1)* | O(1) |

> *Log approach may have floating-point precision issues

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Power of 2 in binary | Exactly one bit set |
| Formula | `n > 0 && (n & (n-1)) == 0` |
| Why it works | n-1 flips the lone 1 and bits below; AND gives 0 |
| Edge case | n = 0 must be checked separately |

---

## Quick Revision Questions

1. **Is 64 a power of 2? Use the formula.**
   <details><summary>Answer</summary>64 = 1000000₂, 63 = 0111111₂. 64 & 63 = 0. Yes, power of 2 ✓</details>

2. **Is 96 a power of 2?**
   <details><summary>Answer</summary>96 = 01100000₂. Two bits set, so NOT a power of 2.</details>

3. **Why do we need the n > 0 check?**
   <details><summary>Answer</summary>Because 0 & (-1) = 0, which falsely passes the test. Zero is not a power of 2.</details>

4. **What does n & (n-1) do in general (not just for power of 2)?**
   <details><summary>Answer</summary>It turns off the rightmost set bit. For powers of 2 with only one set bit, it clears to 0.</details>

5. **What's the time complexity of this check vs a loop approach?**
   <details><summary>Answer</summary>O(1) vs O(log n). The bit trick is a single operation, loop divides by 2 repeatedly.</details>

---

[← Previous: Toggle a Bit](04-toggle-a-bit.md) | [Next: Count Set Bits →](06-count-set-bits.md)
