# Chapter 3.4: Toggle a Bit

> **Unit 3 — Basic Bit Techniques**
> Flipping a specific bit — if it's 1, make it 0; if it's 0, make it 1.

---

## Overview

Toggling (flipping) a bit changes its value to the opposite state. XOR with 1 flips a bit, while XOR with 0 preserves it — so we XOR with a mask targeting the desired position.

---

## The Formula

```
  ┌──────────────────────────────────────────┐
  │  TOGGLE bit k:   n = n ^ (1 << k)       │
  │                                          │
  │  Shorthand:      n ^= (1 << k)          │
  └──────────────────────────────────────────┘
```

## How It Works

```
  XOR Truth:  x ^ 0 = x  (unchanged)
              x ^ 1 = ~x (flipped)

  ┌──────────────────────────────────────────────────┐
  │  Case 1: Bit is 1 → becomes 0                   │
  │  n    = 0 0 1 0 1 0 1 0     (42)                │
  │  mask = 0 0 0 0 1 0 0 0     (1 << 3)            │
  │         ─────────────────                        │
  │  n^mask= 0 0 1 0 0 0 1 0    (34) ← 1→0         │
  │                                                  │
  │  Case 2: Bit is 0 → becomes 1                   │
  │  n    = 0 0 1 0 0 0 1 0     (34)                │
  │  mask = 0 0 0 0 1 0 0 0     (1 << 3)            │
  │         ─────────────────                        │
  │  n^mask= 0 0 1 0 1 0 1 0    (42) ← 0→1         │
  └──────────────────────────────────────────────────┘
  
  Toggle is its own inverse! Toggle twice → back to original.
```

---

## The Four Operations Compared

```
  ┌───────────┬────────────────┬─────────┬──────────────┐
  │ Operation │ Formula        │ Operator│ Effect       │
  ├───────────┼────────────────┼─────────┼──────────────┤
  │ Check     │ (n >> k) & 1   │   &     │ Read only    │
  │ Set       │ n | (1 << k)   │   |     │ Force → 1    │
  │ Clear     │ n & ~(1 << k)  │   &~    │ Force → 0    │
  │ Toggle    │ n ^ (1 << k)   │   ^     │ Flip 0↔1     │
  └───────────┴────────────────┴─────────┴──────────────┘
```

---

## Pseudocode

```
FUNCTION toggleBit(n, k):
    RETURN n ^ (1 << k)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Formula | `n ^ (1 << k)` |
| Mechanism | XOR with 1 flips, XOR with 0 preserves |
| Self-inverse | Toggling twice restores original |
| Toggle multiple | `n ^ mask` flips all bits where mask=1 |

---

## Quick Revision Questions

1. **Toggle bit 0 of n = 10. What's the result?**
   <details><summary>Answer</summary>10 = 1010, toggle bit 0: 1010 ^ 0001 = 1011 = 11</details>

2. **If you toggle a bit twice, what's the result?**
   <details><summary>Answer</summary>The original value. a ^ b ^ b = a (XOR is self-inverse)</details>

3. **Toggle ALL bits of an 8-bit number n.**
   <details><summary>Answer</summary>`n ^ 0xFF` — XOR with all-ones flips every bit (same as ~n for 8-bit)</details>

4. **What operator is used for toggling?**
   <details><summary>Answer</summary>XOR (^)</details>

5. **Toggle bits 0, 2, 4 of n = 0. What do you get?**
   <details><summary>Answer</summary>0 ^ (1|4|16) = 0 ^ 21 = 21 = 10101₂</details>

---

[← Previous: Clear a Bit](03-clear-a-bit.md) | [Next: Check Power of 2 →](05-check-power-of-2.md)
