# Chapter 3.2: Set a Bit

> **Unit 3 — Basic Bit Techniques**
> Turning ON a specific bit without touching any other bits.

---

## Overview

Setting a bit means forcing it to **1** regardless of its current value. The key insight is that OR with 1 produces 1, and OR with 0 leaves the bit unchanged — so we use OR with a mask.

---

## The Formula

```
  ┌──────────────────────────────────────┐
  │  SET bit k:   n = n | (1 << k)      │
  │                                      │
  │  Shorthand:   n |= (1 << k)         │
  └──────────────────────────────────────┘
```

## How It Works

```
  OR Truth:  x | 0 = x  (unchanged)
             x | 1 = 1  (forced ON)

  So: OR with a mask that has 1 ONLY at position k

  ┌─────────────────────────────────────────────────┐
  │  n    = 0 1 0 0 1 0 0 0     (72)               │
  │  mask = 0 0 0 0 0 1 0 0     (1 << 2 = 4)       │
  │         ─────────────────                       │
  │ result= 0 1 0 0 1 1 0 0     (76)               │
  │                   ↑                             │
  │              This bit SET to 1                  │
  │              All others UNCHANGED               │
  └─────────────────────────────────────────────────┘
```

### Step-by-Step Trace: Set bit 4 in n = 35

```
  n = 35 = 00100011
  k = 4

  Step 1: Create mask
    1 << 4 = 00010000

  Step 2: OR
    00100011   (35)
  | 00010000   (mask)
  ──────────
    00110011   (51)

  Result: 51
  Bit 4 changed from 0 → 1
  All other bits unchanged ✓
```

### Idempotent Property

```
  Setting an ALREADY SET bit does nothing extra:

  n = 42 = 00101010    (bit 3 is already 1)
  Set bit 3:
    00101010 | 00001000 = 00101010 = 42

  No change! Setting is SAFE to repeat. ✓
```

---

## Setting Multiple Bits at Once

```
  Set bits 1, 3, and 5:
  
  mask = (1 << 1) | (1 << 3) | (1 << 5)
       = 00000010 | 00001000 | 00100000
       = 00101010 = 42

  n |= mask;    // Sets all three bits in one operation
```

---

## Pseudocode

```
FUNCTION setBit(n, k):
    RETURN n | (1 << k)

FUNCTION setMultipleBits(n, positions[]):
    mask = 0
    FOR each pos in positions:
        mask = mask | (1 << pos)
    RETURN n | mask
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Set single bit | O(1) | O(1) |
| Set k bits | O(k) to build mask, O(1) to apply | O(1) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Formula | `n \| (1 << k)` |
| Mechanism | OR with 1 forces ON, OR with 0 preserves |
| Idempotent | Setting already-set bit is harmless |
| Multiple bits | Build combined mask, OR once |

---

## Quick Revision Questions

1. **Set bit 6 in n = 15. What's the result?**
   <details><summary>Answer</summary>15 = 00001111, mask = 01000000. 00001111 | 01000000 = 01001111 = 79</details>

2. **What happens if you set a bit that's already 1?**
   <details><summary>Answer</summary>Nothing changes. OR with 1 where bit is already 1 keeps it as 1.</details>

3. **Write a single expression to set bits 0, 2, and 4.**
   <details><summary>Answer</summary>`n | ((1<<0) | (1<<2) | (1<<4))` = `n | 21` = `n | 0b10101`</details>

4. **Why do we use OR instead of AND for setting?**
   <details><summary>Answer</summary>OR with 1 forces a bit ON. AND with 1 would only preserve it (can't turn 0→1 with AND).</details>

5. **Set bit 31 (MSB of 32-bit int). What mask do you use?**
   <details><summary>Answer</summary>`1 << 31` = 0x80000000 = 2,147,483,648 (be careful with signed integers — this sets the sign bit!)</details>

---

[← Previous: Check if Bit is Set](01-check-if-bit-is-set.md) | [Next: Clear a Bit →](03-clear-a-bit.md)
