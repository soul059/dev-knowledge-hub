# Chapter 6: Count of Divisors

[← Previous: Prime Factorization](05-prime-factorization.md) | [Back to README](../README.md) | [Next: Unit 3 — Euclidean Algorithm →](../03-GCD-and-LCM/01-euclidean-algorithm.md)

---

## Overview

Counting divisors efficiently is a frequent problem in competitive programming and number theory. We explore multiple approaches — from brute force to formula-based methods using prime factorization — and learn how to compute divisor counts for ranges of numbers.

---

## 6.1 The Divisor Function τ(n)

```
  ┌──────────────────────────────────────────────────────────┐
  │  τ(n) = number of positive divisors of n                 │
  │  (also written d(n) or σ₀(n))                            │
  │                                                          │
  │  If n = p₁^a₁ × p₂^a₂ × ... × pₖ^aₖ, then:            │
  │                                                          │
  │  τ(n) = (a₁ + 1)(a₂ + 1)...(aₖ + 1)                    │
  └──────────────────────────────────────────────────────────┘
```

### Why This Formula Works

```
  n = p₁^a₁ × p₂^a₂ × ... × pₖ^aₖ

  Any divisor d of n has the form:
  d = p₁^b₁ × p₂^b₂ × ... × pₖ^bₖ

  where 0 ≤ bᵢ ≤ aᵢ for each i

  Choices for b₁: 0, 1, ..., a₁  → (a₁ + 1) options
  Choices for b₂: 0, 1, ..., a₂  → (a₂ + 1) options
  ...
  Choices for bₖ: 0, 1, ..., aₖ  → (aₖ + 1) options

  By multiplication principle:
  τ(n) = (a₁ + 1) × (a₂ + 1) × ... × (aₖ + 1)
```

### Examples

```
  n = 12 = 2² × 3¹
  τ(12) = (2+1)(1+1) = 3 × 2 = 6
  Divisors: {1, 2, 3, 4, 6, 12}  ✓ (6 divisors)

  n = 60 = 2² × 3¹ × 5¹
  τ(60) = (2+1)(1+1)(1+1) = 3 × 2 × 2 = 12
  Divisors: {1,2,3,4,5,6,10,12,15,20,30,60}  ✓

  n = 1000 = 2³ × 5³
  τ(1000) = (3+1)(3+1) = 4 × 4 = 16

  n = p (prime)
  τ(p) = (1+1) = 2  [only 1 and p]

  n = p² (prime squared)
  τ(p²) = (2+1) = 3  [1, p, p²]
```

---

## 6.2 Method 1: Brute Force O(n)

```
FUNCTION countDivisorsNaive(n):
    count = 0
    FOR i = 1 TO n:
        IF n % i == 0:
            count += 1
    RETURN count
```

---

## 6.3 Method 2: √n Approach

```
FUNCTION countDivisors(n):
    count = 0
    FOR i = 1 TO √n:
        IF n % i == 0:
            count += 1           // i is a divisor
            IF i ≠ n / i:
                count += 1       // n/i is also a divisor
    RETURN count

Time: O(√n)
```

### Trace: countDivisors(72)

```
  √72 ≈ 8.49, check i = 1 to 8

  i=1: 72%1=0 → count=1, 72/1=72≠1 → count=2    {1, 72}
  i=2: 72%2=0 → count=3, 72/2=36≠2 → count=4    {1,2,36,72}
  i=3: 72%3=0 → count=5, 72/3=24≠3 → count=6    {1,2,3,24,36,72}
  i=4: 72%4=0 → count=7, 72/4=18≠4 → count=8    {1,2,3,4,18,24,36,72}
  i=5: 72%5=2 → skip
  i=6: 72%6=0 → count=9, 72/6=12≠6 → count=10   {1,2,3,4,6,12,18,24,36,72}
  i=7: 72%7=2 → skip
  i=8: 72%8=0 → count=11, 72/8=9≠8 → count=12   {1,2,3,4,6,8,9,12,18,24,36,72}

  Result: τ(72) = 12

  Verification: 72 = 2³ × 3²
  τ = (3+1)(2+1) = 12  ✓
```

---

## 6.4 Method 3: Formula from Factorization

```
FUNCTION countDivisorsFormula(n):
    result = 1
    d = 2
    WHILE d * d ≤ n:
        IF n % d == 0:
            exponent = 0
            WHILE n % d == 0:
                exponent += 1
                n /= d
            result *= (exponent + 1)
        d += 1
    IF n > 1:
        result *= 2    // remaining prime has exponent 1 → (1+1)
    RETURN result

Time: O(√n)
```

---

## 6.5 Divisor Count Sieve (for Range [1, N])

```
FUNCTION divisorCountSieve(N):
    d[1..N] = all zeros
    
    FOR i = 1 TO N:
        FOR j = i, 2i, 3i, ..., ≤ N:
            d[j] += 1
    
    RETURN d

Time: O(N log N)  [Harmonic series: N/1 + N/2 + ... + N/N ≈ N ln N]
Space: O(N)
```

### Trace: Divisor Sieve for N = 12

```
  i=1:  d[1]++, d[2]++, ..., d[12]++     (add 1 to all)
  i=2:  d[2]++, d[4]++, d[6]++, d[8]++, d[10]++, d[12]++
  i=3:  d[3]++, d[6]++, d[9]++, d[12]++
  i=4:  d[4]++, d[8]++, d[12]++
  i=5:  d[5]++, d[10]++
  i=6:  d[6]++, d[12]++
  i=7:  d[7]++
  i=8:  d[8]++
  i=9:  d[9]++
  i=10: d[10]++
  i=11: d[11]++
  i=12: d[12]++

  Result:
  n:    1  2  3  4  5  6  7  8  9  10  11  12
  d(n): 1  2  2  3  2  4  2  4  3   4   2   6
```

---

## 6.6 Highly Composite Numbers Revisited

```
  Numbers with maximum divisors for their size:

  n       τ(n)    Factorization       Divisor Grid
  ────────────────────────────────────────────────────
  1         1     -                   •
  2         2     2                   • •
  4         3     2²                  • • •
  6         4     2 × 3              • • • •
  12        6     2² × 3             • • • • • •
  24        8     2³ × 3             • • • • • • • •
  36        9     2² × 3²            • • • • • • • • •
  48       10     2⁴ × 3             • • • • • • • • • •
  60       12     2² × 3 × 5        • • • • • • • • • • • •
  120      16     2³ × 3 × 5        ████████████████

  Key Pattern:
  - Use small primes with decreasing exponents
  - Exponents go: a₁ ≥ a₂ ≥ a₃ ≥ ...
  - Why? Small primes "cost" less (contribute less to n)
    but each +1 to exponent multiplies divisor count
```

---

## 6.7 Maximum Divisors for Given Size

```
  Approximate maximum τ(n) for n ≤ N:

  N               max τ(n)    Example n
  ──────────────────────────────────────
  10²                 12      60
  10³                 32      840
  10⁴                 64      7560
  10⁵                128      83160
  10⁶                240      720720
  10⁹               1344      ~10⁹
  10¹²              6720      ~10¹²
  10¹⁸             103680     ~10¹⁸

  Rule of thumb: max τ(n) ≈ n^(1/(log log n))
  For n ≤ 10⁶: max τ(n) < 300
  For n ≤ 10⁹: max τ(n) < 1500
```

---

## 6.8 Problem-Solving Patterns

### Pattern 1: Sum of Divisor Counts in Range

```
  Sum of τ(i) for i = 1 to n

  Answer: Σ τ(i) = Σᵢ₌₁ⁿ ⌊n/i⌋

  This counts for each d from 1 to n, how many numbers
  in [1,n] are divisible by d.

  For n = 6:
  ⌊6/1⌋ + ⌊6/2⌋ + ⌊6/3⌋ + ⌊6/4⌋ + ⌊6/5⌋ + ⌊6/6⌋
  = 6 + 3 + 2 + 1 + 1 + 1 = 14

  Verify: τ(1)+τ(2)+τ(3)+τ(4)+τ(5)+τ(6) = 1+2+2+3+2+4 = 14  ✓
```

### Pattern 2: Number with Exactly k Divisors

```
  Smallest n with exactly k divisors:
  
  k=1:  n=1
  k=2:  n=2          (primes have exactly 2)
  k=3:  n=4=2²       (p² has 3 divisors)
  k=4:  n=6=2×3      (p×q has 4 divisors)
  k=6:  n=12=2²×3    ((2+1)(1+1)=6)
  k=12: n=60=2²×3×5  ((2+1)(1+1)(1+1)=12)

  Strategy: Factor k, assign largest exponents to smallest primes
```

### Pattern 3: Count Pairs with Divisor Relationship

```
  Count pairs (a,b) where a | b in array.
  
  1. Count frequency of each number
  2. For each number x, count multiples of x in array
  3. Sum up: answer = Σ freq[x] × (freq[x in multiples] - freq[x])
```

---

## 6.9 Complexity Summary

| Method | Time | Space | Use Case |
|--------|------|-------|----------|
| Brute force | O(n) | O(1) | Never |
| √n check | O(√n) | O(1) | Single query |
| Formula (factorize first) | O(√n) | O(1) | Single query |
| Divisor sieve | O(N log N) | O(N) | All numbers [1, N] |
| SPF + formula | O(log n) per query | O(N) precompute | Multiple queries |

---

## Summary Table

| Concept | Formula/Method | Key Insight |
|---------|---------------|-------------|
| τ(n) formula | Π(aᵢ + 1) from factorization | Each exponent gives (a+1) choices |
| √n counting | Pair up d and n/d | Count pairs below √n |
| Sieve approach | For each i, mark all multiples | O(N log N) total |
| Max τ(n) | ~n^(1/log log n) | Grows slowly; < 1500 for n ≤ 10⁹ |
| Smallest with k divs | Assign largest exponents to smallest primes | Greedy strategy |

---

## Quick Revision Questions

1. **Derive τ(n) for n = 2⁴ × 3² × 7¹.** How many divisors?
2. **Trace the √n method** for counting divisors of 100.
3. **Explain the divisor sieve** — why is it O(N log N)?
4. **What is the smallest number** with exactly 24 divisors?
5. **Why do highly composite numbers** always use consecutive smallest primes?
6. **If n has 1000 divisors**, what can you say about n's prime factorization?

---

[← Previous: Prime Factorization](05-prime-factorization.md) | [Back to README](../README.md) | [Next: Unit 3 — Euclidean Algorithm →](../03-GCD-and-LCM/01-euclidean-algorithm.md)
