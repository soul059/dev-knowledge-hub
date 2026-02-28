# Chapter 1: Euclidean Algorithm (Deep Dive)

[← Previous: Count of Divisors](../02-Prime-Numbers/06-count-of-divisors.md) | [Back to README](../README.md) | [Next: Extended Euclidean →](02-extended-euclidean-algorithm.md)

---

## Overview

This chapter provides an in-depth treatment of the Euclidean Algorithm — its proof of correctness, both recursive and iterative forms, worst-case analysis, and practical implementation details.

---

## 1.1 The Algorithm

```
  ┌──────────────────────────────────────────────────────────┐
  │  EUCLIDEAN ALGORITHM                                     │
  │                                                          │
  │  Recurrence: GCD(a, b) = GCD(b, a mod b)                │
  │  Base case:  GCD(a, 0) = a                               │
  │                                                          │
  │  Recursive:                                              │
  │    gcd(a, b) = (b == 0) ? a : gcd(b, a % b)             │
  │                                                          │
  │  Iterative:                                              │
  │    while b ≠ 0: a, b = b, a % b                         │
  │    return a                                              │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.2 Proof of Correctness

```
  Theorem: GCD(a, b) = GCD(b, a mod b)

  Let d = GCD(a, b) and r = a mod b = a - ⌊a/b⌋ × b

  Forward: d | a and d | b ⟹ d | (a - qb) = r ⟹ d | GCD(b, r)
  Backward: Let d' = GCD(b, r). d' | b and d' | r
            a = qb + r ⟹ d' | a ⟹ d' | GCD(a, b) = d

  So d | d' and d' | d ⟹ d = d'  ∎

  Termination:
  a mod b < b, so b strictly decreases each step.
  Since b ≥ 0 and is an integer, it must reach 0 in finite steps.
```

---

## 1.3 Comprehensive Traces

### Trace 1: GCD(270, 192)

```
  Step │  a    │  b    │  q  │  r = a mod b
  ─────┼───────┼───────┼─────┼─────────────
    1  │  270  │  192  │  1  │  78
    2  │  192  │   78  │  2  │  36
    3  │   78  │   36  │  2  │   6
    4  │   36  │    6  │  6  │   0
    5  │    6  │    0  │  —  │   — (done)

  GCD(270, 192) = 6

  Verify: 270 = 6 × 45, 192 = 6 × 32, GCD(45,32) = 1 ✓
```

### Trace 2: GCD(17, 13) — Coprime Numbers

```
  Step │  a   │  b   │  r
  ─────┼──────┼──────┼────
    1  │  17  │  13  │  4
    2  │  13  │   4  │  1
    3  │   4  │   1  │  0
    4  │   1  │   0  │ done

  GCD(17, 13) = 1  (coprime!)
```

### Trace 3: Worst Case GCD(F₁₁, F₁₀) = GCD(89, 55)

```
  Step │  a   │  b   │  q  │  r
  ─────┼──────┼──────┼─────┼────
    1  │  89  │  55  │  1  │ 34
    2  │  55  │  34  │  1  │ 21
    3  │  34  │  21  │  1  │ 13
    4  │  21  │  13  │  1  │  8
    5  │  13  │   8  │  1  │  5
    6  │   8  │   5  │  1  │  3
    7  │   5  │   3  │  1  │  2
    8  │   3  │   2  │  1  │  1
    9  │   2  │   1  │  2  │  0
   10  │   1  │   0  │  —  │ done

  GCD = 1 (consecutive Fibonacci numbers are always coprime)
  9 steps! (maximum for numbers this size)

  Notice: q = 1 every step until the last (slowest possible reduction)
```

---

## 1.4 Complexity Analysis

```
  ┌──────────────────────────────────────────────────────────┐
  │  TIME: O(log(min(a, b)))                                 │
  │                                                          │
  │  Proof: After TWO consecutive steps:                     │
  │    a₂ = b₀                                              │
  │    b₂ = a₀ mod b₀ < b₀/something                       │
  │                                                          │
  │  Key lemma: a mod b < a/2  (always!)                    │
  │                                                          │
  │  Case 1: b ≤ a/2  ⟹  a mod b < b ≤ a/2                │
  │  Case 2: b > a/2  ⟹  a mod b = a - b < a/2            │
  │                                                          │
  │  So every 2 steps, the pair (a,b) shrinks by ≥ half     │
  │  ⟹ at most 2 × log₂(min(a,b)) steps                   │
  │                                                          │
  │  SPACE: O(1) iterative, O(log n) recursive (stack)      │
  └──────────────────────────────────────────────────────────┘
```

### Step Count for Different Inputs

```
  (a, b)              Steps    Notes
  ──────────────────────────────────
  (100, 10)              1     b divides a
  (100, 97)              5     close numbers
  (gcd=1, coprime)     varies  depends on structure
  (F(n+1), F(n))         n     WORST CASE
  (10⁹, 10⁹-1)         ~60    large coprime
  (10¹⁸, 10¹⁷)         ~120   very large
```

---

## 1.5 Iterative vs Recursive

```
  ITERATIVE (preferred):                RECURSIVE:
  
  function gcd(a, b):                   function gcd(a, b):
      while b ≠ 0:                          if b == 0: return a
          a, b = b, a % b                   return gcd(b, a % b)
      return a
                                        Pros: elegant, concise
  Pros: no stack overflow,              Cons: stack overflow for
        slightly faster                       very deep recursion
  
  Stack depth comparison for GCD(F(47), F(46)):
  Iterative: 0 stack frames  ✓
  Recursive: 45 stack frames  (still fine for this size)
  
  For safe coding: prefer iterative in production
```

---

## 1.6 GCD with Special Inputs

```
  GCD(0, 0)  = 0    (by convention — no greatest common divisor)
  GCD(a, 0)  = a    (everything divides 0)
  GCD(0, b)  = b
  GCD(a, a)  = a    (trivial)
  GCD(a, 1)  = 1    (1 divides everything)
  GCD(-a, b) = GCD(|a|, |b|)  (take absolute values)
```

---

## 1.7 Applications

### Computing LCM

```
  LCM(a, b) = a / GCD(a, b) * b    // divide first to avoid overflow

  Example: LCM(12, 18) = 12 / GCD(12,18) * 18 = 12/6 * 18 = 2 * 18 = 36
```

### Fraction Simplification

```
  Simplify 84/126:
  GCD(84, 126):
    126 = 84 × 1 + 42
     84 = 42 × 2 + 0
  GCD = 42
  
  84/126 = (84/42)/(126/42) = 2/3
```

### Checking Coprimality

```
  Two numbers are coprime if GCD(a, b) = 1.
  Coprime pairs: (8,15), (14,25), (7,11), (1, anything)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Recurrence | GCD(a,b) = GCD(b, a mod b) |
| Base case | GCD(a, 0) = a |
| Time | O(log min(a,b)) |
| Worst case inputs | Consecutive Fibonacci numbers |
| Steps bound | ≤ 5 × digits of smaller number (Lamé) |
| Preferred form | Iterative (no stack overflow) |

---

## Quick Revision Questions

1. **Prove that** `a mod b < a/2` for any positive a, b.
2. **Why are consecutive Fibonacci numbers** the worst case?
3. **Trace GCD(1071, 462)** showing all steps with quotients and remainders.
4. **What is GCD(0, 0)?** Why is this a special case?
5. **Implement GCD iteratively** — what are the advantages over recursive?
6. **How many steps** does GCD(10⁹, 10⁹ - 1) take approximately?

---

[← Previous: Count of Divisors](../02-Prime-Numbers/06-count-of-divisors.md) | [Back to README](../README.md) | [Next: Extended Euclidean →](02-extended-euclidean-algorithm.md)
