# Chapter 2: Iterative Fast Power

[← Previous: Exponentiation by Squaring](01-exponentiation-by-squaring.md) | [Back to README](../README.md) | [Next: Recursive Fast Power →](03-recursive-fast-power.md)

---

## Overview

The iterative version of binary exponentiation avoids recursion overhead and stack space. It processes the exponent's binary digits from right to left, making it the preferred implementation in practice.

---

## 2.1 Algorithm

```
FUNCTION fastPow(base, exp, MOD):
    result = 1
    base = base % MOD
    WHILE exp > 0:
        IF exp & 1:                     // exp is odd (last bit = 1)
            result = (result * base) % MOD
        exp = exp >> 1                   // divide exp by 2
        base = (base * base) % MOD       // square the base
    RETURN result

Time: O(log exp)
Space: O(1)  ← advantage over recursive!
```

---

## 2.2 How It Works: Right-to-Left Binary Scan

```
  Process bits of exp from LEAST significant to MOST significant.
  
  exp = 13 = 1101₂
  
  bit 0 (value 1): 1 → include base^(2⁰) = base^1
  bit 1 (value 2): 0 → skip   base^(2¹) = base^2
  bit 2 (value 4): 1 → include base^(2²) = base^4
  bit 3 (value 8): 1 → include base^(2³) = base^8
  
  result = base^1 × base^4 × base^8 = base^13

  The variable 'base' is squared at each iteration,
  so it naturally holds base^1, base^2, base^4, base^8, ...
```

---

## 2.3 Detailed Trace: 3^13 mod 1000

```
  base = 3, exp = 13 = 1101₂, MOD = 1000

  ┌──────┬────────┬──────┬────────┬─────────────────────────┐
  │ Step │  exp   │ bit? │ result │ base                     │
  ├──────┼────────┼──────┼────────┼─────────────────────────┤
  │ Init │   13   │      │   1    │   3                      │
  │  1   │   13   │  1   │ 1×3=3  │   3                      │
  │      │    6   │      │        │ 3²=9                     │
  │  2   │    6   │  0   │   3    │   9                      │
  │      │    3   │      │        │ 9²=81                    │
  │  3   │    3   │  1   │3×81=243│  81                      │
  │      │    1   │      │        │ 81²=6561 → 561           │
  │  4   │    1   │  1   │243×561 │  561                     │
  │      │        │      │=136323 │                          │
  │      │    0   │      │→ 323   │ 561²=314721 → 721       │
  └──────┴────────┴──────┴────────┴─────────────────────────┘

  Result: 323
  Verify: 3^13 = 1594323 → 1594323 mod 1000 = 323 ✓
```

---

## 2.4 Trace: 2^10 mod 1000

```
  base = 2, exp = 10 = 1010₂, MOD = 1000

  ┌──────┬────────┬──────┬────────┬───────────┐
  │ Step │  exp   │ bit? │ result │ base      │
  ├──────┼────────┼──────┼────────┼───────────┤
  │ Init │   10   │      │   1    │   2       │
  │  1   │   10   │  0   │   1    │   2       │
  │      │    5   │      │        │  2²=4     │
  │  2   │    5   │  1   │ 1×4=4  │   4       │
  │      │    2   │      │        │  4²=16    │
  │  3   │    2   │  0   │   4    │  16       │
  │      │    1   │      │        │ 16²=256   │
  │  4   │    1   │  1   │4×256   │  256      │
  │      │        │      │=1024→24│           │
  │      │    0   │      │        │256²=65536→│
  └──────┴────────┴──────┴────────┴───────────┘

  Result: 24
  Verify: 2^10 = 1024 → 1024 mod 1000 = 24 ✓
```

---

## 2.5 Edge Cases

```
  ┌──────────────────────────────────────────────────────────┐
  │  EDGE CASES TO HANDLE:                                   │
  │                                                          │
  │  1. exp = 0 → result = 1 (loop doesn't execute)        │
  │  2. base = 0, exp > 0 → result = 0                     │
  │  3. base = 0, exp = 0 → convention: 1                  │
  │  4. base = 1 → result = 1 always                       │
  │  5. exp = 1 → result = base % MOD                      │
  │  6. MOD = 1 → result = 0 always                        │
  │  7. base > MOD → base %= MOD at start                  │
  │  8. Negative base → ((base%MOD)+MOD)%MOD               │
  └──────────────────────────────────────────────────────────┘
```

---

## 2.6 Without Modulo (Plain Integer Power)

```
FUNCTION fastPow_nomod(base, exp):
    result = 1
    WHILE exp > 0:
        IF exp & 1:
            result = result * base
        exp >>= 1
        base = base * base
    RETURN result

  ⚠️ This overflows FAST!
  2^63 already exceeds 64-bit signed int.
  Only useful for small exponents or big-integer types.
```

---

## 2.7 Floating-Point Fast Power

```
FUNCTION fastPow_float(base, exp):   // exp is non-negative integer
    result = 1.0
    WHILE exp > 0:
        IF exp & 1:
            result *= base
        exp >>= 1
        base *= base
    RETURN result

  ⚠️ Floating-point precision degrades with large exponents.
  For exact results, always use integer + modular arithmetic.
```

---

## 2.8 Template Code (Language-Agnostic)

```
  // Template: Modular Fast Power
  // Input:  base (integer), exp (non-negative integer), MOD (positive integer)
  // Output: (base^exp) mod MOD
  
  FUNCTION modpow(base, exp, MOD):
      result = 1
      base %= MOD
      IF base < 0:
          base += MOD
      WHILE exp > 0:
          IF exp & 1:
              result = result * base % MOD
          exp >>= 1
          base = base * base % MOD
      RETURN result

  This handles:
  ✓ Large base (reduced immediately)
  ✓ Negative base
  ✓ exp = 0
  ✓ Overflow prevention (mod at each step)
```

---

## 2.9 Performance Comparison

```
  Iterations for various exponent sizes:

  ┌──────────────┬────────────┬───────────────┐
  │ Exponent n   │ Naive O(n) │ Binary O(logn)│
  ├──────────────┼────────────┼───────────────┤
  │ 10           │ 10         │ 4             │
  │ 100          │ 100        │ 7             │
  │ 1000         │ 1000       │ 10            │
  │ 10⁶          │ 10⁶        │ 20            │
  │ 10⁹          │ 10⁹ (TLE!) │ 30            │
  │ 10¹²         │ impossible │ 40            │
  │ 10¹⁸         │ impossible │ 60            │
  └──────────────┴────────────┴───────────────┘

  Binary exponentiation makes O(10¹⁸) exponents trivial!
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Time | O(log n) |
| Space | O(1) — no recursion |
| Bit processing | Right to left (LSB first) |
| Key operation | Square base each step, multiply into result when bit=1 |
| Preferred over recursive | Yes — no stack overflow risk |
| Template | result=1; while exp>0: if odd result*=base; base²; exp>>=1 |

---

## Quick Revision Questions

1. **Why is the iterative version preferred** over recursive in practice?
2. **Trace 5^7 mod 13** using the iterative method.
3. **What is the result** of modpow(x, 0, M) for any x?
4. **How many loop iterations** for modpow(a, 10⁹, M)?
5. **What happens if** you forget `base %= MOD` at the start?
6. **Write the iterative modpow** handling negative base values.

---

[← Previous: Exponentiation by Squaring](01-exponentiation-by-squaring.md) | [Back to README](../README.md) | [Next: Recursive Fast Power →](03-recursive-fast-power.md)
