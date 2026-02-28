# Chapter 6.3: Normal Distribution

[← Previous: Exponential Distribution](02-exponential-distribution.md) | [Back to README](../README.md) | [Next: Standard Normal Distribution →](04-standard-normal-distribution.md)

---

## Overview

The **Normal (Gaussian) distribution** is the most important distribution in statistics. Its bell-shaped curve arises naturally due to the Central Limit Theorem — sums of many independent random variables tend toward normality regardless of their individual distributions.

---

## 1. Definition

$$X \sim N(\mu, \sigma^2)$$

### PDF

$$\boxed{f(x) = \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right), \quad -\infty < x < \infty}$$

### Parameters

| Parameter | Meaning | Effect |
|-----------|---------|--------|
| $\mu$ | Mean (location) | Shifts the bell curve left/right |
| $\sigma^2$ | Variance (scale) | Controls the width/spread |
| $\sigma$ | Standard deviation | Width of the bell at inflection points |

```
  The Bell Curve — N(μ, σ²)
  
  f(x)
  │           ╱╲
  │         ╱    ╲
  │       ╱   ▲    ╲          Inflection points
  │     ╱     │      ╲        at x = μ ± σ
  │   ╱       │        ╲
  │ ╱         │          ╲
  │╱──────────┼────────────╲──────
  └───────────┼───────────────► x
         μ-2σ  μ-σ  μ  μ+σ  μ+2σ
```

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $\mu$ |
| Variance | $\sigma^2$ |
| Median | $\mu$ |
| Mode | $\mu$ |
| Skewness | 0 (perfectly symmetric) |
| Kurtosis | 3 (excess kurtosis = 0) |
| MGF | $e^{\mu t + \sigma^2 t^2/2}$ |
| Inflection points | $x = \mu \pm \sigma$ |

---

## 3. The 68–95–99.7 Rule (Empirical Rule)

$$P(\mu - k\sigma < X < \mu + k\sigma)$$

| Range | Probability |
|:-----:|:-----------:|
| $\mu \pm 1\sigma$ | **68.27%** |
| $\mu \pm 2\sigma$ | **95.45%** |
| $\mu \pm 3\sigma$ | **99.73%** |

```
  68-95-99.7 Rule
  
              99.7%
         ◄────────────────────►
              95.4%
           ◄──────────────►
              68.3%
             ◄────────►
  
            ╱──────────╲
          ╱  ╱────────╲  ╲
        ╱  ╱  ╱──────╲  ╲  ╲
      ╱  ╱  ╱          ╲  ╲  ╲
    ╱  ╱  ╱              ╲  ╲  ╲
  ╱──╱──╱                  ╲──╲──╲───
  ───┼──┼──┼──────┼──────┼──┼──┼───► x
   μ-3σ μ-2σ μ-σ  μ    μ+σ μ+2σ μ+3σ
     │         │   34.1%  │         │
     │    13.6%│          │13.6%    │
     │2.1%     │          │    2.1% │
```

---

## 4. Linear Transformations

If $X \sim N(\mu, \sigma^2)$, then:

$$\boxed{aX + b \sim N(a\mu + b, \; a^2\sigma^2)}$$

### Standardization

$$Z = \frac{X - \mu}{\sigma} \sim N(0, 1)$$

### Sum of Independent Normals

If $X_i \sim N(\mu_i, \sigma_i^2)$ independently:

$$\sum_{i=1}^n X_i \sim N\left(\sum \mu_i, \; \sum \sigma_i^2\right)$$

---

## 5. Why is the Normal So Important?

### Central Limit Theorem (Preview)

For i.i.d. $X_1, X_2, \ldots, X_n$ with mean $\mu$ and variance $\sigma^2$:

$$\frac{\bar{X}_n - \mu}{\sigma/\sqrt{n}} \xrightarrow{d} N(0, 1) \quad \text{as } n \to \infty$$

> Sums (and averages) of many independent variables become approximately Normal, no matter the original distribution!

### Examples of Approximately Normal Data

- Heights of a population
- Measurement errors
- IQ scores
- Stock returns (approximately)
- Sum of many small random effects

---

## 6. Worked Examples

### Example 1: Heights

Men's heights: $\mu = 175$ cm, $\sigma = 7$ cm.

$P(\text{height} > 189)$:

$$Z = \frac{189 - 175}{7} = 2 \implies P(Z > 2) = 1 - 0.9772 = 0.0228$$

About 2.28% of men are taller than 189 cm.

### Example 2: Exam Scores

$\mu = 70$, $\sigma = 10$. Find the score at the 90th percentile.

$$z_{0.90} = 1.2816 \implies x = 70 + 1.2816 \times 10 = 82.8$$

### Example 3: Sum of Independent Normals

$X \sim N(100, 25)$ and $Y \sim N(50, 16)$ independent.

$$X + Y \sim N(150, 41)$$

$$P(X + Y > 160) = P\left(Z > \frac{160-150}{\sqrt{41}}\right) = P(Z > 1.562) \approx 0.0591$$

---

## 7. The Normal PDF — Key Features

| Feature | Detail |
|---------|--------|
| Always positive | $f(x) > 0$ for all $x$ |
| Symmetric | $f(\mu + x) = f(\mu - x)$ |
| Max at $x = \mu$ | $f(\mu) = \frac{1}{\sigma\sqrt{2\pi}}$ |
| Inflection points | At $x = \mu \pm \sigma$ |
| Tails approach 0 | $f(x) \to 0$ as $x \to \pm\infty$ |
| Area = 1 | $\int_{-\infty}^{\infty} f(x)\,dx = 1$ (Gaussian integral) |

---

## 8. Effect of Parameters

```
  Changing μ (location shift):
  
  │       N(0,1)        N(3,1)
  │        ╱╲             ╱╲
  │      ╱    ╲         ╱    ╲
  │    ╱        ╲     ╱        ╲
  └──╱────────────╲─╱────────────╲──► x
    -3  -1   0   1   2   3   4   6
  
  Changing σ (spread):
  
  │     N(0, 0.5²)
  │       ╱╲           σ=0.5 tall & narrow
  │     ╱    ╲
  │    ╱  ╱────╲── N(0, 1²)
  │   ╱ ╱────────╲──── N(0, 2²)
  └──╱╱──────────────╲╲──────────► x
    -4  -2   0    2    4
                          σ=2 short & wide
```

---

## Summary Table

| Concept | Formula |
|---------|---------|
| PDF | $\frac{1}{\sigma\sqrt{2\pi}}e^{-(x-\mu)^2/(2\sigma^2)}$ |
| Mean, Median, Mode | All equal $\mu$ |
| Variance | $\sigma^2$ |
| Standardization | $Z = (X-\mu)/\sigma$ |
| 68–95–99.7 | Within $1\sigma$, $2\sigma$, $3\sigma$ |
| Linear transform | $aX+b \sim N(a\mu+b, a^2\sigma^2)$ |
| Sum of independents | $\sum X_i \sim N(\sum\mu_i, \sum\sigma_i^2)$ |
| MGF | $e^{\mu t + \sigma^2 t^2/2}$ |

---

## Quick Revision Questions

1. If $X \sim N(50, 16)$, find $P(X > 54)$, $P(46 < X < 54)$, and the value $c$ such that $P(X < c) = 0.95$.

2. Use the 68–95–99.7 rule: IQ scores are $N(100, 225)$. What percentage of people have IQs between 85 and 130?

3. If $X \sim N(10, 4)$ and $Y \sim N(20, 9)$ are independent, find the distribution of $X + Y$ and $2X - Y$.

4. Show that the inflection points of the Normal PDF are at $x = \mu \pm \sigma$ by finding where $f''(x) = 0$.

5. Explain why the Normal distribution is so prevalent in nature, referencing the Central Limit Theorem.

6. The weights of packages are $N(500, 100)$ grams. What proportion of packages weigh between 480 and 530 grams?

---

[← Previous: Exponential Distribution](02-exponential-distribution.md) | [Back to README](../README.md) | [Next: Standard Normal Distribution →](04-standard-normal-distribution.md)
