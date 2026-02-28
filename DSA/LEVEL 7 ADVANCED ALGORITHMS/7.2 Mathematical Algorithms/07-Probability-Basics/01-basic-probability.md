# Chapter 1: Basic Probability

[← Previous Unit: Catalan Numbers](../06-Combinatorics/06-catalan-numbers.md) | [Back to README](../README.md) | [Next: Expected Value →](02-expected-value.md)

---

## Overview

Probability underpins randomized algorithms, expected value analysis, and many competitive programming problems. This chapter covers foundations: sample spaces, events, and core rules.

---

## 1.1 Definitions

```
  ┌──────────────────────────────────────────────────────────┐
  │  Experiment:     A process with uncertain outcomes       │
  │  Sample Space S: Set of all possible outcomes            │
  │  Event A:        A subset of S                           │
  │  Probability:    P(A) = |A| / |S|  (for equally likely)  │
  │                                                          │
  │  Axioms:                                                 │
  │  1. 0 ≤ P(A) ≤ 1                                        │
  │  2. P(S) = 1                                             │
  │  3. P(A ∪ B) = P(A) + P(B)  if A ∩ B = ∅               │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.2 Addition Rule

```
  P(A ∪ B) = P(A) + P(B) - P(A ∩ B)
  
  Venn diagram:
  
  ┌───────────────────────────────┐
  │           S                   │
  │   ┌────────────┐              │
  │   │   A  ┌─────┼────┐        │
  │   │      │ A∩B │    │        │
  │   │      │     │  B │        │
  │   └──────┼─────┘    │        │
  │          └──────────┘        │
  └───────────────────────────────┘
  
  Inclusion-Exclusion (3 events):
  P(A∪B∪C) = P(A)+P(B)+P(C)
             - P(A∩B)-P(A∩C)-P(B∩C)
             + P(A∩B∩C)
```

---

## 1.3 Multiplication Rule

```
  Independent events:
  P(A ∩ B) = P(A) × P(B)
  
  Dependent events (conditional probability):
  P(A ∩ B) = P(A) × P(B|A) = P(B) × P(A|B)
  
  where P(B|A) = P(A ∩ B) / P(A)  (probability of B given A)
```

---

## 1.4 Conditional Probability

```
             P(A ∩ B)
  P(A|B) = ──────────
               P(B)

  Example: Two dice rolled. 
  A = sum is 8, B = first die is 3
  
  P(B) = 1/6
  A ∩ B = {(3,5)} → P(A∩B) = 1/36
  P(A|B) = (1/36)/(1/6) = 1/6
  
  Given first die is 3, only one way to get sum 8: second die = 5
  So P = 1/6 ✓
```

---

## 1.5 Complement Rule

```
  P(A') = 1 - P(A)
  
  ★ Very useful! Often easier to count what we DON'T want.
  
  Example: Probability at least one head in 5 coin flips?
  P(at least 1 head) = 1 - P(no heads) = 1 - (1/2)⁵ = 31/32
```

---

## 1.6 Bayes' Theorem

```
              P(B|A) × P(A)
  P(A|B) = ───────────────────
                  P(B)

  Full form (law of total probability in denominator):
  
              P(B|A) × P(A)
  P(A|B) = ────────────────────────────────────
            P(B|A)×P(A) + P(B|A')×P(A')

  Example: Disease test
  - Prevalence P(D) = 1%
  - Sensitivity P(+|D) = 99%
  - Specificity P(-|D') = 95%  →  P(+|D') = 5%
  
  P(D|+) = (0.99 × 0.01) / (0.99×0.01 + 0.05×0.99)
          = 0.0099 / (0.0099 + 0.0495)
          = 0.0099 / 0.0594
          ≈ 16.7%
  
  Even with 99% accurate test, only ~17% chance of disease if positive!
```

---

## 1.7 Counting Probability with Combinatorics

```
  Many problems reduce to: P = favorable / total
  where both are computed using C(n, r), P(n, r), etc.
  
  Example: Probability that a random 5-card hand has all hearts?
  Favorable: C(13, 5) = 1287  (choose 5 hearts from 13)
  Total:     C(52, 5) = 2598960
  P = 1287 / 2598960 ≈ 0.000495
```

---

## 1.8 CP Problem Patterns

```
  ┌──────────────────────────────────────────────────────────┐
  │  Pattern 1: Complement counting                          │
  │  "At least one" → 1 - P(none)                          │
  │                                                          │
  │  Pattern 2: Multiplication principle                     │
  │  Sequential independent events → multiply probabilities  │
  │                                                          │
  │  Pattern 3: Conditional decomposition                    │
  │  Case analysis: P(A) = Σ P(A|Bᵢ) × P(Bᵢ)              │
  │                                                          │
  │  Pattern 4: Modular probability                          │
  │  Answer = P × Q⁻¹ mod M (modular inverse of denominator)│
  └──────────────────────────────────────────────────────────┘
  
  In competitive programming, probability P/Q mod M:
  Answer = P × modInverse(Q, M) % M
```

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Basic probability | P(A) = \|A\| / \|S\| |
| Complement | P(A') = 1 - P(A) |
| Addition | P(A∪B) = P(A)+P(B)-P(A∩B) |
| Multiplication | P(A∩B) = P(A)×P(B\|A) |
| Independence | P(A∩B) = P(A)×P(B) |
| Conditional | P(A\|B) = P(A∩B)/P(B) |
| Bayes | P(A\|B) = P(B\|A)P(A)/P(B) |

---

## Quick Revision Questions

1. **What is** P(at least one 6) when rolling 4 dice?
2. **Two cards** drawn without replacement. P(both aces)?
3. **Explain** why Bayes' theorem is counter-intuitive in the disease test example.
4. **A bag** has 5 red, 3 blue balls. Draw 2. P(both same color)?
5. **How does** competitive programming typically represent probability answers?
6. **Prove** the inclusion-exclusion formula for 3 events.

---

[← Previous Unit: Catalan Numbers](../06-Combinatorics/06-catalan-numbers.md) | [Back to README](../README.md) | [Next: Expected Value →](02-expected-value.md)
