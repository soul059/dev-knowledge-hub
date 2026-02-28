# Chapter 7.1: Rightmost Set Bit Position

> **Unit 7 — Advanced Bit Operations**
> Find the position of the lowest (rightmost) set bit — the starting point for many bit algorithms.

---

## Overview

The rightmost set bit (also called Least Significant Bit or LSB) is the lowest-order bit that is 1. Finding its **position** is needed for tasks like iterating set bits, fenwick trees, and low-level optimizations.

---

## Methods

### Method 1: n & (-n) — Isolate the Bit

```
  n & (-n) gives a number with ONLY the rightmost set bit.

  n     = 0101 1000
  -n    = 1010 1000   (two's complement = ~n + 1)
  n&-n  = 0000 1000   ← only the rightmost set bit!

  To get the POSITION, take log2 of the result:
  position = log2(n & (-n))
```

### Method 2: __builtin_ctz (Count Trailing Zeros)

```
  Available in GCC/Clang:
  pos = __builtin_ctz(n)     // for 32-bit
  pos = __builtin_ctzll(n)   // for 64-bit

  Java:  Integer.numberOfTrailingZeros(n)
  Python: (n & -n).bit_length() - 1
  C#:    BitOperations.TrailingZeroCount(n)
```

### Method 3: Loop

```
FUNCTION rightmostSetBitPos(n):
    IF n == 0: RETURN -1    // no set bit
    pos = 0
    WHILE (n & 1) == 0:
        n = n >> 1
        pos += 1
    RETURN pos
```

---

## Step-by-Step Trace

### Example: n = 40

```
  n = 40 = 0010 1000

  Method 1: n & (-n)
    -40 = 1101 1000  (two's complement)
    40 & (-40) = 0000 1000 = 8
    position = log2(8) = 3  ✓

  Method 3: Loop
    n = 0010 1000, bit 0 = 0, shift → 0001 0100, pos = 1
    n = 0001 0100, bit 0 = 0, shift → 0000 1010, pos = 2
    n = 0000 1010, bit 0 = 0, shift → 0000 0101, pos = 3
    n = 0000 0101, bit 0 = 1 → STOP, pos = 3  ✓
```

### Example: n = 1

```
  n = 1 = 0000 0001
  n & (-n) = 1 & (-1) = 0001 & 1111 = 0001 = 1
  position = log2(1) = 0  ✓  (bit 0 is set)
```

---

## De Bruijn Method (O(1) without builtins)

```
  When hardware ctz isn't available, use a De Bruijn sequence
  for an O(1) lookup:

  // Magic constant and lookup table
  debruijn = 0x077CB531
  table[32] = {0,1,28,2,29,14,24,3,30,22,20,15,25,17,4,8,
               31,27,13,23,21,19,16,7,26,12,18,6,11,5,10,9}

  FUNCTION ctz(n):
      RETURN table[((n & -n) * debruijn) >> 27]
```

---

## Applications

| Use Case | How |
|----------|-----|
| Fenwick tree (BIT) | `i += i & (-i)` to find parent |
| Iterate set bits | Start from lowest, clear with `n & (n-1)` |
| Event systems | Process lowest-priority event first |
| Hardware interrupts | Service lowest-numbered pending interrupt |

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| n & (-n) + log2 | O(1) | O(1) |
| __builtin_ctz | O(1) — hardware instruction | O(1) |
| Loop | O(k) where k = trailing zeros | O(1) |
| De Bruijn | O(1) | O(32) table |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| n & (-n) | Isolates rightmost set bit (as a value) |
| ctz(n) | Returns the position (count of trailing zeros) |
| Position = log2(n & -n) | Converts isolated bit to position index |
| Edge case | n = 0 has no set bit — undefined/special-cased |

---

## Quick Revision Questions

1. **What is the rightmost set bit position of 96 (0110 0000)?**
   <details><summary>Answer</summary>96 & (-96) = 0010 0000 = 32. Position = log2(32) = 5.</details>

2. **What does n & (-n) return for n = 7 (0111)?**
   <details><summary>Answer</summary>7 & (-7) = 0111 & 1001 = 0001 = 1. The rightmost set bit is at position 0.</details>

3. **How is this used in Fenwick trees?**
   <details><summary>Answer</summary>The operation `i += i & (-i)` adds the value of the lowest set bit, which navigates to the parent in the tree. Similarly, `i -= i & (-i)` (or `i &= i-1`) navigates for range queries.</details>

4. **What happens with __builtin_ctz(0)?**
   <details><summary>Answer</summary>Undefined behavior in C/C++. Always check n != 0 first. Java's numberOfTrailingZeros(0) returns 32.</details>

5. **Can you get the position without log2 or builtins?**
   <details><summary>Answer</summary>Yes. Use the De Bruijn lookup table method: multiply the isolated bit by a De Bruijn constant, shift right, and index into a precomputed table. O(1) time.</details>

---

[← Previous: Iterating Subsets of Mask](../06-Subsets-With-Bits/06-iterating-subsets-of-mask.md) | [Next: Leftmost Set Bit Position →](02-leftmost-set-bit-position.md)
