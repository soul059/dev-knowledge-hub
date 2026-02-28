# Chapter 5: Prime Factorization

[← Previous: Segmented Sieve](04-segmented-sieve.md) | [Back to README](../README.md) | [Next: Count of Divisors →](06-count-of-divisors.md)

---

## Overview

Prime factorization — decomposing a number into its prime factors — is one of the most fundamental operations in number theory. It's used in GCD/LCM computation, divisor counting, modular arithmetic, and forms the basis of RSA cryptography's security.

---

## 5.1 Trial Division Method

```
FUNCTION primeFactorize(n):
    factors = {}
    
    d = 2
    WHILE d * d ≤ n:
        WHILE n % d == 0:
            factors[d] += 1
            n = n / d
        d += 1
    
    IF n > 1:
        factors[n] = 1      // remaining prime factor
    
    RETURN factors

Time: O(√n)
Space: O(log n) — at most log₂(n) prime factors
```

### Trace: Factorize 360

```
  n = 360

  d=2: 360%2=0 → n=180, count[2]=1
       180%2=0 → n=90,  count[2]=2
        90%2=0 → n=45,  count[2]=3
        45%2≠0 → move on

  d=3: 45%3=0  → n=15,  count[3]=1
       15%3=0  → n=5,   count[3]=2
        5%3≠0  → move on

  d=4: 4²=16 > 5 → EXIT loop

  n=5 > 1 → count[5]=1

  Result: 360 = 2³ × 3² × 5¹  ✓

  Factorization tree:
           360
          /   \
         2    180
             /   \
            2    90
                /  \
               2   45
                  /  \
                 3   15
                    /  \
                   3    5
```

---

## 5.2 Optimized Trial Division (6k±1)

```
FUNCTION primeFactorizeOpt(n):
    factors = {}
    
    // Handle factor 2
    WHILE n % 2 == 0:
        factors[2] += 1
        n /= 2
    
    // Handle factor 3
    WHILE n % 3 == 0:
        factors[3] += 1
        n /= 3
    
    // Check 6k±1 forms
    d = 5
    WHILE d * d ≤ n:
        WHILE n % d == 0:
            factors[d] += 1
            n /= d
        WHILE n % (d+2) == 0:
            factors[d+2] += 1
            n /= (d+2)
        d += 6
    
    IF n > 1:
        factors[n] = 1
    
    RETURN factors

Time: O(√n / 3) — about 3× faster
```

---

## 5.3 Factorization Using SPF Sieve

For multiple factorizations, pre-compute the Smallest Prime Factor sieve.

```
// Pre-computation: O(n log log n) time, O(n) space
FUNCTION buildSPF(n):
    spf[i] = i for all i in [0..n]
    FOR i = 2 TO √n:
        IF spf[i] == i:     // i is prime
            FOR j = i*i TO n STEP i:
                IF spf[j] == j:
                    spf[j] = i
    RETURN spf

// Each factorization: O(log n) time!
FUNCTION factorizeWithSPF(n, spf):
    factors = {}
    WHILE n > 1:
        p = spf[n]
        WHILE spf[n] == p:
            factors[p] += 1
            n /= p
    RETURN factors
```

### Trace: Factorize 84 Using SPF

```
  SPF table (partial):
  n:   2  3  4  5  6  7  ...  12  ...  14  ...  21  ...  42  ...  84
  spf: 2  3  2  5  2  7  ...   2  ...   2  ...   3  ...   2  ...   2

  Factorize 84:
  Step 1: spf[84] = 2, divide out all 2's
          84/2 = 42, spf[42] = 2, 42/2 = 21, spf[21] = 3 ≠ 2 → STOP
          factors[2] = 2

  Step 2: spf[21] = 3, divide out all 3's
          21/3 = 7, spf[7] = 7 ≠ 3 → STOP
          factors[3] = 1

  Step 3: spf[7] = 7, divide out all 7's
          7/7 = 1 → STOP
          factors[7] = 1

  Result: 84 = 2² × 3 × 7  ✓
  Only 5 divisions! (vs O(√84) ≈ 9 with trial division)
```

---

## 5.4 Comparison of Methods

```
  ┌──────────────────────────────────────────────────────────────┐
  │              METHOD COMPARISON                               │
  ├───────────────────┬──────────┬───────────┬──────────────────┤
  │ Method            │ Time     │ Space     │ Best for         │
  ├───────────────────┼──────────┼───────────┼──────────────────┤
  │ Trial division    │ O(√n)    │ O(log n)  │ Single large n   │
  │ 6k±1 optimized    │ O(√n/3)  │ O(log n)  │ Single large n   │
  │ SPF sieve         │ O(log n) │ O(N) pre  │ Many queries ≤ N │
  │ Pollard's rho     │ O(n^¼)   │ O(1)      │ Very large n     │
  └───────────────────┴──────────┴───────────┴──────────────────┘

  Choosing the right method:
  
           Single query?            Multiple queries?
               │                         │
               ▼                         ▼
         n ≤ 10¹²?              n ≤ 10⁷?
           │    │                  │     │
          YES   NO               YES    NO
           │    │                  │     │
           ▼    ▼                  ▼     ▼
        Trial  Pollard's       SPF    Trial per
        div.   rho             sieve  query
```

---

## 5.5 Properties of Prime Factorization

```
  Given n = p₁^a₁ × p₂^a₂ × ... × pₖ^aₖ

  ┌──────────────────────────────────────────────────────────┐
  │  Number of divisors:    τ(n) = Π(aᵢ + 1)               │
  │  Sum of divisors:       σ(n) = Π(pᵢ^(aᵢ+1) - 1)/(pᵢ-1)│
  │  Euler's totient:       φ(n) = n × Π(1 - 1/pᵢ)        │
  │  Number of prime factors (with mult.): Ω(n) = Σ aᵢ     │
  │  Number of distinct primes: ω(n) = k                    │
  │                                                          │
  │  GCD: take MIN of exponents                              │
  │  LCM: take MAX of exponents                              │
  └──────────────────────────────────────────────────────────┘
```

### Example: n = 720

```
  720 = 2⁴ × 3² × 5¹

  τ(720) = (4+1)(2+1)(1+1) = 5 × 3 × 2 = 30 divisors
  σ(720) = (2⁵-1)/(2-1) × (3³-1)/(3-1) × (5²-1)/(5-1)
         = 31 × 13 × 6 = 2418
  φ(720) = 720 × (1-1/2)(1-1/3)(1-1/5)
         = 720 × 1/2 × 2/3 × 4/5 = 192
  Ω(720) = 4 + 2 + 1 = 7
  ω(720) = 3
```

---

## 5.6 Problem-Solving Patterns

### Pattern 1: GCD via Factorization

```
  GCD(360, 540):
  360 = 2³ × 3² × 5¹
  540 = 2² × 3³ × 5¹
  
  GCD = 2^min(3,2) × 3^min(2,3) × 5^min(1,1)
      = 2² × 3² × 5 = 180
```

### Pattern 2: Check if n is a Perfect Power

```
  n is a perfect k-th power if ALL exponents in its
  factorization are divisible by k.

  Is 1296 a perfect square?
  1296 = 2⁴ × 3⁴
  All exponents (4, 4) divisible by 2? YES → 1296 = 36²

  Is 1296 a perfect 4th power?
  All exponents (4, 4) divisible by 4? YES → 1296 = 6⁴
```

### Pattern 3: Make n a Perfect Square

```
  Multiply by the product of primes with ODD exponents.

  n = 72 = 2³ × 3²
  Odd exponents: 2 has exponent 3 (odd)
  Multiply by 2 → 144 = 2⁴ × 3² → perfect square!
```

---

## 5.7 Factorization of Large Numbers

```
  For n up to 10¹⁸:
  - Trial division finds factors up to 10⁶ (in √n time with n=10¹²)
  - For larger n, need Pollard's rho or other advanced methods

  Practical approach:
  1. Trial division for small factors (up to 10⁶)
  2. If remaining n > 1 and n ≤ 10¹², check if it's prime
  3. If composite, it has exactly 2 prime factors
     (since we removed all factors up to 10⁶,
      and n has at most 2 prime factors > 10⁶)
     Use Pollard's rho or similar
```

---

## Summary Table

| Concept | Method | Complexity |
|---------|--------|------------|
| Trial division | Divide by 2, then odds up to √n | O(√n) |
| 6k±1 optimized | Check 2, 3, then 6k±1 forms | O(√n/3) |
| SPF sieve factorization | Precompute array, divide by spf[n] | O(log n) per query |
| Properties from factors | τ, σ, φ formulas | O(k) given factors |
| Perfect power check | All exponents divisible by k | O(log n) |

---

## Quick Revision Questions

1. **Walk through the trial division** of n = 2520. What is its factorization?
2. **Why is SPF sieve factorization O(log n)?** What's the maximum number of steps?
3. **Given 360 = 2³ × 3² × 5, compute** the number of divisors, sum of divisors, and Euler's totient.
4. **How do you determine** if a number is a perfect cube from its factorization?
5. **Compare trial division vs SPF sieve** — when would you use each?
6. **What is the minimum number to multiply 50 by** to make it a perfect square?

---

[← Previous: Segmented Sieve](04-segmented-sieve.md) | [Back to README](../README.md) | [Next: Count of Divisors →](06-count-of-divisors.md)
