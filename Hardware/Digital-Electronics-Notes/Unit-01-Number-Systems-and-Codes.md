# Unit 1: Number Systems and Codes

---

## Table of Contents
1. [Introduction to Number Systems](#1-introduction-to-number-systems)
2. [Binary Number System](#2-binary-number-system)
3. [Octal Number System](#3-octal-number-system)
4. [Hexadecimal Number System](#4-hexadecimal-number-system)
5. [Number Conversions](#5-number-conversions)
6. [Binary Arithmetic](#6-binary-arithmetic)
7. [Signed Number Representation](#7-signed-number-representation)
8. [Binary Codes](#8-binary-codes)
9. [Error Detection Codes](#9-error-detection-codes)
10. [Summary Tables](#10-summary-tables)
11. [Quick Revision Questions](#11-quick-revision-questions)

---

## 1. Introduction to Number Systems

### What is a Number System?

A **number system** is a mathematical notation for representing numbers using a set of digits or symbols. Each number system has:

- **Base (Radix)**: The number of unique digits used
- **Digits**: Symbols used to represent values
- **Positional Notation**: Each position has a weight based on the base

```
┌─────────────────────────────────────────────────────────────────┐
│                     COMMON NUMBER SYSTEMS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Binary (Base-2)      →  Digits: 0, 1                         │
│   Octal (Base-8)       →  Digits: 0, 1, 2, 3, 4, 5, 6, 7       │
│   Decimal (Base-10)    →  Digits: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 │
│   Hexadecimal (Base-16)→  Digits: 0-9, A, B, C, D, E, F        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Why Different Number Systems?

| System | Primary Use |
|--------|-------------|
| Binary | Internal computer operations (transistors have 2 states) |
| Octal | Compact binary representation (legacy systems) |
| Decimal | Human-readable numbers |
| Hexadecimal | Memory addresses, color codes, compact binary |

### Positional Notation

In any positional number system with base `r`:

```
Number = dₙrⁿ + dₙ₋₁rⁿ⁻¹ + ... + d₁r¹ + d₀r⁰ + d₋₁r⁻¹ + d₋₂r⁻² + ...

Where:
  - d = digit at position
  - r = radix (base)
  - n = position (0 for rightmost integer digit)
```

**Example (Decimal 1234.56):**
```
1234.56₁₀ = 1×10³ + 2×10² + 3×10¹ + 4×10⁰ + 5×10⁻¹ + 6×10⁻²
          = 1000 + 200 + 30 + 4 + 0.5 + 0.06
          = 1234.56
```

---

## 2. Binary Number System

### Fundamentals

**Base**: 2  
**Digits**: 0 and 1 (called **bits** - binary digits)

```
┌─────────────────────────────────────────────────────────────────┐
│                    BINARY NUMBER STRUCTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│      Position:    7    6    5    4    3    2    1    0         │
│      Weight:     128   64   32   16   8    4    2    1         │
│                  2⁷   2⁶   2⁵   2⁴   2³   2²   2¹   2⁰         │
│                                                                 │
│      Example:     1    0    1    1    0    1    0    1         │
│                  128 + 0  + 32 + 16 + 0  + 4  + 0  + 1 = 181   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Binary Terminology

| Term | Definition | Example |
|------|------------|---------|
| Bit | Single binary digit | 0 or 1 |
| Nibble | 4 bits | 1010 |
| Byte | 8 bits | 10110101 |
| Word | System-dependent (16/32/64 bits) | Varies |
| MSB | Most Significant Bit (leftmost) | **1**0110101 |
| LSB | Least Significant Bit (rightmost) | 1011010**1** |

### Powers of 2 (Memorize These!)

```
┌────────────────────────────────────────────┐
│           POWERS OF 2 TABLE                │
├──────┬───────────┬────────────────────────┤
│  2⁰  │     1     │                        │
│  2¹  │     2     │                        │
│  2²  │     4     │                        │
│  2³  │     8     │                        │
│  2⁴  │    16     │                        │
│  2⁵  │    32     │                        │
│  2⁶  │    64     │                        │
│  2⁷  │   128     │                        │
│  2⁸  │   256     │ 1 byte max + 1         │
│  2⁹  │   512     │                        │
│  2¹⁰ │  1024     │ 1 KB (Kilo)            │
│  2¹⁶ │ 65536     │ 64 KB                  │
│  2²⁰ │ 1048576   │ 1 MB (Mega)            │
│  2³⁰ │ ~1 Billion│ 1 GB (Giga)            │
└──────┴───────────┴────────────────────────┘
```

### Fractional Binary Numbers

Binary fractions work with negative powers of 2:

```
Position:    -1     -2     -3     -4
Weight:     0.5   0.25  0.125  0.0625
             2⁻¹    2⁻²    2⁻³    2⁻⁴

Example: 0.1011₂ = 0×2⁰ + 1×2⁻¹ + 0×2⁻² + 1×2⁻³ + 1×2⁻⁴
                 = 0 + 0.5 + 0 + 0.125 + 0.0625
                 = 0.6875₁₀
```

---

## 3. Octal Number System

### Fundamentals

**Base**: 8  
**Digits**: 0, 1, 2, 3, 4, 5, 6, 7

```
┌─────────────────────────────────────────────────────────────────┐
│                     OCTAL NUMBER STRUCTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│      Position:    3      2      1      0                       │
│      Weight:     512    64     8      1                        │
│                  8³     8²     8¹     8⁰                       │
│                                                                 │
│      Example:     2      7      5      3                       │
│                 1024 + 448  + 40   + 3   = 1515₁₀              │
│                                                                 │
│      (2×512) + (7×64) + (5×8) + (3×1) = 1515                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Octal-Binary Relationship

Each octal digit corresponds to **exactly 3 binary bits**:

```
┌─────────────────────────────────────┐
│   OCTAL TO BINARY MAPPING           │
├─────────┬───────────────────────────┤
│ Octal   │ Binary (3 bits)           │
├─────────┼───────────────────────────┤
│   0     │   000                     │
│   1     │   001                     │
│   2     │   010                     │
│   3     │   011                     │
│   4     │   100                     │
│   5     │   101                     │
│   6     │   110                     │
│   7     │   111                     │
└─────────┴───────────────────────────┘

Example: 275₈ = 010 111 101₂ = 10111101₂
                 2   7   5
```

### Why Octal?

- Compact representation of binary (3:1 compression)
- Used in Unix/Linux file permissions (chmod 755)
- Legacy computing systems (PDP-8, early computers)

---

## 4. Hexadecimal Number System

### Fundamentals

**Base**: 16  
**Digits**: 0-9 and A-F (A=10, B=11, C=12, D=13, E=14, F=15)

```
┌─────────────────────────────────────────────────────────────────┐
│                 HEXADECIMAL NUMBER STRUCTURE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│      Position:    3       2       1       0                    │
│      Weight:    4096    256     16      1                      │
│                 16³     16²     16¹     16⁰                    │
│                                                                 │
│      Example:     2       A       F       3                    │
│                 8192 + 2560  + 240   + 3   = 10995₁₀           │
│                                                                 │
│      (2×4096) + (10×256) + (15×16) + (3×1) = 10995             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Hexadecimal-Binary Relationship

Each hex digit corresponds to **exactly 4 binary bits**:

```
┌──────────────────────────────────────────────────────┐
│            HEX TO BINARY MAPPING                     │
├──────┬────────┬─────────────────────────────────────┤
│ Hex  │ Binary │ Decimal                             │
├──────┼────────┼─────────────────────────────────────┤
│  0   │  0000  │   0                                 │
│  1   │  0001  │   1                                 │
│  2   │  0010  │   2                                 │
│  3   │  0011  │   3                                 │
│  4   │  0100  │   4                                 │
│  5   │  0101  │   5                                 │
│  6   │  0110  │   6                                 │
│  7   │  0111  │   7                                 │
│  8   │  1000  │   8                                 │
│  9   │  1001  │   9                                 │
│  A   │  1010  │  10                                 │
│  B   │  1011  │  11                                 │
│  C   │  1100  │  12                                 │
│  D   │  1101  │  13                                 │
│  E   │  1110  │  14                                 │
│  F   │  1111  │  15                                 │
└──────┴────────┴─────────────────────────────────────┘

Example: 2AF3₁₆ = 0010 1010 1111 0011₂
                   2    A    F    3
```

### Why Hexadecimal?

- Very compact (4:1 binary compression)
- Memory addresses
- Color codes (#FF5733 = RGB)
- MAC addresses (AA:BB:CC:DD:EE:FF)
- Assembly language programming

---

## 5. Number Conversions

### Conversion Roadmap

```
                         ┌─────────────┐
                         │   DECIMAL   │
                         │   (Base 10) │
                         └──────┬──────┘
                                │
           ┌────────────────────┼────────────────────┐
           │                    │                    │
           ▼                    ▼                    ▼
    ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
    │   BINARY    │◄────►│   OCTAL     │◄────►│HEXADECIMAL  │
    │   (Base 2)  │      │   (Base 8)  │      │  (Base 16)  │
    └─────────────┘      └─────────────┘      └─────────────┘
           │                    ▲                    │
           │                    │                    │
           └────────────────────┴────────────────────┘
                        (via Binary)
```

### 5.1 Decimal to Binary

**Method**: Repeated Division by 2

```
Example: Convert 45₁₀ to Binary

    45 ÷ 2 = 22  remainder 1  ←── LSB
    22 ÷ 2 = 11  remainder 0
    11 ÷ 2 = 5   remainder 1
     5 ÷ 2 = 2   remainder 1
     2 ÷ 2 = 1   remainder 0
     1 ÷ 2 = 0   remainder 1  ←── MSB

Read remainders BOTTOM to TOP: 45₁₀ = 101101₂

Verification: 32 + 8 + 4 + 1 = 45 ✓
```

**Fractional Part**: Repeated Multiplication by 2

```
Example: Convert 0.625₁₀ to Binary

    0.625 × 2 = 1.25   →  1  ←── First bit after point
    0.25  × 2 = 0.5    →  0
    0.5   × 2 = 1.0    →  1  ←── Last bit (fraction = 0)

Read integer parts TOP to BOTTOM: 0.625₁₀ = 0.101₂

Verification: 0.5 + 0.125 = 0.625 ✓
```

### 5.2 Binary to Decimal

**Method**: Sum of positional weights

```
Example: Convert 10110101₂ to Decimal

Position:  7   6   5   4   3   2   1   0
Binary:    1   0   1   1   0   1   0   1
Weight:  128  64  32  16   8   4   2   1

Calculation: 128 + 32 + 16 + 4 + 1 = 181₁₀
```

### 5.3 Decimal to Octal

**Method**: Repeated Division by 8

```
Example: Convert 247₁₀ to Octal

    247 ÷ 8 = 30  remainder 7  ←── LSB
     30 ÷ 8 = 3   remainder 6
      3 ÷ 8 = 0   remainder 3  ←── MSB

Result: 247₁₀ = 367₈

Verification: 3×64 + 6×8 + 7×1 = 192 + 48 + 7 = 247 ✓
```

### 5.4 Decimal to Hexadecimal

**Method**: Repeated Division by 16

```
Example: Convert 746₁₀ to Hexadecimal

    746 ÷ 16 = 46  remainder 10 (A)  ←── LSB
     46 ÷ 16 = 2   remainder 14 (E)
      2 ÷ 16 = 0   remainder 2       ←── MSB

Result: 746₁₀ = 2EA₁₆

Verification: 2×256 + 14×16 + 10×1 = 512 + 224 + 10 = 746 ✓
```

### 5.5 Binary ↔ Octal (Quick Method)

**Binary to Octal**: Group bits in 3s from right

```
Example: 10111010₂ to Octal

Step 1: Group in 3s:  10 | 111 | 010
Step 2: Pad leftmost:  010 | 111 | 010
Step 3: Convert:        2     7     2

Result: 10111010₂ = 272₈
```

**Octal to Binary**: Expand each digit to 3 bits

```
Example: 547₈ to Binary

    5    →   101
    4    →   100
    7    →   111

Result: 547₈ = 101100111₂
```

### 5.6 Binary ↔ Hexadecimal (Quick Method)

**Binary to Hex**: Group bits in 4s from right

```
Example: 110101011110₂ to Hexadecimal

Step 1: Group in 4s:  1101 | 0101 | 1110
Step 2: Convert:        D      5      E

Result: 110101011110₂ = D5E₁₆
```

**Hex to Binary**: Expand each digit to 4 bits

```
Example: 3AF₁₆ to Binary

    3    →   0011
    A    →   1010
    F    →   1111

Result: 3AF₁₆ = 001110101111₂ = 1110101111₂
```

### 5.7 Octal ↔ Hexadecimal

**Method**: Convert via Binary (fastest)

```
Example: 752₈ to Hexadecimal

Step 1 - Octal to Binary:
    7    →   111
    5    →   101
    2    →   010
    752₈ = 111101010₂

Step 2 - Binary to Hex:
    Group in 4s:  0001 | 1110 | 1010
    Convert:        1      E      A

Result: 752₈ = 1EA₁₆
```

---

## 6. Binary Arithmetic

### 6.1 Binary Addition

**Rules**:
```
    0 + 0 = 0
    0 + 1 = 1
    1 + 0 = 1
    1 + 1 = 10  (0 with carry 1)
    1 + 1 + 1 = 11  (1 with carry 1)
```

**Example**: Add 1011₂ + 1101₂

```
        1 1 1       ← Carries
          1 0 1 1   (11₁₀)
      +   1 1 0 1   (13₁₀)
      ───────────
        1 1 0 0 0   (24₁₀)

Step-by-step:
  Position 0: 1 + 1 = 10 → Write 0, carry 1
  Position 1: 1 + 0 + 1 = 10 → Write 0, carry 1
  Position 2: 0 + 1 + 1 = 10 → Write 0, carry 1
  Position 3: 1 + 1 + 1 = 11 → Write 1, carry 1
  Position 4: 0 + 0 + 1 = 1 → Write 1

Verification: 11 + 13 = 24 ✓
```

### 6.2 Binary Subtraction

**Rules**:
```
    0 - 0 = 0
    1 - 0 = 1
    1 - 1 = 0
    0 - 1 = 1 (with borrow 1 from next position)
```

**Example**: Subtract 1101₂ - 1001₂

```
                    (No borrows needed in this case)
          1 1 0 1   (13₁₀)
      -   1 0 0 1   (9₁₀)
      ───────────
          0 1 0 0   (4₁₀)

Verification: 13 - 9 = 4 ✓
```

**Example with Borrowing**: 1000₂ - 0011₂

```
          0 1 1     ← Modified after borrows
        ╱ ╱ ╱ ╲
          1 0 0 0   (8₁₀)
      -   0 0 1 1   (3₁₀)
      ───────────
          0 1 0 1   (5₁₀)

Step-by-step:
  Position 0: 0 - 1 → Borrow needed
    - Borrow from position 3 (chain borrow)
    - 10 - 1 = 1
  Position 1: 1 - 1 = 0 (after giving borrow)
  Position 2: 1 - 0 = 1 (after receiving borrow)
  Position 3: 0 - 0 = 0 (after giving borrow)

Verification: 8 - 3 = 5 ✓
```

### 6.3 Binary Multiplication

**Rules**:
```
    0 × 0 = 0
    0 × 1 = 0
    1 × 0 = 0
    1 × 1 = 1
```

**Method**: Shift and Add (like decimal multiplication)

**Example**: Multiply 1011₂ × 1101₂

```
                1 0 1 1   (11₁₀)
            ×   1 1 0 1   (13₁₀)
            ───────────
                1 0 1 1   (1011 × 1)
              0 0 0 0     (1011 × 0, shifted)
            1 0 1 1       (1011 × 1, shifted)
          1 0 1 1         (1011 × 1, shifted)
        ─────────────
        1 0 0 0 1 1 1 1   (143₁₀)

Verification: 11 × 13 = 143 ✓
```

### 6.4 Binary Division

**Method**: Long division (like decimal)

**Example**: Divide 1101₂ ÷ 11₂

```
                  1 0 0    Quotient (4₁₀)
              ┌────────
          11  │ 1 1 0 1
              │ 1 1       (11 × 1)
              ├──────
              │   0 0 1   (Remainder so far)
              │   0 0     (11 × 0)
              ├──────
              │     0 1   (Remainder so far)
              │     0 0   (11 × 0)
              └──────
                    0 1   Remainder (1₁₀)

Result: 1101₂ ÷ 11₂ = 100₂ remainder 01₂
        (13₁₀ ÷ 3₁₀ = 4₁₀ remainder 1₁₀) ✓
```

---

## 7. Signed Number Representation

### Why Signed Numbers?

Computers need to represent both positive and negative numbers. Several methods exist:

```
┌─────────────────────────────────────────────────────────────────┐
│                SIGNED NUMBER REPRESENTATIONS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. Sign-Magnitude                                             │
│   2. 1's Complement                                             │
│   3. 2's Complement  ← Most commonly used                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.1 Sign-Magnitude Representation

**Concept**: MSB indicates sign (0 = positive, 1 = negative)

```
For 8-bit representation:

   +25₁₀ = 0 0011001
           ↑ └──┬───┘
         Sign  Magnitude

   -25₁₀ = 1 0011001
           ↑ └──┬───┘
         Sign  Magnitude
```

**Range** (n bits): -(2^(n-1) - 1) to +(2^(n-1) - 1)
- For 8 bits: -127 to +127

**Problems**:
- Two representations for zero: +0 (00000000) and -0 (10000000)
- Complex arithmetic circuits

### 7.2 1's Complement

**Concept**: Negative numbers = Invert all bits of positive number

```
To find 1's complement: Flip all bits (0→1, 1→0)

Example (8-bit):
   +25₁₀ = 00011001
   -25₁₀ = 11100110  (inverted)

   +0₁₀  = 00000000
   -0₁₀  = 11111111  (still two zeros!)
```

**Range** (n bits): -(2^(n-1) - 1) to +(2^(n-1) - 1)
- For 8 bits: -127 to +127

**Identifying Negative Numbers**: MSB = 1 indicates negative

**Problems**:
- Still two representations for zero
- End-around carry needed in arithmetic

### 7.3 2's Complement (Most Important!)

**Concept**: 2's Complement = 1's Complement + 1

```
To find 2's complement:
   Method 1: Invert all bits, then add 1
   Method 2: Starting from right, keep bits until first 1, then invert rest

Example (8-bit):
   +25₁₀ = 00011001

   Method 1 for -25:
   Step 1: Invert → 11100110
   Step 2: Add 1  → 11100111
   
   -25₁₀ = 11100111

   Method 2 for -25 (Shortcut):
   00011001
        ↑ Keep everything from here right
   11100111 ← Invert everything to the left
```

**Range** (n bits): -2^(n-1) to +(2^(n-1) - 1)
- For 8 bits: -128 to +127

**Advantages**:
- Only ONE representation for zero
- Simple arithmetic (add/subtract using same circuit)
- Hardware efficient

### 2's Complement Number Circle

```
                         0 (00000000)
                              │
                    +1        │        -1
                 (00000001)   │   (11111111)
                      \       │       /
                       \      │      /
              +127      \     │     /      -128
           (01111111)    \    │    /    (10000000)
                │         \   │   /         │
                │          \  │  /          │
                │           ╲ │ ╱           │
                └────────────╳─────────────┘
                             │
                      Overflow boundary
                      
   Adding 1 to +127 (01111111) gives -128 (10000000) → OVERFLOW!
```

### 2's Complement Arithmetic

**Addition**:
```
Add +25 and -10 (8-bit):

   +25 = 00011001
   -10 = 11110110  (2's complement of 10)

       00011001
   +   11110110
   ───────────
     1 00001111  ← Ignore carry out
   
   Result: 00001111 = +15₁₀ ✓
```

**Subtraction**: A - B = A + (-B) = A + (2's complement of B)

```
Subtract: 25 - 10

   +25 = 00011001
   -10 = 11110110  (2's complement of 10)

   Same as addition above!
   Result: +15 ✓
```

### Comparison Table

| Property | Sign-Magnitude | 1's Complement | 2's Complement |
|----------|---------------|----------------|----------------|
| Zero representations | 2 (+0, -0) | 2 (+0, -0) | 1 |
| Range (8-bit) | -127 to +127 | -127 to +127 | -128 to +127 |
| Negative of N | Flip MSB | Invert all | Invert + 1 |
| Hardware complexity | High | Medium | Low |
| Modern usage | Rare | Rare | Standard |

---

## 8. Binary Codes

### 8.1 BCD (Binary Coded Decimal)

**Concept**: Each decimal digit represented by 4 bits

```
┌─────────────────────────────────────────────────────────────────┐
│                        BCD ENCODING                             │
├─────────────────────────────────────────────────────────────────┤
│  Decimal  │  BCD    │  Note                                    │
├───────────┼─────────┼───────────────────────────────────────────┤
│     0     │  0000   │                                          │
│     1     │  0001   │                                          │
│     2     │  0010   │                                          │
│     3     │  0011   │                                          │
│     4     │  0100   │                                          │
│     5     │  0101   │                                          │
│     6     │  0110   │                                          │
│     7     │  0111   │                                          │
│     8     │  1000   │                                          │
│     9     │  1001   │                                          │
│   10-15   │  ----   │  Invalid in BCD!                         │
└───────────┴─────────┴───────────────────────────────────────────┘
```

**Example**: Convert 259₁₀ to BCD

```
    2       5       9
   0010    0101    1001
   
259₁₀ = 0010 0101 1001 (BCD)

Note: This is NOT the same as 259₁₀ = 100000011₂ (pure binary)
```

**BCD vs Binary**:

| Decimal | BCD | Binary |
|---------|-----|--------|
| 15 | 0001 0101 | 1111 |
| 99 | 1001 1001 | 1100011 |

**Uses**: Digital displays, financial calculations (no rounding errors)

### 8.2 Gray Code

**Concept**: Only ONE bit changes between consecutive values

```
┌────────────────────────────────────────────────────────────────────┐
│                    GRAY CODE TABLE                                 │
├─────────┬────────┬─────────┬───────────────────────────────────────┤
│ Decimal │ Binary │  Gray   │  Bits Changed from Previous           │
├─────────┼────────┼─────────┼───────────────────────────────────────┤
│    0    │  0000  │  0000   │  -                                    │
│    1    │  0001  │  0001   │  1 bit (position 0)                   │
│    2    │  0010  │  0011   │  1 bit (position 1)                   │
│    3    │  0011  │  0010   │  1 bit (position 0)                   │
│    4    │  0100  │  0110   │  1 bit (position 2)                   │
│    5    │  0101  │  0111   │  1 bit (position 0)                   │
│    6    │  0110  │  0101   │  1 bit (position 1)                   │
│    7    │  0111  │  0100   │  1 bit (position 0)                   │
│    8    │  1000  │  1100   │  1 bit (position 3)                   │
│    9    │  1001  │  1101   │  1 bit (position 0)                   │
│   10    │  1010  │  1111   │  1 bit (position 1)                   │
│   11    │  1011  │  1110   │  1 bit (position 0)                   │
│   12    │  1100  │  1010   │  1 bit (position 2)                   │
│   13    │  1101  │  1011   │  1 bit (position 0)                   │
│   14    │  1110  │  1001   │  1 bit (position 1)                   │
│   15    │  1111  │  1000   │  1 bit (position 0)                   │
└─────────┴────────┴─────────┴───────────────────────────────────────┘
```

**Binary to Gray Code Conversion**:

```
Rule: G[i] = B[i] XOR B[i+1], G[MSB] = B[MSB]

Example: Binary 1011 to Gray

B:  1   0   1   1
    │   │   │   │
    │   ├───┤   │
    │   │   │   │
    ▼   ▼   ▼   ▼
G:  1   1   1   0

Step-by-step:
  G[3] = B[3] = 1 (MSB stays same)
  G[2] = B[3] XOR B[2] = 1 XOR 0 = 1
  G[1] = B[2] XOR B[1] = 0 XOR 1 = 1
  G[0] = B[1] XOR B[0] = 1 XOR 1 = 0

Result: 1011₂ → 1110 (Gray)
```

**Gray to Binary Conversion**:

```
Rule: B[i] = G[i] XOR B[i+1], B[MSB] = G[MSB]

Example: Gray 1110 to Binary

G:  1   1   1   0
    │   │   │   │
    │   ├───┼───┤
    │   │   │   │
    ▼   ▼   ▼   ▼
B:  1   0   1   1

Step-by-step:
  B[3] = G[3] = 1 (MSB stays same)
  B[2] = G[2] XOR B[3] = 1 XOR 1 = 0
  B[1] = G[1] XOR B[2] = 1 XOR 0 = 1
  B[0] = G[0] XOR B[1] = 0 XOR 1 = 1

Result: 1110 (Gray) → 1011₂
```

**Uses**: Rotary encoders, Karnaugh maps, error reduction in analog-to-digital conversion

### 8.3 Excess-3 Code

**Concept**: BCD + 3 (self-complementing code)

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXCESS-3 CODE TABLE                          │
├─────────────────────────────────────────────────────────────────┤
│  Decimal  │   BCD    │  Excess-3  │  9's Complement (in XS-3)  │
├───────────┼──────────┼────────────┼─────────────────────────────┤
│     0     │   0000   │    0011    │    1100 (represents 9)      │
│     1     │   0001   │    0100    │    1011 (represents 8)      │
│     2     │   0010   │    0101    │    1010 (represents 7)      │
│     3     │   0011   │    0110    │    1001 (represents 6)      │
│     4     │   0100   │    0111    │    1000 (represents 5)      │
│     5     │   0101   │    1000    │    0111 (represents 4)      │
│     6     │   0110   │    1001    │    0110 (represents 3)      │
│     7     │   0111   │    1010    │    0101 (represents 2)      │
│     8     │   1000   │    1011    │    0100 (represents 1)      │
│     9     │   1001   │    1100    │    0011 (represents 0)      │
└───────────┴──────────┴────────────┴─────────────────────────────┘

Self-complementing property:
  - 1's complement of XS-3 code = XS-3 code of 9's complement
  - Example: 1's comp of 0011 (0) = 1100 (9)  ✓
```

**Uses**: Simplifies subtraction in decimal arithmetic

---

## 9. Error Detection Codes

### 9.1 Parity Codes

**Concept**: Add extra bit to make total number of 1s even or odd

```
┌─────────────────────────────────────────────────────────────────┐
│                      PARITY TYPES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EVEN PARITY: Total number of 1s (including parity) is EVEN    │
│  ODD PARITY:  Total number of 1s (including parity) is ODD     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Example - Even Parity:
  Data: 1011001 (four 1s - even)
  Parity bit: 0
  Transmitted: 10110010

Example - Odd Parity:
  Data: 1011001 (four 1s)
  Parity bit: 1 (to make five 1s - odd)
  Transmitted: 10110011
```

**Parity Generator Circuit**:

```
        Data bits
           │
    ┌──────┴──────┐
    │             │
    D7 D6 D5 D4 D3 D2 D1 D0
    │  │  │  │  │  │  │  │
    └──┬──┘  │  └──┬──┘  │
       │     │     │     │
      XOR   XOR   XOR   XOR
       │     │     │     │
       └──┬──┘     └──┬──┘
          │           │
         XOR         XOR
          │           │
          └─────┬─────┘
                │
               XOR ───────► Parity bit (for even parity)
```

**Limitations**:
- Detects only ODD number of bit errors
- Cannot detect even number of errors (2, 4, 6...)
- Cannot correct errors

### 9.2 Hamming Code

**Concept**: Multiple parity bits at positions 2^n, can detect AND correct single-bit errors

```
┌─────────────────────────────────────────────────────────────────┐
│                    HAMMING CODE STRUCTURE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Position:    1   2   3   4   5   6   7   8   9   10  11  12   │
│  Type:        P1  P2  D1  P4  D2  D3  D4  P8  D5  D6  D7  D8   │
│                                                                 │
│  P = Parity bit (at positions 2^n: 1, 2, 4, 8, ...)            │
│  D = Data bit                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Hamming Code for 4 Data Bits (7,4 Hamming)**:

```
Position:       1    2    3    4    5    6    7
Binary pos:    001  010  011  100  101  110  111
Type:           P1   P2   D1   P4   D2   D3   D4

Parity coverage:
  P1 (bit 1): Covers positions with bit 0 = 1: 1,3,5,7
  P2 (bit 2): Covers positions with bit 1 = 1: 2,3,6,7
  P4 (bit 4): Covers positions with bit 2 = 1: 4,5,6,7
```

**Example**: Encode data 1011 using Hamming (7,4)

```
Step 1: Place data bits
Position:   1    2    3    4    5    6    7
           P1   P2   D1   P4   D2   D3   D4
            ?    ?    1    ?    0    1    1

Step 2: Calculate parity bits (even parity)

P1 covers positions 1,3,5,7: ?, 1, 0, 1
    P1 XOR 1 XOR 0 XOR 1 = 0  →  P1 = 0

P2 covers positions 2,3,6,7: ?, 1, 1, 1
    P2 XOR 1 XOR 1 XOR 1 = 0  →  P2 = 1

P4 covers positions 4,5,6,7: ?, 0, 1, 1
    P4 XOR 0 XOR 1 XOR 1 = 0  →  P4 = 0

Result: 0 1 1 0 0 1 1 = 0110011

Transmitted codeword: 0110011
```

**Error Detection and Correction**:

```
Received: 0110111 (error in position 5)

Check each parity:
  C1 = P1⊕D1⊕D2⊕D4 = 0⊕1⊕1⊕1 = 1  (Error!)
  C2 = P2⊕D1⊕D3⊕D4 = 1⊕1⊕1⊕1 = 0  (OK)
  C3 = P4⊕D2⊕D3⊕D4 = 0⊕1⊕1⊕1 = 1  (Error!)

Error position = C3 C2 C1 = 101₂ = 5₁₀

Flip bit at position 5: 0110111 → 0110011 (Corrected!)
```

**Hamming Distance**:
- Minimum number of bit positions where two codewords differ
- For error detection of up to `d` errors: minimum distance ≥ d + 1
- For error correction of up to `t` errors: minimum distance ≥ 2t + 1

```
Hamming(7,4) minimum distance = 3
  → Can detect up to 2 errors
  → Can correct 1 error
```

---

## 10. Summary Tables

### Number Systems Summary

| Base | Name | Digits | Conversion To Decimal | Binary Grouping |
|------|------|--------|----------------------|-----------------|
| 2 | Binary | 0, 1 | Σ(digit × 2^pos) | - |
| 8 | Octal | 0-7 | Σ(digit × 8^pos) | 3 bits |
| 10 | Decimal | 0-9 | - | - |
| 16 | Hexadecimal | 0-9, A-F | Σ(digit × 16^pos) | 4 bits |

### Signed Number Summary

| Method | Range (n bits) | Negation | Zeros |
|--------|---------------|----------|-------|
| Sign-Magnitude | ±(2^(n-1) - 1) | Flip MSB | 2 |
| 1's Complement | ±(2^(n-1) - 1) | Invert all | 2 |
| 2's Complement | -2^(n-1) to 2^(n-1)-1 | Invert + 1 | 1 |

### Binary Codes Summary

| Code | Bits per Digit | Property | Use |
|------|---------------|----------|-----|
| BCD | 4 | Direct decimal mapping | Displays, finance |
| Gray | Same as binary | 1-bit change | Encoders, K-maps |
| Excess-3 | 4 | Self-complementing | Arithmetic |

### Error Codes Summary

| Code | Detection | Correction | Overhead |
|------|-----------|------------|----------|
| Even/Odd Parity | Single bit | No | 1 bit |
| Hamming(7,4) | 2 bits | 1 bit | 3 bits per 4 data |

### Quick Conversion Reference

```
┌───────────────────────────────────────────────────────────────┐
│                 CONVERSION QUICK REFERENCE                    │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  Decimal → Binary:   Divide by 2, read remainders upward     │
│  Binary → Decimal:   Multiply each bit by 2^position, sum    │
│                                                               │
│  Binary → Octal:     Group 3 bits from right                 │
│  Octal → Binary:     Expand each digit to 3 bits             │
│                                                               │
│  Binary → Hex:       Group 4 bits from right                 │
│  Hex → Binary:       Expand each digit to 4 bits             │
│                                                               │
│  Binary → Gray:      G[i] = B[i] XOR B[i+1]                  │
│  Gray → Binary:      B[i] = G[i] XOR B[i+1]                  │
│                                                               │
│  Decimal → BCD:      Convert each digit to 4-bit binary      │
│  BCD → Decimal:      Convert each 4-bit group to digit       │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 11. Quick Revision Questions

### Question 1
Convert (10110.101)₂ to decimal.

<details>
<summary>Click for Answer</summary>

```
10110.101₂ = 1×2⁴ + 0×2³ + 1×2² + 1×2¹ + 0×2⁰ + 1×2⁻¹ + 0×2⁻² + 1×2⁻³
           = 16 + 0 + 4 + 2 + 0 + 0.5 + 0 + 0.125
           = 22.625₁₀
```
</details>

### Question 2
What is the 2's complement representation of -45 in 8 bits?

<details>
<summary>Click for Answer</summary>

```
Step 1: +45 in binary = 00101101
Step 2: 1's complement = 11010010
Step 3: Add 1          = 11010011

-45₁₀ = 11010011₂ (in 2's complement)
```
</details>

### Question 3
Convert (3AF)₁₆ to octal.

<details>
<summary>Click for Answer</summary>

```
Step 1: Hex to Binary
  3 = 0011, A = 1010, F = 1111
  3AF₁₆ = 001110101111₂

Step 2: Binary to Octal (group by 3)
  001 110 101 111
   1   6   5   7

(3AF)₁₆ = (1657)₈
```
</details>

### Question 4
Encode the data bits 1010 using Hamming (7,4) code.

<details>
<summary>Click for Answer</summary>

```
Positions: P1, P2, D1, P4, D2, D3, D4
Data:              1       0   1   0

P1 covers 1,3,5,7: P1⊕1⊕0⊕0 = 0 → P1 = 1
P2 covers 2,3,6,7: P2⊕1⊕1⊕0 = 0 → P2 = 0
P4 covers 4,5,6,7: P4⊕0⊕1⊕0 = 0 → P4 = 1

Hamming code: 1 0 1 1 0 1 0 = 1011010
```
</details>

### Question 5
Add (10110)₂ and (01101)₂ and express the result in hexadecimal.

<details>
<summary>Click for Answer</summary>

```
    1 1 1
      1 0 1 1 0   (22₁₀)
  +   0 1 1 0 1   (13₁₀)
  ─────────────
    1 0 0 0 1 1   (35₁₀)

100011₂ = 0010 0011 = 23₁₆
```
</details>

### Question 6
What is the Gray code equivalent of binary 1101?

<details>
<summary>Click for Answer</summary>

```
Binary:   1    1    0    1
          │    │    │    │
         MSB   │    │    │
          │    ▼    ▼    ▼
Gray:     1   1⊕1  1⊕0  0⊕1
          1    0    1    1

Binary 1101 = Gray 1011
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| - | Return to Index | [Unit 2: Boolean Algebra](Unit-02-Boolean-Algebra-and-Logic-Gates.md) |

---

*End of Unit 1*
