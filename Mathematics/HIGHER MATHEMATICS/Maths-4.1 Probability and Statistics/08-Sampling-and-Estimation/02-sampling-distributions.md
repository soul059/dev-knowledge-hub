# Chapter 8.2: Sampling Distributions

[← Previous: Population and Sample](01-population-and-sample.md) | [Back to README](../README.md) | [Next: Central Limit Theorem →](03-central-limit-theorem.md)

---

## Overview

A **sampling distribution** is the probability distribution of a statistic (like $\bar{X}$ or $S^2$) computed from a random sample. Understanding sampling distributions is the bridge between probability theory and statistical inference.

---

## 1. What Is a Sampling Distribution?

A statistic $T = g(X_1, X_2, \ldots, X_n)$ is a function of the sample. Since the sample is random, $T$ is a random variable with its own distribution.

```
  Population (μ, σ²)
       │
       ▼
  Draw sample of size n → Compute X̄₁
  Draw sample of size n → Compute X̄₂     ──► Distribution
  Draw sample of size n → Compute X̄₃         of X̄
  Draw sample of size n → Compute X̄₄
       ⋮                      ⋮
  
  The distribution of ALL possible X̄ values
  = Sampling Distribution of X̄
```

---

## 2. Sampling Distribution of $\bar{X}$

Let $X_1, \ldots, X_n$ be iid with $E[X_i] = \mu$, $\text{Var}(X_i) = \sigma^2$.

$$\boxed{E[\bar{X}] = \mu, \quad \text{Var}(\bar{X}) = \frac{\sigma^2}{n}, \quad \text{SE}(\bar{X}) = \frac{\sigma}{\sqrt{n}}}$$

where SE = **Standard Error** (standard deviation of the sampling distribution).

### Effect of Sample Size

```
  Distribution of X̄ for different n
  
  n = 1 (individual observations)
  ╱╲           Wide, same as population
  
  n = 10
   ╱╲          Narrower
  
  n = 30
    ╱╲         Even narrower
  
  n = 100
    │╲         Very concentrated around μ
    ╱│
  
  ──────────────────────────► x
              μ
  
  As n → ∞: X̄ → μ (concentrates at μ)
```

### Special Cases

| Population | Distribution of $\bar{X}$ |
|-----------|--------------------------|
| Normal: $X_i \sim N(\mu, \sigma^2)$ | $\bar{X} \sim N(\mu, \sigma^2/n)$ **exactly** |
| Any distribution, large $n$ | $\bar{X} \approx N(\mu, \sigma^2/n)$ by CLT |
| Bernoulli($p$) | $\hat{p} = \bar{X} \approx N(p, p(1-p)/n)$ for large $n$ |

---

## 3. Sampling Distribution of $S^2$

Let $X_1, \ldots, X_n \overset{\text{iid}}{\sim} N(\mu, \sigma^2)$.

$$\boxed{\frac{(n-1)S^2}{\sigma^2} \sim \chi^2(n-1)}$$

Properties:
- $E[S^2] = \sigma^2$ (unbiased)
- $\text{Var}(S^2) = \frac{2\sigma^4}{n-1}$

```
  Distribution of (n-1)S²/σ²
  
  f(x)
  │╲
  │ ╲
  │  ╲        χ²(n-1) distribution
  │   ╲
  │    ╲
  │     ╲___________
  └──────────────────► x
  0     n-1
        (mean)
  
  Right-skewed; becomes more symmetric as n increases
```

---

## 4. Key Sampling Distributions

### The "Big Three" for Normal Populations

| Statistic | Distribution | Conditions |
|-----------|-------------|------------|
| $\frac{\bar{X} - \mu}{\sigma/\sqrt{n}}$ | $Z \sim N(0,1)$ | $\sigma$ known |
| $\frac{\bar{X} - \mu}{S/\sqrt{n}}$ | $t(n-1)$ | $\sigma$ unknown, Normal pop |
| $\frac{(n-1)S^2}{\sigma^2}$ | $\chi^2(n-1)$ | Normal population |

### Derivation of the t-distribution

$$T = \frac{\bar{X} - \mu}{S/\sqrt{n}} = \frac{(\bar{X}-\mu)/(\sigma/\sqrt{n})}{S/\sigma} = \frac{Z}{\sqrt{\chi^2(n-1)/(n-1)}}$$

where $Z$ and $\chi^2(n-1)$ are independent.

```
  Why t instead of Z?
  
  When σ known:           When σ unknown:
  
  Z = (X̄ - μ)/(σ/√n)    T = (X̄ - μ)/(S/√n)
  
  ──── N(0,1) ────        ──── t(n-1) ────
  
       │╲                      │╲
      ╱│ ╲                   ╱╱│ ╲╲   ← heavier tails
     ╱ │  ╲                 ╱  │  ╲    (extra uncertainty
    ╱  │   ╲               ╱   │   ╲    from estimating σ)
  ─╱───┼────╲─           ─╱────┼────╲─
       0                       0
  
  As n → ∞: t(n-1) → N(0,1) (S → σ)
```

---

## 5. Two-Sample Results

### Difference of Means

If $X_1, \ldots, X_m \overset{\text{iid}}{\sim} N(\mu_1, \sigma_1^2)$ and $Y_1, \ldots, Y_n \overset{\text{iid}}{\sim} N(\mu_2, \sigma_2^2)$ are independent:

$$\bar{X} - \bar{Y} \sim N\left(\mu_1 - \mu_2, \frac{\sigma_1^2}{m} + \frac{\sigma_2^2}{n}\right)$$

### Ratio of Variances

$$\frac{S_1^2/\sigma_1^2}{S_2^2/\sigma_2^2} \sim F(m-1, n-1)$$

Used to compare variances of two populations.

---

## 6. Finite Population Correction

When sampling **without replacement** from a finite population of size $N$:

$$\text{Var}(\bar{X}) = \frac{\sigma^2}{n} \cdot \underbrace{\frac{N - n}{N - 1}}_{\text{FPC factor}}$$

- If $n/N$ is small (< 5%), FPC $\approx 1$ (ignore it)
- If $n = N$, FPC $= 0$ (no uncertainty — census!)

---

## 7. Order Statistics

Given sample $X_1, \ldots, X_n$, the **order statistics** are:

$$X_{(1)} \leq X_{(2)} \leq \cdots \leq X_{(n)}$$

| Statistic | Notation | Formula |
|-----------|----------|---------|
| Minimum | $X_{(1)}$ | Smallest value |
| Maximum | $X_{(n)}$ | Largest value |
| Range | $R$ | $X_{(n)} - X_{(1)}$ |
| Median | $\tilde{X}$ | $X_{((n+1)/2)}$ if $n$ odd |
| IQR | $Q_3 - Q_1$ | 75th percentile − 25th percentile |

---

## Summary Table

| Statistic | Sampling Distribution | Conditions |
|-----------|----------------------|------------|
| $\bar{X}$ | $N(\mu, \sigma^2/n)$ | Normal pop or large $n$ |
| $(n-1)S^2/\sigma^2$ | $\chi^2(n-1)$ | Normal population |
| $(\bar{X}-\mu)/(S/\sqrt{n})$ | $t(n-1)$ | Normal pop, $\sigma$ unknown |
| $S_1^2/S_2^2$ | $F(m-1, n-1)$ (if $\sigma_1=\sigma_2$) | Normal pops |
| $\hat{p}$ | $\approx N(p, p(1-p)/n)$ | $np \geq 5, n(1-p) \geq 5$ |
| SE of $\bar{X}$ | $\sigma/\sqrt{n}$ | — |

---

## Quick Revision Questions

1. If $X_i \sim N(50, 16)$ and $n = 25$, find the distribution of $\bar{X}$ and $P(\bar{X} > 52)$.

2. Explain why the $t$-distribution has heavier tails than the standard Normal.

3. A sample of $n = 10$ from a Normal population has $S^2 = 15$. Find $P(S^2 > 15)$ if $\sigma^2 = 10$.

4. What happens to the standard error of $\bar{X}$ when the sample size is quadrupled?

5. State the condition under which using FPC is necessary. Calculate the FPC for $N = 500, n = 100$.

---

[← Previous: Population and Sample](01-population-and-sample.md) | [Back to README](../README.md) | [Next: Central Limit Theorem →](03-central-limit-theorem.md)
