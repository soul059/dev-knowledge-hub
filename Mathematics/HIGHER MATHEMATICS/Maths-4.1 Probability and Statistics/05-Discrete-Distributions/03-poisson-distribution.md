# Chapter 5.3: Poisson Distribution

[← Previous: Binomial Distribution](02-binomial-distribution.md) | [Back to README](../README.md) | [Next: Geometric Distribution →](04-geometric-distribution.md)

---

## Overview

The **Poisson distribution** models the number of events occurring in a fixed interval of time or space, when events happen independently at a constant average rate. It is the go-to distribution for **counting rare events**.

---

## 1. Definition

$$\boxed{P(X = k) = \frac{e^{-\lambda}\lambda^k}{k!}, \quad k = 0, 1, 2, \ldots}$$

### Notation

$$X \sim \text{Poisson}(\lambda), \quad \lambda > 0$$

### Conditions for Poisson Model

1. Events occur **independently**
2. Events occur at a **constant average rate** $\lambda$
3. Two events cannot occur at exactly the **same instant**
4. The probability of an event in a small interval is proportional to interval length

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $E[X] = \lambda$ |
| Variance | $\text{Var}(X) = \lambda$ |
| Standard Deviation | $\sigma = \sqrt{\lambda}$ |
| MGF | $M(t) = e^{\lambda(e^t - 1)}$ |
| Mode | $\lfloor \lambda \rfloor$ or $\lfloor \lambda \rfloor - 1$ (if $\lambda$ integer) |
| Skewness | $1/\sqrt{\lambda}$ |

> **Key insight**: Mean = Variance = $\lambda$. This property helps identify Poisson data.

### Derivation of Mean

$$E[X] = \sum_{k=0}^{\infty} k \cdot \frac{e^{-\lambda}\lambda^k}{k!} = e^{-\lambda} \sum_{k=1}^{\infty} \frac{\lambda^k}{(k-1)!} = e^{-\lambda} \cdot \lambda \sum_{j=0}^{\infty} \frac{\lambda^j}{j!} = e^{-\lambda} \cdot \lambda \cdot e^{\lambda} = \lambda$$

---

## 3. Shape of the PMF

```
  Poisson(λ=1)                    Poisson(λ=5)
  
  P(X=k)                          P(X=k)
  0.40│▮                           0.20│       ▮
      │▮ ▮                             │     ▮ ▮ ▮
  0.20│▮ ▮ ▮                       0.15│   ▮ ▮ ▮ ▮ ▮
      │▮ ▮ ▮ ▮                         │   ▮ ▮ ▮ ▮ ▮ ▮
  0.10│▮ ▮ ▮ ▮ ▮                   0.05│ ▮ ▮ ▮ ▮ ▮ ▮ ▮ ▮
      │▮ ▮ ▮ ▮ ▮ ▮                     │ ▮ ▮ ▮ ▮ ▮ ▮ ▮ ▮ ▮ ▮
  0.00└─┴─┴─┴─┴─┴─┴──► k          0.00└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴──► k
      0 1 2 3 4 5                   0 1 2 3 4 5 6 7 8 9 10
  
  Strongly right-skewed            More symmetric for larger λ
```

---

## 4. Poisson as Limit of Binomial

### Poisson Limit Theorem

If $n \to \infty$, $p \to 0$, and $np \to \lambda$:

$$\binom{n}{k}p^k(1-p)^{n-k} \to \frac{e^{-\lambda}\lambda^k}{k!}$$

**Rule of thumb**: Use Poisson approximation when $n \geq 20$ and $p \leq 0.05$ (or $n \geq 100$ and $np \leq 10$).

### Example 1

A book has 500 pages and 10 typos total. Probability of a given page having 0, 1, or 2+ typos?

$\lambda = 10/500 = 0.02$ per page.

$$P(X=0) = e^{-0.02} \approx 0.9802$$
$$P(X=1) = 0.02 \cdot e^{-0.02} \approx 0.0196$$
$$P(X \geq 2) = 1 - P(0) - P(1) \approx 0.0002$$

---

## 5. Worked Examples

### Example 2: Call Center

A call center receives an average of 4 calls per minute.

$X \sim \text{Poisson}(4)$

**(a)** $P(X = 0)$ — no calls in a minute

$$P(X=0) = \frac{e^{-4} \cdot 4^0}{0!} = e^{-4} \approx 0.0183$$

**(b)** $P(X \geq 3)$

$$P(X \geq 3) = 1 - P(0) - P(1) - P(2)$$
$$= 1 - e^{-4}(1 + 4 + 8) = 1 - 13e^{-4} \approx 1 - 0.2381 = 0.7619$$

**(c)** Calls in 5 minutes: $Y \sim \text{Poisson}(20)$

### Example 3: Radioactive Decay

A radioactive source emits particles at rate 3 per second. What is the probability of exactly 5 particles in 2 seconds?

$\lambda = 3 \times 2 = 6$ particles per 2 seconds.

$$P(X=5) = \frac{e^{-6} \cdot 6^5}{5!} = \frac{e^{-6} \cdot 7776}{120} \approx 0.1606$$

---

## 6. Additivity Property

If $X \sim \text{Poisson}(\lambda_1)$ and $Y \sim \text{Poisson}(\lambda_2)$ are independent:

$$\boxed{X + Y \sim \text{Poisson}(\lambda_1 + \lambda_2)}$$

**Proof via MGF**:

$$M_{X+Y}(t) = e^{\lambda_1(e^t-1)} \cdot e^{\lambda_2(e^t-1)} = e^{(\lambda_1+\lambda_2)(e^t-1)}$$

---

## 7. Poisson Process

A **Poisson process** with rate $\lambda$ generates:
- Number of events in time $t$: $N(t) \sim \text{Poisson}(\lambda t)$
- Time between events: $\text{Exponential}(\lambda)$

```
  Poisson Process on a timeline:
  
  Events:  ×    ×  ×        ×       × ×    ×        ×
  ─────────┼────┼──┼────────┼───────┼─┼────┼────────┼────► time
  0        t₁   t₂ t₃      t₄      t₅t₆   t₇       t₈
  
  Inter-arrival times (gaps between ×'s) ~ Exponential(λ)
  
  Count in [0, T]:  N(T) ~ Poisson(λT)
```

---

## 8. Comparison: Binomial vs. Poisson

| Feature | Binomial | Poisson |
|---------|----------|---------|
| Fixed $n$? | Yes | No (infinite support) |
| Trials | Finite, countable | Continuous interval |
| Parameters | $n, p$ | $\lambda$ |
| Mean | $np$ | $\lambda$ |
| Variance | $np(1-p)$ | $\lambda$ |
| Mean = Variance? | Only if $p \to 0$ | **Always** |
| Support | $\{0, 1, \ldots, n\}$ | $\{0, 1, 2, \ldots\}$ |

---

## 9. Real-World Applications

| Scenario | $\lambda$ |
|----------|-----------|
| Emails received per hour | 12 |
| Accidents at intersection per month | 2.5 |
| Mutations per gene per generation | 0.001 |
| Server requests per second | 100 |
| Earthquakes per year (in a region) | 0.8 |
| Typos per page | 0.5 |
| Goals per soccer match | 2.7 |

---

## Summary Table

| Concept | Formula / Value |
|---------|----------------|
| PMF | $e^{-\lambda}\lambda^k / k!$ |
| Mean | $\lambda$ |
| Variance | $\lambda$ |
| MGF | $e^{\lambda(e^t - 1)}$ |
| Sum of indep. Poissons | $\text{Poisson}(\lambda_1 + \lambda_2)$ |
| Binomial approximation | $n$ large, $p$ small, $\lambda = np$ |
| Poisson process | $N(t) \sim \text{Poisson}(\lambda t)$ |

---

## Quick Revision Questions

1. If $X \sim \text{Poisson}(3)$, find $P(X = 0)$, $P(X = 1)$, $P(X \leq 2)$.

2. A website gets 200 hits per hour. What is the probability of 0 hits in a given minute? (State the distribution and $\lambda$ for one minute.)

3. Prove that the Poisson PMF sums to 1 using the Taylor series for $e^{\lambda}$.

4. Hospital admissions average 5 per day. Find the probability of more than 8 admissions in a day.

5. Two independent Poisson processes have rates $\lambda_1 = 3$ and $\lambda_2 = 7$. What is the distribution of the total count? Find $P(\text{total} = 10)$.

6. A dataset has sample mean 4.8 and sample variance 5.1. Is this consistent with a Poisson model? Justify.

---

[← Previous: Binomial Distribution](02-binomial-distribution.md) | [Back to README](../README.md) | [Next: Geometric Distribution →](04-geometric-distribution.md)
