# Chapter 3: LCM Computation

[← Previous: Extended Euclidean](02-extended-euclidean-algorithm.md) | [Back to README](../README.md) | [Next: GCD of Array →](04-gcd-of-array.md)

---

## Overview

LCM (Least Common Multiple) computation is tightly coupled with GCD. This chapter covers efficient LCM calculation, overflow-safe techniques, LCM of arrays, and common problems.

---

## 3.1 LCM Formulas

$$\text{LCM}(a, b) = \frac{|a \times b|}{\gcd(a, b)} = \frac{a}{\gcd(a, b)} \times b$$

```
  ┌──────────────────────────────────────────────────────────┐
  │  ⚠️ OVERFLOW WARNING:                                    │
  │                                                          │
  │  a × b can overflow even when LCM fits in the type!     │
  │                                                          │
  │  WRONG:  lcm = (a * b) / gcd(a, b)    // overflow risk! │
  │  RIGHT:  lcm = (a / gcd(a, b)) * b    // divide first!  │
  │                                                          │
  │  Example: a = 2×10⁹, b = 3×10⁹ (32-bit integer max ~2.1×10⁹)
  │  a × b = 6×10¹⁸ → OVERFLOW in 32-bit!                   │
  │  But a / gcd × b = 2×10⁹ / 10⁹ × 3×10⁹ = 6×10⁹        │
  │  → fits in 64-bit                                       │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.2 Implementation

```
FUNCTION lcm(a, b):
    IF a == 0 OR b == 0:
        RETURN 0
    g = gcd(abs(a), abs(b))
    RETURN abs(a) / g * abs(b)    // divide first!

Time: O(log min(a, b))   — dominated by GCD
Space: O(1)
```

### Trace: LCM(12, 18)

```
  Step 1: g = GCD(12, 18)
          18 = 12 × 1 + 6
          12 = 6 × 2 + 0
          g = 6

  Step 2: LCM = 12 / 6 × 18 = 2 × 18 = 36

  Verify multiples:
  12: 12, 24, [36], 48, ...
  18: 18, [36], 54, ...
  LCM = 36 ✓
```

---

## 3.3 LCM Using Prime Factorization

```
  LCM = take MAXIMUM exponent of each prime.

  Example: LCM(360, 756)
  
  360 = 2³ × 3² × 5¹
  756 = 2² × 3³ × 7¹

  LCM = 2^max(3,2) × 3^max(2,3) × 5^max(1,0) × 7^max(0,1)
      = 2³ × 3³ × 5¹ × 7¹
      = 8 × 27 × 5 × 7
      = 7560

  Contrast with GCD:
  GCD = 2^min(3,2) × 3^min(2,3) = 2² × 3² = 36
  
  Verify: GCD × LCM = 36 × 7560 = 272,160
          a × b    = 360 × 756 = 272,160  ✓
```

---

## 3.4 LCM of Multiple Numbers

```
  LCM(a₁, a₂, ..., aₙ) = LCM(LCM(...LCM(a₁, a₂)...), aₙ)

FUNCTION lcmArray(arr):
    result = arr[0]
    FOR i = 1 TO len(arr) - 1:
        result = lcm(result, arr[i])
    RETURN result
```

### Trace: LCM(4, 6, 10, 15)

```
  Step 1: lcm(4, 6)
          gcd(4, 6) = 2
          lcm = 4/2 × 6 = 12

  Step 2: lcm(12, 10)
          gcd(12, 10) = 2
          lcm = 12/2 × 10 = 60

  Step 3: lcm(60, 15)
          gcd(60, 15) = 15
          lcm = 60/15 × 15 = 60

  LCM(4, 6, 10, 15) = 60

  Via prime factorization:
  4  = 2²
  6  = 2 × 3
  10 = 2 × 5
  15 = 3 × 5
  LCM = 2² × 3 × 5 = 60 ✓
```

---

## 3.5 LCM Properties

```
  ┌──────────────────────────────────────────────────────────┐
  │  LCM PROPERTIES:                                         │
  │                                                          │
  │  1. LCM(a, b) ≥ max(a, b)                               │
  │  2. LCM(a, b) = a × b  ⟺  GCD(a, b) = 1 (coprime)    │
  │  3. LCM(a, 1) = a                                       │
  │  4. LCM(a, a) = a                                       │
  │  5. LCM(a, b) ≤ a × b                                   │
  │  6. LCM(ka, kb) = k × LCM(a, b)                         │
  │  7. a | LCM(a, b) and b | LCM(a, b)                     │
  │  8. If a | c and b | c, then LCM(a, b) | c              │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.6 LCM Growth Behavior

```
  ⚠️ LCM of array can grow VERY fast!

  LCM(1, 2, 3, ..., n):
  n     LCM
  ────────────
  1       1
  2       2
  3       6
  4      12
  5      60
  6      60
  7     420
  8     840
  9    2520
  10   2520
  20   232792560

  Asymptotic: LCM(1..n) ≈ e^n  (grows exponentially!)

  This means LCM of even a small array can overflow.
  Always use 64-bit integers or modular arithmetic!
```

---

## 3.7 Problem-Solving Patterns

### Pattern 1: Synchronization / Scheduling

```
  Problem: Event A repeats every a seconds, B every b seconds.
           When do they coincide?
  
  Answer: Every LCM(a, b) seconds.
  
  Example: Traffic lights: Red every 90s, Green every 60s
  LCM(90, 60) = 180 seconds = 3 minutes
```

### Pattern 2: Smallest Number Divisible by All in Array

```
  Problem: Find smallest positive n divisible by all elements of array.
  Answer: LCM of all elements.
  
  arr = [2, 3, 4, 5, 6]
  LCM = 60
```

### Pattern 3: LCM with Modular Arithmetic

```
  When LCM can overflow, compute LCM modulo M.
  
  But WARNING: LCM mod M ≠ (a/gcd mod M) × (b mod M)
  
  Need to handle carefully:
  - Compute factorization and track max exponents
  - Use modular exponentiation for final result
```

---

## 3.8 Applications

| Application | How LCM Is Used |
|-------------|-----------------|
| Task scheduling | Period of combined schedule |
| Clock synchronization | Tick alignment |
| Fraction addition | Finding LCD |
| Gear ratios | Teeth alignment period |
| Display refresh | Frame synchronization |

---

## Summary Table

| Concept | Formula | Key Note |
|---------|---------|----------|
| LCM from GCD | (a / GCD) × b | Divide first to avoid overflow |
| LCM of array | Reduce pairwise | Can grow exponentially |
| LCM property | a×b = GCD×LCM | Fundamental identity |
| Coprime LCM | LCM = a × b when GCD=1 | Maximum LCM for given product |
| LCM(1..n) | ≈ eⁿ | Grows very fast |

---

## Quick Revision Questions

1. **Why must we divide before multiplying** when computing LCM?
2. **Compute LCM(24, 36, 48)** step by step.
3. **If GCD(a,b) = 12 and LCM(a,b) = 360**, what is a × b?
4. **What is LCM(1, 2, 3, ..., 10)?** Compute it.
5. **Two clocks chime** every 15 and 20 minutes. How often do they chime together?
6. **Can LCM(a, b) ever be less than** max(a, b)? Why or why not?

---

[← Previous: Extended Euclidean](02-extended-euclidean-algorithm.md) | [Back to README](../README.md) | [Next: GCD of Array →](04-gcd-of-array.md)
