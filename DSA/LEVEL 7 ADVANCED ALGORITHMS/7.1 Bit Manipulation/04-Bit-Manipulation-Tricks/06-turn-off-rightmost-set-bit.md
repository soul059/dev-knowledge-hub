# Chapter 4.6: Turn Off the Rightmost Set Bit

> **Unit 4 — Bit Manipulation Tricks**
> The `n & (n - 1)` trick — one of the most powerful single expressions in bit manipulation.

---

## Overview

The expression `n & (n - 1)` removes the **rightmost (lowest) set bit** of n. It's the foundation of Kernighan's bit-counting algorithm and the power-of-2 check.

---

## How It Works

```
  ┌──────────────────────────────────────────────────────┐
  │  Subtracting 1 from n:                               │
  │    - Flips the rightmost set bit to 0                │
  │    - Flips all 0s below it to 1                      │
  │    - Leaves all bits above unchanged                 │
  │                                                      │
  │  AND-ing n with (n-1):                               │
  │    - The rightmost set bit was 1 in n, 0 in (n-1)   │
  │    - The bits below were 0 in n, 1 in (n-1)         │
  │    - Both groups → 0 after AND                       │
  │    - The bits above → unchanged                      │
  └──────────────────────────────────────────────────────┘
```

---

## Visual Proof

```
  n        = xxxx 1 000   (rightmost set bit at position k)
                 ↑
                 rightmost set bit
  
  n - 1    = xxxx 0 111   (bit k flipped, below filled with 1s)
  
  n & (n-1)= xxxx 0 000   ← rightmost set bit removed!
                 ─────
                 all cleared
```

---

## Step-by-Step Trace

### Example 1: n = 52

```
  n     = 0011 0100   (52, rightmost set bit at position 2)
  n - 1 = 0011 0011   (51)
  
  AND:
        0011 0100
      & 0011 0011
      ──────────
        0011 0000   = 48

  Bit 2 was cleared. All other bits unchanged. ✓
```

### Example 2: n = 40

```
  n     = 0010 1000   (40, rightmost set bit at position 3)
  n - 1 = 0010 0111   (39)
  
  AND:
        0010 1000
      & 0010 0111
      ──────────
        0010 0000   = 32

  Bit 3 was cleared. ✓
```

### Example 3: n = 1 (only one bit)

```
  n     = 0000 0001   (1)
  n - 1 = 0000 0000   (0)
  
  AND:
        0000 0001
      & 0000 0000
      ──────────
        0000 0000   = 0

  Single bit cleared → result is 0. This is why n&(n-1)==0 means power of 2!
```

---

## Applications of n & (n-1)

### 1. Check Power of 2

```
  n > 0 AND n & (n-1) == 0
  // Power of 2 has exactly one set bit
  // Clearing it gives 0
```

### 2. Count Set Bits (Kernighan's Algorithm)

```
  count = 0
  WHILE n > 0:
      n = n & (n - 1)    // remove one set bit
      count += 1
  RETURN count
  // Loops exactly (number of set bits) times
```

### 3. Check if n is a Power of 2 Minus 1

```
  n & (n + 1) == 0   // e.g., 7 (0111), 15 (01111), 31 (011111)
  // All bits below some position are set
```

---

## Related: Isolate the Rightmost Set Bit

```
  n & (-n)   or   n & ~(n-1)

  n      = 0011 0100
  -n     = 1100 1100   (two's complement)
  
  n & -n = 0000 0100   ← only the rightmost set bit remains!
```

This is the complementary operation: instead of removing it, you isolate it.

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| n & (n-1) | O(1) | O(1) |
| Kernighan count | O(k) where k = set bits | O(1) |

---

## Summary Table

| Expression | Effect |
|------------|--------|
| `n & (n-1)` | Turn off rightmost set bit |
| `n & (-n)` | Isolate rightmost set bit |
| `n \| (n+1)` | Turn on rightmost zero bit |
| `n & (n+1)` | Turn off trailing ones |

---

## Quick Revision Questions

1. **What is 12 & 11?**
   <details><summary>Answer</summary>12 = 1100, 11 = 1011. 1100 & 1011 = 1000 = 8. Rightmost set bit (bit 2) cleared.</details>

2. **How many times does the Kernighan loop execute for n = 255?**
   <details><summary>Answer</summary>8 times. 255 = 1111 1111 has 8 set bits. Each iteration clears one.</details>

3. **What is 80 & 79?**
   <details><summary>Answer</summary>80 = 0101 0000, 79 = 0100 1111. AND = 0100 0000 = 64. Bit 4 cleared.</details>

4. **How do you isolate the rightmost set bit of n = 40?**
   <details><summary>Answer</summary>40 & (-40). 40 = 0010 1000, -40 = 1101 1000. AND = 0000 1000 = 8. Bit 3 isolated.</details>

5. **Why is n & (n-1) faster than checking each bit for counting set bits?**
   <details><summary>Answer</summary>The naive approach always iterates 32 times (or log n). Kernighan's method iterates exactly k times where k is the number of set bits. For sparse numbers (few 1s), this is much faster.</details>

6. **What happens if n = 0?**
   <details><summary>Answer</summary>0 & (0-1) = 0 & 0xFFFFFFFF = 0 (in unsigned). It's a no-op since there's no set bit to clear. Kernighan's loop wouldn't enter.</details>

---

[← Previous: Next Power of 2](05-next-power-of-2.md) | [Next: Unit 5 — XOR Properties →](../05-XOR-Applications/01-xor-properties.md)
