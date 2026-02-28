# Chapter 5: GCD and LCM (Introduction)

[← Previous: Composite Numbers](04-composite-numbers.md) | [Back to README](../README.md) | [Next: Euclidean Algorithm →](06-euclidean-algorithm.md)

---

## Overview

The Greatest Common Divisor (GCD) and Least Common Multiple (LCM) are two of the most fundamental operations in number theory. They appear in fraction simplification, scheduling problems, modular arithmetic, and countless algorithmic challenges.

---

## 5.1 Greatest Common Divisor (GCD)

```
  ┌──────────────────────────────────────────────────────────┐
  │  GCD(a, b) = the LARGEST positive integer that           │
  │              divides both a and b                         │
  │                                                          │
  │  Also called: HCF (Highest Common Factor)                │
  │                                                          │
  │  If GCD(a, b) = 1, then a and b are COPRIME             │
  │  (or relatively prime)                                   │
  └──────────────────────────────────────────────────────────┘
```

### Visual: Finding GCD(12, 18)

```
  Divisors of 12: {1, 2, 3, 4, 6, 12}
  Divisors of 18: {1, 2, 3, 6, 9, 18}

  Common divisors:
  12:  1   2   3   4       6       12
  18:  1   2   3       6       9       18
       ↑   ↑   ↑       ↑
    Common: {1, 2, 3, 6}

  GCD(12, 18) = 6  (the greatest one)
```

### GCD Using Prime Factorization

```
  Method: Take MINIMUM power of each common prime.

  12 = 2² × 3¹
  18 = 2¹ × 3²

  GCD = 2^min(2,1) × 3^min(1,2) = 2¹ × 3¹ = 6

  Another example:
  60  = 2² × 3¹ × 5¹
  90  = 2¹ × 3² × 5¹

  GCD = 2^min(2,1) × 3^min(1,2) × 5^min(1,1)
      = 2¹ × 3¹ × 5¹ = 30
```

### GCD Properties

```
  ┌──────────────────────────────────────────────────┐
  │  1. GCD(a, 0) = a                               │
  │  2. GCD(a, a) = a                               │
  │  3. GCD(a, b) = GCD(b, a)      [commutative]    │
  │  4. GCD(a, b) = GCD(a-b, b)    if a > b         │
  │  5. GCD(a, b) = GCD(a%b, b)    [Euclidean]      │
  │  6. GCD(ka, kb) = k × GCD(a,b)                  │
  │  7. GCD(a, b) divides (a - b)                   │
  │  8. If d = GCD(a,b), then GCD(a/d, b/d) = 1     │
  └──────────────────────────────────────────────────┘
```

---

## 5.2 Least Common Multiple (LCM)

```
  ┌──────────────────────────────────────────────────────────┐
  │  LCM(a, b) = the SMALLEST positive integer that          │
  │              is divisible by both a and b                 │
  └──────────────────────────────────────────────────────────┘
```

### Visual: Finding LCM(4, 6)

```
  Multiples of 4:  4,  8, 12, 16, 20, 24, 28, 32, 36, ...
  Multiples of 6:  6, 12, 18, 24, 30, 36, 42, ...
                       ↑           ↑       ↑
               Common: 12, 24, 36, ...
  
  LCM(4, 6) = 12  (the smallest common multiple)
```

### LCM Using Prime Factorization

```
  Method: Take MAXIMUM power of each prime.

  12 = 2² × 3¹
  18 = 2¹ × 3²

  LCM = 2^max(2,1) × 3^max(1,2) = 2² × 3² = 36

  Note the symmetry with GCD:
  GCD → min powers → largest common divisor
  LCM → max powers → smallest common multiple
```

---

## 5.3 The GCD-LCM Relationship

$$\text{GCD}(a, b) \times \text{LCM}(a, b) = a \times b$$

$$\text{LCM}(a, b) = \frac{a \times b}{\text{GCD}(a, b)}$$

```
  Proof using prime factorization:
  
  Let a = p₁^a₁ × p₂^a₂ × ...
      b = p₁^b₁ × p₂^b₂ × ...
  
  GCD = Π pᵢ^min(aᵢ,bᵢ)
  LCM = Π pᵢ^max(aᵢ,bᵢ)
  
  GCD × LCM = Π pᵢ^(min(aᵢ,bᵢ) + max(aᵢ,bᵢ))
             = Π pᵢ^(aᵢ + bᵢ)
             = (Π pᵢ^aᵢ) × (Π pᵢ^bᵢ)
             = a × b  ∎
  
  Verification: a=12, b=18
  GCD(12,18) = 6
  LCM(12,18) = 12×18/6 = 216/6 = 36
  Check: 6 × 36 = 216 = 12 × 18  ✓
```

### Venn Diagram Analogy

```
  12 = 2² × 3
  18 = 2  × 3²

  Factor "Venn Diagram":
  
  ┌─────────── 12 ────────────┐
  │                           │
  │   Only in 12    Shared    │  ┌─── Only in 18 ──┐
  │      2¹        2¹ × 3¹   │  │     3¹           │
  │                           │  │                  │
  └───────────────────────────┘  └──────────────────┘
  
  GCD = Shared part      = 2 × 3 = 6
  LCM = Everything       = 2² × 3² = 36
  a × b = All with overlap counted twice
```

---

## 5.4 Computing GCD — Methods Comparison

### Method 1: Listing Divisors

```
  GCD(48, 36):
  Divisors of 48: {1, 2, 3, 4, 6, 8, 12, 16, 24, 48}
  Divisors of 36: {1, 2, 3, 4, 6, 9, 12, 18, 36}
  Common: {1, 2, 3, 4, 6, 12}
  GCD = 12

  Time: O(n) — slow!
```

### Method 2: Prime Factorization

```
  GCD(48, 36):
  48 = 2⁴ × 3¹
  36 = 2² × 3²
  GCD = 2² × 3¹ = 12

  Time: O(√n) for factorization
```

### Method 3: Euclidean Algorithm (Best!)

```
  GCD(48, 36):
  48 = 36 × 1 + 12    →  GCD(48, 36) = GCD(36, 12)
  36 = 12 × 3 + 0     →  GCD(36, 12) = 12

  GCD = 12

  Time: O(log(min(a,b)))  — very fast!
```

---

## 5.5 Applications of GCD and LCM

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    APPLICATIONS                              │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  GCD Applications:                                           │
  │  • Fraction simplification: 12/18 → divide by GCD(12,18)=6 │
  │    → 2/3                                                     │
  │  • Determine if tiles fit perfectly in a room                │
  │  • Cryptography (key generation)                             │
  │  • Checking coprimality                                      │
  │                                                              │
  │  LCM Applications:                                           │
  │  • Adding fractions: LCD = LCM of denominators              │
  │  • Scheduling: when events coincide                          │
  │    Bus A every 15 min, Bus B every 20 min                    │
  │    Both together every LCM(15,20) = 60 min                  │
  │  • Gear ratios                                               │
  │  • Music: beat synchronization                               │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

### Example: Tile Problem

```
  Room: 12m × 18m
  Question: What is the largest square tile that fits perfectly?
  
  Answer: Side = GCD(12, 18) = 6m
  
  ┌──────────────────────────────┐
  │      │      │      │        │
  │  6×6 │  6×6 │  6×6 │        │  18m
  │      │      │      │        │
  ├──────┼──────┼──────┤        │
  │      │      │      │        │
  │  6×6 │  6×6 │  6×6 │        │
  │      │      │      │        │
  └──────┴──────┴──────┘────────┘
           12m
  
  Number of tiles = (12/6) × (18/6) = 2 × 3 = 6 tiles
```

---

## 5.6 GCD/LCM of Multiple Numbers

```
  GCD(a, b, c) = GCD(GCD(a, b), c)
  LCM(a, b, c) = LCM(LCM(a, b), c)

  Example: GCD(12, 18, 24)
  Step 1: GCD(12, 18) = 6
  Step 2: GCD(6, 24) = 6
  Result: 6

  Example: LCM(4, 6, 10)
  Step 1: LCM(4, 6) = 12
  Step 2: LCM(12, 10) = 60
  Result: 60
```

---

## 5.7 Complexity Comparison

| Method | GCD Time | LCM Time | Space |
|--------|----------|----------|-------|
| List all divisors | O(min(a,b)) | O(a×b) | O(min(a,b)) |
| Prime factorization | O(√max(a,b)) | O(√max(a,b)) | O(log n) |
| Euclidean algorithm | O(log min(a,b)) | O(log min(a,b)) | O(1) |

---

## Summary Table

| Concept | Formula | Key Insight |
|---------|---------|-------------|
| GCD definition | Largest common divisor | Min of prime powers |
| LCM definition | Smallest common multiple | Max of prime powers |
| GCD-LCM relation | GCD × LCM = a × b | Fundamental identity |
| LCM from GCD | LCM = a×b / GCD | Avoids factorization |
| Coprime | GCD(a,b) = 1 | No common factors |
| Multiple numbers | Apply associatively | GCD(a,b,c) = GCD(GCD(a,b),c) |

---

## Quick Revision Questions

1. **What is the formal definition** of GCD and LCM?
2. **Prove that** GCD(a,b) × LCM(a,b) = a × b using prime factorization.
3. **Find GCD(84, 120)** using three different methods.
4. **Calculate LCM(15, 20, 35)** step by step.
5. **If two buses depart** every 12 and 18 minutes, when will they depart together?
6. **What does it mean** for two numbers to be coprime? Give 3 examples of coprime pairs.

---

[← Previous: Composite Numbers](04-composite-numbers.md) | [Back to README](../README.md) | [Next: Euclidean Algorithm →](06-euclidean-algorithm.md)
