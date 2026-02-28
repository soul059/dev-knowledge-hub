# Chapter 8.1: Population and Sample

[← Previous: Covariance and Correlation](../07-Joint-Distributions/05-covariance-and-correlation.md) | [Back to README](../README.md) | [Next: Sampling Distributions →](02-sampling-distributions.md)

---

## Overview

**Statistical inference** is the art of drawing conclusions about a large group (population) based on a smaller group (sample). This chapter introduces the foundational terminology and concepts that underpin all of inferential statistics.

---

## 1. Population vs Sample

| Concept | Population | Sample |
|---------|-----------|--------|
| Definition | Entire group of interest | Subset selected from population |
| Size | $N$ (often very large or infinite) | $n$ (manageable) |
| Characteristics | **Parameters** (fixed, unknown) | **Statistics** (computed, known) |
| Notation | Greek letters ($\mu, \sigma, p$) | Roman letters ($\bar{x}, s, \hat{p}$) |
| Goal | What we want to know | What we observe |

```
  ┌─────────────────────────────────────────┐
  │            POPULATION (N)               │
  │                                         │
  │    Parameters: μ, σ², p                 │
  │    (fixed but unknown)                  │
  │                                         │
  │     ┌─────────────────┐                 │
  │     │  SAMPLE (n)     │                 │
  │     │                 │                 │
  │     │  Statistics:    │                 │
  │     │  x̄, s², p̂      │                 │
  │     │  (calculated)   │                 │
  │     └─────────────────┘                 │
  │                                         │
  └─────────────────────────────────────────┘
  
  We INFER population parameters FROM sample statistics
```

---

## 2. Parameters vs Statistics

| Symbol | Name | Type | Formula |
|--------|------|------|---------|
| $\mu$ | Population mean | Parameter | $\mu = \frac{1}{N}\sum_{i=1}^N x_i$ |
| $\sigma^2$ | Population variance | Parameter | $\sigma^2 = \frac{1}{N}\sum(x_i - \mu)^2$ |
| $p$ | Population proportion | Parameter | True proportion with characteristic |
| $\bar{X}$ | Sample mean | Statistic | $\bar{X} = \frac{1}{n}\sum_{i=1}^n X_i$ |
| $S^2$ | Sample variance | Statistic | $S^2 = \frac{1}{n-1}\sum(X_i - \bar{X})^2$ |
| $\hat{p}$ | Sample proportion | Statistic | $\hat{p} = X/n$ where $X$ = count |

### Why $n-1$ in Sample Variance?

The divisor $n - 1$ (called **Bessel's correction**) is used to make $S^2$ an **unbiased estimator** of $\sigma^2$:

$$E[S^2] = \sigma^2$$

Using $n$ instead would systematically underestimate $\sigma^2$.

```
  With divisor n:     E[S²_biased] = (n-1)/n · σ²  < σ²  (biased low)
  With divisor n-1:   E[S²]        = σ²                   (unbiased ✓)
  
  Intuition: We "use up" 1 degree of freedom
  estimating μ by x̄, leaving n-1 for variance.
```

---

## 3. Random Sampling

### Simple Random Sample (SRS)

A sample of size $n$ where every possible sample of size $n$ has equal probability of being selected.

$$P(\text{any specific sample of size } n) = \frac{1}{\binom{N}{n}}$$

### Mathematical Model

In practice, we model each observation as an independent draw from the population:

$$X_1, X_2, \ldots, X_n \overset{\text{iid}}{\sim} F$$

where $F$ is the population distribution.

### Sampling Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| Simple Random | Every subset equally likely | Small, accessible population |
| Stratified | Divide into groups, sample within each | Heterogeneous population |
| Cluster | Randomly select groups, sample all within | Geographically spread |
| Systematic | Every $k$-th element | Ordered lists |

```
  Sampling Methods Illustrated
  
  Simple Random:   Stratified:     Cluster:       Systematic:
  ○ ○ ● ○ ○       ○ ● │ ○ ○       ┌──┐ ┌──┐      ○ ○ ● ○ ○
  ○ ○ ○ ● ○       ● ○ │ ● ○       │●●│ │○○│      ● ○ ○ ○ ●
  ● ○ ○ ○ ○       ────┼────       │●●│ │○○│      ○ ○ ● ○ ○
  ○ ● ○ ○ ●       ○ ○ │ ○ ●       └──┘ └──┘      ● ○ ○ ○ ●
  ○ ○ ○ ○ ○       ○ ● │ ○ ○       ┌──┐ ┌──┐
                                   │○○│ │●●│      Every 3rd
  Random picks    From each       Whole           item
                  stratum         clusters
```

---

## 4. Sampling With and Without Replacement

| Aspect | With Replacement | Without Replacement |
|--------|-----------------|---------------------|
| Same item twice? | Yes | No |
| Independence | Observations independent | Observations dependent |
| Practical use | Infinite populations | Finite populations |
| Math model | iid exactly | iid approximately (if $n \ll N$) |

**Rule of Thumb:** If $n/N < 0.05$ (sample < 5% of population), treat as sampling with replacement (iid).

---

## 5. Common Sample Statistics

### Sample Mean

$$\bar{X} = \frac{1}{n}\sum_{i=1}^n X_i$$

Properties (for iid sample from population with mean $\mu$, variance $\sigma^2$):

$$E[\bar{X}] = \mu, \quad \text{Var}(\bar{X}) = \frac{\sigma^2}{n}$$

### Sample Variance

$$S^2 = \frac{1}{n-1}\sum_{i=1}^n (X_i - \bar{X})^2 = \frac{1}{n-1}\left(\sum X_i^2 - n\bar{X}^2\right)$$

Properties: $E[S^2] = \sigma^2$ (unbiased)

### Sample Standard Deviation

$$S = \sqrt{S^2}$$

Note: $S$ is **not** unbiased for $\sigma$ (Jensen's inequality: $E[\sqrt{S^2}] \neq \sqrt{E[S^2]}$).

### Sample Proportion

$$\hat{p} = \frac{\text{number of successes}}{n}$$

Properties: $E[\hat{p}] = p$, $\text{Var}(\hat{p}) = \frac{p(1-p)}{n}$

---

## 6. Degrees of Freedom

$$\text{df} = n - k$$

where $k$ = number of parameters estimated from the data.

```
  Example: Sample variance
  
  n observations: X₁, X₂, ..., Xₙ
  
  Constraint: Σ(Xᵢ - X̄) = 0  (deviations sum to zero)
  
  So only n-1 deviations are "free" to vary
  
  ──► df = n - 1
  
  Think of it as: knowing n-1 deviations + X̄ determines the last one
```

---

## 7. Real-World Applications

| Application | Population | Sample | Parameter | Statistic |
|------------|-----------|--------|-----------|-----------|
| Election poll | All voters | 1000 surveyed | True vote share $p$ | $\hat{p}$ |
| Quality control | All products | 50 tested | Defect rate $p$ | $\hat{p}$ |
| Medical trial | All patients | 200 participants | Treatment effect $\mu_T - \mu_C$ | $\bar{X}_T - \bar{X}_C$ |
| SAT scores | All test-takers | 500 sampled | Mean score $\mu$ | $\bar{X}$ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Population | Entire group; parameters $\mu, \sigma^2, p$ |
| Sample | Subset; statistics $\bar{X}, S^2, \hat{p}$ |
| SRS | Every subset equally likely |
| iid model | $X_1, \ldots, X_n$ independent, same distribution |
| $E[\bar{X}] = \mu$ | Sample mean is unbiased for $\mu$ |
| $\text{Var}(\bar{X}) = \sigma^2/n$ | Precision improves with $n$ |
| Bessel's correction | Divide by $n-1$ for unbiased variance |
| Degrees of freedom | $n - (\text{parameters estimated})$ |

---

## Quick Revision Questions

1. Distinguish between a parameter and a statistic. Give two examples of each.

2. Why do we divide by $n-1$ rather than $n$ in the sample variance formula?

3. A population has $N = 10000$. You draw $n = 200$. Can you reasonably treat the sample as iid? Justify.

4. Calculate $\bar{X}$ and $S^2$ for the data: $3, 7, 5, 9, 6$.

5. Explain the difference between stratified and cluster sampling. When would you use each?

6. If $X_1, \ldots, X_n$ are iid with $E[X_i] = 10$ and $\text{Var}(X_i) = 25$, find $E[\bar{X}]$ and $\text{Var}(\bar{X})$ for $n = 100$.

---

[← Previous: Covariance and Correlation](../07-Joint-Distributions/05-covariance-and-correlation.md) | [Back to README](../README.md) | [Next: Sampling Distributions →](02-sampling-distributions.md)
