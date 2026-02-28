# Chapter 4.3: Variance and Standard Deviation

[← Previous: Properties of Expectation](02-properties-of-expectation.md) | [Back to README](../README.md) | [Next: Moments and MGF →](04-moments-and-mgf.md)

---

## Overview

While the expected value tells us the **center** of a distribution, **variance** measures how **spread out** the values are around that center. Standard deviation puts the spread in the same units as the data.

---

## 1. Definition of Variance

$$\boxed{\text{Var}(X) = \sigma_X^2 = E\left[(X - \mu)^2\right]}$$

where $\mu = E[X]$.

### Computational Formula (shortcut)

$$\boxed{\text{Var}(X) = E[X^2] - (E[X])^2}$$

**Proof:**
$$\text{Var}(X) = E[(X-\mu)^2] = E[X^2 - 2\mu X + \mu^2] = E[X^2] - 2\mu E[X] + \mu^2 = E[X^2] - \mu^2 \quad \blacksquare$$

### Standard Deviation

$$\boxed{\sigma_X = \sqrt{\text{Var}(X)}}$$

```
  Low Variance                    High Variance
  
  p(x) or f(x)                   p(x) or f(x)
  │     ╱╲                        │
  │    ╱  ╲                        │   ╱╲      ╱╲
  │   ╱    ╲                       │  ╱  ╲  ╱╲╱  ╲
  │  ╱      ╲                      │ ╱    ╲╱      ╲
  │╱          ╲                    │╱                ╲
  └──────┼──────► x                └──────┼───────────► x
         μ                               μ
  
  Values clustered                 Values spread out
  near the mean                    far from the mean
```

---

## 2. Computing Variance — Examples

### Example 1: Fair Die

$E[X] = 3.5$, $E[X^2] = \frac{1+4+9+16+25+36}{6} = \frac{91}{6}$

$$\text{Var}(X) = \frac{91}{6} - (3.5)^2 = \frac{91}{6} - \frac{49}{4} = \frac{182 - 147}{12} = \frac{35}{12} \approx 2.917$$

$$\sigma = \sqrt{35/12} \approx 1.708$$

### Example 2: Bernoulli$(p)$

$E[X] = p$, $E[X^2] = 0^2(1-p) + 1^2(p) = p$

$$\text{Var}(X) = p - p^2 = p(1-p) = pq$$

### Example 3: Exponential($\lambda$)

$E[X] = 1/\lambda$, $E[X^2] = 2/\lambda^2$ (by integration by parts)

$$\text{Var}(X) = \frac{2}{\lambda^2} - \frac{1}{\lambda^2} = \frac{1}{\lambda^2}$$

---

## 3. Properties of Variance

| Property | Formula | Notes |
|----------|---------|-------|
| Non-negative | $\text{Var}(X) \geq 0$ | Always |
| Zero variance | $\text{Var}(X) = 0 \iff X = c$ a.s. | Constant RV |
| Add constant | $\text{Var}(X + c) = \text{Var}(X)$ | Shift doesn't affect spread |
| Scale | $\text{Var}(aX) = a^2 \text{Var}(X)$ | Note: $a^2$, not $a$ |
| Linear | $\text{Var}(aX + b) = a^2 \text{Var}(X)$ | Combined |
| Sum (indep.) | $\text{Var}(X+Y) = \text{Var}(X) + \text{Var}(Y)$ | Only if $X \perp Y$ |
| Sum (general) | $\text{Var}(X+Y) = \text{Var}(X) + \text{Var}(Y) + 2\text{Cov}(X,Y)$ | Always |

### Variance is NOT Linear

```
  ╔════════════════════════════════════════════════════════╗
  ║  IMPORTANT: Variance has DIFFERENT rules than E[X]    ║
  ║                                                        ║
  ║  E[aX + b]   = aE[X] + b        (linear)              ║
  ║  Var(aX + b) = a²Var(X)          (quadratic in a,      ║
  ║                                   constant b vanishes) ║
  ╚════════════════════════════════════════════════════════╝
```

### Proof: $\text{Var}(aX + b) = a^2 \text{Var}(X)$

$$\text{Var}(aX+b) = E[(aX+b)^2] - (E[aX+b])^2$$
$$= E[a^2X^2 + 2abX + b^2] - (aE[X] + b)^2$$
$$= a^2E[X^2] + 2abE[X] + b^2 - a^2(E[X])^2 - 2abE[X] - b^2$$
$$= a^2\left(E[X^2] - (E[X])^2\right) = a^2\text{Var}(X) \quad \blacksquare$$

---

## 4. Variance of Sum of Independent Variables

If $X_1, X_2, \ldots, X_n$ are **pairwise independent**:

$$\boxed{\text{Var}\left(\sum_{i=1}^n X_i\right) = \sum_{i=1}^n \text{Var}(X_i)}$$

### Example 4: Variance of Binomial

$X = X_1 + X_2 + \cdots + X_n$, $X_i \sim \text{Bernoulli}(p)$ independent

$$\text{Var}(X) = \sum_{i=1}^n \text{Var}(X_i) = n \cdot p(1-p) = npq$$

### Example 5: Sum of Two Dice

$S = X + Y$, $X \perp Y$, $\text{Var}(X) = \text{Var}(Y) = 35/12$

$$\text{Var}(S) = 35/12 + 35/12 = 35/6 \approx 5.833$$

---

## 5. Standardization

Given $X$ with mean $\mu$ and standard deviation $\sigma$, the **standardized** variable:

$$\boxed{Z = \frac{X - \mu}{\sigma}}$$

has $E[Z] = 0$ and $\text{Var}(Z) = 1$.

**Proof:**
$$E[Z] = E\left[\frac{X-\mu}{\sigma}\right] = \frac{E[X] - \mu}{\sigma} = 0$$

$$\text{Var}(Z) = \text{Var}\left(\frac{X-\mu}{\sigma}\right) = \frac{1}{\sigma^2}\text{Var}(X) = \frac{\sigma^2}{\sigma^2} = 1$$

```
  Before Standardization        After Standardization
  
  f(x)                          f(z)
  │      ╱╲                     │      ╱╲
  │    ╱    ╲                   │    ╱    ╲
  │  ╱        ╲                 │  ╱        ╲
  │╱            ╲               │╱            ╲
  └──────┼──────► x             └──────┼──────► z
         μ                            0
                                Mean=0, Var=1
```

---

## 6. Coefficient of Variation

$$\boxed{CV = \frac{\sigma}{\mu} \times 100\%}$$

Measures **relative** spread (useful for comparing distributions with different scales).

| Dataset | Mean | SD | CV |
|---------|------|-----|-----|
| Heights (cm) | 170 | 10 | 5.9% |
| Weights (kg) | 70 | 15 | 21.4% |
| Incomes ($) | 50,000 | 20,000 | 40.0% |

> Weights are relatively more variable than heights, and incomes more variable than both.

---

## 7. Variance for Common Distributions

| Distribution | Variance $\sigma^2$ | Std Dev $\sigma$ |
|:------------|:-------------------:|:----------------:|
| Bernoulli($p$) | $p(1-p)$ | $\sqrt{p(1-p)}$ |
| Binomial($n,p$) | $np(1-p)$ | $\sqrt{np(1-p)}$ |
| Poisson($\lambda$) | $\lambda$ | $\sqrt{\lambda}$ |
| Geometric($p$) | $(1-p)/p^2$ | $\sqrt{1-p}/p$ |
| Uniform($a,b$) cont. | $(b-a)^2/12$ | $(b-a)/\sqrt{12}$ |
| Exponential($\lambda$) | $1/\lambda^2$ | $1/\lambda$ |
| Normal($\mu,\sigma^2$) | $\sigma^2$ | $\sigma$ |

---

## 8. Real-World Applications

| Domain | Mean vs. Variance |
|--------|-------------------|
| **Finance** | Mean = expected return, Variance = risk |
| **Manufacturing** | Mean = target value, Variance = quality (lower = better) |
| **Exam scores** | Mean = class performance, Variance = spread of abilities |
| **Climate** | Mean temp = typical, Variance = weather variability |
| **Signal processing** | Mean = signal, Variance = noise power |

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| Variance (definition) | $E[(X-\mu)^2]$ |
| Variance (shortcut) | $E[X^2] - (E[X])^2$ |
| Standard deviation | $\sigma = \sqrt{\text{Var}(X)}$ |
| $\text{Var}(aX+b)$ | $a^2 \text{Var}(X)$ |
| Independent sum | $\text{Var}(\sum X_i) = \sum \text{Var}(X_i)$ |
| Standardization | $Z = (X-\mu)/\sigma$ |
| Bernoulli variance | $p(1-p)$ |
| Binomial variance | $np(1-p)$ |

---

## Quick Revision Questions

1. A random variable has $E[X] = 5$ and $E[X^2] = 30$. Find $\text{Var}(X)$ and $\sigma$.

2. If $\text{Var}(X) = 9$, find $\text{Var}(3X + 7)$ and $\text{Var}(-2X)$.

3. $X$ and $Y$ are independent with $\text{Var}(X) = 4$, $\text{Var}(Y) = 9$. Find $\text{Var}(X - Y)$ and $\text{Var}(2X + 3Y)$.

4. Prove that $\text{Var}(X) \geq 0$ for any random variable, and show $\text{Var}(X) = 0$ implies $X$ is constant.

5. For a Poisson random variable, the mean equals the variance ($\lambda$). If a dataset has mean 5 and variance 5, how is this evidence for a Poisson model?

6. The returns of two stocks have means 8% and 12%, and standard deviations 5% and 15% respectively. Which is "riskier" in absolute and relative (CV) terms?

---

[← Previous: Properties of Expectation](02-properties-of-expectation.md) | [Back to README](../README.md) | [Next: Moments and MGF →](04-moments-and-mgf.md)
