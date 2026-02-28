# Chapter 5.4: Geometric Distribution

[← Previous: Poisson Distribution](03-poisson-distribution.md) | [Back to README](../README.md) | [Next: Negative Binomial Distribution →](05-negative-binomial.md)

---

## Overview

The **Geometric distribution** models the number of trials needed to get the **first success** in a sequence of independent Bernoulli trials. It's the discrete analog of the Exponential distribution and is the only discrete distribution with the **memoryless property**.

---

## 1. Definition

### Version 1: Number of trials until first success (including the success)

$$\boxed{P(X = k) = (1-p)^{k-1} p, \quad k = 1, 2, 3, \ldots}$$

### Version 2: Number of failures before first success

$$P(Y = k) = (1-p)^k p, \quad k = 0, 1, 2, \ldots$$

> $Y = X - 1$. We use **Version 1** throughout (number of trials).

### Notation

$$X \sim \text{Geometric}(p)$$

```
  Geometric(p=0.3) — Version 1
  
  P(X=k)
  0.30│▮
      │▮ ▮
  0.20│▮ ▮
      │▮ ▮ ▮
  0.10│▮ ▮ ▮ ▮
      │▮ ▮ ▮ ▮ ▮ ▮ ▮ ▮ ▮ ▮
  0.00└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴──► k
      1 2 3 4 5 6 7 8 9 10
  
  Always decreasing, most likely on first trial
```

---

## 2. Key Properties

| Property | Version 1 ($k = 1,2,\ldots$) | Version 2 ($k = 0,1,2,\ldots$) |
|----------|:--:|:--:|
| Mean | $1/p$ | $(1-p)/p$ |
| Variance | $(1-p)/p^2$ | $(1-p)/p^2$ |
| MGF | $\frac{pe^t}{1-(1-p)e^t}$ | $\frac{p}{1-(1-p)e^t}$ |

### Derivation of Mean (Version 1)

$$E[X] = \sum_{k=1}^{\infty} k(1-p)^{k-1}p = p \sum_{k=1}^{\infty} k q^{k-1} = p \cdot \frac{1}{(1-q)^2} = p \cdot \frac{1}{p^2} = \frac{1}{p}$$

(Using $\sum_{k=1}^{\infty} kx^{k-1} = \frac{1}{(1-x)^2}$ for $|x| < 1$)

---

## 3. CDF and Survival Function

$$\boxed{P(X > k) = (1-p)^k}$$

$$F(k) = P(X \leq k) = 1 - (1-p)^k$$

### Proof

$P(X > k) = P(\text{first } k \text{ trials all fail}) = (1-p)^k$

This gives us a very clean formula without summing the PMF!

---

## 4. The Memoryless Property

$$\boxed{P(X > s + t \mid X > s) = P(X > t)}$$

> "Given that you've already failed $s$ times, the probability of needing $t$ more trials is the same as starting fresh."

### Proof

$$P(X > s+t \mid X > s) = \frac{P(X > s+t)}{P(X > s)} = \frac{(1-p)^{s+t}}{(1-p)^s} = (1-p)^t = P(X > t) \quad \blacksquare$$

### Uniqueness

The Geometric is the **only** discrete distribution with the memoryless property. (The Exponential is the only continuous one.)

```
  Memoryless Property Visualized
  
  Trial:  F  F  F  F  F  F  F  S
          1  2  3  4  5  6  7  8
                    ▲
                    │
                   "I've failed 4 times already.
                    How many MORE trials do I need?"
                    
                    Answer: Same distribution as if
                    starting from scratch!
                    
                    P(need ≥ t more | already failed 4)
                    = P(need ≥ t from start)
```

---

## 5. Worked Examples

### Example 1: Rolling a Six

$P(\text{six}) = 1/6$. How many rolls until the first six?

$$E[X] = \frac{1}{1/6} = 6 \text{ rolls}$$

$$P(X = 1) = \frac{1}{6} \approx 0.167$$

$$P(X \leq 3) = 1 - \left(\frac{5}{6}\right)^3 \approx 0.421$$

$$P(X > 10) = \left(\frac{5}{6}\right)^{10} \approx 0.162$$

### Example 2: Network Packet

A packet has $P(\text{success}) = 0.95$ on each transmission attempt.

$$E[\text{attempts}] = \frac{1}{0.95} \approx 1.053$$

$$P(\text{needs retransmission}) = P(X > 1) = 0.05$$

$$P(\text{needs} > 3 \text{ attempts}) = (0.05)^3 = 0.000125$$

### Example 3: Waiting for a Pattern

On each attempt, $P(\text{success}) = 0.2$. Given you've already tried 5 times without success, what's the expected additional number of trials?

By memorylessness: $E[\text{additional trials}] = 1/0.2 = 5$ more trials.

---

## 6. Median and Quantiles

### Median

$$\text{Median} = \left\lceil \frac{-1}{\log_2(1-p)} \right\rceil = \left\lceil \frac{\ln 2}{-\ln(1-p)} \right\rceil$$

For $p = 0.5$: Median = 1  
For $p = 0.1$: Median = $\lceil 6.58 \rceil = 7$

### $q$-th Quantile

$$x_q = \left\lceil \frac{\ln(1-q)}{\ln(1-p)} \right\rceil$$

---

## 7. Relationship to Other Distributions

| Connection | Description |
|-----------|-------------|
| **Bernoulli** | Geometric = repeated Bernoullis until success |
| **Negative Binomial** | Sum of $r$ independent Geometrics |
| **Exponential** | Continuous analog (memoryless property) |
| **Binomial** | $P(\text{Geom} > n) = P(\text{Bin}(n,p) = 0) = (1-p)^n$ |

```
  Geometric(p) ←→ Exponential(λ)
  
  Discrete                    Continuous
  Trials until success        Time until event
  P(X > k) = (1-p)^k         P(T > t) = e^{-λt}
  E[X] = 1/p                 E[T] = 1/λ
  Memoryless                  Memoryless
```

---

## 8. Real-World Applications

| Scenario | $p$ | $E[X] = 1/p$ |
|----------|-----|:---:|
| Coin flips until heads | 0.5 | 2 |
| Days until a sale | 0.1 | 10 |
| Interviews until job offer | 0.15 | ≈ 7 |
| Rolls until a six | 1/6 | 6 |
| Attempts until correct password | 1/1000 | 1000 |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| PMF | $(1-p)^{k-1}p$, $k = 1, 2, \ldots$ |
| CDF | $1 - (1-p)^k$ |
| Survival | $P(X > k) = (1-p)^k$ |
| Mean | $1/p$ |
| Variance | $(1-p)/p^2$ |
| Memoryless | $P(X > s+t \mid X > s) = P(X > t)$ |
| MGF | $pe^t/(1-(1-p)e^t)$ |

---

## Quick Revision Questions

1. If a coin has $P(\text{heads}) = 0.4$, find the expected number of flips to get the first heads.

2. A student has probability 0.6 of passing a driving test on each attempt. Find $P(\text{passes on 3rd attempt})$ and $P(\text{needs} > 4 \text{ attempts})$.

3. **Prove** the memoryless property $P(X > s+t \mid X > s) = P(X > t)$.

4. If $X \sim \text{Geometric}(0.1)$, find the median of $X$.

5. A computer generates random numbers from 1 to 100 until it generates the number 42. What is the expected number of attempts? What is the variance?

6. Explain why the Geometric distribution is the only discrete memoryless distribution. (Hint: What constraint does $P(X > s+t) = P(X > s)P(X > t)$ impose on the survival function?)

---

[← Previous: Poisson Distribution](03-poisson-distribution.md) | [Back to README](../README.md) | [Next: Negative Binomial Distribution →](05-negative-binomial.md)
