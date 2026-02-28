# Chapter 1.1: Binary Representation

> **Unit 1 — Binary Number System**
> Understanding how computers store and represent data at the most fundamental level.

---

## Overview

Every digital computer is built from billions of tiny switches (transistors) that can be either **ON** (1) or **OFF** (0). This two-state system forms the basis of the **binary number system** — a base-2 number system that uses only the digits 0 and 1.

Understanding binary representation is the **foundation** of all bit manipulation. Before you can twist, flip, or mask individual bits, you must understand what those bits mean.

---

## What is a Bit?

A **bit** (binary digit) is the smallest unit of data in computing.

```
  ┌─────────────────────────────────────────┐
  │            A SINGLE BIT                 │
  │                                         │
  │    ┌───┐         ┌───┐                  │
  │    │ 0 │   or    │ 1 │                  │
  │    └───┘         └───┘                  │
  │   OFF/False     ON/True                 │
  │   Low Voltage   High Voltage            │
  │   No Current    Current Flowing         │
  └─────────────────────────────────────────┘
```

---

## Grouping Bits

Bits are grouped into larger units:

```
  1 bit      →   0 or 1
  
  4 bits     →   Nibble    →   0000 to 1111        (0 to 15)
  
  8 bits     →   Byte      →   00000000 to 11111111 (0 to 255)
  
  16 bits    →   Half Word →   0 to 65,535
  
  32 bits    →   Word      →   0 to 4,294,967,295
  
  64 bits    →   Double    →   0 to 18,446,744,073,709,551,615
                   Word
```

### Visual: One Byte (8 bits)

```
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │ Bit │ Bit │ Bit │ Bit │ Bit │ Bit │ Bit │ Bit │
  │  7  │  6  │  5  │  4  │  3  │  2  │  1  │  0  │
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │  1  │  0  │  1  │  0  │  0  │  1  │  1  │  0  │   ← Example: 166
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    128    64    32    16     8     4     2     1     ← Place values (powers of 2)
    MSB                                         LSB
  (Most                                      (Least
  Significant                              Significant
    Bit)                                      Bit)
```

---

## Binary Number System — The Theory

### Place Value System

Just as decimal uses powers of 10, binary uses **powers of 2**:

```
  DECIMAL (Base 10):
  ──────────────────
   4   2   5
   │   │   └── 5 × 10⁰ = 5 × 1   =   5
   │   └────── 2 × 10¹ = 2 × 10   =  20
   └────────── 4 × 10² = 4 × 100  = 400
                                     ───
                              Total = 425

  BINARY (Base 2):
  ────────────────
   1   1   0   1   0   1
   │   │   │   │   │   └── 1 × 2⁰ = 1 × 1  =  1
   │   │   │   │   └────── 0 × 2¹ = 0 × 2  =  0
   │   │   │   └────────── 1 × 2² = 1 × 4  =  4
   │   │   └────────────── 0 × 2³ = 0 × 8  =  0
   │   └────────────────── 1 × 2⁴ = 1 × 16 = 16
   └────────────────────── 1 × 2⁵ = 1 × 32 = 32
                                              ──
                                       Total = 53
   So: 110101₂ = 53₁₀
```

---

## Powers of 2 — Must Memorize!

```
  ┌────────┬────────┬───────────────┐
  │ Power  │ Value  │ Common Name   │
  ├────────┼────────┼───────────────┤
  │  2⁰    │      1 │               │
  │  2¹    │      2 │               │
  │  2²    │      4 │               │
  │  2³    │      8 │               │
  │  2⁴    │     16 │ Nibble max+1  │
  │  2⁵    │     32 │               │
  │  2⁶    │     64 │               │
  │  2⁷    │    128 │               │
  │  2⁸    │    256 │ Byte max+1    │
  │  2⁹    │    512 │               │
  │  2¹⁰   │   1024 │ 1 KB (approx) │
  │  2¹⁶   │  65536 │ 64 KB         │
  │  2²⁰   │  ~1 M  │ 1 MB          │
  │  2³⁰   │  ~1 B  │ 1 GB          │
  │  2³²   │  ~4 B  │ 32-bit limit  │
  └────────┴────────┴───────────────┘
```

---

## How Computers Store Integers

### Unsigned Integers (always non-negative)

With **n bits**, we can represent values from **0** to **2ⁿ − 1**.

```
  n = 8 bits (1 byte)
  ┌────────────────────────────────────────┐
  │  Range: 0 to 255   (2⁸ − 1 = 255)    │
  │                                        │
  │  00000000₂ =   0₁₀  (minimum)         │
  │  00000001₂ =   1₁₀                    │
  │  01111111₂ = 127₁₀                    │
  │  10000000₂ = 128₁₀                    │
  │  11111111₂ = 255₁₀  (maximum)         │
  └────────────────────────────────────────┘
```

### Number Line for 4-bit Unsigned
```
  0    1    2    3    4    5    6    7    8    9   10   11   12   13   14   15
  │────│────│────│────│────│────│────│────│────│────│────│────│────│────│────│
 0000 0001 0010 0011 0100 0101 0110 0111 1000 1001 1010 1011 1100 1101 1110 1111
```

---

## Binary Representation of Common Values

```
  ┌──────────┬───────────┬──────────┐
  │ Decimal  │  Binary   │  Hex     │
  ├──────────┼───────────┼──────────┤
  │    0     │ 0000 0000 │   0x00   │
  │    1     │ 0000 0001 │   0x01   │
  │    2     │ 0000 0010 │   0x02   │
  │    5     │ 0000 0101 │   0x05   │
  │   10     │ 0000 1010 │   0x0A   │
  │   15     │ 0000 1111 │   0x0F   │
  │   16     │ 0001 0000 │   0x10   │
  │   42     │ 0010 1010 │   0x2A   │
  │  100     │ 0110 0100 │   0x64   │
  │  127     │ 0111 1111 │   0x7F   │
  │  128     │ 1000 0000 │   0x80   │
  │  255     │ 1111 1111 │   0xFF   │
  └──────────┴───────────┴──────────┘
```

---

## Hexadecimal — Binary's Best Friend

Hexadecimal (base 16) is used because each hex digit maps **exactly** to 4 binary bits:

```
  ┌──────┬────────┐    ┌──────┬────────┐
  │ Hex  │ Binary │    │ Hex  │ Binary │
  ├──────┼────────┤    ├──────┼────────┤
  │  0   │  0000  │    │  8   │  1000  │
  │  1   │  0001  │    │  9   │  1001  │
  │  2   │  0010  │    │  A   │  1010  │
  │  3   │  0011  │    │  B   │  1011  │
  │  4   │  0100  │    │  C   │  1100  │
  │  5   │  0101  │    │  D   │  1101  │
  │  6   │  0110  │    │  E   │  1110  │
  │  7   │  0111  │    │  F   │  1111  │
  └──────┴────────┘    └──────┴────────┘

  Example:  0xA3  →  1010 0011
                     ││││ ││││
                     A=10  3=3
                     
            1010 0011₂ = 163₁₀
```

---

## Real-World Applications

1. **Memory Addressing** — RAM addresses are binary numbers
2. **Character Encoding** — ASCII uses 7 bits per character (A = 01000001)
3. **Color Representation** — RGB colors use 8 bits per channel (24 bits total)
4. **Network Protocols** — IP addresses are 32-bit binary numbers
5. **File Formats** — Every file is a sequence of bytes (8-bit groups)

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Bit | Smallest unit: 0 or 1 |
| Byte | 8 bits, range 0–255 |
| Binary system | Base-2, uses powers of 2 |
| Place values | 2⁰, 2¹, 2², 2³, ... from right to left |
| MSB | Most Significant Bit (leftmost) |
| LSB | Least Significant Bit (rightmost) |
| n-bit range (unsigned) | 0 to 2ⁿ − 1 |
| Hexadecimal | Base-16, each hex digit = 4 bits |

---

## Quick Revision Questions

1. **How many distinct values can 6 bits represent?**
   <details><summary>Answer</summary>2⁶ = 64 values (0 to 63)</details>

2. **What is the binary representation of decimal 42?**
   <details><summary>Answer</summary>101010₂ (32 + 8 + 2 = 42)</details>

3. **What is the maximum unsigned value in 16 bits?**
   <details><summary>Answer</summary>2¹⁶ − 1 = 65,535</details>

4. **Convert hexadecimal 0xFF to decimal.**
   <details><summary>Answer</summary>15 × 16 + 15 = 255</details>

5. **What is the MSB of the byte 01011010?**
   <details><summary>Answer</summary>0 (the leftmost bit)</details>

6. **How many bytes are needed to represent a 32-bit integer?**
   <details><summary>Answer</summary>4 bytes (32 ÷ 8 = 4)</details>

---

[← Back to README](../README.md) | [Next: Decimal to Binary →](02-decimal-to-binary.md)
