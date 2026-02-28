# Chapter 1.2: Classical, Empirical, and Axiomatic Probability

[← Previous: Sample Space and Events](01-sample-space-and-events.md) | [Back to README](../README.md) | [Next: Addition and Multiplication Rules →](03-addition-and-multiplication-rules.md)

---

## Overview

There are three major approaches to defining probability. The **classical** approach assumes equally likely outcomes, the **empirical** approach uses observed frequencies, and the **axiomatic** approach provides a rigorous mathematical foundation using axioms. Understanding all three is essential for building a complete picture of probability theory.

---

## 1. Classical (A Priori) Probability

### Definition

If an experiment has $n$ **equally likely**, **mutually exclusive**, and **exhaustive** outcomes, and $m$ of these are favorable to event $A$, then:

$$P(A) = \frac{m}{n} = \frac{\text{Number of favorable outcomes}}{\text{Total number of outcomes}}$$

### Assumptions (Limitations)
1. All outcomes must be **equally likely** (fair coin, unbiased die, etc.)
2. The sample space must be **finite**
3. Cannot handle infinitely many outcomes or unequal probabilities

### Examples

**Example 1: Fair Die**
- $S = \{1,2,3,4,5,6\}$, $n = 6$
- $A$ = "rolling an even number" = $\{2,4,6\}$, $m = 3$
- $P(A) = \frac{3}{6} = \frac{1}{2}$

**Example 2: Deck of Cards**
- $n = 52$ (total cards)
- $A$ = "drawing an Ace" → $m = 4$
- $P(A) = \frac{4}{52} = \frac{1}{13}$

**Example 3: Two Dice Sum**
- Total outcomes = 36
- $A$ = "sum = 9" = $\{(3,6),(4,5),(5,4),(6,3)\}$ → $m = 4$
- $P(A) = \frac{4}{36} = \frac{1}{9}$

```
  Classical Probability — Decision Flow
  
  ┌─────────────────────────┐
  │  Is sample space finite? │
  └──────────┬──────────────┘
             │
        Yes  │  No → Cannot use classical approach
             ▼
  ┌──────────────────────────────┐
  │ Are all outcomes equally     │
  │ likely?                      │
  └──────────┬───────────────────┘
             │
        Yes  │  No → Cannot use classical approach
             ▼
  ┌──────────────────────────────┐
  │  P(A) = m / n                │
  │  (favorable / total)         │
  └──────────────────────────────┘
```

---

## 2. Empirical (Relative Frequency) Probability

### Definition

If an experiment is repeated $N$ times and event $A$ occurs $n_A$ times, then the **empirical probability** of $A$ is:

$$P(A) = \lim_{N \to \infty} \frac{n_A}{N}$$

In practice (for finite $N$):

$$\hat{P}(A) = \frac{n_A}{N} \quad \text{(estimate)}$$

### Key Idea
- Based on **observed data** from actual experiments
- The more trials $N$, the closer $\hat{P}(A)$ gets to the true probability
- This is the **Law of Large Numbers** in action

### Example: Coin Toss Experiment

| Number of Tosses ($N$) | Heads Count ($n_H$) | Relative Frequency $\frac{n_H}{N}$ |
|:-----------------------:|:-------------------:|:-----------------------------------:|
| 10 | 6 | 0.600 |
| 100 | 53 | 0.530 |
| 1,000 | 498 | 0.498 |
| 10,000 | 5,012 | 0.501 |
| 100,000 | 49,987 | 0.500 |

```
  Relative Frequency → Converges to True Probability
  
  Freq
  1.0 │
      │ *
  0.8 │
      │    *
  0.6 │       *
      │          *        *
  0.5 │─ ─ ─ ─ ─ ─ ─*─ ─ ─*─ ─ ─*─ ─ ─  ← True P = 0.5
      │                             *
  0.4 │
      │
  0.2 │
      │
  0.0 └─────────────────────────────────►
       10   100   1K   10K   100K    N
```

### Classical vs. Empirical

| Feature | Classical | Empirical |
|---------|-----------|-----------|
| Basis | Theoretical reasoning | Observed data |
| Requires | Equally likely outcomes | Repeated experiments |
| Sample space | Must be finite | Any |
| Example | Fair die → $P(1) = 1/6$ | Loaded die → observe frequency |
| Accuracy | Exact (if assumptions hold) | Approximate (depends on $N$) |

---

## 3. Axiomatic (Kolmogorov's) Probability

### Background

In 1933, the Russian mathematician **Andrey Kolmogorov** established probability on a rigorous foundation using three axioms. This approach works for **any** sample space — finite, countably infinite, or uncountable.

### The Three Axioms

Let $S$ be a sample space and $P$ be a function assigning a real number $P(A)$ to every event $A$. Then $P$ is a **probability measure** if:

$$\boxed{\text{Axiom 1 (Non-negativity):} \quad P(A) \geq 0 \quad \text{for every event } A}$$

$$\boxed{\text{Axiom 2 (Normalization):} \quad P(S) = 1}$$

$$\boxed{\text{Axiom 3 (Countable Additivity):} \quad \text{If } A_1, A_2, \ldots \text{ are mutually exclusive, then } P\left(\bigcup_{i=1}^{\infty} A_i\right) = \sum_{i=1}^{\infty} P(A_i)}$$

### Logical Structure

```
  ┌────────────────────────────────────────────────────────┐
  │              KOLMOGOROV'S AXIOMS                        │
  │                                                        │
  │   Axiom 1: P(A) ≥ 0          → No negative probs      │
  │                                                        │
  │   Axiom 2: P(S) = 1          → Total prob = 1          │
  │                                                        │
  │   Axiom 3: P(A₁ ∪ A₂ ∪ ...) → Additive for disjoint   │
  │            = P(A₁) + P(A₂) + ...  events               │
  └────────────────────────────────────────────────────────┘
              │
              │ From these axioms, we can DERIVE:
              ▼
  ┌────────────────────────────────────────────────────────┐
  │   • P(∅) = 0                                           │
  │   • P(Aᶜ) = 1 - P(A)                                  │
  │   • 0 ≤ P(A) ≤ 1                                      │
  │   • If A ⊆ B, then P(A) ≤ P(B)                        │
  │   • P(A ∪ B) = P(A) + P(B) - P(A ∩ B)                 │
  └────────────────────────────────────────────────────────┘
```

---

## 4. Theorems Derived from the Axioms

### Theorem 1: Probability of the Empty Set

$$P(\emptyset) = 0$$

**Proof:** $S = S \cup \emptyset$ and $S \cap \emptyset = \emptyset$  
By Axiom 3: $P(S) = P(S) + P(\emptyset) \implies P(\emptyset) = 0$

### Theorem 2: Complement Rule

$$P(A^c) = 1 - P(A)$$

**Proof:** $S = A \cup A^c$ and $A \cap A^c = \emptyset$  
By Axiom 3: $P(S) = P(A) + P(A^c) = 1 \implies P(A^c) = 1 - P(A)$

### Theorem 3: Probability Bounds

$$0 \leq P(A) \leq 1$$

**Proof:** From Axiom 1, $P(A) \geq 0$. From Theorem 2, $P(A^c) = 1 - P(A) \geq 0 \implies P(A) \leq 1$.

### Theorem 4: Monotonicity

If $A \subseteq B$, then $P(A) \leq P(B)$.

**Proof:** $B = A \cup (B \setminus A)$ where $A \cap (B \setminus A) = \emptyset$  
$P(B) = P(A) + P(B \setminus A) \geq P(A)$ (since $P(B \setminus A) \geq 0$)

### Theorem 5: Addition Rule (General)

$$P(A \cup B) = P(A) + P(B) - P(A \cap B)$$

*Detailed proof covered in the next chapter.*

---

## 5. Probability Space — The Complete Picture

A probability space is a triple $(\Omega, \mathcal{F}, P)$ where:

| Component | Name | Description |
|-----------|------|-------------|
| $\Omega$ | Sample Space | Set of all outcomes |
| $\mathcal{F}$ | Event Space ($\sigma$-algebra) | Collection of events (subsets of $\Omega$) |
| $P$ | Probability Measure | Function $\mathcal{F} \to [0,1]$ satisfying Kolmogorov's axioms |

### Properties of $\mathcal{F}$ ($\sigma$-algebra)

1. $\Omega \in \mathcal{F}$
2. If $A \in \mathcal{F}$, then $A^c \in \mathcal{F}$
3. If $A_1, A_2, \ldots \in \mathcal{F}$, then $\bigcup_{i=1}^{\infty} A_i \in \mathcal{F}$

**For finite sample spaces:** $\mathcal{F}$ is usually the power set (set of all subsets) of $\Omega$.

---

## 6. Comparison of All Three Approaches

| Feature | Classical | Empirical | Axiomatic |
|---------|-----------|-----------|-----------|
| **Foundation** | Equally likely outcomes | Experimental data | Mathematical axioms |
| **Formula** | $m/n$ | $n_A/N$ | Axioms 1, 2, 3 |
| **Sample Space** | Finite only | Any | Any |
| **Equally likely?** | Required | Not required | Not required |
| **Precision** | Exact | Approximate | Exact |
| **Limitations** | Fails for unequal probs | Needs many trials | Abstract |
| **Year** | 17th century | 19th century | 1933 (Kolmogorov) |
| **Use Case** | Games of chance | Quality control, surveys | All theoretical work |

---

## 7. Worked Example: All Three Approaches

**Problem:** What is the probability that a randomly selected day in a year falls on a Monday?

### Classical Approach
- $S$ = {Mon, Tue, Wed, Thu, Fri, Sat, Sun}
- Assuming all days equally likely: $P(\text{Mon}) = \frac{1}{7} \approx 0.1429$

### Empirical Approach
- In 2025, there are 52 Mondays out of 365 days
- $\hat{P}(\text{Mon}) = \frac{52}{365} \approx 0.1425$

### Axiomatic Approach
- Define $P(\{\text{Mon}\}) = 1/7$ (assign probability satisfying axioms)
- Verify: $P(S) = 7 \times (1/7) = 1$ ✓
- $P(\text{Mon}) \geq 0$ ✓
- All axioms satisfied ✓

---

## 8. Real-World Applications

| Approach | Application |
|----------|------------|
| **Classical** | Casino games, lottery odds, card games |
| **Empirical** | Insurance risk tables, clinical drug trials, weather forecasting |
| **Axiomatic** | Machine learning models, stochastic processes, financial mathematics |

---

## Summary Table

| Concept | Key Formula / Statement |
|---------|------------------------|
| Classical Probability | $P(A) = m/n$ (equally likely outcomes) |
| Empirical Probability | $P(A) = \lim_{N\to\infty} n_A / N$ |
| Axiom 1 (Non-negativity) | $P(A) \geq 0$ |
| Axiom 2 (Normalization) | $P(S) = 1$ |
| Axiom 3 (Additivity) | $P(\bigcup A_i) = \sum P(A_i)$ for disjoint events |
| Complement Rule | $P(A^c) = 1 - P(A)$ |
| Bounds | $0 \leq P(A) \leq 1$ |
| Impossible Event | $P(\emptyset) = 0$ |
| Monotonicity | $A \subseteq B \implies P(A) \leq P(B)$ |
| Probability Space | $(\Omega, \mathcal{F}, P)$ |

---

## Quick Revision Questions

1. **State** the three axioms of Kolmogorov's probability. Why is the classical definition considered a special case of the axiomatic approach?

2. A bag contains 5 red, 3 blue, and 2 green balls. Using the classical approach, find $P(\text{Red})$, $P(\text{Not Green})$.

3. In 500 trials of an experiment, event $A$ occurred 120 times. What is the empirical probability of $A$? What happens to this estimate as $N \to \infty$?

4. **Prove** that $P(A \cup B) \leq P(A) + P(B)$ using the axioms. When does equality hold?

5. Explain why the classical definition fails for a biased coin with $P(H) = 0.7$. Which approach handles this?

6. What is a $\sigma$-algebra? Why is it needed in the axiomatic framework?

---

[← Previous: Sample Space and Events](01-sample-space-and-events.md) | [Back to README](../README.md) | [Next: Addition and Multiplication Rules →](03-addition-and-multiplication-rules.md)
