# Chapter 10.3: Correlation Coefficient

[← Previous: Least Squares Method](02-least-squares-method.md) | [Back to README](../README.md) | [Next: Multiple Regression →](04-multiple-regression.md)

---

## Overview

The **correlation coefficient** measures the strength and direction of the linear relationship between two variables. Unlike regression, which predicts one variable from another, correlation treats both variables symmetrically.

---

## 1. Pearson Correlation Coefficient ($r$)

### Population Correlation

$$\boxed{\rho = \frac{\text{Cov}(X,Y)}{\sigma_X \sigma_Y} = \frac{E[(X-\mu_X)(Y-\mu_Y)]}{\sigma_X \sigma_Y}}$$

### Sample Correlation

$$\boxed{r = \frac{S_{XY}}{\sqrt{S_{XX} \cdot S_{YY}}} = \frac{\sum(X_i - \bar{X})(Y_i - \bar{Y})}{\sqrt{\sum(X_i - \bar{X})^2 \cdot \sum(Y_i - \bar{Y})^2}}}$$

### Computational Formula

$$r = \frac{n\sum X_iY_i - \sum X_i \sum Y_i}{\sqrt{[n\sum X_i^2 - (\sum X_i)^2][n\sum Y_i^2 - (\sum Y_i)^2]}}$$

---

## 2. Properties of $r$

| Property | Description |
|----------|-------------|
| **Range** | $-1 \le r \le 1$ |
| **Sign** | Matches direction of relationship |
| **Unit-free** | Invariant under linear transformation $aX + b$ (if $a > 0$) |
| **Symmetric** | $r(X,Y) = r(Y,X)$ |
| **$r = 0$** | No LINEAR relationship (nonlinear may exist!) |

```
  r = −1        r ≈ −0.7      r ≈ 0         r ≈ 0.7       r = 1
  
  ·              ··             · ·  ·        ··             ·
   ·            · ·           ·  ··  ·         ··           ·
    ·          ·  ··         ··· · ··          · ··         ·
     ·        ··  ·          · ·· · ·           ··        ·
      ·       · ··          ·  · · ·            · ·      ·
       ·        ··          · ·· ·               ··     ·
  
  Perfect     Strong        No linear      Strong       Perfect
  negative    negative      correlation    positive     positive
```

---

## 3. Relationship to Regression

$$\boxed{r = \hat{\beta}_1 \cdot \frac{S_X}{S_Y}}$$

where $S_X$ and $S_Y$ are the sample standard deviations.

$$\boxed{R^2 = r^2}$$

| Connection | Formula |
|-----------|---------|
| Slope → correlation | $r = \hat{\beta}_1 \cdot S_X / S_Y$ |
| Correlation → slope | $\hat{\beta}_1 = r \cdot S_Y / S_X$ |
| $R^2$ in simple regression | $R^2 = r^2$ |
| Sign agreement | $\text{sign}(r) = \text{sign}(\hat{\beta}_1)$ |

---

## 4. Worked Example

| $X$ | $Y$ | $X^2$ | $Y^2$ | $XY$ |
|:---:|:---:|:-----:|:-----:|:----:|
| 2 | 4 | 4 | 16 | 8 |
| 4 | 3 | 16 | 9 | 12 |
| 6 | 7 | 36 | 49 | 42 |
| 8 | 8 | 64 | 64 | 64 |
| 10 | 9 | 100 | 81 | 90 |
| **30** | **31** | **220** | **219** | **216** |

$n = 5$

**Numerator:** $5(216) - (30)(31) = 1080 - 930 = 150$

**Denominator:** $\sqrt{[5(220) - 900][5(219) - 961]} = \sqrt{(200)(34)} = \sqrt{6800} = 82.46$

$$r = \frac{150}{82.46} = 0.819$$

**Interpretation:** Strong positive linear relationship. $R^2 = 0.671$, so $X$ explains 67.1% of variation in $Y$.

---

## 5. Hypothesis Test for $\rho$

### Test: $H_0: \rho = 0$ (no linear correlation)

$$\boxed{T = \frac{r\sqrt{n-2}}{\sqrt{1-r^2}} \sim t(n-2)}$$

### Example

$r = 0.819$, $n = 5$

$$T = \frac{0.819\sqrt{3}}{\sqrt{1 - 0.671}} = \frac{0.819 \times 1.732}{0.574} = \frac{1.419}{0.574} = 2.47$$

$t_{0.025, 3} = 3.182$. Since $2.47 < 3.182$: **Fail to reject $H_0$** at $\alpha = 0.05$.

(Small sample → insufficient evidence despite seemingly strong $r$.)

### Fisher's $z$-Transformation (for $H_0: \rho = \rho_0$)

$$z = \frac{1}{2}\ln\frac{1+r}{1-r} = \tanh^{-1}(r)$$

$$z \dot\sim N\left(\frac{1}{2}\ln\frac{1+\rho}{1-\rho}, \frac{1}{n-3}\right)$$

Useful for constructing confidence intervals for $\rho$ and testing $\rho = \rho_0 \ne 0$.

---

## 6. Spearman Rank Correlation ($r_s$)

A non-parametric measure based on **ranks** rather than raw values.

$$\boxed{r_s = 1 - \frac{6\sum d_i^2}{n(n^2 - 1)}}$$

where $d_i = \text{rank}(X_i) - \text{rank}(Y_i)$.

### When to Use Spearman

| Scenario | Use |
|----------|-----|
| Data is ordinal (ranks, ratings) | Spearman ✓ |
| Relationship is monotonic but nonlinear | Spearman ✓ |
| Outliers or skewed data | Spearman ✓ |
| Data is interval/ratio, linear relationship | Pearson ✓ |

### Example

| Student | Math Rank ($R_X$) | Science Rank ($R_Y$) | $d_i$ | $d_i^2$ |
|:-------:|:-:|:-:|:-:|:-:|
| A | 1 | 2 | −1 | 1 |
| B | 2 | 1 | 1 | 1 |
| C | 3 | 3 | 0 | 0 |
| D | 4 | 5 | −1 | 1 |
| E | 5 | 4 | 1 | 1 |

$\sum d_i^2 = 4$

$$r_s = 1 - \frac{6(4)}{5(24)} = 1 - \frac{24}{120} = 1 - 0.2 = 0.8$$

Strong positive monotonic relationship.

---

## 7. Important Warnings

### Correlation ≠ Causation

```
  ┌──────────────────────────────────────────────┐
  │  r(ice cream sales, drowning deaths) = 0.85  │
  │                                              │
  │        Ice Cream ───✕──→ Drowning?           │
  │            ↑                  ↑               │
  │            └── Hot Weather ──┘                │
  │                (lurking variable)             │
  └──────────────────────────────────────────────┘
```

### Common Pitfalls

| Pitfall | Problem |
|---------|---------|
| **Nonlinear data** | $r \approx 0$ even with strong curved relationship |
| **Outliers** | Single outlier can drastically change $r$ |
| **Restricted range** | Correlation is suppressed when data range is narrow |
| **Aggregation (ecological fallacy)** | Group-level $r$ ≠ individual-level $r$ |
| **$r = 0 \not\Rightarrow$ independent** | Only for normal distributions |

```
  Example: r ≈ 0 but STRONG relationship!
  
  Y │     ·       ·
    │   ·           ·
    │  ·             ·
    │ ·               ·
    │·                 ·
    └──────────────────── X
    
  Parabolic relationship → Pearson r ≈ 0
```

---

## 8. Comparing Pearson and Spearman

| Feature | Pearson ($r$) | Spearman ($r_s$) |
|---------|:-------:|:---------:|
| Data type | Continuous | Ordinal or continuous |
| Measures | Linear association | Monotonic association |
| Sensitive to outliers | Yes | Less so |
| Assumes normality | Yes (for test) | No |
| Uses raw values | Yes | No (ranks) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Pearson $r$ | $S_{XY}/\sqrt{S_{XX} \cdot S_{YY}}$, range $[-1, 1]$ |
| Interpretation | Sign = direction, $\|r\|$ = strength |
| Test $\rho = 0$ | $T = r\sqrt{n-2}/\sqrt{1-r^2} \sim t(n-2)$ |
| Fisher's $z$ | $z = \tanh^{-1}(r)$, approximately normal |
| Spearman $r_s$ | $1 - 6\sum d_i^2/[n(n^2-1)]$, rank-based |
| $R^2 = r^2$ | Proportion of variance explained |
| Correlation ≠ Causation | Always consider lurking variables |

---

## Quick Revision Questions

1. Compute $r$ for: $n=4, \sum X=10, \sum Y=12, \sum X^2=30, \sum Y^2=40, \sum XY=35$.

2. If $r = 0.9$ and $n = 10$, test $H_0: \rho = 0$ at $\alpha = 0.05$.

3. Calculate the Spearman rank correlation if rankings are: $(1,1), (2,3), (3,2), (4,4)$.

4. Give an example where $r = 0$ but $X$ and $Y$ are clearly related.

5. Explain the difference between Pearson's $r$ and Spearman's $r_s$. When would you prefer each?

---

[← Previous: Least Squares Method](02-least-squares-method.md) | [Back to README](../README.md) | [Next: Multiple Regression →](04-multiple-regression.md)
