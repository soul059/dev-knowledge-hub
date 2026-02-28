# Chapter 9.3: Z-Test and t-Test

[← Previous: Type I and Type II Errors](02-type-I-and-type-II-errors.md) | [Back to README](../README.md) | [Next: Chi-Square Test →](04-chi-square-test.md)

---

## Overview

The **Z-test** and **t-test** are the workhorses of hypothesis testing for means. The Z-test is used when $\sigma$ is known; the t-test when $\sigma$ is unknown and must be estimated from the sample.

---

## 1. One-Sample Z-Test

### When to Use

- Testing a claim about a population mean $\mu$
- $\sigma$ is **known**
- Population is Normal OR $n \geq 30$ (CLT)

### Test Statistic

$$\boxed{Z = \frac{\bar{X} - \mu_0}{\sigma / \sqrt{n}}}$$

Under $H_0$: $Z \sim N(0,1)$.

### Decision Rules

| Alternative | Reject $H_0$ if | Critical Value(s) |
|-------------|-----------------|-------------------|
| $H_1: \mu > \mu_0$ | $Z > z_\alpha$ | $z_{0.05} = 1.645$ |
| $H_1: \mu < \mu_0$ | $Z < -z_\alpha$ | $-z_{0.05} = -1.645$ |
| $H_1: \mu \neq \mu_0$ | $|Z| > z_{\alpha/2}$ | $z_{0.025} = 1.960$ |

### Example

Claim: Average weight of packages is 500g. Sample: $n = 36$, $\bar{x} = 506$, $\sigma = 12$.

$$H_0: \mu = 500 \quad \text{vs} \quad H_1: \mu \neq 500 \quad (\alpha = 0.05)$$

$$Z = \frac{506 - 500}{12/\sqrt{36}} = \frac{6}{2} = 3.0$$

Since $|3.0| > 1.96$: **Reject $H_0$**. Evidence that mean weight differs from 500g.

---

## 2. One-Sample t-Test

### When to Use

- Testing a claim about $\mu$
- $\sigma$ is **unknown** (estimated by $S$)
- Population is Normal OR $n \geq 30$

### Test Statistic

$$\boxed{T = \frac{\bar{X} - \mu_0}{S / \sqrt{n}}}$$

Under $H_0$: $T \sim t(n-1)$.

### Example

Claim: Mean reaction time is 200ms. Sample: $n = 16$, $\bar{x} = 215$, $s = 30$.

$$H_0: \mu = 200 \quad \text{vs} \quad H_1: \mu > 200 \quad (\alpha = 0.05)$$

$$T = \frac{215 - 200}{30/\sqrt{16}} = \frac{15}{7.5} = 2.0$$

$t_{0.05, 15} = 1.753$. Since $2.0 > 1.753$: **Reject $H_0$**.

```
  t-distribution with df = 15
  
           ╱╲
          ╱  ╲
         ╱    ╲
        ╱      ╲
       ╱  Fail  ╲
      ╱    to    ╲
     ╱   reject   ╲█████
  ──╱──────────────╲█████──
    0            1.753  2.0
                  ↑      ↑
               t_crit  t_calc
                    
                → Reject H₀
```

---

## 3. Two-Sample Z-Test (Independent Samples)

### Test Statistic ($\sigma_1, \sigma_2$ Known)

$$\boxed{Z = \frac{(\bar{X}_1 - \bar{X}_2) - (\mu_1 - \mu_2)_0}{\sqrt{\frac{\sigma_1^2}{n_1} + \frac{\sigma_2^2}{n_2}}}}$$

Under $H_0: \mu_1 - \mu_2 = 0$, this simplifies to:

$$Z = \frac{\bar{X}_1 - \bar{X}_2}{\sqrt{\frac{\sigma_1^2}{n_1} + \frac{\sigma_2^2}{n_2}}}$$

---

## 4. Two-Sample t-Test (Independent Samples)

### Case 1: Equal Variances (Pooled t-Test)

$$\boxed{T = \frac{\bar{X}_1 - \bar{X}_2}{S_p\sqrt{\frac{1}{n_1} + \frac{1}{n_2}}}, \quad \text{df} = n_1 + n_2 - 2}$$

where $S_p^2 = \frac{(n_1 - 1)S_1^2 + (n_2 - 1)S_2^2}{n_1 + n_2 - 2}$.

### Case 2: Unequal Variances (Welch's t-Test)

$$\boxed{T = \frac{\bar{X}_1 - \bar{X}_2}{\sqrt{\frac{S_1^2}{n_1} + \frac{S_2^2}{n_2}}}}$$

$$\text{df} = \frac{\left(\frac{S_1^2}{n_1} + \frac{S_2^2}{n_2}\right)^2}{\frac{(S_1^2/n_1)^2}{n_1-1} + \frac{(S_2^2/n_2)^2}{n_2-1}} \quad \text{(Welch-Satterthwaite)}$$

### Example

| | Group 1 | Group 2 |
|:-:|:-:|:-:|
| $n$ | 15 | 12 |
| $\bar{x}$ | 78 | 72 |
| $s$ | 10 | 8 |

$H_0: \mu_1 = \mu_2$ vs $H_1: \mu_1 \neq \mu_2$, $\alpha = 0.05$. Assume equal variances.

$$S_p^2 = \frac{14(100) + 11(64)}{25} = \frac{1400 + 704}{25} = 84.16$$

$$T = \frac{78 - 72}{\sqrt{84.16(1/15 + 1/12)}} = \frac{6}{\sqrt{84.16 \times 0.1500}} = \frac{6}{\sqrt{12.624}} = \frac{6}{3.553} = 1.69$$

$t_{0.025, 25} = 2.060$. Since $1.69 < 2.060$: **Fail to reject $H_0$**.

---

## 5. Paired t-Test

For **matched pairs** (before/after, twins, same subject twice):

$$\boxed{T = \frac{\bar{D} - 0}{S_D / \sqrt{n}}, \quad \text{df} = n - 1}$$

where $D_i = X_{i,1} - X_{i,2}$ and $\bar{D}, S_D$ are the mean and SD of differences.

### Example

| Subject | Before | After | $D_i$ |
|:-:|:-:|:-:|:-:|
| 1 | 180 | 170 | 10 |
| 2 | 200 | 185 | 15 |
| 3 | 160 | 155 | 5 |
| 4 | 190 | 175 | 15 |
| 5 | 175 | 165 | 10 |

$\bar{D} = 11$, $S_D = 4.18$, $n = 5$

$$T = \frac{11 - 0}{4.18/\sqrt{5}} = \frac{11}{1.87} = 5.88$$

$t_{0.025, 4} = 2.776$. Since $5.88 > 2.776$: **Reject $H_0$**. Significant decrease.

---

## 6. Choosing the Right Test

```
  Comparing means?
       │
  ┌────┴────┐
  │One      │Two
  │sample   │samples
  │         │
  σ known?  │
  ├──┬──┤   Are data paired?
  Yes No    ├──┬──┤
  │   │     Yes No
  Z   t     │   │
  test test  Paired  Are variances
             t-test  equal?
                     ├──┬──┤
                     Yes No
                     │   │
                   Pooled Welch's
                   t-test t-test
```

---

## 7. Assumptions and Robustness

| Test | Assumptions |
|------|-------------|
| Z-test | $\sigma$ known, Normal or large $n$ |
| One-sample t | Normal population or large $n$; $\sigma$ unknown |
| Two-sample t | Both Normal or large $n$; independent samples |
| Pooled t | Equal variances ($\sigma_1 = \sigma_2$) |
| Welch's t | Unequal variances OK |
| Paired t | Differences are Normal; observations are paired |

**Robustness:** t-tests are fairly robust to non-normality for $n \geq 30$, but sensitive to outliers and heavy tails for small $n$.

---

## Summary Table

| Test | Statistic | Distribution | df |
|------|-----------|-------------|-----|
| One-sample Z | $(\bar{X}-\mu_0)/(\sigma/\sqrt{n})$ | $N(0,1)$ | — |
| One-sample t | $(\bar{X}-\mu_0)/(S/\sqrt{n})$ | $t$ | $n-1$ |
| Pooled two-sample t | $(\bar{X}_1-\bar{X}_2)/(S_p\sqrt{1/n_1+1/n_2})$ | $t$ | $n_1+n_2-2$ |
| Welch's t | $(\bar{X}_1-\bar{X}_2)/\sqrt{S_1^2/n_1+S_2^2/n_2}$ | $t$ | Welch df |
| Paired t | $\bar{D}/(S_D/\sqrt{n})$ | $t$ | $n-1$ |

---

## Quick Revision Questions

1. $\bar{x} = 105$, $\sigma = 15$, $n = 49$. Test $H_0: \mu = 100$ vs $H_1: \mu > 100$ at $\alpha = 0.05$.

2. When would you use a paired t-test instead of a two-sample t-test?

3. Two groups: $n_1 = 20$, $\bar{x}_1 = 50$, $s_1 = 8$; $n_2 = 25$, $\bar{x}_2 = 45$, $s_2 = 10$. Perform a pooled two-sample t-test at $\alpha = 0.05$.

4. Explain why the t-distribution is used instead of Z when $\sigma$ is unknown.

5. What is Welch's t-test and when is it preferred over the pooled t-test?

6. A sample of 8 differences has $\bar{d} = 3.5$, $s_d = 2.1$. Test if the true mean difference is positive at $\alpha = 0.05$.

---

[← Previous: Type I and Type II Errors](02-type-I-and-type-II-errors.md) | [Back to README](../README.md) | [Next: Chi-Square Test →](04-chi-square-test.md)
