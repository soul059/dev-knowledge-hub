# Chapter 9.4: Error Detection

> **Unit 9 — Practical Applications**
> Use parity bits, checksums, and CRC to detect data corruption through bit-level operations.

---

## Overview

When data travels across networks or sits on disk, bits can flip. Error detection schemes add redundancy so corrupted data can be identified. All these schemes are built on bitwise operations.

---

## Technique 1: Parity Bits

```
  Add 1 extra bit so the total number of 1-bits is even (even parity)
  or odd (odd parity).

  Data:    1011001  (four 1-bits → even)
  Even parity bit: 0  → total 1-bits = 4 (even)  ✓
  Transmitted: 10110010

  Data:    1011101  (five 1-bits → odd)
  Even parity bit: 1  → total 1-bits = 6 (even)  ✓
  Transmitted: 10111011
```

### Computing Parity

```
FUNCTION parityBit(data):
    parity = 0
    WHILE data:
        parity ^= 1          // flip for each set bit
        data &= (data - 1)   // clear lowest set bit
    RETURN parity

// Alternatively, using popcount:
FUNCTION parityBit(data):
    RETURN popcount(data) & 1
```

### Trace: data = 0b1011001

```
  popcount(1011001) = 4
  4 & 1 = 0
  Even parity bit = 0  ✓
```

### XOR-based Parity for Bytes

```
  // Parity of a byte using XOR folding:
  FUNCTION byteParity(b):
      b ^= (b >> 4)    // fold upper 4 into lower 4
      b ^= (b >> 2)    // fold into 2 bits
      b ^= (b >> 1)    // fold into 1 bit
      RETURN b & 1

  Trace: b = 0b10110010
    b ^= b>>4: 10110010 ^ 00001011 = 10111001
    b ^= b>>2: 10111001 ^ 00101110 = 10010111
    b ^= b>>1: 10010111 ^ 01001011 = 11011100
    b & 1:     0
    Parity = 0 (even number of 1-bits)  ✓
```

### Limitation

```
  Parity detects:     odd number of bit flips
  Parity MISSES:      even number of bit flips (e.g., 2 flips)
  Detection rate:     ~50% of all errors
```

---

## Technique 2: Checksums

```
  Sum all data units and append the sum.
  Receiver recomputes and compares.

  Internet Checksum (RFC 1071):
  1. Split data into 16-bit words
  2. Sum with ones' complement addition
  3. Take ones' complement of the sum
```

### Ones' Complement Addition

```
FUNCTION onesComplementAdd(a, b):
    sum = a + b
    // Wrap carry around
    WHILE sum > 0xFFFF:
        sum = (sum & 0xFFFF) + (sum >> 16)
    RETURN sum

FUNCTION internetChecksum(data[], length):
    sum = 0
    FOR i = 0 TO length-1 STEP 2:
        word = (data[i] << 8) | data[i+1]
        sum = onesComplementAdd(sum, word)
    RETURN ~sum & 0xFFFF    // ones' complement
```

### Trace

```
  Data: [0x45, 0x00, 0x00, 0x3C]

  Word 1: 0x4500
  Word 2: 0x003C

  Sum: 0x4500 + 0x003C = 0x453C
  Checksum: ~0x453C = 0xBAC3

  Verification: 0x4500 + 0x003C + 0xBAC3 = 0xFFFF  ✓
  (ones' complement sum of data + checksum = all 1s)
```

---

## Technique 3: CRC (Cyclic Redundancy Check)

```
  Treats data as a polynomial over GF(2) (binary field).
  Divides by a generator polynomial using XOR.
  Remainder = CRC value.

  Much stronger than parity or simple checksums!
```

### CRC Computation

```
  Data:      1101011011   (M)
  Generator: 10011        (G), degree 4 → CRC is 4 bits

  Step 1: Append 4 zero bits to data
           11010110110000

  Step 2: XOR-divide by generator

  11010110110000
  10011
  ─────
   10011
   10011
   ─────
    00001011
        10011
        ─────
         00100000
           10011
           ─────
            01100  ← Remainder = CRC
```

### CRC Algorithm (Bit-by-bit)

```
FUNCTION crc(data, dataLen, generator, crcLen):
    // Shift data left by crcLen bits (append zeros)
    register = 0
    FOR i = 0 TO dataLen + crcLen - 1:
        // Shift in next bit
        IF i < dataLen:
            bit = (data >> (dataLen - 1 - i)) & 1
        ELSE:
            bit = 0
        register = (register << 1) | bit

        // If MSB is 1, XOR with generator
        IF register & (1 << crcLen):
            register ^= generator
    RETURN register    // This is the CRC
```

### Common CRC Polynomials

```
  ┌─────────────┬─────────────────────┬────────────┐
  │ Name        │ Polynomial          │ Hex        │
  ├─────────────┼─────────────────────┼────────────┤
  │ CRC-8       │ x^8+x^2+x+1        │ 0x07       │
  │ CRC-16-CCITT│ x^16+x^12+x^5+1    │ 0x1021     │
  │ CRC-32      │ x^32+x^26+...+1    │ 0x04C11DB7 │
  └─────────────┴─────────────────────┴────────────┘
```

---

## Hamming Code (Error Correction)

```
  Goes beyond detection — can also CORRECT single-bit errors.

  Parity bits at positions that are powers of 2: 1, 2, 4, 8, ...
  Each parity bit covers specific data positions.

  Position:  1  2  3  4  5  6  7
  Type:      p1 p2 d1 p3 d2 d3 d4
  
  p1 covers: positions with bit 0 set: 1,3,5,7
  p2 covers: positions with bit 1 set: 2,3,6,7
  p3 covers: positions with bit 2 set: 4,5,6,7
```

### Error Detection and Correction

```
  // Check each parity group
  syndrome = 0
  FOR i = 0 TO numParityBits-1:
      parity = 0
      FOR each position j where bit i of j is set:
          parity ^= data[j]
      IF parity != 0:
          syndrome |= (1 << i)

  // syndrome is the position of the error (0 = no error)
  IF syndrome != 0:
      data[syndrome] ^= 1    // flip the erroneous bit
```

---

## Comparison of Error Detection Methods

```
  ┌────────────────┬───────────┬──────────┬──────────────────┐
  │ Method         │ Overhead  │ Detects  │ Corrects         │
  ├────────────────┼───────────┼──────────┼──────────────────┤
  │ Parity bit     │ 1 bit     │ 1-bit    │ No               │
  │ Checksum       │ 16-32 bit │ Most     │ No               │
  │ CRC-32         │ 32 bits   │ Burst≤32 │ No               │
  │ Hamming(7,4)   │ 3 bits/4  │ 2-bit    │ 1-bit            │
  └────────────────┴───────────┴──────────┴──────────────────┘
```

---

## Quick Revision Questions

1. **How does parity detect errors?**
   <details><summary>Answer</summary>A parity bit ensures the total number of 1-bits is even (or odd). If a single bit flips, the parity changes, signaling an error. It uses XOR, since XOR of all bits gives the parity.</details>

2. **Why does parity miss double-bit errors?**
   <details><summary>Answer</summary>Two flipped bits change the parity twice, returning it to its original value. The total number of 1-bits changes by 0 or ±2, which preserves even/odd parity.</details>

3. **What mathematical operation underlies CRC?**
   <details><summary>Answer</summary>Polynomial division over GF(2), the binary Galois field. XOR serves as both addition and subtraction in GF(2). The CRC is the remainder of this division.</details>

4. **How does Hamming code locate the error position?**
   <details><summary>Answer</summary>Each parity bit checks a specific subset of positions (those with a particular bit set in their index). The syndrome — the binary number formed by the failing parity checks — directly gives the position of the erroneous bit.</details>

5. **Why is CRC-32 preferred over simple checksums for file integrity?**
   <details><summary>Answer</summary>CRC-32 detects all burst errors up to 32 bits, all odd-count bit errors, and most other error patterns. Simple checksums miss many error patterns, especially those that cancel out in the sum (e.g., two bytes swapped).</details>

---

[← Previous: Compression Basics](03-compression-basics.md) | [Next: Bloom Filter Bits →](05-bloom-filter-bits.md)
