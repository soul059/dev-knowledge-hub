# Chapter 1.2: Converting Decimal to Binary

> **Unit 1 — Binary Number System**
> Master the fundamental skill of translating between human-readable decimals and machine-level binary.

---

## Overview

Converting decimal (base-10) to binary (base-2) is one of the most essential skills in computer science. There are multiple methods, and understanding all of them gives you both **mathematical insight** and **programming efficiency**.

---

## Method 1: Repeated Division by 2

The most common method. Divide the number by 2 repeatedly and collect remainders.

### Algorithm (Pseudocode)

```
FUNCTION decimalToBinary(n):
    result = ""
    WHILE n > 0:
        remainder = n % 2        // 0 or 1
        result = str(remainder) + result  // prepend
        n = n / 2                // integer division
    RETURN result
```

### Step-by-Step Trace: Convert 45₁₀ to Binary

```
  Step │ n   │ n ÷ 2 │ n % 2 │ Binary (building right to left)
  ─────┼─────┼───────┼───────┼─────────────────────────────────
    1  │  45 │   22  │   1   │                              1
    2  │  22 │   11  │   0   │                           0  1
    3  │  11 │    5  │   1   │                        1  0  1
    4  │   5 │    2  │   1   │                     1  1  0  1
    5  │   2 │    1  │   0   │                  0  1  1  0  1
    6  │   1 │    0  │   1   │               1  0  1  1  0  1
       │     │       │       │               ↑              ↑
       │     │       │       │              MSB            LSB
  
  READ REMAINDERS BOTTOM TO TOP:  101101₂
  
  Verify: 32 + 0 + 8 + 4 + 0 + 1 = 45 ✓
```

### Visual Flow Diagram

```
  45 ÷ 2 = 22 remainder 1  ──┐
  22 ÷ 2 = 11 remainder 0  ──┤
  11 ÷ 2 =  5 remainder 1  ──┤
   5 ÷ 2 =  2 remainder 1  ──┼──► Read upwards: 1 0 1 1 0 1
   2 ÷ 2 =  1 remainder 0  ──┤
   1 ÷ 2 =  0 remainder 1  ──┘
                     STOP (n = 0)
```

---

## Method 2: Subtraction of Powers of 2

Start from the largest power of 2 that fits, subtract, and mark its position as 1.

### Step-by-Step Trace: Convert 100₁₀ to Binary

```
  Powers of 2:  128  64  32  16   8   4   2   1
  
  100 ≥ 128?   No  → 0
  100 ≥  64?  Yes  → 1   (100 - 64 = 36 remaining)
   36 ≥  32?  Yes  → 1   (36 - 32 = 4 remaining)
    4 ≥  16?   No  → 0
    4 ≥   8?   No  → 0
    4 ≥   4?  Yes  → 1   (4 - 4 = 0 remaining)
    0 ≥   2?   No  → 0
    0 ≥   1?   No  → 0
    
  Result:          0   1   1   0   0   1   0   0
  
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │  0  │  1  │  1  │  0  │  0  │  1  │  0  │  0  │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
   128    64    32    16     8     4     2     1
   
  100₁₀ = 01100100₂
  Verify: 64 + 32 + 4 = 100 ✓
```

---

## Method 3: Using Bitwise Operations (Programmatic)

The most efficient method in code. Extract bits one at a time using AND and right-shift.

### Pseudocode

```
FUNCTION decimalToBinary(n):
    bits = []
    FOR i = 31 DOWN TO 0:          // for 32-bit integers
        bit = (n >> i) & 1         // extract bit at position i
        bits.append(bit)
    RETURN bits
```

### Trace: Convert 13₁₀ to Binary (8-bit)

```
  n = 13  →  binary: 00001101

  i = 7:  (13 >> 7) & 1 = (0) & 1 = 0    → bit 7 = 0
  i = 6:  (13 >> 6) & 1 = (0) & 1 = 0    → bit 6 = 0
  i = 5:  (13 >> 5) & 1 = (0) & 1 = 0    → bit 5 = 0
  i = 4:  (13 >> 4) & 1 = (0) & 1 = 0    → bit 4 = 0
  i = 3:  (13 >> 3) & 1 = (1) & 1 = 1    → bit 3 = 1
  i = 2:  (13 >> 2) & 1 = (3) & 1 = 1    → bit 2 = 1
  i = 1:  (13 >> 1) & 1 = (6) & 1 = 0    → bit 1 = 0
  i = 0:  (13 >> 0) & 1 = (13)& 1 = 1    → bit 0 = 1

  Result: 00001101₂ ✓
```

---

## Method 4: Recursive Approach

### Pseudocode

```
FUNCTION toBinary(n):
    IF n == 0:
        RETURN "0"
    IF n == 1:
        RETURN "1"
    RETURN toBinary(n / 2) + str(n % 2)
```

### Recursion Tree: Convert 11₁₀

```
  toBinary(11)
  └── toBinary(5) + "1"       // 11 % 2 = 1
      └── toBinary(2) + "1"   // 5  % 2 = 1
          └── toBinary(1) + "0"// 2  % 2 = 0
              └── returns "1"
          returns "10"
      returns "101"
  returns "1011"
  
  11₁₀ = 1011₂  ✓   (8 + 2 + 1 = 11)
```

---

## Converting Fractions (Decimal → Binary)

For fractional parts, **multiply by 2** and take the integer part:

### Trace: Convert 0.625₁₀ to Binary

```
  Step │ Value  │ ×2    │ Integer Part │ Binary Fraction
  ─────┼────────┼───────┼──────────────┼────────────────
    1  │ 0.625  │ 1.25  │     1        │  0.1
    2  │ 0.25   │ 0.50  │     0        │  0.10
    3  │ 0.50   │ 1.00  │     1        │  0.101
       │        │       │              │  STOP (0 remaining)
  
  0.625₁₀ = 0.101₂
  
  Verify: 1/2 + 0/4 + 1/8 = 0.5 + 0 + 0.125 = 0.625 ✓
```

---

## Complexity Analysis

| Method | Time Complexity | Space Complexity | Notes |
|--------|----------------|------------------|-------|
| Division by 2 | O(log n) | O(log n) | log₂(n) divisions |
| Powers of 2 | O(log n) | O(log n) | Check each power |
| Bitwise extraction | O(k) | O(k) | k = bit width (e.g., 32) |
| Recursive | O(log n) | O(log n) | Stack depth = log₂(n) |

> All methods are O(log n) since a number n has ⌊log₂(n)⌋ + 1 bits.

---

## Common Pitfalls

```
  ╔════════════════════════════════════════════════════╗
  ║  ⚠ WATCH OUT FOR THESE MISTAKES                  ║
  ╠════════════════════════════════════════════════════╣
  ║                                                    ║
  ║  1. Reading remainders in wrong order              ║
  ║     → Read bottom-to-top, NOT top-to-bottom       ║
  ║                                                    ║
  ║  2. Forgetting to handle n = 0                     ║
  ║     → 0₁₀ = 0₂ (special case)                     ║
  ║                                                    ║
  ║  3. Not adding leading zeros when needed           ║
  ║     → 5 in 8-bit = 00000101, not just 101         ║
  ║                                                    ║
  ║  4. Integer overflow in programming                ║
  ║     → Use appropriate data types                   ║
  ╚════════════════════════════════════════════════════╝
```

---

## Practice Conversions

Convert the following to binary:

| Decimal | Your Answer | Actual Binary |
|---------|-------------|---------------|
| 7 | ___ | 111 |
| 23 | ___ | 10111 |
| 64 | ___ | 1000000 |
| 100 | ___ | 1100100 |
| 255 | ___ | 11111111 |
| 1000 | ___ | 1111101000 |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Division method | Divide by 2, collect remainders bottom→top |
| Subtraction method | Subtract largest fitting power of 2 |
| Bitwise method | Use (n >> i) & 1 for each position |
| Recursive method | toBinary(n/2) + (n%2) |
| Fraction conversion | Multiply by 2, take integer part |
| Time complexity | O(log n) for all methods |
| Special case | n = 0 → "0" |

---

## Quick Revision Questions

1. **Convert 73₁₀ to binary using the division method.**
   <details><summary>Answer</summary>73 → 36 R1, 36 → 18 R0, 18 → 9 R0, 9 → 4 R1, 4 → 2 R0, 2 → 1 R0, 1 → 0 R1. Result: 1001001₂ (64+8+1=73)</details>

2. **How many bits are needed to represent the number 200?**
   <details><summary>Answer</summary>⌊log₂(200)⌋ + 1 = 7 + 1 = 8 bits</details>

3. **What is the binary of 0.75₁₀?**
   <details><summary>Answer</summary>0.75 × 2 = 1.5 → 1; 0.5 × 2 = 1.0 → 1. Result: 0.11₂ (1/2 + 1/4 = 0.75)</details>

4. **Which method is most efficient to implement in code?**
   <details><summary>Answer</summary>Bitwise extraction: (n >> i) & 1 — uses hardware-level operations</details>

5. **What is the time complexity of decimal-to-binary conversion?**
   <details><summary>Answer</summary>O(log n) — the number of digits in binary is ⌊log₂(n)⌋ + 1</details>

6. **Convert 128₁₀ to binary without calculating.**
   <details><summary>Answer</summary>128 = 2⁷, so it's just 10000000₂ (a 1 followed by 7 zeros)</details>

---

[← Previous: Binary Representation](01-binary-representation.md) | [Next: Binary to Decimal →](03-binary-to-decimal.md)
