# Chapter 3: Recursive Fast Power

[← Previous: Iterative Fast Power](02-iterative-fast-power.md) | [Back to README](../README.md) | [Next: Matrix Exponentiation →](04-matrix-exponentiation.md)

---

## Overview

The recursive version of fast exponentiation provides a cleaner, more intuitive implementation. While it uses O(log n) stack space, it's excellent for understanding the divide-and-conquer nature and serves as the basis for matrix exponentiation and other generalizations.

---

## 3.1 Two Recursive Approaches

### Approach A: Halve-and-Square

```
FUNCTION modpow(base, exp, MOD):
    IF exp == 0: RETURN 1
    half = modpow(base, exp / 2, MOD)
    IF exp % 2 == 0:
        RETURN (half * half) % MOD
    ELSE:
        RETURN (half * half % MOD) * base % MOD
```

### Approach B: Even-Odd Split

```
FUNCTION modpow(base, exp, MOD):
    IF exp == 0: RETURN 1
    IF exp % 2 == 0:
        RETURN modpow(base * base % MOD, exp / 2, MOD)
    ELSE:
        RETURN base * modpow(base, exp - 1, MOD) % MOD
```

```
  ┌──────────────────────────────────────────────────────────┐
  │  Approach A: Better! Only ONE recursive call.            │
  │    → Depth = log₂(n), total multiplications = O(log n)  │
  │                                                          │
  │  Approach B: Can be TWO recursive calls for odd n.       │
  │    → exp-1 then (exp-1)/2... still O(log n) but         │
  │      slightly more overhead.                             │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.2 Trace: modpow(2, 13, 1000) — Approach A

```
  Call stack (going down):
  
  modpow(2, 13, 1000)         exp=13 (odd)
    modpow(2, 6, 1000)        exp=6 (even)
      modpow(2, 3, 1000)      exp=3 (odd)
        modpow(2, 1, 1000)    exp=1 (odd)
          modpow(2, 0, 1000)  exp=0 → return 1
        half=1, 1×1×2 = 2 → return 2
      half=2, 2×2×2 = 8 → return 8
    half=8, 8×8 = 64 → return 64
  half=64, 64×64×2 = 8192 mod 1000 = 192 → return 192

  2^13 = 8192, 8192 mod 1000 = 192 ✓
  
  Recursion depth: 5 (= ⌊log₂(13)⌋ + 1)
```

---

## 3.3 Trace: modpow(7, 20, 100) — Approach A

```
  modpow(7, 20)
    modpow(7, 10)
      modpow(7, 5)
        modpow(7, 2)
          modpow(7, 1)
            modpow(7, 0) = 1
          half=1, odd: 1×1×7 = 7
        half=7, even: 7×7 = 49
      half=49, odd: 49×49×7 = 2401×7 = 16807 mod 100 = 7
    half=7, even: 7×7 = 49
  half=49, even: 49×49 = 2401 mod 100 = 1

  7^20 mod 100 = 1

  Verify: 7^4 = 2401 → last 2 digits: 01
  7^20 = (7^4)^5 = (...01)^5 = ...01 ✓
```

---

## 3.4 Common Mistake: Double Recursion

```
  ⚠️ WRONG — causes EXPONENTIAL time!

  FUNCTION power_WRONG(base, exp):
      IF exp == 0: RETURN 1
      IF exp % 2 == 0:
          RETURN power_WRONG(base, exp/2) * power_WRONG(base, exp/2)
      ELSE:
          RETURN power_WRONG(base, exp/2) * power_WRONG(base, exp/2) * base

  This calls power_WRONG TWICE → recursion tree doubles!
  T(n) = 2T(n/2) + O(1) → O(n) ← NO IMPROVEMENT!

  ✓ CORRECT: Store in 'half' variable, reuse:
  half = power(base, exp/2)
  RETURN half * half  ← single call!
```

---

## 3.5 Recursion Tree Visualization

```
  CORRECT (single recursive call):

  power(a, 13)
  └── power(a, 6)
      └── power(a, 3)
          └── power(a, 1)
              └── power(a, 0) = 1

  Total nodes: 5 = O(log n)  ✓


  WRONG (double recursive call):

  power(a, 4)
  ├── power(a, 2)
  │   ├── power(a, 1)
  │   │   ├── power(a, 0) = 1
  │   │   └── power(a, 0) = 1
  │   └── power(a, 1)
  │       ├── power(a, 0) = 1
  │       └── power(a, 0) = 1
  └── power(a, 2)
      ├── power(a, 1)
      │   ├── power(a, 0) = 1
      │   └── power(a, 0) = 1
      └── power(a, 1)
          ├── power(a, 0) = 1
          └── power(a, 0) = 1

  Total nodes: 15 for n=4.  For n=1000 → 10⁶ nodes! ✗
```

---

## 3.6 Recursive vs Iterative Comparison

```
  ┌──────────────────────┬────────────────┬────────────────┐
  │ Aspect               │ Recursive      │ Iterative      │
  ├──────────────────────┼────────────────┼────────────────┤
  │ Time                 │ O(log n)       │ O(log n)       │
  │ Space                │ O(log n) stack │ O(1)           │
  │ Readability          │ More intuitive │ Slightly less  │
  │ Stack overflow risk  │ Yes (n > 2^30) │ None           │
  │ Generalizes to       │ Matrices, etc. │ Matrices too   │
  │ Competition use      │ Common         │ More common    │
  │ Teaching tool         │ Excellent      │ After recursive│
  └──────────────────────┴────────────────┴────────────────┘
```

---

## 3.7 Tail-Call Optimization (Approach B)

```
  Approach B is actually tail-recursive for the even case:

  modpow(base, exp, MOD):
      IF exp == 0: RETURN 1
      IF exp % 2 == 0:
          RETURN modpow(base * base % MOD, exp / 2, MOD)  // tail call!
      ELSE:
          RETURN base * modpow(base, exp - 1, MOD) % MOD

  Some compilers can optimize the even-case tail call.
  The odd case reduces to even case in one step,
  so effectively: O(log n) depth with partial TCO.
```

---

## 3.8 Applications of Recursive Structure

```
  The recursive formulation naturally extends to:

  1. MATRIX exponentiation (Chapter 4)
     Replace 'multiply numbers' with 'multiply matrices'

  2. POLYNOMIAL exponentiation
     Replace 'multiply numbers' with 'convolve polynomials'

  3. PERMUTATION power
     Replace 'multiply' with 'compose permutation'

  4. TRANSFORMATION composition
     Apply a transformation n times via binary lifting

  The recursive structure makes these generalizations obvious.
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Approach A | half = power(a, n/2); return half*half [*a] |
| Approach B | Even: power(a², n/2); Odd: a * power(a, n-1) |
| Preferred | Approach A (single recursive call) |
| Common mistake | Calling power(a, n/2) twice instead of storing |
| Time complexity | O(log n) |
| Space complexity | O(log n) recursion stack |
| Best use | Teaching, proof, matrix exponentiation |

---

## Quick Revision Questions

1. **What's wrong** with `return power(a, n/2) * power(a, n/2)`?
2. **Trace modpow(5, 8, 100)** using the recursive approach.
3. **How deep** is the recursion stack for n = 10⁹?
4. **Which approach** is partially tail-call optimizable?
5. **Why is Approach A preferred** over Approach B?
6. **Draw the recursion tree** for power(a, 11) (correct version).

---

[← Previous: Iterative Fast Power](02-iterative-fast-power.md) | [Back to README](../README.md) | [Next: Matrix Exponentiation →](04-matrix-exponentiation.md)
