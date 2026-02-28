# Chapter 6.8: F-Distribution

[← Previous: t-Distribution](07-t-distribution.md) | [Back to README](../README.md) | [Next: Joint PMF and PDF →](../07-Joint-Distributions/01-joint-pmf-and-pdf.md)

---

## Overview

The **F-distribution** arises as the ratio of two chi-square variables and is fundamental in ANOVA (comparing group means), testing equality of variances, and regression analysis.

---

## 1. Definition

If $U \sim \chi^2(d_1)$ and $V \sim \chi^2(d_2)$ are independent:

$$\boxed{F = \frac{U/d_1}{V/d_2} \sim F(d_1, d_2)}$$

$d_1$ = numerator df, $d_2$ = denominator df

### PDF

$$f(x) = \frac{\sqrt{\frac{(d_1 x)^{d_1} d_2^{d_2}}{(d_1 x + d_2)^{d_1+d_2}}}}{x \cdot B(d_1/2, d_2/2)}, \quad x > 0$$

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $\frac{d_2}{d_2 - 2}$ for $d_2 > 2$ |
| Variance | $\frac{2d_2^2(d_1+d_2-2)}{d_1(d_2-2)^2(d_2-4)}$ for $d_2 > 4$ |
| Mode | $\frac{d_1-2}{d_1} \cdot \frac{d_2}{d_2+2}$ for $d_1 > 2$ |
| Support | $(0, \infty)$ |
| Skewness | Right-skewed (always) |

```
  F-distribution shapes:
  
  f(x)
  │╲  F(1, 5)
  │  ╲
  │    ╲     ╱╲  F(5, 10)
  │      ╲─╱    ╲
  │     ╱   ╲  ╱╲╲── F(10, 30)
  │   ╱       ╳    ╲
  │ ╱       ╱   ╲    ╲
  └╱──────╱───────╲────╲──► x
  0    1    2    3    4    5
  
  Larger df → more concentrated near 1
  Always right-skewed, support on (0, ∞)
```

---

## 3. Key Relationships

| Relationship | Formula |
|:--:|:--|
| Reciprocal | $1/F(d_1, d_2) \sim F(d_2, d_1)$ |
| t-squared | $t(k)^2 \sim F(1, k)$ |
| Chi-square limit | $d_1 \cdot F(d_1, \infty) \sim \chi^2(d_1)$ |

### Reciprocal Property

$$\boxed{F_{1-\alpha}(d_1, d_2) = \frac{1}{F_\alpha(d_2, d_1)}}$$

This lets you find lower critical values from upper critical values.

---

## 4. Applications

### Testing Equality of Two Variances

$H_0: \sigma_1^2 = \sigma_2^2$ vs $H_1: \sigma_1^2 \neq \sigma_2^2$

$$F = \frac{S_1^2}{S_2^2} \sim F(n_1-1, n_2-1) \quad \text{under } H_0$$

Reject if $F > F_{\alpha/2}(n_1-1, n_2-1)$ or $F < F_{1-\alpha/2}(n_1-1, n_2-1)$.

### ANOVA (Analysis of Variance)

$$F = \frac{\text{MSB}}{\text{MSW}} = \frac{\text{Between-group variance}}{\text{Within-group variance}} \sim F(k-1, N-k)$$

### Regression Significance

$$F = \frac{\text{MSR}}{\text{MSE}} = \frac{R^2/p}{(1-R^2)/(n-p-1)} \sim F(p, n-p-1)$$

---

## 5. Critical F-Values (Selected, $\alpha = 0.05$)

| $d_1 \backslash d_2$ | 5 | 10 | 20 | 30 | $\infty$ |
|:--:|:--:|:--:|:--:|:--:|:--:|
| 1 | 6.61 | 4.96 | 4.35 | 4.17 | 3.84 |
| 2 | 5.79 | 4.10 | 3.49 | 3.32 | 3.00 |
| 5 | 5.05 | 3.33 | 2.71 | 2.53 | 2.21 |
| 10 | 4.74 | 2.98 | 2.35 | 2.16 | 1.83 |
| 20 | 4.56 | 2.77 | 2.12 | 1.93 | 1.57 |

---

## 6. Worked Example

### Comparing Two Variances

Two manufacturers' bolt strengths:
- Manufacturer A: $n_1 = 16$, $s_1^2 = 25$
- Manufacturer B: $n_2 = 21$, $s_2^2 = 12$

$$F = \frac{25}{12} = 2.083, \quad df = (15, 20)$$

$F_{0.025}(15, 20) \approx 2.57$

Since $2.083 < 2.57$, we fail to reject $H_0: \sigma_1^2 = \sigma_2^2$ at $\alpha = 0.05$.

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Definition | $(U/d_1)/(V/d_2)$, $U, V$ indep. $\chi^2$ |
| Mean | $d_2/(d_2-2)$ |
| Reciprocal | $1/F(d_1,d_2) \sim F(d_2,d_1)$ |
| t-squared | $t(k)^2 = F(1,k)$ |
| Variance test | $F = S_1^2/S_2^2$ |
| ANOVA F-stat | MSB/MSW |

---

## Quick Revision Questions

1. If $U \sim \chi^2(6)$ and $V \sim \chi^2(10)$ independently, what is the distribution of $(U/6)/(V/10)$?

2. Given $F \sim F(5, 20)$, find $E[F]$.

3. Two samples: $n_1 = 10$, $s_1^2 = 30$ and $n_2 = 15$, $s_2^2 = 15$. Test $H_0: \sigma_1^2 = \sigma_2^2$ at $\alpha = 0.05$.

4. Why does the F-distribution only take positive values?

5. Explain the relationship $t(k)^2 = F(1, k)$ and its implication for two-sided t-tests.

6. In a one-way ANOVA with 4 groups and 40 total observations, what are the degrees of freedom for the F-statistic?

---

[← Previous: t-Distribution](07-t-distribution.md) | [Back to README](../README.md) | [Next: Joint PMF and PDF →](../07-Joint-Distributions/01-joint-pmf-and-pdf.md)
