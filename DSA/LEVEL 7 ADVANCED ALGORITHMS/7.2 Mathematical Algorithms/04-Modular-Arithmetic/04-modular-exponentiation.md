# Chapter 4: Modular Exponentiation

[← Previous: Modular Division & Inverse](03-modular-division-inverse.md) | [Back to README](../README.md) | [Next: Properties of Modulo →](05-properties-of-modulo.md)

---

## Overview

Computing $a^n \bmod M$ efficiently is one of the most important primitives in competitive programming and cryptography. Naive O(n) is too slow; binary exponentiation gives us O(log n).

---

## 4.1 The Problem

```
  Compute: 2^100 mod (10⁹+7)

  2^100 ≈ 1.27 × 10³⁰ → doesn't fit ANY integer type!
  But 2^100 mod (10⁹+7) = a small number < 10⁹+7

  ┌──────────────────────────────────────────────────────────┐
  │  Naive approach: multiply a, n times → O(n) TOO SLOW!   │
  │  For n = 10¹⁸, that's 10¹⁸ multiplications.            │
  │                                                          │
  │  Binary exponentiation: O(log n) ≈ 60 multiplications  │
  │  for n = 10¹⁸!                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.2 Binary Exponentiation (Exponentiation by Squaring)

```
  Key insight: express n in binary and use:
  
  a^n = a^(b₀ + b₁×2 + b₂×4 + ... + bₖ×2^k)
      = a^b₀ × (a²)^b₁ × (a⁴)^b₂ × ... × (a^(2^k))^bₖ

  Where bᵢ ∈ {0, 1} are the binary digits of n.

  Example: n = 13 = 1101₂
  a^13 = a^8 × a^4 × a^1

  Only multiply when the bit is 1!
```

---

## 4.3 Iterative Implementation

```
FUNCTION modPow(base, exp, M):
    result = 1
    base = base % M
    WHILE exp > 0:
        IF exp is odd:           // last bit is 1
            result = (result * base) % M
        exp = exp >> 1           // divide exp by 2
        base = (base * base) % M // square the base
    RETURN result

Time: O(log exp)
Space: O(1)
```

---

## 4.4 Trace: 3^13 mod 100

```
  base = 3, exp = 13 = 1101₂, M = 100

  ┌─────┬──────┬──────┬────────┬────────┬──────────────────┐
  │ Step│ base │ exp  │ odd?   │ result │ Action           │
  ├─────┼──────┼──────┼────────┼────────┼──────────────────┤
  │  0  │  3   │  13  │        │   1    │ Init             │
  │  1  │  3   │  13  │  Yes   │  1×3=3 │ result *= base   │
  │     │  9   │   6  │        │        │ base²=9, exp>>=1 │
  │  2  │  9   │   6  │  No    │   3    │ skip             │
  │     │  81  │   3  │        │        │ base²=81,exp>>=1 │
  │  3  │  81  │   3  │  Yes   │3×81=243│ result *= base   │
  │     │      │      │        │ =43    │ 243 % 100 = 43   │
  │     │ 6561 │   1  │        │        │ 81²=6561         │
  │     │  61  │      │        │        │ 6561%100 = 61    │
  │  4  │  61  │   1  │  Yes   │43×61   │ result *= base   │
  │     │      │      │        │=2623   │ 2623%100 = 23    │
  │     │ 3721 │   0  │        │        │ 61²=3721         │
  │     │  21  │      │        │        │ 3721%100 = 21    │
  └─────┴──────┴──────┴────────┴────────┴──────────────────┘

  Result: 23
  Verify: 3^13 = 1594323. 1594323 mod 100 = 23 ✓
```

---

## 4.5 Recursive Implementation

```
FUNCTION modPow_recursive(base, exp, M):
    IF exp == 0:
        RETURN 1
    IF exp is even:
        half = modPow_recursive(base, exp / 2, M)
        RETURN (half * half) % M
    ELSE:
        RETURN (base * modPow_recursive(base, exp - 1, M)) % M

Time: O(log exp)
Space: O(log exp)  — recursion stack
```

### Recursion Tree for 3^13

```
  modPow(3, 13)
  └── 3 × modPow(3, 12)
      └── half = modPow(3, 6)
          └── half = modPow(3, 3)
              └── 3 × modPow(3, 2)
                  └── half = modPow(3, 1)
                      └── 3 × modPow(3, 0)
                          └── return 1

  Unwind:
  modPow(3, 0) = 1
  modPow(3, 1) = 3 × 1 = 3
  modPow(3, 2) = 3 × 3 = 9
  modPow(3, 3) = 3 × 9 = 27
  modPow(3, 6) = 27 × 27 = 729 mod 100 = 29
  modPow(3, 12) = 29 × 29 = 841 mod 100 = 41
  modPow(3, 13) = 3 × 41 = 123 mod 100 = 23 ✓
```

---

## 4.6 Comparison: Iterative vs Recursive

```
  ┌──────────────────┬─────────────────┬────────────────────┐
  │ Aspect           │ Iterative       │ Recursive          │
  ├──────────────────┼─────────────────┼────────────────────┤
  │ Time             │ O(log n)        │ O(log n)           │
  │ Space            │ O(1)            │ O(log n) stack     │
  │ Overflow risk    │ Same            │ Same               │
  │ Code clarity     │ Slightly less   │ More intuitive     │
  │ Stack overflow   │ No risk         │ For huge exponents │
  │ Preferred        │ ✓ (in practice) │ Conceptual/proofs  │
  └──────────────────┴─────────────────┴────────────────────┘
```

---

## 4.7 Applications

### RSA Encryption

```
  Encrypt: c = m^e mod n    (modPow!)
  Decrypt: m = c^d mod n    (modPow!)
  
  where n ≈ 2^2048 (very large)
```

### Modular Inverse via Fermat

```
  a⁻¹ mod p = modPow(a, p - 2, p)
  (when p is prime)
```

### Last k Digits of a^n

```
  Last k digits of a^n = a^n mod 10^k
  
  Example: Last 3 digits of 7^100
  = modPow(7, 100, 1000)
  = 1 (7^100 ends in 001)
```

### Cycle Detection in Modular Sequences

```
  Sequence: aₙ = a^n mod M
  This sequence is periodic (Pigeonhole Principle)!
  Period divides φ(M) (Euler's totient).
```

---

## 4.8 Common Pitfalls

```
  ┌──────────────────────────────────────────────────────────┐
  │  1. base = 0, exp = 0 → Define as 1 (convention)       │
  │  2. Forgetting base %= M at start → wrong for base > M │
  │  3. Using int instead of long long in C++               │
  │     → (result * base) overflows silently!               │
  │  4. Negative exponent → not applicable in modular       │
  │     arithmetic (use modular inverse instead)             │
  │  5. exp = 0 should return 1, not 0                      │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.9 Towers of Exponents (a^b^c mod M)

```
  How to compute a^(b^c) mod M?
  
  By Euler's theorem:
  a^x ≡ a^(x mod φ(M)) (mod M)  when gcd(a, M) = 1
  
  So:
  a^(b^c) mod M = a^(b^c mod φ(M)) mod M
  
  Recursively:
  b^c mod φ(M) = b^(c mod φ(φ(M))) mod φ(M)
  
  Eventually φ reaches 1, and anything mod 1 = 0.
  
  This is an ADVANCED technique (Unit 10 covers more).
```

---

## Summary Table

| Concept | Complexity | Key Point |
|---------|-----------|-----------|
| Naive a^n mod M | O(n) | Too slow for n > 10⁷ |
| Binary exponentiation | O(log n) | Standard approach |
| Iterative version | O(log n) time, O(1) space | Preferred |
| Recursive version | O(log n) time, O(log n) space | Intuitive |
| Modular inverse | modPow(a, p-2, p) | Fermat's theorem |
| RSA | modPow on huge numbers | Real-world crypto |

---

## Quick Revision Questions

1. **How many multiplications** does binary exponentiation need for a^(10¹⁸)?
2. **Trace 2^10 mod 7** using iterative binary exponentiation.
3. **Why do we take base % M** at the very start?
4. **What should modPow(x, 0, M)** return and why?
5. **How do you find** the last 4 digits of 13^256?
6. **How does** a^(b^c) mod M differ from (a^b)^c mod M?

---

[← Previous: Modular Division & Inverse](03-modular-division-inverse.md) | [Back to README](../README.md) | [Next: Properties of Modulo →](05-properties-of-modulo.md)
