# Chapter 4: Power Function

## Overview
Computing x^n recursively introduces an important optimization — **fast exponentiation** (exponentiation by squaring), which reduces time from O(n) to O(log n). This technique appears frequently in competitive programming and cryptography.

---

## Naive Recursive Approach

### Pseudocode
```
function power(x, n):
    if n == 0: return 1          // Base case: x^0 = 1
    return x * power(x, n - 1)   // x^n = x × x^(n-1)
```

### Trace: power(2, 5)

```
power(2, 5) = 2 * power(2, 4)
                    = 2 * power(2, 3)
                              = 2 * power(2, 2)
                                        = 2 * power(2, 1)
                                                  = 2 * power(2, 0)
                                                            = 1  ← BASE
                                                  = 2 * 1 = 2
                                        = 2 * 2 = 4
                              = 2 * 4 = 8
                    = 2 * 8 = 16
            = 2 * 16 = 32 ✓

Complexity: O(n) time, O(n) space — can we do better?
```

---

## Optimized: Fast Exponentiation (Exponentiation by Squaring)

### Key Insight

```
┌──────────────────────────────────────────────┐
│          FAST POWER KEY INSIGHT               │
│                                              │
│  x^n = (x^(n/2))^2        if n is EVEN       │
│  x^n = x * (x^(n/2))^2   if n is ODD        │
│                                              │
│  Example:                                    │
│  2^10 = (2^5)^2                              │
│  2^5  = 2 * (2^2)^2                          │
│  2^2  = (2^1)^2                              │
│  2^1  = 2 * (2^0)^2                          │
│  2^0  = 1                                    │
│                                              │
│  Only 4 steps instead of 10!                 │
└──────────────────────────────────────────────┘
```

### Pseudocode
```
function fastPower(x, n):
    if n == 0: return 1              // Base case
    
    half = fastPower(x, n / 2)      // Compute half power ONCE
    
    if n is even:
        return half * half           // Square it
    else:
        return x * half * half       // x times square
```

### Trace: fastPower(2, 10)

```
fastPower(2, 10)
│  n=10 (even)
│  half = fastPower(2, 5)
│         │  n=5 (odd)
│         │  half = fastPower(2, 2)
│         │         │  n=2 (even)
│         │         │  half = fastPower(2, 1)
│         │         │         │  n=1 (odd)
│         │         │         │  half = fastPower(2, 0)
│         │         │         │         return 1  ← BASE
│         │         │         │  return 2 * 1 * 1 = 2
│         │         │  return 2 * 2 = 4
│         │  return 2 * 4 * 4 = 32
│  return 32 * 32 = 1024  ✓  (2^10 = 1024)

Only 5 calls instead of 11!
```

### Visual Comparison

```
NAIVE (2^10):                    FAST (2^10):
power(2,10)                      fastPower(2,10)
  power(2,9)                       fastPower(2,5)
    power(2,8)                       fastPower(2,2)
      power(2,7)                       fastPower(2,1)
        power(2,6)                       fastPower(2,0) ← BASE
          power(2,5)               
            power(2,4)           4 recursive calls
              power(2,3)         vs
                power(2,2)       10 recursive calls!
                  power(2,1)
                    power(2,0)

10 calls                         4 calls
O(n)                             O(log n)
```

---

## Handling Negative Exponents

```
function power(x, n):
    if n < 0:
        return 1.0 / power(x, -n)    // x^(-n) = 1/x^n
    if n == 0: return 1
    half = power(x, n / 2)
    if n is even:
        return half * half
    else:
        return x * half * half
```

---

## Complexity Analysis

```
┌──────────────────────────────────────────────────┐
│           COMPLEXITY COMPARISON                   │
│                                                  │
│  Naive:                                          │
│    Time:  O(n)                                   │
│    Space: O(n)                                   │
│    Recurrence: T(n) = T(n-1) + O(1)             │
│                                                  │
│  Fast Power:                                     │
│    Time:  O(log n)                               │
│    Space: O(log n)                               │
│    Recurrence: T(n) = T(n/2) + O(1)             │
│                                                  │
│  Why O(log n)?                                   │
│  n halves each step: n → n/2 → n/4 → ... → 1   │
│  Steps = log₂(n)                                │
│                                                  │
│  For n = 1,000,000:                              │
│    Naive:  1,000,000 operations                  │
│    Fast:   about 20 operations!                  │
└──────────────────────────────────────────────────┘
```

---

## Iterative Fast Power

```
function fastPowerIter(x, n):
    result = 1
    while n > 0:
        if n is odd:
            result = result * x
        x = x * x          // Square the base
        n = n / 2           // Halve the exponent
    return result
```

### Trace: fastPowerIter(2, 13)

```
13 in binary = 1101

┌──────┬───────┬──────────┬────────┐
│ Step │   n   │    x     │ result │
├──────┼───────┼──────────┼────────┤
│ init │  13   │    2     │   1    │
│  1   │  13→6 │  2→4     │ 1*2=2  │  (13 odd, multiply)
│  2   │  6→3  │  4→16    │   2    │  (6 even, skip)
│  3   │  3→1  │  16→256  │ 2*16=32│  (3 odd, multiply)
│  4   │  1→0  │  256→... │32*256= │  (1 odd, multiply)
│      │       │          │ 8192   │
└──────┴───────┴──────────┴────────┘

2^13 = 8192 ✓

Connection to binary:
13 = 1101₂ = 2³ + 2² + 2⁰ = 8 + 4 + 1
2^13 = 2^8 × 2^4 × 2^1 = 256 × 16 × 2 = 8192
```

---

## Application: Modular Exponentiation

Used in cryptography (RSA) — compute (x^n) % mod:

```
function modPow(x, n, mod):
    if n == 0: return 1
    half = modPow(x % mod, n / 2, mod)
    if n is even:
        return (half * half) % mod
    else:
        return (x * half * half) % mod
```

---

## Summary Table

| Aspect | Naive | Fast Power |
|--------|-------|------------|
| Base case | n == 0 → return 1 | n == 0 → return 1 |
| Recursive case | x * power(x, n-1) | half² or x * half² |
| Time | O(n) | O(log n) |
| Space | O(n) | O(log n) |
| Key insight | — | x^n = (x^(n/2))² |
| Use case | Simple | Large exponents, crypto |

---

## Quick Revision Questions

1. **What is the time complexity** of naive vs fast power?
2. **Why does halving the exponent** lead to O(log n)?
3. **How do you handle** negative exponents?
4. **Trace fastPower(3, 8)** step by step.
5. **How is binary representation** connected to fast exponentiation?
6. **What is modular exponentiation** and where is it used?

---

[← Previous: Sum of Array](03-sum-of-array.md) | [Next: Print N to 1 and 1 to N →](05-print-n-to-1-and-1-to-n.md)

[← Back to README](../README.md)
