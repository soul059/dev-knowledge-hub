# Chapter 10.2: Least Squares Method

[← Previous: Linear Regression](01-linear-regression.md) | [Back to README](../README.md) | [Next: Correlation Coefficient →](03-correlation-coefficient.md)

---

## Overview

The **method of least squares** finds the line that minimizes the total squared error between observed and predicted values. It provides the mathematical foundation for regression analysis.

---

## 1. The Least Squares Criterion

Find $\hat{\beta}_0$ and $\hat{\beta}_1$ that minimize:

$$\boxed{Q(\beta_0, \beta_1) = \sum_{i=1}^n e_i^2 = \sum_{i=1}^n (Y_i - \beta_0 - \beta_1 X_i)^2}$$

```
  Why SQUARED errors?
  
  Y │     · (e₃)          ┌────────────────────────────┐
    │    ╱ ·               │ Squaring errors:           │
    │   ╱ · (e₂)          │ 1. Positive and negative   │
    │  ╱·                  │    errors don't cancel     │
    │ ╱  · (e₁)           │ 2. Large errors penalized  │
    │╱                     │    more than small ones    │
    └──────────── X        │ 3. Yields smooth, solvable │
                           │    optimization problem    │
  Minimize: Σeᵢ²           └────────────────────────────┘
```

---

## 2. Derivation of Normal Equations

Taking partial derivatives and setting them to zero:

$$\frac{\partial Q}{\partial \beta_0} = -2\sum(Y_i - \beta_0 - \beta_1 X_i) = 0$$

$$\frac{\partial Q}{\partial \beta_1} = -2\sum X_i(Y_i - \beta_0 - \beta_1 X_i) = 0$$

### Normal Equations

$$\boxed{\sum Y_i = n\hat{\beta}_0 + \hat{\beta}_1 \sum X_i}$$

$$\boxed{\sum X_i Y_i = \hat{\beta}_0 \sum X_i + \hat{\beta}_1 \sum X_i^2}$$

### Solutions

$$\boxed{\hat{\beta}_1 = \frac{n\sum X_i Y_i - \sum X_i \sum Y_i}{n\sum X_i^2 - (\sum X_i)^2} = \frac{S_{XY}}{S_{XX}}}$$

$$\boxed{\hat{\beta}_0 = \bar{Y} - \hat{\beta}_1 \bar{X}}$$

where:
- $S_{XY} = \sum X_i Y_i - n\bar{X}\bar{Y} = \sum(X_i - \bar{X})(Y_i - \bar{Y})$
- $S_{XX} = \sum X_i^2 - n\bar{X}^2 = \sum(X_i - \bar{X})^2$

---

## 3. Worked Example

| $X_i$ | $Y_i$ | $X_i^2$ | $X_iY_i$ |
|:-:|:-:|:-:|:-:|
| 1 | 2 | 1 | 2 |
| 2 | 4 | 4 | 8 |
| 3 | 5 | 9 | 15 |
| 4 | 4 | 16 | 16 |
| 5 | 5 | 25 | 25 |
| **15** | **20** | **55** | **66** |

$n = 5$, $\bar{X} = 3$, $\bar{Y} = 4$

$$\hat{\beta}_1 = \frac{5(66) - (15)(20)}{5(55) - (15)^2} = \frac{330 - 300}{275 - 225} = \frac{30}{50} = 0.6$$

$$\hat{\beta}_0 = 4 - 0.6(3) = 2.2$$

$$\hat{Y} = 2.2 + 0.6X$$

### Residuals

| $X_i$ | $Y_i$ | $\hat{Y}_i$ | $e_i = Y_i - \hat{Y}_i$ | $e_i^2$ |
|:-:|:-:|:-:|:-:|:-:|
| 1 | 2 | 2.8 | −0.8 | 0.64 |
| 2 | 4 | 3.4 | 0.6 | 0.36 |
| 3 | 5 | 4.0 | 1.0 | 1.00 |
| 4 | 4 | 4.6 | −0.6 | 0.36 |
| 5 | 5 | 5.2 | −0.2 | 0.04 |
| | | | $\sum = 0$ ✓ | $SSE = 2.40$ |

---

## 4. Partitioning Total Variation

$$\boxed{SST = SSR + SSE}$$

| Term | Full Name | Formula | Measures |
|------|-----------|---------|----------|
| $SST$ | Total Sum of Squares | $\sum(Y_i - \bar{Y})^2$ | Total variation in $Y$ |
| $SSR$ | Regression Sum of Squares | $\sum(\hat{Y}_i - \bar{Y})^2$ | Variation explained by $X$ |
| $SSE$ | Error Sum of Squares | $\sum(Y_i - \hat{Y}_i)^2 = \sum e_i^2$ | Unexplained variation |

```
  Total variation = Explained + Unexplained
  
  Y │    ·
    │    ↕ SST = (Yᵢ - Ȳ)           = total distance
    │    │
    │····│·····Ȳ····· ← grand mean
    │    │
    │    ↕ SSR = (Ŷᵢ - Ȳ)           = explained
    │    │
    │────●──── Ŷᵢ (on regression line)
    │    ↕ eᵢ = (Yᵢ - Ŷᵢ)           = unexplained
    │    ·
    └─────────────── X
```

---

## 5. Coefficient of Determination ($R^2$)

$$\boxed{R^2 = \frac{SSR}{SST} = 1 - \frac{SSE}{SST}}$$

| $R^2$ Value | Interpretation |
|------------|----------------|
| $R^2 = 0$ | $X$ explains none of the variation in $Y$ |
| $R^2 = 0.5$ | $X$ explains 50% of the variation |
| $R^2 = 1$ | $X$ explains all variation (perfect fit) |

**For simple regression:** $R^2 = r^2$ (squared correlation coefficient).

### Example (continued)

$SST = \sum(Y_i - 4)^2 = 4 + 0 + 1 + 0 + 1 = 6$

$SSE = 2.40$ (from residuals above)

$SSR = SST - SSE = 6 - 2.40 = 3.60$

$$R^2 = \frac{3.60}{6} = 0.60$$

$X$ explains 60% of the variation in $Y$.

---

## 6. Estimating $\sigma^2$

$$\boxed{S^2 = MSE = \frac{SSE}{n-2} = \frac{\sum e_i^2}{n-2}}$$

The divisor is $n - 2$ because we estimated 2 parameters ($\beta_0, \beta_1$).

### Standard Error of Regression

$$S = \sqrt{MSE} = \sqrt{\frac{SSE}{n-2}}$$

For our example: $S^2 = 2.40/3 = 0.80$, $S = 0.894$.

---

## 7. Inference on Regression Coefficients

### Sampling Distribution of $\hat{\beta}_1$

$$\hat{\beta}_1 \sim N\left(\beta_1, \frac{\sigma^2}{S_{XX}}\right)$$

$$\text{SE}(\hat{\beta}_1) = \frac{S}{\sqrt{S_{XX}}}$$

### Hypothesis Test for Slope

$$H_0: \beta_1 = 0 \quad (\text{no linear relationship})$$

$$T = \frac{\hat{\beta}_1}{\text{SE}(\hat{\beta}_1)} \sim t(n-2)$$

### Confidence Interval for $\beta_1$

$$\hat{\beta}_1 \pm t_{\alpha/2, n-2} \cdot \text{SE}(\hat{\beta}_1)$$

### Example

$\hat{\beta}_1 = 0.6$, $S = 0.894$, $S_{XX} = 50$, $n = 5$

$$\text{SE}(\hat{\beta}_1) = \frac{0.894}{\sqrt{50}} = 0.126$$

$$T = \frac{0.6}{0.126} = 4.76, \quad \text{df} = 3$$

$t_{0.025, 3} = 3.182$. Since $4.76 > 3.182$: **Reject $H_0$**. Significant linear relationship.

---

## 8. ANOVA Table for Regression

| Source | SS | df | MS | F |
|--------|-----|------|------|------|
| Regression | $SSR$ | 1 | $MSR = SSR/1$ | $F = MSR/MSE$ |
| Error | $SSE$ | $n-2$ | $MSE = SSE/(n-2)$ | |
| Total | $SST$ | $n-1$ | | |

The $F$-test for overall regression significance is equivalent to the $t$-test for $\beta_1 = 0$ (in simple regression, $F = T^2$).

---

## 9. Prediction Intervals

### Confidence Interval for $E[Y|X_0]$ (mean response)

$$\hat{Y}_0 \pm t_{\alpha/2, n-2} \cdot S\sqrt{\frac{1}{n} + \frac{(X_0 - \bar{X})^2}{S_{XX}}}$$

### Prediction Interval for new $Y_0$ (individual value)

$$\hat{Y}_0 \pm t_{\alpha/2, n-2} \cdot S\sqrt{1 + \frac{1}{n} + \frac{(X_0 - \bar{X})^2}{S_{XX}}}$$

```
  Prediction interval is WIDER than confidence interval
  
  Y │        ╱  ╱  ╱
    │      ╱  ╱  ╱      ← Prediction interval (individual)
    │    ╱ ╱ ╱ ╱
    │  ╱ ╱·╱·╱          ← Confidence interval (mean)
    │╱ ╱·╱·╱
    │╱╱·╱·
    │╱·╱·               Both narrowest at X = X̄
    └──────────── X
           X̄
```

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Minimize | $Q = \sum(Y_i - \beta_0 - \beta_1 X_i)^2$ |
| $\hat{\beta}_1$ | $S_{XY}/S_{XX}$ |
| $\hat{\beta}_0$ | $\bar{Y} - \hat{\beta}_1\bar{X}$ |
| $SST = SSR + SSE$ | Total = Explained + Unexplained |
| $R^2$ | $SSR/SST = 1 - SSE/SST$ |
| $MSE$ | $SSE/(n-2)$ |
| Test $\beta_1 = 0$ | $T = \hat{\beta}_1/\text{SE}(\hat{\beta}_1) \sim t(n-2)$ |

---

## Quick Revision Questions

1. Derive the normal equations by setting partial derivatives of $Q$ to zero.

2. From data: $n=6, \sum X=21, \sum Y=30, \sum X^2=91, \sum XY=120$. Find $\hat{\beta}_0$ and $\hat{\beta}_1$.

3. If $SST = 200$ and $SSE = 50$, find $R^2$ and interpret it.

4. Explain why the prediction interval is wider than the confidence interval for the mean.

5. Test $H_0: \beta_1 = 0$ given $\hat{\beta}_1 = 2.5$, $\text{SE}(\hat{\beta}_1) = 0.8$, $n = 12$.

6. What does $R^2 = 0.85$ mean in context?

---

[← Previous: Linear Regression](01-linear-regression.md) | [Back to README](../README.md) | [Next: Correlation Coefficient →](03-correlation-coefficient.md)
