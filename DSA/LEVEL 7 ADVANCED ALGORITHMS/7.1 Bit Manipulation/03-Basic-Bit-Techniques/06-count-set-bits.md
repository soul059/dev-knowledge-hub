# Chapter 3.6: Count Set Bits (Brian Kernighan's Algorithm)

> **Unit 3 — Basic Bit Techniques**
> Efficiently counting the number of 1-bits (popcount / Hamming weight) in an integer.

---

## Overview

Counting set bits (also called **popcount** or **Hamming weight**) is a fundamental operation used in error correction, data compression, cryptography, and countless algorithms. Brian Kernighan's algorithm is the elegant O(k) solution where k is the number of set bits.

---

## Method 1: Naive — Check Each Bit

```
FUNCTION countBits_naive(n):
    count = 0
    WHILE n > 0:
        count += n & 1       // check LSB
        n = n >> 1           // shift right
    RETURN count
```

### Trace: n = 42 (00101010)

```
  Step │   n (binary)  │ n & 1 │ count │ n >> 1
  ─────┼───────────────┼───────┼───────┼────────────
    1  │ 00101010 (42) │   0   │   0   │ 00010101
    2  │ 00010101 (21) │   1   │   1   │ 00001010
    3  │ 00001010 (10) │   0   │   1   │ 00000101
    4  │ 00000101  (5) │   1   │   2   │ 00000010
    5  │ 00000010  (2) │   0   │   2   │ 00000001
    6  │ 00000001  (1) │   1   │   3   │ 00000000
  
  Result: 3 set bits ✓
  Time: O(total bits) = O(32) for 32-bit int
```

> This always takes O(log n) or O(bit_width) iterations, regardless of how many bits are set.

---

## Method 2: Brian Kernighan's Algorithm

### The Key Insight

```
  n & (n - 1) turns off the RIGHTMOST set bit.
  
  Repeat until n becomes 0 → count the iterations!
  
  ┌───────────────────────────────────────────┐
  │  Each iteration removes exactly ONE set   │
  │  bit. So iterations = number of set bits! │
  └───────────────────────────────────────────┘
```

### Why n & (n-1) Removes the Rightmost Set Bit

```
  n     = XXXX1000    (rightmost set bit at position 3)
  n - 1 = XXXX0111    (borrow flips the 1 and trailing 0s)
  n&(n-1)= XXXX0000   (rightmost 1 is cleared!)
  
  The X bits are unchanged — only the rightmost 1 vanishes.
```

### Pseudocode

```
FUNCTION countBits_kernighan(n):
    count = 0
    WHILE n != 0:
        n = n & (n - 1)     // remove rightmost set bit
        count++
    RETURN count
```

### Step-by-Step Trace: n = 52 (00110100)

```
  ┌─────────────────────────────────────────────────────┐
  │  Initial: n = 52 = 00110100  (3 bits set)          │
  │                                                     │
  │  Iteration 1:                                       │
  │    n     = 00110100                                 │
  │    n-1   = 00110011                                 │
  │    n&(n-1)= 00110000   ← rightmost 1 (pos 2) gone  │
  │    count = 1                                        │
  │                                                     │
  │  Iteration 2:                                       │
  │    n     = 00110000                                 │
  │    n-1   = 00101111                                 │
  │    n&(n-1)= 00100000   ← rightmost 1 (pos 4) gone  │
  │    count = 2                                        │
  │                                                     │
  │  Iteration 3:                                       │
  │    n     = 00100000                                 │
  │    n-1   = 00011111                                 │
  │    n&(n-1)= 00000000   ← rightmost 1 (pos 5) gone  │
  │    count = 3                                        │
  │                                                     │
  │  n == 0 → STOP                                      │
  │  Result: 3 set bits ✓                               │
  │  Only 3 iterations (vs 6 for naive on same number!) │
  └─────────────────────────────────────────────────────┘
```

### Visual: Bits Vanishing One by One

```
  00110100   (3 bits set)
       ↓     n & (n-1)
  00110000   (2 bits set)
       ↓     n & (n-1)
  00100000   (1 bit set)
       ↓     n & (n-1)
  00000000   (0 bits) → STOP! Count = 3
```

---

## Method 3: Lookup Table

For maximum speed, precompute counts for all byte values:

```
  // Precompute: table[i] = number of set bits in byte value i
  table[0] = 0, table[1] = 1, table[2] = 1, table[3] = 2, ...
  table[255] = 8

  FUNCTION countBits_table(n):   // 32-bit integer
      RETURN table[n & 0xFF]
           + table[(n >> 8) & 0xFF]
           + table[(n >> 16) & 0xFF]
           + table[(n >> 24) & 0xFF]
```

---

## Method 4: Parallel Bit Counting (Divide and Conquer)

```
  // Count bits in a 32-bit integer — O(1) with 5 operations
  
  FUNCTION popcount(n):
      n = n - ((n >> 1) & 0x55555555)           // pairs
      n = (n & 0x33333333) + ((n >> 2) & 0x33333333)  // nibbles
      n = (n + (n >> 4)) & 0x0F0F0F0F           // bytes
      RETURN (n * 0x01010101) >> 24              // sum all bytes

  This is what __builtin_popcount() uses internally!
```

---

## Complexity Comparison

```
  ┌──────────────────────┬────────────┬───────┬──────────────────┐
  │ Method               │ Time       │ Space │ Notes            │
  ├──────────────────────┼────────────┼───────┼──────────────────┤
  │ Naive (shift)        │ O(log n)   │ O(1)  │ Always 32 iters  │
  │                      │ = O(32)    │       │ for 32-bit       │
  │ Kernighan            │ O(k)       │ O(1)  │ k = set bits     │
  │                      │            │       │ Best when sparse │
  │ Lookup table         │ O(1)       │ O(256)│ 4 table lookups  │
  │ Parallel/popcount    │ O(1)       │ O(1)  │ Hardware support │
  └──────────────────────┴────────────┴───────┴──────────────────┘
```

---

## Applications of Popcount

```
  ╔══════════════════════════════════════════════════════╗
  ║  WHERE POPCOUNT IS USED                              ║
  ╠══════════════════════════════════════════════════════╣
  ║                                                      ║
  ║  1. Hamming Distance: popcount(a ^ b)                ║
  ║     Count of positions where two values differ       ║
  ║                                                      ║
  ║  2. Subset Size: popcount(mask)                      ║
  ║     Count elements in a bitmask-represented set      ║
  ║                                                      ║
  ║  3. Election/Majority: More than n/2 bits set?       ║
  ║                                                      ║
  ║  4. Error Detection: Count parity bits               ║
  ║                                                      ║
  ║  5. Chess: Count pieces on bitboard                   ║
  ╚══════════════════════════════════════════════════════╝
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Popcount / Hamming weight | Number of 1-bits |
| Naive method | Shift right and check LSB each step: O(log n) |
| Kernighan's trick | `n &= (n-1)` removes rightmost 1: O(k) |
| Lookup table | Precompute for bytes: O(1) time, O(256) space |
| Parallel counting | Divide-and-conquer bit tricks: O(1) |
| Built-in | `__builtin_popcount()` in C/C++, `Integer.bitCount()` in Java |

---

## Quick Revision Questions

1. **How many set bits does 255 have?**
   <details><summary>Answer</summary>8 — 255 = 11111111₂, all bits set</details>

2. **How many iterations does Kernighan's algorithm take for n = 240?**
   <details><summary>Answer</summary>240 = 11110000₂ → 4 iterations (one per set bit)</details>

3. **What does n & (n-1) do?**
   <details><summary>Answer</summary>Turns off (clears) the rightmost set bit of n</details>

4. **What is the Hamming distance between 13 and 10?**
   <details><summary>Answer</summary>13 ^ 10 = 1101 ^ 1010 = 0111 = 7. popcount(7) = 3. Distance = 3 bits differ.</details>

5. **Why is Kernighan's better than naive for sparse numbers?**
   <details><summary>Answer</summary>Kernighan iterates only k times (k = set bits). For n = 1024 (one bit set), it takes 1 iteration vs 11 for naive.</details>

6. **What hardware instruction does popcount map to?**
   <details><summary>Answer</summary>POPCNT instruction on x86 CPUs (SSE4.2). Counts bits in one cycle.</details>

---

[← Previous: Check Power of 2](05-check-power-of-2.md) | [Next Unit: Bit Manipulation Tricks →](../04-Bit-Manipulation-Tricks/01-swap-without-temp.md)
