# Chapter 2: Modular Multiplication

[← Previous: Modular Addition](01-modular-addition.md) | [Back to README](../README.md) | [Next: Modular Division & Inverse →](03-modular-division-inverse.md)

---

## Overview

Modular multiplication is more overflow-prone than addition. This chapter covers safe multiplication under a modulus, 128-bit tricks, and the foundation for modular exponentiation.

---

## 2.1 Basic Formula

$$(a \times b) \bmod M = \big((a \bmod M) \times (b \bmod M)\big) \bmod M$$

```
FUNCTION modMul(a, b, M):
    RETURN ((a % M) * (b % M)) % M

Time: O(1)
Space: O(1)
```

---

## 2.2 Overflow Analysis

```
  ┌──────────────────────────────────────────────────────────┐
  │  CRITICAL OVERFLOW CONCERN:                              │
  │                                                          │
  │  M = 10⁹ + 7                                            │
  │  a % M ≤ 10⁹ + 6                                        │
  │  b % M ≤ 10⁹ + 6                                        │
  │                                                          │
  │  (a%M) × (b%M) can be up to ≈ 10¹⁸                      │
  │                                                          │
  │  32-bit int max:  ~2.1 × 10⁹   → OVERFLOW!              │
  │  64-bit long max: ~9.2 × 10¹⁸  → SAFE ✓                │
  │                                                          │
  │  So for M ≈ 10⁹: MUST use 64-bit integer!              │
  │                                                          │
  │  For M ≈ 10¹⁸: Even 64-bit overflows!                   │
  │  → Need 128-bit or special multiplication               │
  └──────────────────────────────────────────────────────────┘
```

---

## 2.3 Multiplication Table Under Modulus

```
  Multiplication table mod 7:

     × │ 0  1  2  3  4  5  6
    ───┼─────────────────────
     0 │ 0  0  0  0  0  0  0
     1 │ 0  1  2  3  4  5  6
     2 │ 0  2  4  6  1  3  5
     3 │ 0  3  6  2  5  1  4
     4 │ 0  4  1  5  2  6  3
     5 │ 0  5  3  1  6  4  2
     6 │ 0  6  5  4  3  2  1

  Notice: Every non-zero row is a permutation of {1,...,6}
  This happens because 7 is PRIME!
  (Every non-zero element has a multiplicative inverse)
```

---

## 2.4 Safe Multiplication for Large Modulus

When M can be up to 10¹⁸, use "binary multiplication" (multiply-by-doubling):

```
  Idea: Same as binary exponentiation, but with addition!
  
  a × b mod M:
  - Like Russian peasant multiplication
  - Double a, halve b, add when b is odd

FUNCTION modMulSafe(a, b, M):
    a = a % M
    result = 0
    WHILE b > 0:
        IF b is odd:
            result = (result + a) % M
        a = (a + a) % M      // double a (mod M)
        b = b >> 1             // halve b
    RETURN result

Time: O(log b)
Space: O(1)
```

### Trace: 7 × 13 mod 100

```
  a = 7, b = 13 = 1101₂, M = 100

  ┌─────┬──────┬──────┬────────┬───────────────────┐
  │ Step│  a   │  b   │ b odd? │ result             │
  ├─────┼──────┼──────┼────────┼───────────────────┤
  │  0  │  7   │  13  │        │ 0                  │
  │  1  │  7   │  13  │  Yes   │ 0+7 = 7            │
  │     │  14  │   6  │        │                     │
  │  2  │  14  │   6  │  No    │ 7 (unchanged)       │
  │     │  28  │   3  │        │                     │
  │  3  │  28  │   3  │  Yes   │ 7+28 = 35           │
  │     │  56  │   1  │        │                     │
  │  4  │  56  │   1  │  Yes   │ 35+56 = 91          │
  │     │  12  │   0  │        │                     │
  └─────┴──────┴──────┴────────┴───────────────────┘

  Result: 91
  Verify: 7 × 13 = 91. 91 mod 100 = 91 ✓
```

---

## 2.5 Language-Specific Overflow Handling

```
  ┌──────────────────────────────────────────────────────────┐
  │  C/C++:                                                  │
  │    long long a, b, M;                                    │
  │    result = (a % M) * (b % M) % M;  // OK for M ≤ 10⁹  │
  │    result = (__int128)a * b % M;     // for M ≤ 10¹⁸    │
  │                                                          │
  │  Java:                                                   │
  │    long result = (a % M) * (b % M) % M;  // M ≤ 10⁹    │
  │    BigInteger for larger M                               │
  │    Math.multiplyHigh(a, b) for 128-bit (Java 9+)        │
  │                                                          │
  │  Python:                                                 │
  │    result = (a * b) % M   // no overflow (arbitrary int) │
  │                                                          │
  │  JavaScript:                                             │
  │    BigInt(a) * BigInt(b) % BigInt(M)  // for large values│
  └──────────────────────────────────────────────────────────┘
```

---

## 2.6 Modular Product of Array

```
FUNCTION modProduct(arr, M):
    result = 1
    FOR each x in arr:
        result = (result * (x % M)) % M
    RETURN result

  ⚠️ If any element is divisible by M,
     the entire product becomes 0 mod M.

  Example: arr = [3, 5, 7, 11], M = 13
  Step 1: result = 1 × 3 = 3
  Step 2: result = 3 × 5 = 15 % 13 = 2
  Step 3: result = 2 × 7 = 14 % 13 = 1
  Step 4: result = 1 × 11 = 11
  Answer: 11

  Verify: 3 × 5 × 7 × 11 = 1155
  1155 mod 13 = 1155 - 88×13 = 1155 - 1144 = 11 ✓
```

---

## 2.7 Factorial Modulo M

```
FUNCTION factorialMod(n, M):
    result = 1
    FOR i = 2 TO n:
        result = (result * i) % M
    RETURN result

  n!  mod (10⁹+7):
  ┌─────┬──────────────────┐
  │  n  │  n! mod 10⁹+7    │
  ├─────┼──────────────────┤
  │  1  │  1               │
  │  5  │  120             │
  │ 10  │  3628800         │
  │ 20  │  146326063       │
  │100  │  437918130       │
  └─────┴──────────────────┘

  Note: If n ≥ M, then n! mod M = 0
  (because M is a factor in the product 1×2×...×n)
```

---

## 2.8 Problem-Solving Patterns

### Pattern 1: Compute Large Products Mod M

```
  Given: n very large numbers (could be > 10⁹)
  Find: their product mod M

  Key: Take mod at EACH multiplication step.
  NEVER compute the full product then mod!
```

### Pattern 2: Repeated Squaring Preview

```
  a^n mod M:
  Naive: multiply a, n times → O(n)
  Fast:  binary exponentiation → O(log n)

  a^13 = a^8 × a^4 × a^1    (13 = 1101₂)

  → Covered in detail in Unit 5!
```

### Pattern 3: Counting with Multiplication

```
  Many counting problems involve:
  • nCr mod M (combinations)
  • n! mod M (permutations)
  • Catalan numbers mod M
  
  All need modular multiplication as the primitive.
```

---

## 2.9 Pitfalls

```
  ┌──────────────────────────────────────────────────────────┐
  │  COMMON MISTAKES:                                        │
  │                                                          │
  │  1. Forgetting to mod intermediate results               │
  │     → overflow and wrong answer                          │
  │                                                          │
  │  2. Using int instead of long long in C++                │
  │     → silent overflow, hard to debug                     │
  │                                                          │
  │  3. Modding by 0                                         │
  │     → runtime error / undefined behavior                 │
  │                                                          │
  │  4. Assuming (a*b)%M = ((a%M)*(b%M))%M works for        │
  │     integers only. For floats, modular arithmetic        │
  │     does NOT apply!                                      │
  │                                                          │
  │  5. Not handling negative intermediate values            │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Formula | Overflow Risk |
|---------|---------|---------------|
| Basic mod mul | (a%M × b%M) % M | If M > √(max_int) |
| Safe for M ≤ 10⁹ | Use 64-bit int | Product ≤ 10¹⁸ |
| Safe for M ≤ 10¹⁸ | Binary multiplication or __int128 | Need special handling |
| Array product | Take mod at each step | Never accumulate |
| Factorial mod | Iterate with mod | n ≥ M gives 0 |

---

## Quick Revision Questions

1. **What is the maximum value** of (a%M) × (b%M) when M = 10⁹+7?
2. **Why does 64-bit suffice** for M = 10⁹+7 but not for M = 10¹⁸?
3. **Trace 5 × 11 mod 7** using binary multiplication.
4. **For what value of n** does n! mod M become 0?
5. **Write modular product** of array [2, 3, 5, 7] mod 11.
6. **Why can't we use** modular arithmetic with floating-point numbers?

---

[← Previous: Modular Addition](01-modular-addition.md) | [Back to README](../README.md) | [Next: Modular Division & Inverse →](03-modular-division-inverse.md)
