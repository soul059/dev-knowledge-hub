# Chapter 1: Exponentiation by Squaring

[← Previous Unit: Linear Congruences](../04-Modular-Arithmetic/06-linear-congruences.md) | [Back to README](../README.md) | [Next: Iterative Fast Power →](02-iterative-fast-power.md)

---

## Overview

Exponentiation by Squaring is the technique of computing $a^n$ in $O(\log n)$ multiplications instead of $O(n)$. It's one of the most fundamental algorithmic optimizations — applicable to numbers, matrices, polynomials, and any monoid.

---

## 1.1 The Core Idea

```
  ┌──────────────────────────────────────────────────────────┐
  │  OBSERVATION:                                            │
  │                                                          │
  │  a^n = ┌ 1                        if n = 0              │
  │        │ (a^(n/2))² × a           if n is odd            │
  │        └ (a^(n/2))²               if n is even           │
  │                                                          │
  │  Each step HALVES the exponent → O(log n) steps!        │
  │                                                          │
  │  Instead of: a × a × a × ... × a  (n times)            │
  │  We do:      square, square, ... (log n times)          │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.2 Visual: How 3^13 Decomposes

```
  13 = 1101₂ = 8 + 4 + 1

  3^13 = 3^8 × 3^4 × 3^1

  Computing the powers of 3:
  3^1  = 3
  3^2  = 9        (square 3^1)
  3^4  = 81       (square 3^2)
  3^8  = 6561     (square 3^4)

  3^13 = 6561 × 81 × 3 = 1594323

  Total multiplications: 3 squarings + 2 multiplies = 5
  Naive: 12 multiplications

  ┌────────────────────────────────────────┐
  │  Exponent bits:  1  1  0  1           │
  │  Powers:         8  4  2  1           │
  │  Include?        ✓  ✓  ✗  ✓           │
  │  Result: 3^8 × 3^4 × 3^1 = 3^13     │
  └────────────────────────────────────────┘
```

---

## 1.3 Recursive Implementation

```
FUNCTION power(a, n):
    IF n == 0:
        RETURN 1
    half = power(a, n / 2)   // integer division
    IF n is even:
        RETURN half × half
    ELSE:
        RETURN half × half × a

Time: O(log n) multiplications
Space: O(log n) recursion stack
```

### Trace: power(2, 10)

```
  power(2, 10)
  └── half = power(2, 5)
      └── half = power(2, 2)
          └── half = power(2, 1)
              └── half = power(2, 0) = 1
              │   return 1 × 1 × 2 = 2
          │   return 2 × 2 = 4
      │   return 4 × 4 × 2 = 34... wait
      
  Let me trace carefully:
  
  power(2, 10):
    half = power(2, 5)
      half = power(2, 2)
        half = power(2, 1)
          half = power(2, 0) = 1
        n=1 is odd: 1 × 1 × 2 = 2
      n=2 is even: 2 × 2 = 4
    n=5 is odd: 4 × 4 × 2 = 34... 

  Hmm, let me recompute:
  power(2, 0) = 1
  power(2, 1): half=1, odd → 1×1×2 = 2
  power(2, 2): half=2, even → 2×2 = 4
  power(2, 5): half=4, odd → 4×4×2 = 32
  power(2, 10): half=32, even → 32×32 = 1024

  2^10 = 1024 ✓

  Only 4 recursive calls (= ⌊log₂(10)⌋ + 1)!
```

---

## 1.4 Why It Works: Mathematical Proof

```
  Proof by strong induction on n:
  
  Base: n = 0 → a^0 = 1 ✓
  
  Inductive step: Assume works for all k < n.
  
  Case 1: n even (n = 2m)
    power(a, n) = half × half where half = power(a, m)
    By IH: half = a^m
    power(a, n) = a^m × a^m = a^(2m) = a^n ✓
  
  Case 2: n odd (n = 2m + 1)
    power(a, n) = half × half × a where half = power(a, m)
    By IH: half = a^m
    power(a, n) = a^m × a^m × a = a^(2m+1) = a^n ✓
```

---

## 1.5 Complexity Analysis

```
  ┌──────────────────────────────────────────────────────────┐
  │  At each step, exponent is HALVED.                       │
  │  n → n/2 → n/4 → ... → 1 → 0                          │
  │  Number of steps = ⌊log₂(n)⌋ + 1                       │
  │                                                          │
  │  Each step does:                                         │
  │  • 1 recursive call                                      │
  │  • 1 multiplication (squaring)                           │
  │  • possibly 1 more multiplication (if odd)               │
  │                                                          │
  │  Total multiplications: at most 2 × log₂(n)             │
  │                                                          │
  │  COMPARISON:                                             │
  │  n = 10⁶:  Naive = 10⁶ mults, Binary = 40 mults       │
  │  n = 10⁹:  Naive = 10⁹ mults, Binary = 60 mults       │
  │  n = 10¹⁸: Naive = 10¹⁸ mults, Binary = 120 mults     │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.6 Generalization: Any Associative Operation

```
  Binary exponentiation works for ANY associative operation!

  ┌────────────────────┬───────────────┬───────────────┐
  │ Domain             │ Operation     │ "Power"       │
  ├────────────────────┼───────────────┼───────────────┤
  │ Integers           │ ×             │ a^n           │
  │ Integers mod M     │ × mod M       │ a^n mod M     │
  │ Matrices           │ Matrix mult   │ A^n           │
  │ Strings            │ Concatenation │ s repeated n× │
  │ Permutations       │ Composition   │ π^n           │
  │ Transformations    │ Composition   │ f^n           │
  └────────────────────┴───────────────┴───────────────┘

  Requirement: operation must be ASSOCIATIVE
  (a ⊕ b) ⊕ c = a ⊕ (b ⊕ c)
```

---

## 1.7 Comparison: Naive vs Binary Exponentiation

```
  ┌──────────────────┬──────────────────┬──────────────────┐
  │ Aspect           │ Naive            │ Binary           │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Time             │ O(n)             │ O(log n)         │
  │ For n=10⁹        │ ~1 second        │ ~30 operations   │
  │ For n=10¹⁸       │ Years            │ ~60 operations   │
  │ Code             │ 3 lines          │ 6-8 lines        │
  │ Applicable to    │ Numbers only     │ Any associative  │
  │                  │ (practically)    │ operation        │
  └──────────────────┴──────────────────┴──────────────────┘
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Core idea | Halve exponent at each step |
| Time | O(log n) multiplications |
| Space (recursive) | O(log n) stack |
| Even case | a^n = (a^(n/2))² |
| Odd case | a^n = (a^(n/2))² × a |
| Generalization | Works for any associative operation |
| Key application | Modular exponentiation |

---

## Quick Revision Questions

1. **How many multiplications** to compute 5^100 using binary exponentiation?
2. **Write the recurrence** for power(a, n) and identify base case.
3. **Trace power(3, 7)** showing each recursive call and return value.
4. **Why must the operation be associative** for this technique to work?
5. **Compare the number of multiplications** for 2^1000 naive vs binary.
6. **Give three examples** of domains where binary exponentiation applies.

---

[← Previous Unit: Linear Congruences](../04-Modular-Arithmetic/06-linear-congruences.md) | [Back to README](../README.md) | [Next: Iterative Fast Power →](02-iterative-fast-power.md)
