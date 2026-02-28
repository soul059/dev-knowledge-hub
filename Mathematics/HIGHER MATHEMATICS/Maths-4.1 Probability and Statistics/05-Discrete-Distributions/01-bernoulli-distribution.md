# Chapter 5.1: Bernoulli Distribution

[← Previous: Chebyshev's Inequality](../04-Expectation-and-Variance/05-chebyshev-inequality.md) | [Back to README](../README.md) | [Next: Binomial Distribution →](02-binomial-distribution.md)

---

## Overview

The **Bernoulli distribution** is the simplest discrete distribution — a single trial with exactly two outcomes: success (1) or failure (0). It is the building block for many other distributions including the Binomial.

---

## 1. Definition

A random variable $X$ has a **Bernoulli distribution** with parameter $p$ if:

$$\boxed{P(X = x) = p^x (1-p)^{1-x}, \quad x \in \{0, 1\}}$$

| Outcome | $x$ | Probability |
|---------|:---:|:-----------:|
| Failure | 0 | $q = 1-p$ |
| Success | 1 | $p$ |

### Notation

$$X \sim \text{Bernoulli}(p), \quad 0 \leq p \leq 1$$

```
  PMF of Bernoulli(p)
  
  P(X=x)
  │
  │  ▮                  ▮
  │  ▮ 1-p           p  ▮
  │  ▮                  ▮
  │  ▮                  ▮
  └──┼──────────────────┼──► x
     0                  1
```

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $E[X] = p$ |
| Variance | $\text{Var}(X) = p(1-p) = pq$ |
| Standard Deviation | $\sigma = \sqrt{pq}$ |
| MGF | $M(t) = q + pe^t$ |
| Skewness | $\frac{1-2p}{\sqrt{pq}}$ |
| Kurtosis | $\frac{1-6pq}{pq}$ |
| Mode | 0 if $p < 0.5$; 1 if $p > 0.5$ |

### Derivations

**Mean**: $E[X] = 0 \cdot (1-p) + 1 \cdot p = p$

**Variance**: $E[X^2] = 0^2(1-p) + 1^2(p) = p$, so $\text{Var}(X) = p - p^2 = p(1-p)$

**MGF**: $M(t) = (1-p)e^{0 \cdot t} + p \cdot e^{1 \cdot t} = (1-p) + pe^t$

---

## 3. Maximum Variance

The variance $p(1-p)$ is maximized when $p = 0.5$:

$$\max_{p} p(1-p) = 0.5 \times 0.5 = 0.25$$

```
  Var(X) = p(1-p)
  
  Var
  0.25│        ╱╲
      │      ╱    ╲
  0.20│    ╱        ╲
      │  ╱            ╲
  0.10│╱                ╲
      │                   ╲
  0.00└──┼──┼──┼──┼──┼──┼──► p
     0  0.1 0.2 0.3 0.5  0.8  1.0
  
  Maximum uncertainty at p = 0.5
```

---

## 4. CDF

$$F(x) = \begin{cases} 0 & x < 0 \\ 1-p & 0 \leq x < 1 \\ 1 & x \geq 1 \end{cases}$$

```
  CDF of Bernoulli(p)
  
  F(x)
  1.0 │                    ●━━━━━━━━━
      │                    │
  1-p │        ●━━━━━━━━━━━○
      │        │
  0.0 │━━━━━━━━○
      └────────┼───────────┼──────► x
               0           1
```

---

## 5. Bernoulli as a Building Block

```
  ┌─────────────────────────────────────────────────────┐
  │  Bernoulli(p) → Building block for:                 │
  │                                                      │
  │  • Binomial(n,p)   = Sum of n Bernoullis             │
  │  • Geometric(p)    = # trials until 1st success      │
  │  • Neg. Binomial   = # trials until r-th success     │
  │  • Bernoulli Process = sequence of Bernoulli trials  │
  └─────────────────────────────────────────────────────┘
```

### Indicator Random Variables

Every indicator variable $\mathbb{1}_A$ is Bernoulli($P(A)$):

$$\mathbb{1}_A = \begin{cases} 1 & \text{if event } A \text{ occurs} \\ 0 & \text{otherwise} \end{cases}$$

---

## 6. Real-World Examples

| Scenario | Success ($X=1$) | $p$ |
|----------|-----------------|-----|
| Coin flip | Heads | 0.5 |
| Quality inspection | Item is defective | 0.02 |
| Email filter | Email is spam | 0.45 |
| Medical test | Test positive | sensitivity |
| Free throw | Ball goes in | 0.85 |
| Click-through | User clicks ad | 0.03 |

---

## 7. Relationship to Other Distributions

| From Bernoulli to... | Connection |
|---------------------|------------|
| **Binomial($n,p$)** | Sum of $n$ i.i.d. Bernoulli($p$) |
| **Geometric($p$)** | Number of Bernoulli trials until first success |
| **N. Binomial($r,p$)** | Number of trials until $r$ successes |
| **Beta distribution** | Conjugate prior for $p$ in Bayesian stats |
| **Multinoulli** | Generalization to $k > 2$ categories |

---

## Summary Table

| Concept | Value |
|---------|-------|
| PMF | $p^x(1-p)^{1-x}$, $x \in \{0,1\}$ |
| Mean | $p$ |
| Variance | $p(1-p)$ |
| Max variance | $0.25$ at $p = 0.5$ |
| MGF | $(1-p) + pe^t$ |
| Support | $\{0, 1\}$ |
| Parameters | $0 \leq p \leq 1$ |

---

## Quick Revision Questions

1. If $X \sim \text{Bernoulli}(0.3)$, compute $E[X]$, $\text{Var}(X)$, and $P(X \geq 1)$.

2. Show that the sum of the PMF values equals 1: $P(X=0) + P(X=1) = 1$.

3. Derive the MGF and use it to verify $E[X] = p$.

4. If an event has probability 0.7, write the indicator variable and its distribution.

5. Prove that $\text{Var}(X) \leq 0.25$ for any Bernoulli random variable and state when equality holds.

6. A manufacturing process has defect rate $p = 0.01$. Model a single inspection as a Bernoulli RV and state the mean and standard deviation.

---

[← Previous: Chebyshev's Inequality](../04-Expectation-and-Variance/05-chebyshev-inequality.md) | [Back to README](../README.md) | [Next: Binomial Distribution →](02-binomial-distribution.md)
