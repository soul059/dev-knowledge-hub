# Chapter 3.3: Clear a Bit

> **Unit 3 — Basic Bit Techniques**
> Turning OFF a specific bit without disturbing any other bits.

---

## Overview

Clearing a bit means forcing it to **0** regardless of its current value. We use AND with an **inverted mask** — a mask with 0 only at the target position and 1 everywhere else.

---

## The Formula

```
  ┌──────────────────────────────────────────┐
  │  CLEAR bit k:   n = n & ~(1 << k)       │
  │                                          │
  │  Shorthand:     n &= ~(1 << k)          │
  └──────────────────────────────────────────┘
```

## How It Works

```
  AND Truth:  x & 1 = x  (unchanged)
              x & 0 = 0  (forced OFF)

  ~(1 << k) creates a mask with 0 at position k, 1 everywhere else.

  ┌──────────────────────────────────────────────────┐
  │  n      = 0 1 0 0 1 1 0 0     (76)              │
  │  1 << 2 = 0 0 0 0 0 1 0 0                       │
  │ ~(1<<2) = 1 1 1 1 1 0 1 1     (inverted mask)   │
  │           ─────────────────                      │
  │  result = 0 1 0 0 1 0 0 0     (72)              │
  │                   ↑                              │
  │            This bit CLEARED to 0                 │
  │            All others UNCHANGED                  │
  └──────────────────────────────────────────────────┘
```

### Step-by-Step Trace: Clear bit 3 in n = 42

```
  n = 42 = 00101010
  k = 3

  Step 1: Create mask
    1 << 3  = 00001000
  Step 2: Invert mask
    ~(1<<3) = 11110111
  Step 3: AND
    00101010
  & 11110111
  ──────────
    00100010 = 34

  Bit 3 changed from 1 → 0
  All other bits unchanged ✓
```

---

## Clearing Multiple Bits

```
  Clear bits 1 and 3:
  
  mask = (1 << 1) | (1 << 3) = 00001010
  ~mask = 11110101
  n &= ~mask;    // Both bits cleared at once
```

## Clearing a Range of Bits

```
  Clear bits 2 through 5 of n:
  
  range_mask = ((1 << 4) - 1) << 2 = 00111100
  n &= ~range_mask;        // = n & 11000011
```

---

## Pseudocode

```
FUNCTION clearBit(n, k):
    RETURN n & ~(1 << k)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Formula | `n & ~(1 << k)` |
| Mechanism | AND with 0 forces OFF, AND with 1 preserves |
| Inverted mask | `~(1 << k)` has 0 at position k, 1 elsewhere |
| Idempotent | Clearing already-clear bit is harmless |

---

## Quick Revision Questions

1. **Clear bit 5 in n = 63. What's the result?**
   <details><summary>Answer</summary>63 = 00111111, ~(1<<5) = 11011111. 00111111 & 11011111 = 00011111 = 31</details>

2. **Why do we invert the mask before ANDing?**
   <details><summary>Answer</summary>We need 0 at the target position to clear it, and 1 everywhere else to preserve. Inverting `(1 << k)` achieves exactly this.</details>

3. **What happens when you clear a bit that's already 0?**
   <details><summary>Answer</summary>Nothing changes. AND with 0 when bit is already 0 keeps it at 0.</details>

4. **Write an expression to clear bits 0 through 3.**
   <details><summary>Answer</summary>`n & ~0xF` or `n & (~0 << 4)` — both produce a mask of 1s in the upper bits and 0s in the lower 4.</details>

5. **What is `~(1 << 0)` in 8-bit?**
   <details><summary>Answer</summary>~00000001 = 11111110 = 0xFE = 254 (unsigned)</details>

---

[← Previous: Set a Bit](02-set-a-bit.md) | [Next: Toggle a Bit →](04-toggle-a-bit.md)
