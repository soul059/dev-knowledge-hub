# Chapter 5.2: Binomial Distribution

[← Previous: Bernoulli Distribution](01-bernoulli-distribution.md) | [Back to README](../README.md) | [Next: Poisson Distribution →](03-poisson-distribution.md)

---

## Overview

The **Binomial distribution** counts the number of successes in $n$ independent Bernoulli trials, each with the same success probability $p$. It is one of the most widely used discrete distributions.

---

## 1. Definition and Conditions

$X \sim \text{Binomial}(n, p)$ if:

1. **Fixed** number of trials: $n$
2. Each trial is **independent**
3. Each trial has exactly **two outcomes** (success/failure)
4. **Same** probability of success $p$ in each trial

$$\boxed{P(X = k) = \binom{n}{k} p^k (1-p)^{n-k}, \quad k = 0, 1, 2, \ldots, n}$$

where $\binom{n}{k} = \frac{n!}{k!(n-k)!}$

### Interpretation

$$P(X = k) = \underbrace{\binom{n}{k}}_{\text{ways to choose}} \times \underbrace{p^k}_{\text{k successes}} \times \underbrace{(1-p)^{n-k}}_{\text{n-k failures}}$$

---

## 2. Shape of the PMF

```
  Binomial(10, 0.3)                Binomial(10, 0.5)
  
  P(X=k)                           P(X=k)
  0.30│ ▮                           0.25│       ▮
      │ ▮ ▮                             │     ▮ ▮ ▮
  0.20│ ▮ ▮ ▮                       0.20│     ▮ ▮ ▮
      │ ▮ ▮ ▮ ▮                         │   ▮ ▮ ▮ ▮ ▮
  0.10│ ▮ ▮ ▮ ▮ ▮                   0.10│   ▮ ▮ ▮ ▮ ▮
      │ ▮ ▮ ▮ ▮ ▮ ▮                     │ ▮ ▮ ▮ ▮ ▮ ▮ ▮
  0.00└─┴─┴─┴─┴─┴─┴─┴─┴──► k       0.00└─┴─┴─┴─┴─┴─┴─┴─┴──► k
      0 1 2 3 4 5 6 7              0 1 2 3 4 5 6 7 8
  
  Right-skewed (p < 0.5)            Symmetric (p = 0.5)
```

---

## 3. Key Properties

| Property | Value |
|----------|-------|
| Mean | $E[X] = np$ |
| Variance | $\text{Var}(X) = np(1-p) = npq$ |
| Standard Deviation | $\sigma = \sqrt{npq}$ |
| MGF | $M(t) = (q + pe^t)^n$ |
| Mode | $\lfloor (n+1)p \rfloor$ or $\lfloor (n+1)p \rfloor - 1$ |
| Skewness | $(1-2p)/\sqrt{npq}$ |

### Derivation of Mean and Variance

$X = X_1 + X_2 + \cdots + X_n$ where $X_i \sim \text{Bernoulli}(p)$ independent.

$$E[X] = \sum_{i=1}^n E[X_i] = np$$

$$\text{Var}(X) = \sum_{i=1}^n \text{Var}(X_i) = np(1-p)$$

---

## 4. Worked Examples

### Example 1: Quality Control

A batch of parts has 5% defect rate. In a sample of 20, find:

$X \sim \text{Binomial}(20, 0.05)$

**(a)** $P(X = 0)$ — no defectives

$$P(X=0) = \binom{20}{0}(0.05)^0(0.95)^{20} = (0.95)^{20} \approx 0.3585$$

**(b)** $P(X \leq 2)$

$$= P(0) + P(1) + P(2) = 0.3585 + 20(0.05)(0.95)^{19} + \binom{20}{2}(0.05)^2(0.95)^{18}$$
$$\approx 0.3585 + 0.3774 + 0.1887 = 0.9245$$

**(c)** Expected number of defectives

$$E[X] = 20 \times 0.05 = 1$$

### Example 2: Free Throws

A basketball player makes 80% of free throws. In 10 attempts:

$X \sim \text{Binomial}(10, 0.8)$

$$P(X = 10) = (0.8)^{10} \approx 0.1074$$

$$P(X \geq 8) = P(8) + P(9) + P(10) = \binom{10}{8}(0.8)^8(0.2)^2 + \binom{10}{9}(0.8)^9(0.2) + (0.8)^{10}$$
$$\approx 0.3020 + 0.2684 + 0.1074 = 0.6778$$

---

## 5. Cumulative Probabilities

| Type | Formula |
|------|---------|
| Exactly $k$ | $P(X = k) = \binom{n}{k}p^k q^{n-k}$ |
| At most $k$ | $P(X \leq k) = \sum_{i=0}^{k} \binom{n}{i}p^i q^{n-i}$ |
| At least $k$ | $P(X \geq k) = 1 - P(X \leq k-1)$ |
| Between $a$ and $b$ | $P(a \leq X \leq b) = \sum_{i=a}^{b} \binom{n}{i}p^i q^{n-i}$ |

---

## 6. Important Relationships

### Additivity

If $X \sim \text{Bin}(n_1, p)$ and $Y \sim \text{Bin}(n_2, p)$ are independent:

$$X + Y \sim \text{Bin}(n_1 + n_2, p)$$

### Symmetry

If $X \sim \text{Bin}(n, p)$ then $n - X \sim \text{Bin}(n, 1-p)$

$$P(X = k) = P(n - X = n - k)$$

### Binomial → Poisson (Poisson Limit Theorem)

As $n \to \infty$, $p \to 0$, $np \to \lambda$:

$$\binom{n}{k}p^k(1-p)^{n-k} \to \frac{e^{-\lambda}\lambda^k}{k!}$$

### Normal Approximation

For large $n$: $X \sim \text{Bin}(n,p) \approx N(np, npq)$ when $np \geq 5$ and $n(1-p) \geq 5$.

```
  Approximation Roadmap
  
  ┌──────────────────┐
  │  Binomial(n, p)  │
  └────────┬─────────┘
           │
     ┌─────┴──────┐
     │             │
     ▼             ▼
  n large       n large
  p small       p moderate
  np → λ        np ≥ 5, nq ≥ 5
     │             │
     ▼             ▼
  ┌────────┐  ┌──────────┐
  │Poisson │  │ Normal   │
  │(λ=np)  │  │(np, npq) │
  └────────┘  └──────────┘
```

---

## 7. Recursive Formula

$$\frac{P(X = k+1)}{P(X = k)} = \frac{n-k}{k+1} \cdot \frac{p}{1-p}$$

This is useful for computing successive probabilities without recalculating binomial coefficients.

---

## 8. Real-World Applications

| Application | $n$ | $p$ | Question |
|-------------|-----|-----|----------|
| Drug efficacy | 100 patients | 0.7 cure rate | P(≥80 cured)? |
| Election poll | 1000 voters | 0.52 support | P(majority)? |
| Network packets | 50 packets | 0.01 loss rate | P(all arrive)? |
| Genetics | 4 children | 0.25 recessive | P(≥1 affected)? |
| A/B testing | 500 visitors | 0.03 click rate | P(≥20 clicks)? |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| PMF | $\binom{n}{k}p^k(1-p)^{n-k}$ |
| Mean | $np$ |
| Variance | $np(1-p)$ |
| MGF | $(q + pe^t)^n$ |
| Sum of indep. Bin | $\text{Bin}(n_1+n_2, p)$ |
| Poisson approx. | $n$ large, $p$ small, $\lambda = np$ |
| Normal approx. | $np \geq 5$, $nq \geq 5$ |

---

## Quick Revision Questions

1. A fair coin is tossed 8 times. Find the probability of getting exactly 5 heads.

2. In a multiple-choice test with 20 questions and 4 options each, a student guesses randomly. Find the expected score and the probability of passing (≥10 correct).

3. Derive $E[X] = np$ for the Binomial distribution using the indicator variable decomposition.

4. If $X \sim \text{Bin}(100, 0.04)$, approximate $P(X \geq 6)$ using the Poisson approximation.

5. Show that the Binomial PMF sums to 1 using the Binomial Theorem: $(p + q)^n = 1$.

6. A production line has a 2% defect rate. In a batch of 50 items, what is the probability of finding 0, 1, or 2 defective items?

---

[← Previous: Bernoulli Distribution](01-bernoulli-distribution.md) | [Back to README](../README.md) | [Next: Poisson Distribution →](03-poisson-distribution.md)
