# Chapter 3: Prime Numbers

[← Previous: Factors and Multiples](02-factors-and-multiples.md) | [Back to README](../README.md) | [Next: Composite Numbers →](04-composite-numbers.md)

---

## Overview

Prime numbers are the "atoms" of number theory — every integer greater than 1 can be uniquely expressed as a product of primes. Understanding primes is fundamental to cryptography, hashing, and countless algorithmic problems.

---

## 3.1 Definition and Basics

```
  ┌──────────────────────────────────────────────────────────┐
  │  PRIME NUMBER:                                           │
  │    A natural number p > 1 that has exactly               │
  │    TWO distinct positive divisors: 1 and p itself.       │
  │                                                          │
  │  NOT PRIME:                                              │
  │    • 1 is NOT prime (only one divisor)                   │
  │    • 0 is NOT prime                                      │
  │    • Negative numbers are NOT prime                      │
  │    • 2 is the ONLY even prime                            │
  └──────────────────────────────────────────────────────────┘
```

### First 25 Primes

```
  2  3  5  7  11  13  17  19  23  29  31  37  41  43  47
  53  59  61  67  71  73  79  83  89  97

  Prime number line up to 30:
  ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
  │1 │2 │3 │4 │5 │6 │7 │8 │9 │10│11│12│13│14│15│16│17│18│19│20│21│22│23│24│25│26│27│28│29│30│
  └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
      P  P     P     P        P     P        P        P     P        P        P        P
  
  P = Prime,  unmarked = composite or 1
```

---

## 3.2 Fundamental Theorem of Arithmetic

$$\text{Every integer } n > 1 \text{ has a UNIQUE prime factorization (up to order).}$$

```
  Examples:
  
  60  = 2² × 3 × 5           (unique!)
  100 = 2² × 5²              (unique!)
  13  = 13                    (prime itself)
  360 = 2³ × 3² × 5          (unique!)
  
  Factorization Tree for 60:
  
            60
           / \
          2   30
             / \
            2   15
               / \
              3   5
  
  Reading leaves: 2 × 2 × 3 × 5 = 60 = 2² × 3 × 5
```

---

## 3.3 Properties of Prime Numbers

```
  ┌──────────────────────────────────────────────────────────────┐
  │                 PRIME NUMBER PROPERTIES                      │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  1. There are INFINITELY many primes (Euclid's proof)       │
  │                                                              │
  │  2. 2 is the only even prime                                 │
  │                                                              │
  │  3. Every prime > 3 is of the form 6k ± 1                   │
  │     (because 6k, 6k+2, 6k+4 are even; 6k+3 divisible by 3) │
  │                                                              │
  │  4. If p is prime and p | (a × b), then p | a or p | b      │
  │     (Euclid's lemma)                                         │
  │                                                              │
  │  5. Prime Counting Function π(n) ≈ n / ln(n)                │
  │     (Prime Number Theorem)                                   │
  │                                                              │
  │  6. Bertrand's Postulate: For n > 1, there exists a prime   │
  │     p such that n < p < 2n                                   │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

### Why Every Prime > 3 is 6k ± 1

```
  Any integer can be written as:
  6k, 6k+1, 6k+2, 6k+3, 6k+4, 6k+5

  6k   = 2(3k)     → divisible by 2 and 3 → NOT prime
  6k+1 = ?         → COULD be prime
  6k+2 = 2(3k+1)   → divisible by 2 → NOT prime
  6k+3 = 3(2k+1)   → divisible by 3 → NOT prime
  6k+4 = 2(3k+2)   → divisible by 2 → NOT prime
  6k+5 = 6(k+1)-1  → COULD be prime (same as 6k-1)

  So all primes > 3 are of the form 6k-1 or 6k+1

  Verify: 5=6(1)-1 ✓  7=6(1)+1 ✓  11=6(2)-1 ✓  13=6(2)+1 ✓
```

---

## 3.4 Primality Testing — Naive Approach

### Basic: Check All Numbers up to n-1

```
FUNCTION isPrimeNaive(n):
    IF n < 2:
        RETURN false
    FOR i = 2 TO n - 1:
        IF n % i == 0:
            RETURN false
    RETURN true

Time: O(n)  — Very slow!
```

### Better: Check up to √n

```
FUNCTION isPrime(n):
    IF n < 2: RETURN false
    IF n < 4: RETURN true         // 2 and 3 are prime
    IF n % 2 == 0: RETURN false   // even numbers
    IF n % 3 == 0: RETURN false   // multiples of 3
    
    i = 5
    WHILE i * i ≤ n:
        IF n % i == 0 OR n % (i + 2) == 0:
            RETURN false
        i = i + 6                 // 6k±1 optimization
    RETURN true

Time: O(√n)
```

### Trace: Is 97 Prime?

```
  n = 97
  97 ≥ 2  ✓
  97 is odd  ✓
  97 % 3 = 1 ≠ 0  ✓
  
  i = 5:  5² = 25 ≤ 97? YES
          97 % 5  = 2 ≠ 0  ✓
          97 % 7  = 6 ≠ 0  ✓
  i = 11: 11² = 121 ≤ 97? NO → STOP
  
  97 is PRIME ✓  (checked only: 2, 3, 5, 7 — that's it!)
```

### Why √n is Sufficient

```
  If n is composite, n = a × b  where 1 < a ≤ b < n

  If both a > √n and b > √n:
     a × b > √n × √n = n  → contradiction!

  Therefore, at least one factor ≤ √n.

  So we only need to check potential divisors up to √n.

  √97 ≈ 9.85 → check: 2, 3, 5, 7  (primes up to 9)
```

---

## 3.5 Euclid's Proof: Infinitely Many Primes

```
  Proof by Contradiction:

  Assume there are finitely many primes: p₁, p₂, ..., pₖ

  Construct N = p₁ × p₂ × ... × pₖ + 1

  Case 1: N is prime → N is a prime not in our list → contradiction!
  Case 2: N is composite → some prime q divides N
          q must be one of p₁, ..., pₖ (our list has ALL primes)
          But N = p₁×p₂×...×pₖ + 1
          So N mod pᵢ = 1 for all i → no pᵢ divides N → contradiction!

  Therefore, there must be infinitely many primes. ∎

  Example: primes = {2, 3, 5}
  N = 2×3×5 + 1 = 31 → 31 is prime!  (not in list)
```

---

## 3.6 Prime Distribution

```
  Prime Counting Function π(n):

  n        π(n)     n/ln(n)    Ratio π(n)/(n/ln(n))
  ──────────────────────────────────────────────────
  10         4       4.3          0.93
  100       25      21.7          1.15
  1,000    168     144.8          1.16
  10,000  1,229   1,085.7        1.13
  100,000 9,592   8,685.9        1.10
  1M      78,498  72,382.4       1.08
  
  As n → ∞, the ratio → 1 (Prime Number Theorem)
  
  Density visualization (primes per 100 numbers):
  
  Range       Count   ██████████████████████████
  [1,100]       25    █████████████████████████
  [100,200]     21    █████████████████████
  [200,300]     16    ████████████████
  [300,400]     16    ████████████████
  [400,500]     17    █████████████████
  [900,1000]    14    ██████████████
  
  Primes become sparser as numbers grow larger.
```

---

## 3.7 Twin Primes and Special Primes

| Type | Definition | Examples |
|------|-----------|----------|
| **Twin primes** | Pairs (p, p+2) both prime | (3,5), (5,7), (11,13), (17,19) |
| **Cousin primes** | Pairs (p, p+4) both prime | (3,7), (7,11), (13,17) |
| **Mersenne primes** | 2ⁿ - 1 where result is prime | 3, 7, 31, 127, 8191 |
| **Fermat primes** | 2^(2ⁿ) + 1 where result is prime | 3, 5, 17, 257, 65537 |
| **Sophie Germain** | p and 2p+1 both prime | 2, 3, 5, 11, 23 |

---

## 3.8 Common Patterns in Problems

### Pattern: Smallest Prime Factor

```
  Problem: Find SPF for all numbers 1 to n (used in fast factorization)
  
  Build during sieve:
  n:    1  2  3  4  5  6  7  8  9  10  11  12
  SPF:  1  2  3  2  5  2  7  2  3   2  11   2
  
  Use SPF for O(log n) factorization:
  60 → SPF[60]=2 → 60/2=30 → SPF[30]=2 → 30/2=15 → SPF[15]=3 → 15/3=5 → prime
  Result: 2² × 3 × 5  ✓
```

### Pattern: Prime Gap

```
  Gap between consecutive primes:
  2→3: gap=1    3→5: gap=2    5→7: gap=2    7→11: gap=4
  
  Gaps can be arbitrarily large:
  n! + 2, n! + 3, ..., n! + n  are all composite → gap ≥ n-1
```

---

## 3.9 Complexity Analysis

| Operation | Time Complexity | Space |
|-----------|----------------|-------|
| isPrime (naive) | O(n) | O(1) |
| isPrime (√n check) | O(√n) | O(1) |
| isPrime (6k±1) | O(√n) but ~3× faster | O(1) |
| Sieve (covered in Unit 2) | O(n log log n) | O(n) |
| Trial division factorization | O(√n) | O(log n) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Prime definition | Exactly 2 divisors: 1 and itself |
| Fundamental theorem | Unique prime factorization |
| Only even prime | 2 |
| Form of primes > 3 | 6k ± 1 |
| Trial division bound | Check up to √n |
| π(n) approximation | n / ln(n) |
| Infinitely many primes | Euclid's proof |
| Smallest prime factor | Useful for fast factorization |

---

## Quick Revision Questions

1. **Why is 1 not considered a prime number?** What would break if it were?
2. **Explain why checking up to √n is sufficient** for primality testing.
3. **What is the 6k±1 optimization** and why does it work?
4. **Outline Euclid's proof** that there are infinitely many primes.
5. **Approximately how many primes** are there below 1,000,000?
6. **What is a Mersenne prime?** Give 3 examples.

---

[← Previous: Factors and Multiples](02-factors-and-multiples.md) | [Back to README](../README.md) | [Next: Composite Numbers →](04-composite-numbers.md)
