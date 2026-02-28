# Chapter 8.5: Confidence Intervals

[← Previous: Point Estimation](04-point-estimation.md) | [Back to README](../README.md) | [Next: Null and Alternative Hypothesis →](../09-Hypothesis-Testing/01-null-and-alternative-hypothesis.md)

---

## Overview

A point estimate gives a single number, but says nothing about its precision. A **confidence interval** provides a *range* of plausible values for the parameter, along with a measure of reliability.

---

## 1. Definition

A **$(1-\alpha) \times 100\%$ confidence interval** for parameter $\theta$ is a random interval $(\hat{\theta}_L, \hat{\theta}_U)$ such that:

$$\boxed{P(\hat{\theta}_L \leq \theta \leq \hat{\theta}_U) = 1 - \alpha}$$

| Symbol | Meaning | Common Values |
|--------|---------|---------------|
| $1 - \alpha$ | Confidence level | 0.90, 0.95, 0.99 |
| $\alpha$ | Significance level | 0.10, 0.05, 0.01 |
| $\alpha/2$ | Tail probability | 0.05, 0.025, 0.005 |

```
  Interpretation of a 95% CI
  
  ──────────────────────────────── θ (true parameter)
           │
  ├────────┼────────┤  CI from sample 1 — CONTAINS θ ✓
     ├─────┼──────┤    CI from sample 2 — CONTAINS θ ✓
            ├──────────┤  CI from sample 3 — MISSES θ ✗
  ├─────────┼───────┤  CI from sample 4 — CONTAINS θ ✓
       ├────┼─────┤    CI from sample 5 — CONTAINS θ ✓
  
  95% of all such intervals will contain θ.
  Any SINGLE interval either does or doesn't — we just don't know which.
```

---

## 2. CI for Mean — $\sigma$ Known (Z-Interval)

$$\boxed{\bar{X} \pm z_{\alpha/2} \cdot \frac{\sigma}{\sqrt{n}}}$$

| Confidence Level | $z_{\alpha/2}$ |
|-----------------|----------------|
| 90% | 1.645 |
| 95% | 1.960 |
| 99% | 2.576 |

### Example

$\bar{x} = 82$, $\sigma = 10$, $n = 25$, 95% CI:

$$82 \pm 1.96 \cdot \frac{10}{\sqrt{25}} = 82 \pm 3.92 = (78.08, 85.92)$$

### Margin of Error

$$\boxed{E = z_{\alpha/2} \cdot \frac{\sigma}{\sqrt{n}}}$$

### Required Sample Size

To achieve margin of error $E$:

$$\boxed{n = \left\lceil\left(\frac{z_{\alpha/2} \cdot \sigma}{E}\right)^2\right\rceil}$$

---

## 3. CI for Mean — $\sigma$ Unknown (t-Interval)

$$\boxed{\bar{X} \pm t_{\alpha/2, n-1} \cdot \frac{S}{\sqrt{n}}}$$

where $t_{\alpha/2, n-1}$ is the critical value from the $t$-distribution with $n-1$ degrees of freedom.

### Example

$\bar{x} = 45.2$, $s = 6.8$, $n = 20$, 95% CI:

$t_{0.025, 19} = 2.093$

$$45.2 \pm 2.093 \cdot \frac{6.8}{\sqrt{20}} = 45.2 \pm 3.18 = (42.02, 48.38)$$

### When to Use Z vs t

```
  Is σ known?
       │
  ┌────┴────┐
  │Yes      │No
  │         │
  Z-interval  Is population Normal?
  X̄ ± z·σ/√n     │
              ┌───┴───┐
              │Yes    │No
              │       │
           t-interval  Is n ≥ 30?
           X̄ ± t·S/√n    │
                     ┌────┴────┐
                     │Yes      │No
                     │         │
                  t-interval  Non-parametric
                  (CLT applies) methods
```

---

## 4. CI for Proportion

For large $n$ ($n\hat{p} \geq 5$ and $n(1-\hat{p}) \geq 5$):

$$\boxed{\hat{p} \pm z_{\alpha/2}\sqrt{\frac{\hat{p}(1-\hat{p})}{n}}}$$

### Example

Survey: 420 out of 1000 support a policy. 95% CI for true proportion:

$$\hat{p} = 0.42, \quad 0.42 \pm 1.96\sqrt{\frac{0.42 \times 0.58}{1000}} = 0.42 \pm 0.031$$

$$= (0.389, 0.451)$$

### Sample Size for Proportion

$$n = \left\lceil\frac{z_{\alpha/2}^2 \cdot p(1-p)}{E^2}\right\rceil$$

Conservative (worst case, $p = 0.5$):

$$n = \left\lceil\frac{z_{\alpha/2}^2}{4E^2}\right\rceil$$

---

## 5. CI for Variance (Normal Population)

$$\boxed{\left(\frac{(n-1)S^2}{\chi^2_{\alpha/2, n-1}}, \quad \frac{(n-1)S^2}{\chi^2_{1-\alpha/2, n-1}}\right)}$$

Note: This interval is **not symmetric** around $S^2$ because $\chi^2$ is skewed.

### Example

$n = 25$, $s^2 = 18$, 95% CI for $\sigma^2$:

$\chi^2_{0.025, 24} = 39.36$, $\chi^2_{0.975, 24} = 12.40$

$$\left(\frac{24 \times 18}{39.36}, \frac{24 \times 18}{12.40}\right) = (10.98, 34.84)$$

---

## 6. CI for Difference of Means

### Two Independent Samples, $\sigma_1, \sigma_2$ Known:

$$(\bar{X}_1 - \bar{X}_2) \pm z_{\alpha/2}\sqrt{\frac{\sigma_1^2}{n_1} + \frac{\sigma_2^2}{n_2}}$$

### Two Independent Samples, Equal Unknown Variances:

$$(\bar{X}_1 - \bar{X}_2) \pm t_{\alpha/2, n_1+n_2-2} \cdot S_p\sqrt{\frac{1}{n_1} + \frac{1}{n_2}}$$

where $S_p^2 = \frac{(n_1-1)S_1^2 + (n_2-1)S_2^2}{n_1 + n_2 - 2}$ is the **pooled variance**.

---

## 7. Common Misinterpretations

| Wrong Statement | Why It's Wrong |
|----------------|----------------|
| "95% probability that $\mu$ is in (a, b)" | $\mu$ is fixed; the interval is random |
| "95% of data falls in the CI" | CI is about $\mu$, not individual values |
| "If we repeat, 95% of intervals contain the same $\bar{x}$" | Different samples give different $\bar{x}$ |

**Correct:** "If we repeat the sampling procedure many times, 95% of the constructed intervals will contain the true $\mu$."

---

## 8. Factors Affecting CI Width

| Factor | Effect on Width |
|--------|----------------|
| $\uparrow$ Confidence level $(1-\alpha)$ | Wider (more certain → wider net) |
| $\uparrow$ Sample size $n$ | Narrower ($\propto 1/\sqrt{n}$) |
| $\uparrow$ Variability $\sigma$ | Wider |

```
  90% CI:  ├─────────────────┤
  95% CI:  ├───────────────────────┤
  99% CI:  ├─────────────────────────────────┤
  
  Higher confidence → Wider interval
  
  n = 25:  ├───────────────────────┤
  n = 100: ├──────────────┤
  n = 400: ├────────┤
  
  Larger sample → Narrower interval
```

---

## Summary Table

| Parameter | CI Formula | Distribution Used |
|-----------|-----------|-------------------|
| $\mu$ ($\sigma$ known) | $\bar{X} \pm z_{\alpha/2}\sigma/\sqrt{n}$ | $Z$ |
| $\mu$ ($\sigma$ unknown) | $\bar{X} \pm t_{\alpha/2,n-1}S/\sqrt{n}$ | $t(n-1)$ |
| $p$ | $\hat{p} \pm z_{\alpha/2}\sqrt{\hat{p}(1-\hat{p})/n}$ | $Z$ (large $n$) |
| $\sigma^2$ | $((n-1)S^2/\chi^2_U, (n-1)S^2/\chi^2_L)$ | $\chi^2(n-1)$ |
| $\mu_1 - \mu_2$ | $(\bar{X}_1-\bar{X}_2) \pm t \cdot \text{SE}$ | $t$ or $Z$ |

---

## Quick Revision Questions

1. Construct a 99% CI for $\mu$ given: $\bar{x} = 50$, $\sigma = 8$, $n = 64$.

2. How large a sample is needed to estimate $\mu$ within $\pm 2$ with 95% confidence if $\sigma = 12$?

3. In a sample of 500, 230 prefer brand A. Find a 95% CI for the true proportion.

4. Explain why a 99% CI is wider than a 95% CI for the same data.

5. $n = 16$, $\bar{x} = 30$, $s = 5$. Find the 95% $t$-interval for $\mu$. What assumption is needed?

6. Correct this misinterpretation: "There is a 95% chance that the population mean is between 42.02 and 48.38."

---

[← Previous: Point Estimation](04-point-estimation.md) | [Back to README](../README.md) | [Next: Null and Alternative Hypothesis →](../09-Hypothesis-Testing/01-null-and-alternative-hypothesis.md)
