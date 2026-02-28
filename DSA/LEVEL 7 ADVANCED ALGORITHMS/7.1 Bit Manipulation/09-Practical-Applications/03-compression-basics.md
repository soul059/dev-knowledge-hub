# Chapter 9.3: Compression Basics

> **Unit 9 — Practical Applications**
> Understand how bit-level manipulation powers data compression — from fixed-width encoding to Huffman trees.

---

## Overview

Compression reduces data size by encoding information using fewer bits. All compression algorithms ultimately work at the bit level, making bit manipulation an essential tool.

---

## Why Bit-Level Operations?

```
  ASCII uses 8 bits per character, but:
  
  'A'-'Z' = 26 letters → need only 5 bits (2^5 = 32)
  Digits '0'-'9' = 10  → need only 4 bits (2^4 = 16)
  Boolean values        → need only 1 bit

  ASCII 'A':  01000001  (8 bits, wastes 3 bits)
  Packed 'A': 00000     (5 bits, no waste)
```

---

## Technique 1: Bit Packing

```
  Store multiple small values in a single integer.

  Pack four 3-bit values into one 12-bit integer:
  
  Values: 5(101), 3(011), 7(111), 2(010)
  
  Packed:  101 011 111 010
           ─── ─── ─── ───
           v0  v1  v2  v3

  Storage: 4 bytes → 2 bytes (50% savings)
```

### Pack/Unpack Operations

```
FUNCTION pack(value, packed, position, width):
    // Clear target bits, then set them
    mask = ((1 << width) - 1) << position
    packed = packed & (~mask)
    packed = packed | ((value & ((1 << width) - 1)) << position)
    RETURN packed

FUNCTION unpack(packed, position, width):
    RETURN (packed >> position) & ((1 << width) - 1)
```

### Trace: Pack values 5, 3, 7, 2 (3-bit each)

```
  Start: packed = 0

  Pack 5 at position 0: packed = 0 | (5 << 0) = 000000000101
  Pack 3 at position 3: packed = 101 | (3 << 3) = 000000011101 → wait
  
  Step by step:
    packed = 000000000101
    3 << 3 = 000000011000
    OR:      000000011101

  Pack 7 at position 6:
    7 << 6 = 000111000000
    OR:      000111011101

  Pack 2 at position 9:
    2 << 9 = 010000000000
    OR:      010111011101

  Unpack position 6, width 3:
    010111011101 >> 6 = 000000010111
    & 0b111           = 000000000111 = 7  ✓
```

---

## Technique 2: Variable-Length Encoding

```
  Frequent symbols → short codes
  Rare symbols     → long codes

  Fixed-width:   A=00, B=01, C=10, D=11  (always 2 bits)
  Variable:      A=0,  B=10, C=110, D=111

  Message "AAAAB":
    Fixed:    00 00 00 00 01           = 10 bits
    Variable: 0  0  0  0  10          = 6 bits
    Savings: 40%

  Key requirement: no code is a prefix of another (prefix-free)
```

---

## Technique 3: Huffman Encoding

```
  Build optimal variable-length codes based on frequency.

  Input frequencies: A=45, B=13, C=12, D=16, E=9, F=5

  Step 1: Build priority queue (min-heap by frequency)
  Step 2: Repeatedly merge two smallest nodes
  Step 3: Assign 0/1 to left/right edges
```

### Huffman Tree Construction

```
  Frequencies: A(45) B(13) C(12) D(16) E(9) F(5)

  Merge F(5)+E(9) → FE(14)
  Merge C(12)+B(13) → CB(25)
  Merge FE(14)+D(16) → FED(30)
  Merge CB(25)+FED(30) → CBFED(55)
  Merge A(45)+CBFED(55) → root(100)

             (100)
            /     \
          A(45)  (55)
                /    \
             (25)    (30)
            /   \   /    \
          C(12) B(13) (14) D(16)
                     /   \
                   F(5)  E(9)

  Codes:
    A = 0        (1 bit)   ← most frequent
    C = 100      (3 bits)
    B = 101      (3 bits)
    F = 1100     (4 bits)
    E = 1101     (4 bits)  ← least frequent
    D = 111      (3 bits)
```

### Weighted Path Length

```
  Average bits = Σ (frequency × code_length) / total
  = (45×1 + 12×3 + 13×3 + 5×4 + 9×4 + 16×3) / 100
  = (45 + 36 + 39 + 20 + 36 + 48) / 100
  = 224 / 100
  = 2.24 bits per symbol (vs. 3 bits fixed for 6 symbols)
```

---

## Technique 4: Run-Length Encoding (RLE) at Bit Level

```
  Encode runs of identical bits.

  Input:  11111 000 1111111 00 111
  Counts: 5     3   7       2   3
  
  Encode counts in binary:
  Original: 20 bits
  RLE:      5 numbers → can be stored more compactly

  Works best for data with long runs (e.g., fax images).
```

---

## Bitstream I/O

```
  // Write bits to a byte buffer
  FUNCTION writeBits(buffer, bitPos, value, numBits):
      FOR i = numBits-1 DOWN TO 0:
          bit = (value >> i) & 1
          byteIndex = bitPos / 8
          bitIndex  = 7 - (bitPos % 8)    // MSB first
          IF bit:
              buffer[byteIndex] |= (1 << bitIndex)
          ELSE:
              buffer[byteIndex] &= ~(1 << bitIndex)
          bitPos += 1
      RETURN bitPos

  // Read bits from a byte buffer
  FUNCTION readBits(buffer, bitPos, numBits):
      value = 0
      FOR i = 0 TO numBits-1:
          byteIndex = bitPos / 8
          bitIndex  = 7 - (bitPos % 8)
          bit = (buffer[byteIndex] >> bitIndex) & 1
          value = (value << 1) | bit
          bitPos += 1
      RETURN value, bitPos
```

---

## Summary Table

| Technique | Best For | Savings |
|-----------|----------|---------|
| Bit packing | Fixed small values | Up to 8× |
| Variable-length encoding | Known frequency distribution | 20-50% |
| Huffman coding | Optimal prefix-free encoding | Near-entropy |
| Run-length encoding | Long runs of same bit/byte | Varies widely |

---

## Quick Revision Questions

1. **Why is bit packing useful even without compression?**
   <details><summary>Answer</summary>Bit packing reduces memory usage by storing multiple small values in one integer. For example, four 3-bit values fit in a single 12-bit field instead of four separate bytes. This also improves cache performance.</details>

2. **What makes Huffman coding "optimal"?**
   <details><summary>Answer</summary>Huffman coding produces the optimal prefix-free code — no other prefix-free encoding can achieve a shorter average code length for the given symbol frequencies. It approaches the theoretical entropy limit.</details>

3. **Why must variable-length codes be prefix-free?**
   <details><summary>Answer</summary>If code A is a prefix of code B, the decoder can't tell when A ends — it might be the start of B. Prefix-free codes allow unambiguous decoding by reading bit by bit until a complete code is found.</details>

4. **How do you extract a 5-bit value starting at bit position 11?**
   <details><summary>Answer</summary>Shift right by the position and mask: `(data >> 11) & 0x1F` where 0x1F = (1 << 5) - 1 = 0b11111. This isolates exactly 5 bits starting from position 11.</details>

5. **When does run-length encoding perform poorly?**
   <details><summary>Answer</summary>When the data has frequent alternations (e.g., 010101...). Each run has length 1, so the RLE overhead (storing count per run) actually increases the data size. RLE is best for long runs of identical values.</details>

---

[← Previous: Permission Systems](02-permission-systems.md) | [Next: Error Detection →](04-error-detection.md)
