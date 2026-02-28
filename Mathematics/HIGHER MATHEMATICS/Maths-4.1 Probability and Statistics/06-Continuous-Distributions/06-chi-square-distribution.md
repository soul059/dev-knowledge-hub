# Chapter 6.6: Chi-Square Distribution

[← Previous: Gamma and Beta Distributions](05-gamma-and-beta-distributions.md) | [Back to README](../README.md) | [Next: t-Distribution →](07-t-distribution.md)

---

## Overview

The **Chi-Square ($\chi^2$) distribution** arises from summing squares of standard Normal variables. It is fundamental to hypothesis testing (goodness-of-fit, independence) and confidence intervals for variance.

---

## 1. Definition

If $Z_1, Z_2, \ldots, Z_k$ are i.i.d. $N(0,1)$, then:

$$\boxed{X = Z_1^2 + Z_2^2 + \cdots + Z_k^2 \sim \chi^2(k)}$$

$k$ = **degrees of freedom** (df)

### PDF

$$f(x) = \frac{1}{2^{k/2}\Gamma(k/2)} x^{k/2-1} e^{-x/2}, \quad x > 0$$

This is Gamma$(k/2, 1/2)$.

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $E[X] = k$ |
| Variance | $\text{Var}(X) = 2k$ |
| Mode | $\max(k-2, 0)$ |
| MGF | $(1-2t)^{-k/2}$, $t < 1/2$ |
| Skewness | $\sqrt{8/k}$ |
| Kurtosis | $3 + 12/k$ |

> Mean = df, Variance = 2 × df. As $k$ increases, $\chi^2$ becomes more symmetric.

---

## 3. Shape by Degrees of Freedom

```
  Chi-Square PDF for different k:
  
  f(x)
  │╲  k=1
  │  ╲
  │    ╲
  │      ╲     ╱╲  k=3
  │        ╲─╱    ╲
  │       ╱    ╱╲   ╲── k=5
  │     ╱    ╱    ╲    ╲
  │   ╱    ╱   ╱╲  ╲     ╲── k=10
  │ ╱    ╱   ╱    ╲  ╲      ╲
  └╱───╱───╱────────╲──╲──────╲──► x
  0  2  4  6  8  10 12 14 16
  
  k=1: Very right-skewed, mode at 0
  k=2: Exponential (special case)
  k≥3: Mode shifts right, becomes more symmetric
  Large k: Approaches Normal(k, 2k)
```

---

## 4. Additivity

If $X_1 \sim \chi^2(k_1)$ and $X_2 \sim \chi^2(k_2)$ are independent:

$$\boxed{X_1 + X_2 \sim \chi^2(k_1 + k_2)}$$

---

## 5. Applications in Statistics

### Sample Variance Distribution

If $X_1, \ldots, X_n \sim N(\mu, \sigma^2)$ i.i.d., the sample variance is $S^2 = \frac{1}{n-1}\sum(X_i - \bar{X})^2$

$$\boxed{\frac{(n-1)S^2}{\sigma^2} \sim \chi^2(n-1)}$$

### Confidence Interval for Variance

$$\left(\frac{(n-1)S^2}{\chi^2_{\alpha/2}}, \quad \frac{(n-1)S^2}{\chi^2_{1-\alpha/2}}\right)$$

### Chi-Square Goodness-of-Fit Test

$$\chi^2 = \sum_{i=1}^{k} \frac{(O_i - E_i)^2}{E_i} \sim \chi^2(k-1)$$

where $O_i$ = observed, $E_i$ = expected frequencies.

### Chi-Square Test of Independence

For an $r \times c$ contingency table:

$$\chi^2 = \sum \frac{(O_{ij} - E_{ij})^2}{E_{ij}} \sim \chi^2((r-1)(c-1))$$

---

## 6. Critical Values (Selected)

| df | $\chi^2_{0.95}$ | $\chi^2_{0.05}$ | $\chi^2_{0.025}$ | $\chi^2_{0.975}$ |
|:--:|:--:|:--:|:--:|:--:|
| 1 | 0.004 | 3.841 | 5.024 | 0.001 |
| 5 | 1.145 | 11.070 | 12.833 | 0.831 |
| 10 | 3.940 | 18.307 | 20.483 | 3.247 |
| 20 | 10.851 | 31.410 | 34.170 | 9.591 |
| 30 | 18.493 | 43.773 | 46.979 | 16.791 |

---

## 7. Relationship to Other Distributions

| From | To | Relationship |
|------|-----|-------------|
| $N(0,1)$ | $\chi^2(1)$ | $Z^2 \sim \chi^2(1)$ |
| $\text{Gamma}(k/2, 1/2)$ | $\chi^2(k)$ | Same distribution |
| $\text{Exp}(1/2)$ | $\chi^2(2)$ | Special case |
| $\chi^2(k)/k$ → | $N(0,1)$ | CLT as $k \to \infty$ |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Definition | Sum of $k$ squared $N(0,1)$ |
| Mean | $k$ |
| Variance | $2k$ |
| As Gamma | Gamma$(k/2, 1/2)$ |
| Additivity | $\chi^2(k_1) + \chi^2(k_2) = \chi^2(k_1+k_2)$ |
| Sample variance | $(n-1)S^2/\sigma^2 \sim \chi^2(n-1)$ |
| GOF test | $\sum(O_i-E_i)^2/E_i$ |

---

## Quick Revision Questions

1. If $Z_1, Z_2, Z_3, Z_4$ are i.i.d. $N(0,1)$, what is the distribution of $Z_1^2 + Z_2^2 + Z_3^2 + Z_4^2$? Find its mean and variance.

2. Show that $\chi^2(2) = \text{Exp}(1/2)$ by comparing their PDFs.

3. A die is rolled 120 times with results: 1→18, 2→22, 3→17, 4→25, 5→19, 6→19. Perform a chi-square goodness-of-fit test for fairness at $\alpha = 0.05$.

4. From a $N(\mu, \sigma^2)$ population, $n = 25$, $S^2 = 36$. Find a 95% CI for $\sigma^2$.

5. Why does a chi-square distribution with large df approximate a Normal?

6. For a 3×4 contingency table, how many degrees of freedom does the chi-square test of independence have?

---

[← Previous: Gamma and Beta Distributions](05-gamma-and-beta-distributions.md) | [Back to README](../README.md) | [Next: t-Distribution →](07-t-distribution.md)
