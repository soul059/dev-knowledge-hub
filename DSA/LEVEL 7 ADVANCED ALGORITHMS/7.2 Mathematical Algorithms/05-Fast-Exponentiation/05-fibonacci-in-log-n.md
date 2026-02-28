# Chapter 5: Fibonacci in O(log n)

[← Previous: Matrix Exponentiation](04-matrix-exponentiation.md) | [Back to README](../README.md) | [Next: Applications of Fast Exponentiation →](06-applications.md)

---

## Overview

Computing Fibonacci numbers is the classic application of matrix exponentiation. This chapter covers three O(log n) methods: matrix method, fast doubling formulas, and Pisano periods for modular Fibonacci.

---

## 5.1 Methods Comparison

```
  ┌────────────────────┬───────────┬────────────┬──────────────┐
  │ Method             │ Time      │ Space      │ Multiplies   │
  ├────────────────────┼───────────┼────────────┼──────────────┤
  │ Naive recursion    │ O(2ⁿ)    │ O(n)       │ Exponential  │
  │ DP / Iteration     │ O(n)      │ O(1)       │ n            │
  │ Matrix exponent.   │ O(log n)  │ O(1)       │ ~12 log n    │
  │ Fast doubling      │ O(log n)  │ O(log n)*  │ ~5 log n     │
  │ * = recursion stack, O(1) iterative variant exists         │
  └────────────────────┴───────────┴────────────┴──────────────┘
```

---

## 5.2 Method 1: Matrix Exponentiation (Review)

```
  ┌        ┐       ┌     ┐ⁿ   ┌      ┐
  │ F(n+1) │   =   │ 1 1 │  × │ F(1) │
  │  F(n)  │       │ 1 0 │    │ F(0) │
  └        ┘       └     ┘    └      ┘

  F(n) = bottom-left element of [[1,1],[1,0]]^n

  Uses matrix fast power from Chapter 4.
  Each step: 8 multiplications + 4 additions (2×2 matmul)
```

---

## 5.3 Method 2: Fast Doubling Formulas

```
  ┌──────────────────────────────────────────────────────────┐
  │  FAST DOUBLING:                                          │
  │                                                          │
  │  F(2k)   = F(k) × [2F(k+1) - F(k)]                     │
  │  F(2k+1) = F(k)² + F(k+1)²                              │
  │                                                          │
  │  These formulas compute F(n) using only F(k) and F(k+1) │
  │  where k = n/2. Recurse on k → O(log n) depth!          │
  └──────────────────────────────────────────────────────────┘

  Derivation (from matrix identity):
  [[1,1],[1,0]]^(2k) = ([[1,1],[1,0]]^k)²

  Expanding and extracting elements gives the formulas.
```

### Implementation

```
FUNCTION fastDoubling(n, MOD):
    IF n == 0: RETURN (0, 1)    // (F(0), F(1))
    
    (a, b) = fastDoubling(n / 2, MOD)    // a = F(k), b = F(k+1)
    
    c = a * (2*b - a) % MOD              // F(2k)
    d = (a*a + b*b) % MOD                // F(2k+1)
    
    IF n is even:
        RETURN (c, d)           // (F(2k), F(2k+1))
    ELSE:
        RETURN (d, (c + d) % MOD)  // (F(2k+1), F(2k+2))

// To get F(n):
(fib_n, _) = fastDoubling(n, MOD)
// fib_n is the answer
```

---

## 5.4 Trace: F(13) via Fast Doubling

```
  n = 13 → binary: 1101

  fastDoubling(13)
    fastDoubling(6)
      fastDoubling(3)
        fastDoubling(1)
          fastDoubling(0) → (0, 1)     F(0)=0, F(1)=1
        k=0: a=0, b=1
        c = 0×(2×1-0) = 0              F(0) = 0
        d = 0²+1² = 1                  F(1) = 1
        n=1 odd → return (1, 0+1) = (1, 1)   F(1)=1, F(2)=1
      k=1: a=1, b=1
      c = 1×(2×1-1) = 1                F(2) = 1
      d = 1²+1² = 2                    F(3) = 2
      n=3 odd → return (2, 1+2) = (2, 3)     F(3)=2, F(4)=3
    k=3: a=2, b=3
    c = 2×(2×3-2) = 2×4 = 8            F(6) = 8
    d = 2²+3² = 4+9 = 13               F(7) = 13
    n=6 even → return (8, 13)           F(6)=8, F(7)=13
  k=6: a=8, b=13
  c = 8×(2×13-8) = 8×18 = 144          F(12) = 144
  d = 8²+13² = 64+169 = 233            F(13) = 233
  n=13 odd → return (233, 144+233) = (233, 377)

  F(13) = 233 ✓

  Fibonacci: 0,1,1,2,3,5,8,13,21,34,55,89,144,233,...
  F(13) = 233 ✓
```

---

## 5.5 Why Fast Doubling Is Faster

```
  Matrix method (per step):
  - 2×2 matrix multiply = 8 multiplications + 4 additions

  Fast doubling (per step):
  - 3 multiplications + 2 additions + 1 subtraction
  
  Both are O(log n) steps, but fast doubling has
  SMALLER constant factor (~3 mults vs ~8 mults per step).

  ┌──────────────┬──────────────┬──────────────┐
  │ n            │ Matrix mults │ Fast doubling│
  ├──────────────┼──────────────┼──────────────┤
  │ 10⁹          │ ~240         │ ~90          │
  │ 10¹⁸         │ ~480         │ ~180         │
  └──────────────┴──────────────┴──────────────┘
```

---

## 5.6 Pisano Period

```
  F(n) mod M is PERIODIC! The period is called the Pisano period π(M).

  ┌───────┬──────────────────────────────────────────────────┐
  │  M    │ Sequence F(n) mod M                              │
  ├───────┼──────────────────────────────────────────────────┤
  │  2    │ 0,1,1,0,1,1,0,1,1,...    π(2)=3                 │
  │  3    │ 0,1,1,2,0,2,2,1,...      π(3)=8                 │
  │  5    │ 0,1,1,2,3,0,3,3,1,4,0,4,4,3,2,0,2,2,4,1,...    │
  │       │                           π(5)=20               │
  │  10   │ 0,1,1,2,3,5,8,3,1,4,5,9,4,3,7,0,7,7,4,1,...   │
  │       │                           π(10)=60              │
  └───────┴──────────────────────────────────────────────────┘

  Properties:
  • π(M) ≤ 6M (always ≤ 6 times M)
  • π(10) = 60 → last digit of Fibonacci repeats every 60
  • π(10^k) = 15 × 10^(k-1) for k ≥ 1
  • F(n) mod M = F(n mod π(M)) mod M
```

---

## 5.7 Fibonacci Identities

```
  ┌──────────────────────────────────────────────────────────┐
  │  USEFUL IDENTITIES:                                      │
  │                                                          │
  │  1. F(m+n) = F(m-1)F(n) + F(m)F(n+1)                   │
  │  2. F(2n) = F(n)[2F(n+1) - F(n)]                       │
  │  3. gcd(F(m), F(n)) = F(gcd(m, n))                     │
  │  4. F(n-1)F(n+1) - F(n)² = (-1)^n  (Cassini's identity)│
  │  5. F(n) | F(kn) for all k ≥ 1                          │
  │  6. Σ F(i) for i=0..n = F(n+2) - 1                     │
  │  7. Σ F(i)² for i=0..n = F(n)×F(n+1)                   │
  └──────────────────────────────────────────────────────────┘
```

---

## 5.8 Generalized: k-th Order Linear Recurrence in O(k³ log n)

```
  Tribonacci: T(n) = T(n-1) + T(n-2) + T(n-3)

  Matrix: ┌         ┐
          │ 1  1  1  │
          │ 1  0  0  │
          │ 0  1  0  │
          └         ┘

  Time: O(3³ × log n) = O(27 log n)

  For arbitrary k-th order:
  Matrix is k×k, time is O(k³ log n)
```

---

## 5.9 Problem-Solving Tips

```
  When you see "compute F(10¹⁸) mod M":
  
  1. Use fast doubling (fastest, fewest operations)
  2. Or matrix exponentiation (more general)
  
  When you see "compute F(n) mod small_M":
  
  1. Find Pisano period π(M)
  2. Compute F(n mod π(M)) mod M
  
  When you see "GCD of Fibonacci numbers":
  
  gcd(F(a), F(b)) = F(gcd(a, b))
```

---

## Summary Table

| Method | Time | Space | Best For |
|--------|------|-------|----------|
| Matrix exponentiation | O(8 log n) | O(1) | General recurrences |
| Fast doubling | O(3 log n) | O(log n) | Fibonacci specifically |
| Pisano period | O(π(M) + log n) | O(1) | F(n) mod small M |
| Iterative DP | O(n) | O(1) | n ≤ 10⁷ |

---

## Quick Revision Questions

1. **Write the fast doubling formulas** for F(2k) and F(2k+1).
2. **Trace F(10)** using fast doubling.
3. **What is the Pisano period** for M = 10? What does it mean?
4. **Prove Cassini's identity** F(n-1)F(n+1) - F(n)² = (-1)ⁿ for n=5.
5. **Which method** requires fewer multiplications per step: matrix or fast doubling?
6. **How would you compute** gcd(F(100), F(75))?

---

[← Previous: Matrix Exponentiation](04-matrix-exponentiation.md) | [Back to README](../README.md) | [Next: Applications of Fast Exponentiation →](06-applications.md)
