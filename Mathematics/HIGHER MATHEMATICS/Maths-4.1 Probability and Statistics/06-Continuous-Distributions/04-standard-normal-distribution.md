# Chapter 6.4: Standard Normal Distribution and Z-Tables

[← Previous: Normal Distribution](03-normal-distribution.md) | [Back to README](../README.md) | [Next: Gamma and Beta Distributions →](05-gamma-and-beta-distributions.md)

---

## Overview

The **Standard Normal distribution** $Z \sim N(0, 1)$ is the normalized version of any Normal distribution. Standardization allows us to use a single Z-table for probabilities of all Normal distributions.

---

## 1. Definition

$$Z \sim N(0, 1)$$

$$\boxed{f(z) = \frac{1}{\sqrt{2\pi}} e^{-z^2/2}, \quad -\infty < z < \infty}$$

### Standard Normal CDF (Phi Function)

$$\Phi(z) = P(Z \leq z) = \int_{-\infty}^{z} \frac{1}{\sqrt{2\pi}} e^{-t^2/2}\, dt$$

> This integral has **no closed form** — values are obtained from tables or software.

---

## 2. Z-Table (Selected Values)

| $z$ | $\Phi(z)$ | $z$ | $\Phi(z)$ | $z$ | $\Phi(z)$ |
|:---:|:---------:|:---:|:---------:|:---:|:---------:|
| -3.00 | 0.0013 | -0.50 | 0.3085 | 1.50 | 0.9332 |
| -2.50 | 0.0062 | -0.25 | 0.4013 | 1.645 | 0.9500 |
| -2.33 | 0.0099 | 0.00 | 0.5000 | 1.96 | 0.9750 |
| -2.00 | 0.0228 | 0.25 | 0.5987 | 2.00 | 0.9772 |
| -1.96 | 0.0250 | 0.50 | 0.6915 | 2.33 | 0.9901 |
| -1.645 | 0.0500 | 0.674 | 0.7500 | 2.50 | 0.9938 |
| -1.28 | 0.1003 | 1.00 | 0.8413 | 2.576 | 0.9950 |
| -1.00 | 0.1587 | 1.28 | 0.8997 | 3.00 | 0.9987 |

---

## 3. Standardization

For $X \sim N(\mu, \sigma^2)$:

$$\boxed{Z = \frac{X - \mu}{\sigma} \sim N(0, 1)}$$

$$P(X \leq x) = P\left(Z \leq \frac{x - \mu}{\sigma}\right) = \Phi\left(\frac{x - \mu}{\sigma}\right)$$

```
  Standardization Process
  
  X ~ N(μ, σ²)                     Z ~ N(0, 1)
  
       ╱╲                              ╱╲
     ╱    ╲                           ╱    ╲
   ╱        ╲          Z=(X-μ)/σ    ╱        ╲
  ╱────────────╲    ──────────►   ╱────────────╲
  ──┼──────┼────► x              ──┼──────┼────► z
    μ-σ    μ+σ                    -1      +1
       μ                              0
```

---

## 4. Symmetry Properties

$$\boxed{\Phi(-z) = 1 - \Phi(z)}$$

$$P(Z > z) = 1 - \Phi(z)$$

$$P(-a < Z < a) = 2\Phi(a) - 1$$

$$P(|Z| > a) = 2(1 - \Phi(a))$$

```
  Symmetry of the Standard Normal
  
        Φ(-z) = shaded left        1-Φ(z) = shaded right
  
       ╱╲                              ╱╲
     ╱    ╲                           ╱    ╲
   ▓▓      ╲                        ╱      ▓▓
  ▓▓▓▓──────╲──                  ──╱──────▓▓▓▓
  ──┼───┼───── z                  z ─┼───┼──── 
   -z   0                         0  z
  
  These two areas are EQUAL:  Φ(-z) = 1 - Φ(z)
```

---

## 5. Common Probability Calculations

### Type 1: $P(X \leq x)$ — Left tail

$$P(X \leq 65) \text{ where } X \sim N(70, 25)$$
$$z = \frac{65-70}{5} = -1.0 \implies \Phi(-1) = 0.1587$$

### Type 2: $P(X > x)$ — Right tail

$$P(X > 78) = 1 - \Phi\left(\frac{78-70}{5}\right) = 1 - \Phi(1.6) = 1 - 0.9452 = 0.0548$$

### Type 3: $P(a < X < b)$ — Between

$$P(62 < X < 78) = \Phi(1.6) - \Phi(-1.6) = 0.9452 - 0.0548 = 0.8904$$

### Type 4: Find $x$ given probability (Inverse)

Find $x$ such that $P(X < x) = 0.9$

$$z_{0.9} = 1.2816 \implies x = 70 + 1.2816(5) = 76.41$$

---

## 6. Critical Z-Values

These appear constantly in confidence intervals and hypothesis testing:

| Confidence Level | $\alpha$ (two-tailed) | $z_{\alpha/2}$ |
|:----------------:|:---------------------:|:--------------:|
| 80% | 0.20 | 1.282 |
| 90% | 0.10 | **1.645** |
| 95% | 0.05 | **1.960** |
| 98% | 0.02 | 2.326 |
| 99% | 0.01 | **2.576** |
| 99.9% | 0.001 | 3.291 |

> Memorize: $z_{0.025} = 1.96$ and $z_{0.05} = 1.645$.

---

## 7. Percentiles and Quantiles

The $p$-th percentile $z_p$ satisfies $\Phi(z_p) = p$:

| Percentile | $z_p$ |
|:----------:|:-----:|
| 1st | -2.326 |
| 5th | -1.645 |
| 10th | -1.282 |
| 25th (Q1) | -0.674 |
| 50th (Median) | 0.000 |
| 75th (Q3) | 0.674 |
| 90th | 1.282 |
| 95th | 1.645 |
| 99th | 2.326 |

### IQR for Standard Normal

$$\text{IQR} = Q_3 - Q_1 = 0.674 - (-0.674) = 1.349$$

For $N(\mu, \sigma^2)$: $\text{IQR} = 1.349\sigma$

---

## 8. Worked Examples

### Example 1: Grading on a Curve

Exam scores $\sim N(72, 64)$. The top 10% get A's. Minimum score for A?

$z_{0.90} = 1.282$

$$x = 72 + 1.282 \times 8 = 82.3$$

### Example 2: Manufacturing Tolerance

Parts must be $50 \pm 0.5$ mm. $X \sim N(50, 0.04)$.

$$P(49.5 < X < 50.5) = \Phi\left(\frac{0.5}{0.2}\right) - \Phi\left(\frac{-0.5}{0.2}\right) = \Phi(2.5) - \Phi(-2.5)$$
$$= 0.9938 - 0.0062 = 0.9876$$

About 1.24% of parts will be out of spec.

### Example 3: Normal Approximation to Binomial

$X \sim \text{Bin}(100, 0.4)$. Find $P(X \leq 45)$.

$\mu = 40$, $\sigma = \sqrt{24} \approx 4.899$

With continuity correction: $P(X \leq 45.5)$

$$z = \frac{45.5 - 40}{4.899} = 1.122 \implies P \approx \Phi(1.122) \approx 0.8691$$

---

## 9. Continuity Correction

When approximating a discrete distribution with the Normal:

| Discrete probability | Normal approximation |
|---------------------|---------------------|
| $P(X = k)$ | $P(k-0.5 < Y < k+0.5)$ |
| $P(X \leq k)$ | $P(Y < k+0.5)$ |
| $P(X < k)$ | $P(Y < k-0.5)$ |
| $P(X \geq k)$ | $P(Y > k-0.5)$ |
| $P(X > k)$ | $P(Y > k+0.5)$ |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Standard Normal PDF | $\frac{1}{\sqrt{2\pi}}e^{-z^2/2}$ |
| CDF (Phi) | $\Phi(z) = P(Z \leq z)$ — from table |
| Standardization | $Z = (X-\mu)/\sigma$ |
| Symmetry | $\Phi(-z) = 1 - \Phi(z)$ |
| Between | $P(a<Z<b) = \Phi(b) - \Phi(a)$ |
| 95% CI $z$-value | $z_{0.025} = 1.96$ |
| Inverse | $x = \mu + z_p \cdot \sigma$ |

---

## Quick Revision Questions

1. Find $P(Z > 1.5)$, $P(-1.2 < Z < 0.8)$, and $P(|Z| > 2)$ using the Z-table.

2. If $X \sim N(100, 225)$, find $P(X > 130)$ and the 95th percentile of $X$.

3. Explain why $\Phi(-z) = 1 - \Phi(z)$ and illustrate with a diagram.

4. A machine fills bottles with mean 500 ml and SD 5 ml (Normal). What proportion of bottles contain between 490 and 510 ml?

5. Using Normal approximation with continuity correction, find $P(X \geq 60)$ where $X \sim \text{Bin}(100, 0.5)$.

6. Why do we need the continuity correction when using the Normal to approximate a discrete distribution?

---

[← Previous: Normal Distribution](03-normal-distribution.md) | [Back to README](../README.md) | [Next: Gamma and Beta Distributions →](05-gamma-and-beta-distributions.md)
