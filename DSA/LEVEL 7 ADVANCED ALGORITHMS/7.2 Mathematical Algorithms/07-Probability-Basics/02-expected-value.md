# Chapter 2: Expected Value

[← Previous: Basic Probability](01-basic-probability.md) | [Back to README](../README.md) | [Next: Random Algorithms →](03-random-algorithms.md)

---

## Overview

**Expected value** (expectation) is the average outcome of a random variable over many trials. It is the single most important concept for analyzing randomized algorithms and probabilistic problems.

$$E[X] = \sum_{x} x \cdot P(X = x)$$

---

## 2.1 Definition and Intuition

```
  ┌──────────────────────────────────────────────────────────┐
  │  E[X] = Σ xᵢ × P(X = xᵢ)                              │
  │                                                          │
  │  "Weighted average of all possible outcomes"             │
  │                                                          │
  │  Example: Fair die                                       │
  │  E[X] = 1×(1/6) + 2×(1/6) + ... + 6×(1/6) = 3.5       │
  │                                                          │
  │  You never roll 3.5, but on average that's the outcome. │
  └──────────────────────────────────────────────────────────┘
```

---

## 2.2 Linearity of Expectation

```
  ★ THE MOST POWERFUL PROPERTY ★
  
  E[X + Y] = E[X] + E[Y]
  
  This holds ALWAYS — even if X and Y are NOT independent!
  
  E[aX + b] = a × E[X] + b
  
  Example: Roll 2 dice. Expected sum?
  E[sum] = E[die₁] + E[die₂] = 3.5 + 3.5 = 7
  (No need to enumerate all 36 outcomes!)
```

---

## 2.3 Indicator Random Variables

```
  Xᵢ = 1 if event i occurs, 0 otherwise
  E[Xᵢ] = P(event i occurs)
  
  ★ Combined with linearity, this is incredibly powerful!
  
  Example: Expected number of heads in n fair coin flips
  Let Xᵢ = 1 if flip i is heads
  E[Σ Xᵢ] = Σ E[Xᵢ] = Σ (1/2) = n/2
  
  Example: Expected number of fixed points in a random permutation
  Let Xᵢ = 1 if element i stays in position i
  E[Xᵢ] = 1/n  (only 1 out of n positions is "fixed")
  E[total fixed points] = Σ E[Xᵢ] = n × (1/n) = 1
  
  → On average, exactly 1 fixed point in any permutation!
```

---

## 2.4 Classic Expected Value Problems

### Coupon Collector Problem

```
  There are n distinct coupons. Each trial gives a random coupon.
  Expected trials to collect ALL n?
  
  Phase i: You have i-1 distinct, need one more new one.
  P(new coupon) = (n - i + 1) / n
  Expected trials in phase i = n / (n - i + 1)
  
  E[total] = Σ n/(n-i+1) for i = 1 to n
           = n × (1 + 1/2 + 1/3 + ... + 1/n)
           = n × Hₙ   (harmonic number)
           ≈ n × ln(n) + 0.577n
  
  n=10: E ≈ 10 × 2.93 ≈ 29.3 trials
  n=50: E ≈ 50 × 4.50 ≈ 225 trials
```

### Geometric Distribution

```
  Repeat until first success. P(success) = p on each trial.
  
  E[trials] = 1/p
  
  Example: Expected rolls until a 6?
  E = 1/(1/6) = 6 rolls
```

### Hat-Check Problem

```
  n people check hats. Hats returned randomly.
  Expected matches (derangement complement)?
  
  By indicator variables: E = n × (1/n) = 1
  Exactly 1 expected match regardless of n!
```

---

## 2.5 Conditional Expectation

```
  E[X] = Σ E[X | Aᵢ] × P(Aᵢ)
  (law of total expectation)
  
  Example: Expected rolls of a fair die to get a 6.
  Let E = answer.
  
  On each roll:
  - With P=1/6: success, done in 1 roll
  - With P=5/6: failure, expected remaining is E again
  
  E = (1/6)(1) + (5/6)(1 + E)
  E = 1/6 + 5/6 + 5E/6
  E/6 = 1
  E = 6 ✓
```

---

## 2.6 Expected Value in Competitive Programming

```
  ┌──────────────────────────────────────────────────────────┐
  │  Problem types:                                          │
  │                                                          │
  │  1. "Expected number of operations"                      │
  │     → Set up recurrence using conditional expectation    │
  │                                                          │
  │  2. "Expected value of some sum"                         │
  │     → Use linearity + indicator variables                │
  │                                                          │
  │  3. Output as P/Q mod 10⁹+7                             │
  │     → Compute P × Q⁻¹ mod 10⁹+7                        │
  └──────────────────────────────────────────────────────────┘
  
  Computing expected value mod p:
  If E[X] = a/b, output a × modInverse(b, p) % p
```

---

## 2.7 Variance (Brief)

```
  Var(X) = E[X²] - (E[X])²
  SD(X) = √Var(X)
  
  If X, Y independent:
  Var(X + Y) = Var(X) + Var(Y)
  
  Fair die: Var = E[X²] - 3.5²
  E[X²] = (1+4+9+16+25+36)/6 = 91/6
  Var = 91/6 - 49/4 = 35/12 ≈ 2.92
```

---

## Summary Table

| Concept | Formula/Property |
|---------|-----------------|
| Expected value | E[X] = Σ x P(X=x) |
| Linearity | E[X+Y] = E[X]+E[Y] always |
| Indicator trick | E[Xᵢ] = P(event i) |
| Geometric | E = 1/p (trials until success) |
| Coupon collector | E = n × Hₙ ≈ n ln n |
| Conditional | E[X] = Σ E[X\|Aᵢ] P(Aᵢ) |
| Mod output | a/b mod p = a × b⁻¹ mod p |

---

## Quick Revision Questions

1. **Compute** the expected sum when rolling 3 fair dice.
2. **What is** the expected number of coin flips to get 2 consecutive heads?
3. **Prove** using indicator variables that expected fixed points in a permutation = 1.
4. **How many** trials are expected to collect all 6 types of cereal box toys?
5. **Solve** the expected value of: get a 6, then a 5 on a die (in sequence).
6. **Express** E[X] = 3/7 as an answer modulo 10⁹+7.

---

[← Previous: Basic Probability](01-basic-probability.md) | [Back to README](../README.md) | [Next: Random Algorithms →](03-random-algorithms.md)
