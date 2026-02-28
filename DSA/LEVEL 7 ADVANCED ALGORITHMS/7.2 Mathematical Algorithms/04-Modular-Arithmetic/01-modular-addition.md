# Chapter 1: Modular Addition and Subtraction

[← Previous Unit: Binary GCD](../03-GCD-and-LCM/06-binary-gcd.md) | [Back to README](../README.md) | [Next: Modular Multiplication →](02-modular-multiplication.md)

---

## Overview

Modular arithmetic is the backbone of competitive programming. This chapter introduces the concept of modulo, the modular world, and how addition and subtraction work under a modulus — the foundation for all subsequent modular operations.

---

## 1.1 What Is Modular Arithmetic?

```
  "Clock arithmetic" — numbers wrap around after reaching M.

  ┌──────────────────────────────────────────┐
  │  a mod M = remainder when a ÷ M          │
  │                                          │
  │  17 mod 5 = 2  (17 = 5×3 + 2)           │
  │  23 mod 7 = 2  (23 = 7×3 + 2)           │
  │  100 mod 10 = 0  (100 = 10×10 + 0)      │
  └──────────────────────────────────────────┘

  Clock analogy (M = 12):
  
       12/0
      /    \
    11      1
   |          |
  10          2
   |          |
    9       3
      \    /
     8  7  6  5  4

  14 o'clock = 14 mod 12 = 2 o'clock
  27 o'clock = 27 mod 12 = 3 o'clock
```

---

## 1.2 Why Use Modular Arithmetic?

```
  In competitive programming:
  
  "Print the answer modulo 10⁹ + 7"
  
  WHY?
  ┌──────────────────────────────────────────────────────────┐
  │  1. Answers can be ASTRONOMICALLY large                  │
  │     (e.g., 100! has 158 digits)                         │
  │                                                          │
  │  2. We need a way to "compress" the answer              │
  │     while still verifying correctness                    │
  │                                                          │
  │  3. 10⁹ + 7 = 1000000007 is:                            │
  │     ✓ Prime (required for modular inverse)              │
  │     ✓ Fits in 32-bit signed integer                     │
  │     ✓ (10⁹+7)² fits in 64-bit signed long              │
  │     ✓ Large enough to avoid accidental collisions       │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.3 Congruence Notation

```
  a ≡ b (mod M) means M | (a - b), i.e., a and b have the
  same remainder when divided by M.

  17 ≡ 2 (mod 5)    → both leave remainder 2
  23 ≡ 2 (mod 7)    → both leave remainder 2
  -3 ≡ 4 (mod 7)    → -3 + 7 = 4

  ⚠️ In most languages: -3 % 7 may return -3 (not 4!)
  Fix: ((a % M) + M) % M   → always gives [0, M-1]
```

---

## 1.4 Modular Addition

$$( a + b ) \bmod M = \big( (a \bmod M) + (b \bmod M) \big) \bmod M$$

```
FUNCTION modAdd(a, b, M):
    RETURN ((a % M) + (b % M)) % M

  ┌──────────────────────────────────────────────────────────┐
  │  OVERFLOW WARNING:                                       │
  │                                                          │
  │  If M ≈ 10⁹, then (a%M) + (b%M) can be up to 2×10⁹-2  │
  │  This OVERFLOWS 32-bit signed integer!                   │
  │                                                          │
  │  Solutions:                                              │
  │  • Use 64-bit integer (long long)                        │
  │  • Subtract M if sum ≥ M (branchless in practice)       │
  └──────────────────────────────────────────────────────────┘

  Fast version (no modulo, just comparison):
  FUNCTION modAdd_fast(a, b, M):  // assumes 0 ≤ a, b < M
      result = a + b
      IF result >= M:
          result -= M
      RETURN result
```

### Trace: (123456789 + 987654321) mod 10⁹+7

```
  M = 10⁹ + 7 = 1000000007

  a % M = 123456789
  b % M = 987654321

  sum = 123456789 + 987654321 = 1111111110
  sum >= M → 1111111110 - 1000000007 = 111111103

  Answer: 111111103
```

---

## 1.5 Modular Subtraction

$$( a - b ) \bmod M = \big( (a \bmod M) - (b \bmod M) + M \big) \bmod M$$

```
FUNCTION modSub(a, b, M):
    RETURN ((a % M) - (b % M) + M) % M

  WHY + M?
  Because (a%M - b%M) can be NEGATIVE!

  Example: (3 - 7) mod 5
  Direct: 3 - 7 = -4 → -4 mod 5 = ?
  
  In math: -4 mod 5 = 1  (since -4 + 5 = 1)
  In code: -4 % 5 = -4   (C/C++/Java give negative!)

  Fix: (-4 + 5) % 5 = 1 % 5 = 1 ✓
```

### Trace: (5 - 8) mod 7

```
  a % M = 5
  b % M = 8 % 7 = 1

  diff = 5 - 1 = 4
  4 ≥ 0, so result = 4 % 7 = 4

  Alternative with +M: (5 - 1 + 7) % 7 = 11 % 7 = 4 ✓
```

### Trace: (3 - 10) mod 7

```
  a % M = 3
  b % M = 10 % 7 = 3

  diff = 3 - 3 = 0
  Result = 0

  Or: (3 - 3 + 7) % 7 = 7 % 7 = 0 ✓
```

---

## 1.6 Handling Negative Numbers

```
  ┌──────────────────────────────────────────────────────────┐
  │  NEGATIVE MODULO PITFALL:                                │
  │                                                          │
  │  Language behavior for -7 % 3:                           │
  │  • C/C++:  -7 % 3 = -1   (truncated toward zero)       │
  │  • Java:   -7 % 3 = -1   (same as C)                   │
  │  • Python: -7 % 3 =  2   (floored division)            │
  │  • Math:   -7 mod 3 = 2  (always non-negative)         │
  │                                                          │
  │  UNIVERSAL FIX:                                          │
  │  result = ((a % M) + M) % M                             │
  │  This ALWAYS gives result in [0, M-1]                   │
  └──────────────────────────────────────────────────────────┘

  Detailed example: -7 mod 3
  C++: -7 % 3 = -1
  Fix: ((-1) + 3) % 3 = 2 % 3 = 2 ✓
```

---

## 1.7 Properties of Modular Arithmetic

```
  Let ≡ denote congruence mod M:

  ┌──────────────────────────────────────────────────────────┐
  │  If a ≡ a' and b ≡ b' (mod M), then:                   │
  │                                                          │
  │  1. (a + b) ≡ (a' + b')  (mod M)    ← addition OK      │
  │  2. (a - b) ≡ (a' - b')  (mod M)    ← subtraction OK   │
  │  3. (a × b) ≡ (a' × b')  (mod M)    ← multiplication OK│
  │  4. (a ^ n) ≡ (a' ^ n)   (mod M)    ← power OK         │
  │                                                          │
  │  ⚠️ DIVISION DOES NOT WORK THIS WAY!                    │
  │  a/b mod M ≠ (a mod M) / (b mod M)                      │
  │  → Need modular inverse (covered in Chapter 4)          │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.8 Modular Arithmetic as Equivalence Classes

```
  mod 5 creates 5 equivalence classes:

  [0]: ..., -10, -5, 0, 5, 10, 15, 20, ...
  [1]: ...,  -9, -4, 1, 6, 11, 16, 21, ...
  [2]: ...,  -8, -3, 2, 7, 12, 17, 22, ...
  [3]: ...,  -7, -2, 3, 8, 13, 18, 23, ...
  [4]: ...,  -6, -1, 4, 9, 14, 19, 24, ...

  Addition table (mod 5):
  
     + │ 0  1  2  3  4
    ───┼──────────────
     0 │ 0  1  2  3  4
     1 │ 1  2  3  4  0
     2 │ 2  3  4  0  1
     3 │ 3  4  0  1  2
     4 │ 4  0  1  2  3

  Every row is a permutation of {0,1,2,3,4}!
```

---

## 1.9 Common Patterns

### Pattern 1: Running Sum Modulo M

```
  Problem: Given array, find prefix sums mod M.
  
  arr = [3, 7, 2, 5, 1], M = 6
  
  prefix[0] = 3 % 6 = 3
  prefix[1] = (3 + 7) % 6 = 10 % 6 = 4
  prefix[2] = (4 + 2) % 6 = 6 % 6 = 0
  prefix[3] = (0 + 5) % 6 = 5
  prefix[4] = (5 + 1) % 6 = 0

  Key: Take mod at EACH step to prevent overflow.
```

### Pattern 2: Counting Subarrays with Sum Divisible by M

```
  If prefix[i] ≡ prefix[j] (mod M) with i < j,
  then sum(arr[i+1..j]) ≡ 0 (mod M).

  Count pairs of equal prefix-mod values!
  Use a frequency map of prefix_sum % M.
```

---

## Summary Table

| Operation | Formula | Watch Out |
|-----------|---------|-----------|
| (a + b) mod M | ((a%M) + (b%M)) % M | sum may exceed M |
| (a - b) mod M | ((a%M) - (b%M) + M) % M | result can be negative |
| Negative fix | ((x % M) + M) % M | C/C++/Java need this |
| Fast add | if sum ≥ M: sum -= M | avoids modulo operator |
| Why 10⁹+7 | Prime, fits in int, square fits in long | Standard in CP |

---

## Quick Revision Questions

1. **Why is 10⁹ + 7** used so frequently in competitive programming?
2. **What does -13 mod 5 equal** mathematically vs in C++?
3. **Write the universal fix** for negative modulo in C++.
4. **Can modular addition overflow** a 32-bit integer? How to prevent it?
5. **How do you count subarrays** with sum divisible by M using prefix sums?
6. **Why doesn't modular division** work like modular addition?

---

[← Previous Unit: Binary GCD](../03-GCD-and-LCM/06-binary-gcd.md) | [Back to README](../README.md) | [Next: Modular Multiplication →](02-modular-multiplication.md)
