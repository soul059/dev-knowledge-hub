# Chapter 1: Probability Basics

> **Unit 3: Probability Theory** | **Topic**: Sample Space, Events, Axioms & Rules of Probability

---

## 1. Overview

Probability is the mathematical framework for quantifying uncertainty. In machine learning, nearly every
model — from logistic regression to deep neural networks — relies on probabilistic reasoning to make
predictions, estimate parameters, and evaluate confidence. This chapter covers the foundational language
and rules of probability that underpin all subsequent topics.

---

## 2. Sample Space and Events

### 2.1 Sample Space (Ω)

The **sample space** is the set of all possible outcomes of a random experiment.

| Experiment               | Sample Space Ω                          |
|--------------------------|-----------------------------------------|
| Coin flip                | {H, T}                                  |
| Rolling a die            | {1, 2, 3, 4, 5, 6}                      |
| Two coin flips           | {HH, HT, TH, TT}                       |
| Pixel intensity (8-bit)  | {0, 1, 2, ..., 255}                     |

### 2.2 Events

An **event** is any subset of the sample space.

- **Simple event**: A single outcome, e.g., {3} when rolling a die.
- **Compound event**: Multiple outcomes, e.g., {2, 4, 6} (rolling an even number).
- **Certain event**: Ω itself — always occurs.
- **Impossible event**: ∅ — never occurs.

---

## 3. Axioms of Probability (Kolmogorov's Axioms)

For any event A in sample space Ω:

```
Axiom 1 (Non-negativity):   P(A) ≥ 0
Axiom 2 (Normalization):    P(Ω) = 1
Axiom 3 (Additivity):       If A ∩ B = ∅, then P(A ∪ B) = P(A) + P(B)
```

Everything in probability theory is derived from these three axioms.

---

## 4. Fundamental Rules

### 4.1 Complement Rule

```
P(A') = 1 - P(A)
```

*"The probability that something does NOT happen equals 1 minus the probability it does."*

**Example**: If P(rain) = 0.3, then P(no rain) = 1 − 0.3 = 0.7.

### 4.2 Addition Rule (Union of Events)

```
General:              P(A ∪ B) = P(A) + P(B) - P(A ∩ B)
Mutually exclusive:   P(A ∪ B) = P(A) + P(B)          [when A ∩ B = ∅]
```

#### Venn Diagram — General Addition Rule

```
    ┌─────────────────────────────────────┐
    │              Ω (Sample Space)       │
    │                                     │
    │     ┌───────────┐                   │
    │     │     A     ╔═══╗    B          │
    │     │           ║A∩B║              │
    │     │           ╚═══╝    ┌────┐    │
    │     └───────────┘        │    │    │
    │                          └────┘    │
    │                                     │
    └─────────────────────────────────────┘
    P(A ∪ B) = P(A) + P(B) - P(A ∩ B)
```

#### Venn Diagram — Mutually Exclusive Events

```
    ┌─────────────────────────────────────┐
    │              Ω                       │
    │                                     │
    │     ┌─────────┐   ┌─────────┐      │
    │     │    A     │   │    B     │      │
    │     │         │   │         │      │
    │     └─────────┘   └─────────┘      │
    │           No overlap!               │
    └─────────────────────────────────────┘
    P(A ∪ B) = P(A) + P(B)
```

### 4.3 Multiplication Rule (Intersection of Events)

```
General:       P(A ∩ B) = P(A) · P(B|A)
Independent:   P(A ∩ B) = P(A) · P(B)       [when A ⊥ B]
```

---

## 5. Independent Events

Two events A and B are **independent** if the occurrence of one does not affect the other:

```
P(A ∩ B) = P(A) · P(B)
Equivalently: P(A|B) = P(A)
```

**Example**: Flipping two fair coins — getting heads on the first coin is independent of the second.

### Test for Independence

Given P(A) = 0.4, P(B) = 0.5, P(A ∩ B) = 0.2:
- P(A) · P(B) = 0.4 × 0.5 = 0.2 = P(A ∩ B) ✓ → **Independent**

Given P(A) = 0.4, P(B) = 0.5, P(A ∩ B) = 0.3:
- P(A) · P(B) = 0.4 × 0.5 = 0.2 ≠ 0.3 → **Not independent**

---

## 6. Mutually Exclusive Events

Two events are **mutually exclusive** (disjoint) if they cannot occur simultaneously:

```
A ∩ B = ∅   →   P(A ∩ B) = 0
```

> ⚠️ **Key insight**: If A and B both have positive probability, they **cannot** be both
> independent AND mutually exclusive. (If mutually exclusive, P(A∩B) = 0 ≠ P(A)·P(B).)

---

## 7. Conditional Probability

The probability of A **given** B has occurred:

```
P(A|B) = P(A ∩ B) / P(B),    where P(B) > 0
```

### Step-by-Step Example

**Problem**: A bag has 5 red and 3 blue balls. You draw 2 balls without replacement.
What is P(2nd is red | 1st is red)?

```
Step 1: After drawing 1 red ball → 4 red, 3 blue remain (7 total)
Step 2: P(2nd red | 1st red) = 4/7 ≈ 0.571
```

### Law of Total Probability

If B₁, B₂, ..., Bₙ partition Ω:

```
P(A) = Σᵢ P(A|Bᵢ) · P(Bᵢ)
```

This is essential for computing marginal probabilities and is the foundation of Bayes' theorem.

---

## 8. Python Code: Simulating Probability

```python
import random

# Simulate P(A ∪ B) with 100,000 trials
random.seed(42)
n_trials = 100_000
count_union = 0

for _ in range(n_trials):
    a_occurs = random.random() < 0.4   # P(A) = 0.4
    b_occurs = random.random() < 0.5   # P(B) = 0.5  (independent)
    if a_occurs or b_occurs:
        count_union += 1

estimated = count_union / n_trials
theoretical = 0.4 + 0.5 - 0.4 * 0.5  # = 0.7

print(f"Estimated P(A ∪ B): {estimated:.4f}")
print(f"Theoretical P(A ∪ B): {theoretical:.4f}")
```

```python
# Conditional probability simulation
random.seed(42)
n_trials = 100_000
both, b_count = 0, 0

for _ in range(n_trials):
    die = random.randint(1, 6)
    a = die >= 4          # Event A: die ≥ 4
    b = die % 2 == 0      # Event B: die is even
    if b:
        b_count += 1
        if a:
            both += 1

print(f"P(A|B) estimated: {both/b_count:.4f}")   # Theoretical: 2/3
print(f"P(A|B) theoretical: {2/3:.4f}")
```

---

## 9. Real-World ML Applications

| Concept                 | ML Application                                              |
|-------------------------|-------------------------------------------------------------|
| Sample space            | All possible classifications for an input                   |
| Conditional probability | P(spam \| words in email) — Naive Bayes classifier          |
| Independence            | Naive Bayes assumes feature independence given class         |
| Addition rule           | Computing total error across disjoint failure modes         |
| Complement rule         | P(at least one detection) = 1 − P(no detections)           |
| Law of total probability| Marginalizing over latent variables in generative models    |

---

## 10. Summary Table

| Concept                 | Formula                                        | Key Condition          |
|-------------------------|-------------------------------------------------|------------------------|
| Complement              | P(A') = 1 − P(A)                              | Always valid           |
| Addition (general)      | P(A∪B) = P(A) + P(B) − P(A∩B)                | Always valid           |
| Addition (ME)           | P(A∪B) = P(A) + P(B)                          | A ∩ B = ∅              |
| Multiplication (gen.)   | P(A∩B) = P(A) · P(B\|A)                       | Always valid           |
| Multiplication (indep.) | P(A∩B) = P(A) · P(B)                          | A ⊥ B                  |
| Conditional             | P(A\|B) = P(A∩B) / P(B)                       | P(B) > 0               |
| Total probability       | P(A) = Σ P(A\|Bᵢ)P(Bᵢ)                       | {Bᵢ} partitions Ω      |

---

## 11. Quick Revision Questions

1. **State the three axioms of probability.** What do they guarantee?
2. **A fair die is rolled.** What is P(even or greater than 4)? Use the addition rule.
3. **Are "rolling a 3" and "rolling an even number" mutually exclusive?** Explain.
4. **If P(A) = 0.6 and P(B) = 0.5, and A, B are independent,** find P(A ∩ B) and P(A ∪ B).
5. **A deck of 52 cards:** P(King | Face card) = ?
6. **Why can't two events with positive probability be both mutually exclusive and independent?**

---

## 12. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| — | [Chapter 2: Random Variables](02-random-variables.md) |

---

*Unit 3 · Probability Theory · Chapter 1 of 9*
