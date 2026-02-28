# Chapter 4.5: Chebyshev's Inequality

[← Previous: Moments and MGF](04-moments-and-mgf.md) | [Back to README](../README.md) | [Next: Bernoulli Distribution →](../05-Discrete-Distributions/01-bernoulli-distribution.md)

---

## Overview

**Chebyshev's inequality** gives a universal bound on how far a random variable can deviate from its mean, using only the mean and variance — no knowledge of the actual distribution required. It also leads to the **Law of Large Numbers**, one of the most fundamental theorems in probability.

---

## 1. Markov's Inequality

### Statement

If $X \geq 0$ and $a > 0$:

$$\boxed{P(X \geq a) \leq \frac{E[X]}{a}}$$

### Proof

$$E[X] = \int_0^\infty x f(x)\, dx \geq \int_a^\infty x f(x)\, dx \geq \int_a^\infty a f(x)\, dx = a \cdot P(X \geq a)$$

$$\therefore P(X \geq a) \leq \frac{E[X]}{a} \quad \blacksquare$$

### Example 1

Average salary in a company is $\$50{,}000$. What can we say about the fraction earning $\geq \$200{,}000$?

$$P(X \geq 200{,}000) \leq \frac{50{,}000}{200{,}000} = 0.25$$

At most 25% earn $\geq \$200{,}000$.

---

## 2. Chebyshev's Inequality

### Statement

For any random variable $X$ with mean $\mu$ and variance $\sigma^2$:

$$\boxed{P(|X - \mu| \geq k\sigma) \leq \frac{1}{k^2}}$$

Equivalently:

$$P(|X - \mu| \geq c) \leq \frac{\sigma^2}{c^2}$$

Or in complementary form:

$$P(|X - \mu| < k\sigma) \geq 1 - \frac{1}{k^2}$$

### Derivation from Markov

Apply Markov's inequality to $Y = (X - \mu)^2 \geq 0$:

$$P(|X-\mu| \geq c) = P((X-\mu)^2 \geq c^2) \leq \frac{E[(X-\mu)^2]}{c^2} = \frac{\sigma^2}{c^2}$$

Set $c = k\sigma$: $P(|X-\mu| \geq k\sigma) \leq \frac{1}{k^2} \quad \blacksquare$

---

## 3. Chebyshev Bounds Table

| $k$ (std devs) | Upper bound $1/k^2$ | Lower bound $1 - 1/k^2$ |
|:-:|:-:|:-:|
| 1 | 1.000 (100%) | 0.000 (0%) |
| 1.5 | 0.444 (44.4%) | 0.556 (55.6%) |
| 2 | 0.250 (25%) | 0.750 (75%) |
| 3 | 0.111 (11.1%) | 0.889 (88.9%) |
| 4 | 0.063 (6.3%) | 0.938 (93.8%) |
| 5 | 0.040 (4.0%) | 0.960 (96.0%) |
| 10 | 0.010 (1.0%) | 0.990 (99.0%) |

```
  Chebyshev's Guarantee (Distribution-Free)
  
  At least 75% of data within 2σ of mean
  At least 88.9% of data within 3σ of mean
  
  │        ╱──────────────╲
  │      ╱                  ╲
  │    ╱                      ╲
  │  ╱                          ╲
  │╱                              ╲
  └──┼────┼────────┼────┼──────────► x
     μ-3σ μ-2σ    μ    μ+2σ  μ+3σ
     ◄────── ≥ 75% ──────►
     ◄────────── ≥ 88.9% ──────────►
```

---

## 4. Chebyshev vs. Normal (Empirical Rule)

| Range | Chebyshev (ANY dist.) | Normal (empirical) |
|:-----:|:---------------------:|:------------------:|
| $\mu \pm 1\sigma$ | $\geq 0\%$ | $\approx 68.3\%$ |
| $\mu \pm 2\sigma$ | $\geq 75\%$ | $\approx 95.4\%$ |
| $\mu \pm 3\sigma$ | $\geq 88.9\%$ | $\approx 99.7\%$ |

> Chebyshev is **conservative** but applies to **every** distribution. The Normal (empirical) rule is tighter but only for bell-shaped distributions.

---

## 5. Worked Examples

### Example 2

Exam scores have $\mu = 72$, $\sigma = 8$. What fraction of students scored between 56 and 88?

$56 = 72 - 16 = 72 - 2(8) = \mu - 2\sigma$  
$88 = 72 + 16 = 72 + 2(8) = \mu + 2\sigma$

$$P(56 \leq X \leq 88) = P(|X - 72| \leq 16) \geq 1 - \frac{1}{2^2} = 0.75$$

At least **75%** of students scored between 56 and 88.

### Example 3

A machine produces bolts with mean length 10 cm and SD 0.1 cm. At most what proportion of bolts differ from 10 cm by more than 0.3 cm?

$$P(|X - 10| \geq 0.3) \leq \frac{(0.1)^2}{(0.3)^2} = \frac{0.01}{0.09} = \frac{1}{9} \approx 11.1\%$$

### Example 4: One-Sided Chebyshev (Cantelli's Inequality)

$$P(X - \mu \geq k\sigma) \leq \frac{1}{1 + k^2}$$

This is tighter than Chebyshev for one-sided bounds.

For $k=2$: $P(X - \mu \geq 2\sigma) \leq \frac{1}{5} = 20\%$ (vs. Chebyshev's 25%/2 = 12.5% obtained by halving)

---

## 6. Weak Law of Large Numbers (WLLN)

### Setup

Let $X_1, X_2, \ldots, X_n$ be i.i.d. with mean $\mu$ and variance $\sigma^2$.

Define the sample mean: $\bar{X}_n = \frac{1}{n}\sum_{i=1}^n X_i$

Then: $E[\bar{X}_n] = \mu$, $\text{Var}(\bar{X}_n) = \sigma^2/n$

### Statement (WLLN)

$$\boxed{\forall \epsilon > 0: \quad P(|\bar{X}_n - \mu| \geq \epsilon) \to 0 \text{ as } n \to \infty}$$

### Proof via Chebyshev

$$P(|\bar{X}_n - \mu| \geq \epsilon) \leq \frac{\text{Var}(\bar{X}_n)}{\epsilon^2} = \frac{\sigma^2}{n\epsilon^2} \to 0 \text{ as } n \to \infty \quad \blacksquare$$

```
  Law of Large Numbers — Visualization
  
  Sample Mean X̄_n converges to μ as n grows:
  
  X̄_n
  │  *
  │    *  *                 * = X̄_n values
  │      * * *
  │         * * *
  │           * * * * *
  │══════════════════════════ μ (true mean)
  │             * * * * *
  │           * * *
  │         * *
  │       *
  │
  └──────────────────────────► n
       10   100   1000   10000
  
  Spread of X̄_n shrinks as n increases
  (σ/√n → 0)
```

---

## 7. Other Probability Inequalities

| Inequality | Statement | Uses |
|-----------|-----------|------|
| **Markov** | $P(X \geq a) \leq E[X]/a$ | Only mean, $X \geq 0$ |
| **Chebyshev** | $P(\|X-\mu\| \geq k\sigma) \leq 1/k^2$ | Mean + variance |
| **Chernoff** | $P(X \geq a) \leq \min_t e^{-ta} M_X(t)$ | MGF (tightest) |
| **Cantelli** | $P(X-\mu \geq k\sigma) \leq 1/(1+k^2)$ | One-sided Chebyshev |
| **Hoeffding** | $P(\|X̄-\mu\| \geq t) \leq 2e^{-2nt^2/(b-a)^2}$ | Bounded i.i.d. |
| **Jensen** | $E[g(X)] \geq g(E[X])$ if convex | Convexity |

```
  Hierarchy of Bounds (tightest to loosest):
  
  Exact probability    (need full distribution)
       │
  Chernoff bound       (need MGF)
       │
  Hoeffding bound      (need bounded range)
       │
  Chebyshev inequality (need mean + variance)
       │
  Markov inequality    (need mean only)
```

---

## 8. Real-World Applications

| Application | Inequality Used |
|-------------|----------------|
| **Polling accuracy** | WLLN: larger sample → closer to true proportion |
| **Quality control** | Chebyshev: bound fraction of defective items |
| **Casino profits** | WLLN: house edge guarantees long-run profit |
| **Insurance** | WLLN: average claim converges to expected claim |
| **Algorithm analysis** | Chernoff: tail bounds for randomized algorithms |
| **Clinical trials** | Chebyshev: bound on treatment effect deviation |

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| Markov's inequality | $P(X \geq a) \leq E[X]/a$ |
| Chebyshev's inequality | $P(\|X-\mu\| \geq k\sigma) \leq 1/k^2$ |
| Complementary form | $P(\|X-\mu\| < k\sigma) \geq 1 - 1/k^2$ |
| 2$\sigma$ rule (Chebyshev) | $\geq 75\%$ within $\mu \pm 2\sigma$ |
| 3$\sigma$ rule (Chebyshev) | $\geq 88.9\%$ within $\mu \pm 3\sigma$ |
| $\text{Var}(\bar{X}_n)$ | $\sigma^2/n$ |
| WLLN | $\bar{X}_n \xrightarrow{P} \mu$ |
| Cantelli (one-sided) | $P(X-\mu \geq k\sigma) \leq 1/(1+k^2)$ |

---

## Quick Revision Questions

1. A dataset has mean 100 and standard deviation 15. Using Chebyshev's inequality, at least what percentage of values lie between 55 and 145?

2. Apply Markov's inequality: If the average number of bugs per 1000 lines of code is 5, what is the maximum probability of having $\geq 25$ bugs?

3. A factory produces resistors with mean 100Ω and SD 5Ω. The spec requires $100 \pm 12$Ω. Using Chebyshev, what is the maximum rejection rate?

4. You flip a fair coin 10,000 times. Using the WLLN (with Chebyshev), find an upper bound on $P(|\hat{p} - 0.5| \geq 0.01)$ where $\hat{p} = \bar{X}_n$.

5. Compare the Chebyshev bound with the actual probability for $P(|X| \geq 2)$ when $X \sim N(0,1)$.

6. Explain intuitively why Chebyshev's inequality is loose (conservative) for most practical distributions.

---

[← Previous: Moments and MGF](04-moments-and-mgf.md) | [Back to README](../README.md) | [Next: Bernoulli Distribution →](../05-Discrete-Distributions/01-bernoulli-distribution.md)
