# Chapter 6.5: Gamma and Beta Distributions

[← Previous: Standard Normal Distribution](04-standard-normal-distribution.md) | [Back to README](../README.md) | [Next: Chi-Square Distribution →](06-chi-square-distribution.md)

---

## Overview

The **Gamma distribution** generalizes the Exponential to model waiting times for multiple events. The **Beta distribution** models probabilities and proportions on $[0,1]$. Both are versatile families used extensively in Bayesian statistics and reliability engineering.

---

## 1. Gamma Function

$$\boxed{\Gamma(\alpha) = \int_0^{\infty} t^{\alpha-1} e^{-t}\, dt, \quad \alpha > 0}$$

### Key Properties

| Property | Value |
|----------|-------|
| $\Gamma(n) = (n-1)!$ | For positive integers |
| $\Gamma(\alpha+1) = \alpha\Gamma(\alpha)$ | Recursive |
| $\Gamma(1) = 1$ | |
| $\Gamma(1/2) = \sqrt{\pi}$ | |
| $\Gamma(n + 1/2)$ | $\frac{(2n)!}{4^n \cdot n!}\sqrt{\pi}$ |

---

## 2. Gamma Distribution

$$X \sim \text{Gamma}(\alpha, \beta) \quad (\alpha = \text{shape}, \; \beta = \text{rate})$$

### PDF

$$\boxed{f(x) = \frac{\beta^\alpha}{\Gamma(\alpha)} x^{\alpha-1} e^{-\beta x}, \quad x > 0}$$

> Alternative parameterization: $\theta = 1/\beta$ (scale), so $f(x) = \frac{x^{\alpha-1}e^{-x/\theta}}{\theta^\alpha \Gamma(\alpha)}$

### Key Properties

| Property | Shape-Rate $(\alpha, \beta)$ | Shape-Scale $(\alpha, \theta)$ |
|----------|:--:|:--:|
| Mean | $\alpha/\beta$ | $\alpha\theta$ |
| Variance | $\alpha/\beta^2$ | $\alpha\theta^2$ |
| Mode | $(\alpha-1)/\beta$ for $\alpha \geq 1$ | $(\alpha-1)\theta$ |
| MGF | $\left(\frac{\beta}{\beta-t}\right)^\alpha, t<\beta$ | $(1-\theta t)^{-\alpha}$ |

```
  Gamma PDF shapes (β = 1):
  
  f(x)
  │╲  α=1 (Exponential)
  │  ╲
  │    ╲  ╱╲  α=2
  │      ╳    ╲
  │    ╱   ╲  ╱╲  α=5
  │  ╱       ╳    ╲
  │╱       ╱   ╲    ╲
  └──────╱───────╲────╲───► x
  0    2    4    6    8   10
  
  α=1: Exponential (monotone decreasing)
  α>1: Bell-shaped, skewed right
  Larger α → more symmetric, peak moves right
```

---

## 3. Special Cases of Gamma

| Distribution | Gamma Parameters |
|:--:|:--:|
| Exponential($\lambda$) | Gamma$(1, \lambda)$ |
| Chi-square($k$) | Gamma$(k/2, 1/2)$ |
| Erlang($n, \lambda$) | Gamma$(n, \lambda)$, $n$ integer |

### Additivity

If $X_i \sim \text{Gamma}(\alpha_i, \beta)$ independent (same rate $\beta$):

$$\sum X_i \sim \text{Gamma}\left(\sum \alpha_i, \; \beta\right)$$

---

## 4. Beta Distribution

$$X \sim \text{Beta}(\alpha, \beta), \quad \alpha > 0, \beta > 0$$

### PDF

$$\boxed{f(x) = \frac{x^{\alpha-1}(1-x)^{\beta-1}}{B(\alpha, \beta)}, \quad 0 < x < 1}$$

where $B(\alpha, \beta) = \frac{\Gamma(\alpha)\Gamma(\beta)}{\Gamma(\alpha+\beta)}$ is the **Beta function**.

### Key Properties

| Property | Value |
|----------|-------|
| Mean | $\frac{\alpha}{\alpha+\beta}$ |
| Variance | $\frac{\alpha\beta}{(\alpha+\beta)^2(\alpha+\beta+1)}$ |
| Mode | $\frac{\alpha-1}{\alpha+\beta-2}$ for $\alpha, \beta > 1$ |
| Support | $[0, 1]$ |

---

## 5. Shapes of the Beta Distribution

```
  Beta(α, β) — Extremely flexible on [0, 1]
  
  α=0.5, β=0.5 (U-shaped)     α=1, β=1 (Uniform)      α=2, β=5 (left-skewed)
  
  │╲              ╱│           │──────────────│          │  ╱╲
  │  ╲          ╱  │           │              │          │╱    ╲
  │    ╲      ╱    │           │              │          │       ╲
  │      ╲  ╱      │           │              │          │         ╲
  └────────────────┘           └──────────────┘          └──────────╲──
  0                1           0              1          0           1
  
  α=5, β=2 (right-skewed)     α=5, β=5 (symmetric)     α=1, β=3 (J-shaped)
  
  │        ╱╲                  │       ╱╲                │╲
  │      ╱    ╲│               │     ╱    ╲              │  ╲
  │    ╱       │               │   ╱        ╲            │    ╲
  │  ╱         │               │ ╱            ╲          │      ╲───
  └╱───────────┘               └╱──────────────╲         └──────────
  0            1               0                1        0          1
```

### Special Cases

| Parameters | Distribution |
|:--:|:--:|
| Beta(1, 1) | Uniform(0, 1) |
| Beta(1/2, 1/2) | Arcsine distribution |
| Beta($\alpha$, 1) | Power function |

---

## 6. Beta as Conjugate Prior

In Bayesian statistics, the Beta is the **conjugate prior** for the Bernoulli/Binomial:

$$\text{Prior: } p \sim \text{Beta}(\alpha, \beta)$$
$$\text{Data: } k \text{ successes in } n \text{ trials}$$
$$\text{Posterior: } p \mid \text{data} \sim \text{Beta}(\alpha + k, \; \beta + n - k)$$

```
  Bayesian Updating with Beta-Binomial
  
  Prior Beta(2,2)     + 7 successes    = Posterior Beta(9,5)
                        in 12 trials
  
       ╱╲                                    ╱╲
     ╱    ╲         Observe data           ╱    ╲
   ╱        ╲      ──────────────►      ╱         ╲
  ╱────────────╲                       ╱──────────  ╲
  0     0.5     1                     0    0.6  0.8  1
  
  Symmetric,              ──────►     Peaked near 7/12 ≈ 0.58
  uncertain                           More concentrated
```

---

## 7. Gamma–Beta Relationship

If $X \sim \text{Gamma}(\alpha, 1)$ and $Y \sim \text{Gamma}(\beta, 1)$ independently:

$$\frac{X}{X+Y} \sim \text{Beta}(\alpha, \beta)$$

---

## 8. Real-World Applications

### Gamma Distribution

| Application | $\alpha$ | Use |
|-------------|----------|-----|
| Time until $k$-th event | $k$ | Poisson process |
| Insurance claims | Shape | Right-skewed monetary data |
| Rainfall amounts | Shape | Climate modeling |
| Queuing wait times | $n$ | Service systems |

### Beta Distribution

| Application | $\alpha, \beta$ | Use |
|-------------|----------------|-----|
| Batting averages | Hits, misses | Baseball statistics |
| Click-through rates | Clicks, non-clicks | A/B testing |
| Allele frequencies | Observed counts | Population genetics |
| Project completion | Shape | PERT estimation |

---

## Summary Table

| Concept | Gamma | Beta |
|---------|-------|------|
| Support | $(0, \infty)$ | $(0, 1)$ |
| Mean | $\alpha/\beta$ | $\alpha/(\alpha+\beta)$ |
| Variance | $\alpha/\beta^2$ | $\frac{\alpha\beta}{(\alpha+\beta)^2(\alpha+\beta+1)}$ |
| Special cases | Exponential, Chi-square | Uniform, Arcsine |
| Key use | Waiting times | Probabilities/proportions |
| Bayesian role | Prior for rates | Prior for probabilities |

---

## Quick Revision Questions

1. Show that $\text{Gamma}(1, \lambda)$ is the Exponential($\lambda$) distribution.

2. If $X \sim \text{Gamma}(3, 2)$, find $E[X]$ and $\text{Var}(X)$.

3. Sketch the Beta(2, 2) and Beta(0.5, 0.5) distributions. Which is symmetric? Which is U-shaped?

4. A coin has unknown probability $p$ of heads. Prior: $p \sim \text{Beta}(1, 1)$. You observe 7 heads in 10 flips. What is the posterior distribution? What is the posterior mean?

5. Verify that $\Gamma(5) = 24 = 4!$ using the recursive property.

6. If $X_1, X_2, X_3$ are i.i.d. $\text{Exp}(2)$, what is the distribution of $X_1 + X_2 + X_3$? Find its mean and variance.

---

[← Previous: Standard Normal Distribution](04-standard-normal-distribution.md) | [Back to README](../README.md) | [Next: Chi-Square Distribution →](06-chi-square-distribution.md)
