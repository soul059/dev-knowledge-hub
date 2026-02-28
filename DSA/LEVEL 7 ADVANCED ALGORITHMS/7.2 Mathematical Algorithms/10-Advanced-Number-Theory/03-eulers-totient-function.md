# Chapter 3: Euler's Totient Function

[← Previous: Chinese Remainder Theorem](02-chinese-remainder-theorem.md) | [Back to README](../README.md) | [Next: Multiplicative Inverse →](04-multiplicative-inverse.md)

---

## Overview

**Euler's totient function** $\phi(n)$ counts integers in $[1, n]$ that are coprime to $n$. It generalises Fermat's Little Theorem to composite moduli and underpins RSA, order theory, and many counting arguments.

---

## 3.1 Definition

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │   φ(n) = count of integers k, 1 ≤ k ≤ n, gcd(k,n) = 1 │
  │                                                          │
  │   Examples:                                              │
  │   φ(1)  = 1       {1}                                   │
  │   φ(6)  = 2       {1, 5}                                │
  │   φ(7)  = 6       {1,2,3,4,5,6}   (prime!)             │
  │   φ(8)  = 4       {1, 3, 5, 7}                         │
  │   φ(12) = 4       {1, 5, 7, 11}                        │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.2 Formula via Prime Factorisation

```
  If n = p1^a1 × p2^a2 × ... × pk^ak, then:
  
    φ(n) = n × (1 - 1/p1) × (1 - 1/p2) × ... × (1 - 1/pk)
  
  Equivalently:
  
    φ(n) = n × Π (1 - 1/pi)  for each distinct prime pi | n
  
  ┌──────────────────────────────────────────────────┐
  │  Example: φ(12)                                  │
  │  12 = 2² × 3¹                                   │
  │  φ(12) = 12 × (1 - 1/2) × (1 - 1/3)           │
  │        = 12 × 1/2 × 2/3                         │
  │        = 4  ✓                                    │
  │                                                  │
  │  Example: φ(36)                                  │
  │  36 = 2² × 3²                                   │
  │  φ(36) = 36 × 1/2 × 2/3 = 12                   │
  └──────────────────────────────────────────────────┘
```

---

## 3.3 Key Properties

```
  ┌──────────────────────────────────────────────────────────┐
  │  Property                        │  Formula             │
  ├──────────────────────────────────┼──────────────────────┤
  │  p prime                         │  φ(p) = p - 1       │
  │  p^k (prime power)               │  p^k - p^(k-1)     │
  │  Multiplicative                  │  φ(mn) = φ(m)φ(n)  │
  │    (when gcd(m,n)=1)             │    if gcd(m,n) = 1   │
  │  Divisor sum                     │  Σ φ(d) = n         │
  │    (d | n)                       │    for all d | n     │
  │  Euler's theorem                 │  a^φ(n) ≡ 1 (mod n)│
  │    (gcd(a,n) = 1)               │                      │
  └──────────────────────────────────┴──────────────────────┘
```

### Divisor Sum Property Trace

```
  n = 6:  divisors of 6 are {1, 2, 3, 6}
  φ(1) + φ(2) + φ(3) + φ(6) = 1 + 1 + 2 + 2 = 6 ✓
  
  n = 12: divisors of 12 are {1, 2, 3, 4, 6, 12}
  φ(1)+φ(2)+φ(3)+φ(4)+φ(6)+φ(12) = 1+1+2+2+2+4 = 12 ✓
```

---

## 3.4 Single Value Computation

```
FUNCTION eulerTotient(n):
    result = n
    p = 2
    WHILE p × p ≤ n:
        IF n % p == 0:
            WHILE n % p == 0:
                n = n / p
            result = result - result / p     // result *= (1 - 1/p)
        p++
    IF n > 1:
        result = result - result / n         // remaining prime factor
    RETURN result
```

### Trace: φ(60)

```
  n = 60, result = 60
  
  p=2: 60%2==0 → divide out: n=15
       result = 60 - 60/2 = 30
  
  p=3: 15%3==0 → divide out: n=5
       result = 30 - 30/3 = 20
  
  p=4: 4×4=16 > 5 → exit loop
  
  n=5 > 1: result = 20 - 20/5 = 16
  
  φ(60) = 16 ✓
  
  Verify: 60 = 2²×3×5
  φ(60) = 60 × (1-1/2) × (1-1/3) × (1-1/5) = 60 × 1/2 × 2/3 × 4/5 = 16 ✓
```

**Time:** $O(\sqrt{n})$ | **Space:** $O(1)$

---

## 3.5 Sieve of Totients

```
  Compute φ(i) for all i from 1 to n simultaneously.
  
  Similar to Sieve of Eratosthenes:
  
  FUNCTION totientSieve(n):
      phi[0..n]
      FOR i = 0 TO n:
          phi[i] = i            // initialise to identity
      
      FOR p = 2 TO n:
          IF phi[p] == p:       // p is prime (not yet modified)
              FOR j = p TO n STEP p:
                  phi[j] = phi[j] - phi[j] / p    // multiply by (1 - 1/p)
      
      RETURN phi
```

### Trace: Sieve for n = 12

```
  Initial: phi = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
  
  p=2 (prime): update j=2,4,6,8,10,12
    phi[2]  = 2  - 2/2  = 1
    phi[4]  = 4  - 4/2  = 2
    phi[6]  = 6  - 6/2  = 3
    phi[8]  = 8  - 8/2  = 4
    phi[10] = 10 - 10/2 = 5
    phi[12] = 12 - 12/2 = 6
  
  p=3 (prime): update j=3,6,9,12
    phi[3]  = 3  - 3/3  = 2
    phi[6]  = 3  - 3/3  = 2
    phi[9]  = 9  - 9/3  = 6
    phi[12] = 6  - 6/3  = 4
  
  p=5 (prime): update j=5,10
    phi[5]  = 5  - 5/5  = 4
    phi[10] = 5  - 5/5  = 4
  
  p=7 (prime): update j=7
    phi[7]  = 7  - 7/7  = 6
  
  p=11 (prime): update j=11
    phi[11] = 11 - 11/11 = 10
  
  Result: phi = [0, 1, 1, 2, 2, 4, 2, 6, 4, 6, 4, 10, 4]
                       ✓  ✓  ✓  ✓  ✓  ✓  ✓  ✓  ✓   ✓  ✓
```

**Time:** $O(n \log \log n)$ | **Space:** $O(n)$

---

## 3.6 Euler's Theorem

```
  ┌──────────────────────────────────────────────────────────┐
  │  If gcd(a, n) = 1, then:                                │
  │                                                          │
  │          a^φ(n) ≡ 1  (mod n)                            │
  │                                                          │
  │  Generalises Fermat's Little Theorem (where n = prime): │
  │  φ(p) = p-1  ⟹  a^(p-1) ≡ 1 (mod p)                  │
  │                                                          │
  │  Application: a^(-1) ≡ a^(φ(n)-1) (mod n)              │
  │  (works for any n, not just primes!)                    │
  └──────────────────────────────────────────────────────────┘
  
  Example: 3^φ(10) mod 10
    φ(10) = 10 × (1-1/2)(1-1/5) = 4
    3^4 = 81 mod 10 = 1 ✓
```

---

## 3.7 Exponent Reduction (General)

```
  For arbitrary a and n:
  
  ┌──────────────────────────────────────────────────────────┐
  │  If gcd(a,n) = 1:                                       │
  │    a^k ≡ a^(k mod φ(n)) (mod n)                        │
  │                                                          │
  │  If gcd(a,n) ≠ 1 and k ≥ log₂(n):                     │
  │    a^k ≡ a^(φ(n) + (k mod φ(n))) (mod n)               │
  │                                                          │
  │  ★ This is useful for towers of exponents:              │
  │    2^(2^(2^...)) mod n                                   │
  │    Reduce top-down using φ repeatedly!                  │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.8 Applications

```
  1. RSA cryptosystem:  d = e^(-1) mod φ(n) = e^(-1) mod (p-1)(q-1)
  2. Modular inverse for composite moduli
  3. Counting generators of cyclic groups (φ(n) generators for Z_n)
  4. Primitive root existence (exists iff n ∈ {1,2,4,p^k,2p^k})
  5. Reduced residue systems
  6. Iterated power towers (tetration) modulo n
```

---

## Summary Table

| Operation | Time | Method |
|-----------|------|--------|
| Single φ(n) | O(√n) | Trial division |
| All φ(1..n) | O(n log log n) | Sieve |
| Using φ(n) for mod inverse | O(√n + log n) | Compute φ, then fast power |
| Exponent reduction | O(√n) | a^k mod n → a^(k mod φ(n)) mod n |

---

## Quick Revision Questions

1. **Compute** φ(30) using the product formula.
2. **Prove** that Σ φ(d) = n for d dividing n, using n = 10.
3. **Why is** totient multiplicative and what condition is required?
4. **Implement** a totient sieve and verify φ(15) = 8.
5. **Use** Euler's theorem to find 7^222 mod 10.
6. **Explain** how φ is used in RSA key generation.

---

[← Previous: Chinese Remainder Theorem](02-chinese-remainder-theorem.md) | [Back to README](../README.md) | [Next: Multiplicative Inverse →](04-multiplicative-inverse.md)
