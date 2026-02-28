# Chapter 6.7: Student's t-Distribution

[← Previous: Chi-Square Distribution](06-chi-square-distribution.md) | [Back to README](../README.md) | [Next: F-Distribution →](08-f-distribution.md)

---

## Overview

The **t-distribution** arises when estimating the mean of a Normal population using the sample standard deviation instead of the known population $\sigma$. It has heavier tails than the Normal, reflecting the extra uncertainty from estimating $\sigma$.

---

## 1. Definition

If $Z \sim N(0,1)$ and $V \sim \chi^2(k)$ are independent, then:

$$\boxed{T = \frac{Z}{\sqrt{V/k}} \sim t(k)}$$

$k$ = **degrees of freedom**

### PDF

$$f(t) = \frac{\Gamma\left(\frac{k+1}{2}\right)}{\sqrt{k\pi}\;\Gamma\left(\frac{k}{2}\right)} \left(1 + \frac{t^2}{k}\right)^{-(k+1)/2}, \quad -\infty < t < \infty$$

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $0$ (for $k > 1$) |
| Variance | $\frac{k}{k-2}$ (for $k > 2$) |
| Symmetry | Symmetric about 0 |
| Tails | Heavier than Normal |
| As $k \to \infty$ | $t(k) \to N(0,1)$ |
| Mode | 0 |

---

## 3. t vs. Normal Comparison

```
  Comparison of t(k) and N(0,1)
  
  f(t)
  │        N(0,1) — taller peak
  │        ╱╲
  │      ╱    ╲
  │    ╱╱──────╲╲ ← t(5) — lower peak, heavier tails
  │  ╱╱──────────╲╲ ← t(2)
  │╱╱──────────────╲╲
  └────────────────────────► t
  -4  -2    0    2    4
  
  Key differences:
  • t has HEAVIER TAILS (more probability in extremes)
  • t has LOWER PEAK
  • t → Normal as df → ∞
```

| df ($k$) | $\text{Var}(T)$ | $t_{0.025}$ | Compared to $z_{0.025}=1.96$ |
|:--------:|:-:|:-:|:--|
| 1 | $\infty$ | 12.706 | Much wider |
| 5 | 1.667 | 2.571 | Wider |
| 10 | 1.250 | 2.228 | Wider |
| 30 | 1.071 | 2.042 | Close |
| 100 | 1.020 | 1.984 | Very close |
| $\infty$ | 1.000 | 1.960 | = Normal |

---

## 4. Main Application: One-Sample t-Test

Given $X_1, \ldots, X_n \sim N(\mu, \sigma^2)$ i.i.d., with $\sigma$ **unknown**:

$$\boxed{T = \frac{\bar{X} - \mu}{S/\sqrt{n}} \sim t(n-1)}$$

where $S = \sqrt{\frac{1}{n-1}\sum(X_i - \bar{X})^2}$

### Confidence Interval for $\mu$

$$\boxed{\bar{X} \pm t_{\alpha/2, n-1} \cdot \frac{S}{\sqrt{n}}}$$

### When to Use t vs. z

```
  ┌─────────────────────────┐
  │  Is σ known?            │
  │                         │
  │  YES ──► Use z-test     │
  │          (Normal)       │
  │                         │
  │  NO ───► Use t-test     │
  │          (t-distribution)│
  │                         │
  │  n ≥ 30? t ≈ z anyway   │
  └─────────────────────────┘
```

---

## 5. Critical t-Values (Selected)

| df | $t_{0.10}$ | $t_{0.05}$ | $t_{0.025}$ | $t_{0.005}$ |
|:--:|:--:|:--:|:--:|:--:|
| 1 | 3.078 | 6.314 | 12.706 | 63.657 |
| 5 | 1.476 | 2.015 | 2.571 | 4.032 |
| 10 | 1.372 | 1.812 | 2.228 | 3.169 |
| 20 | 1.325 | 1.725 | 2.086 | 2.845 |
| 30 | 1.310 | 1.697 | 2.042 | 2.750 |
| $\infty$ | 1.282 | 1.645 | 1.960 | 2.576 |

---

## 6. Worked Examples

### Example 1: Confidence Interval

A sample of $n = 16$ light bulbs has $\bar{x} = 1200$ hours, $s = 100$ hours. Find a 95% CI for $\mu$.

$df = 15$, $t_{0.025, 15} = 2.131$

$$1200 \pm 2.131 \times \frac{100}{\sqrt{16}} = 1200 \pm 53.3 = (1146.7, 1253.3)$$

### Example 2: Hypothesis Test

Claim: $\mu = 500$. Sample: $n = 25$, $\bar{x} = 510$, $s = 20$.

$$T = \frac{510 - 500}{20/\sqrt{25}} = \frac{10}{4} = 2.5$$

$df = 24$, $t_{0.025, 24} = 2.064$

Since $|T| = 2.5 > 2.064$, reject $H_0$ at $\alpha = 0.05$.

---

## 7. Special Cases

| Distribution | Connection |
|:--:|:--|
| $t(1)$ | Cauchy distribution (no mean, no variance) |
| $t(\infty)$ | Standard Normal $N(0,1)$ |
| $t(k)^2$ | $F(1, k)$ distribution |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Definition | $T = Z/\sqrt{V/k}$ |
| Mean | 0 |
| Variance | $k/(k-2)$ for $k > 2$ |
| t-statistic | $(\bar{X}-\mu)/(S/\sqrt{n}) \sim t(n-1)$ |
| CI for $\mu$ | $\bar{X} \pm t_{\alpha/2} \cdot S/\sqrt{n}$ |
| Convergence | $t(k) \to N(0,1)$ as $k \to \infty$ |

---

## Quick Revision Questions

1. Why do we use the t-distribution instead of the Normal when $\sigma$ is unknown?

2. A sample of $n = 10$ has $\bar{x} = 45$, $s = 6$. Find a 99% CI for $\mu$.

3. As degrees of freedom increase, the t-distribution approaches the Normal. At what df is the difference practically negligible?

4. Show that $\text{Var}(T) = k/(k-2)$ is always $> 1$ for $k > 2$, confirming heavier tails than Normal.

5. Test $H_0: \mu = 100$ vs. $H_1: \mu \neq 100$ with $n = 20$, $\bar{x} = 104$, $s = 8$ at $\alpha = 0.05$.

6. Why does $t(1)$ have no mean? What famous distribution is it?

---

[← Previous: Chi-Square Distribution](06-chi-square-distribution.md) | [Back to README](../README.md) | [Next: F-Distribution →](08-f-distribution.md)
