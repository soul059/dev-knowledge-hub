# Chapter 10.1: Linear Regression

[← Previous: P-Values](../09-Hypothesis-Testing/06-p-values.md) | [Back to README](../README.md) | [Next: Least Squares Method →](02-least-squares-method.md)

---

## Overview

**Linear regression** models the relationship between a dependent variable $Y$ and one or more independent variables $X$. It is one of the most widely used tools in statistics and data science.

---

## 1. Simple Linear Regression Model

$$\boxed{Y = \beta_0 + \beta_1 X + \varepsilon}$$

| Symbol | Name | Meaning |
|--------|------|---------|
| $Y$ | Response (dependent) variable | What we predict |
| $X$ | Predictor (independent) variable | What we use to predict |
| $\beta_0$ | Y-intercept | Value of $Y$ when $X = 0$ |
| $\beta_1$ | Slope | Change in $Y$ per unit change in $X$ |
| $\varepsilon$ | Error term | Random noise, $\varepsilon \sim N(0, \sigma^2)$ |

```
  Y
  │            ·    ·
  │         ·  ╱ ·
  │       · ╱·       ← Scatter around the line
  │     · ╱·  ·
  │    ╱·  ·          Y = β₀ + β₁X  (regression line)
  │  ╱· ·
  │╱·  ·
  │·
  β₀───────────────► X
  (intercept)
  
  β₁ = slope = rise/run = ΔY/ΔX
```

---

## 2. The Regression Line

The **fitted** (estimated) regression line:

$$\boxed{\hat{Y} = \hat{\beta}_0 + \hat{\beta}_1 X}$$

where $\hat{\beta}_0$ and $\hat{\beta}_1$ are estimated from data (details in next chapter).

### Key Formulas

$$\boxed{\hat{\beta}_1 = \frac{\sum(X_i - \bar{X})(Y_i - \bar{Y})}{\sum(X_i - \bar{X})^2} = \frac{S_{XY}}{S_{XX}}}$$

$$\boxed{\hat{\beta}_0 = \bar{Y} - \hat{\beta}_1\bar{X}}$$

**The regression line always passes through $(\bar{X}, \bar{Y})$.**

---

## 3. Interpretation

### Slope $\hat{\beta}_1$

"For each one-unit increase in $X$, $Y$ changes by $\hat{\beta}_1$ units **on average**."

| $\hat{\beta}_1$ | Meaning |
|-----------------|---------|
| $> 0$ | Positive association: $X \uparrow \Rightarrow Y \uparrow$ |
| $< 0$ | Negative association: $X \uparrow \Rightarrow Y \downarrow$ |
| $= 0$ | No linear relationship |

### Intercept $\hat{\beta}_0$

"The predicted $Y$ when $X = 0$." (May not be meaningful if $X = 0$ is outside the data range.)

---

## 4. Residuals

$$\boxed{e_i = Y_i - \hat{Y}_i = Y_i - (\hat{\beta}_0 + \hat{\beta}_1 X_i)}$$

```
  Y │
    │           · (Yᵢ)
    │           ↑ eᵢ = residual
    │           ↓
    │      ────●──── regression line (Ŷᵢ)
    │     ╱
    │   ╱
    │ ╱
    └─────────────── X
              Xᵢ
  
  Residual = observed - predicted
  eᵢ > 0: point ABOVE line (underestimate)
  eᵢ < 0: point BELOW line (overestimate)
```

### Properties of Residuals

1. $\sum_{i=1}^n e_i = 0$ (residuals sum to zero)
2. $\sum_{i=1}^n X_i e_i = 0$ (residuals uncorrelated with $X$)
3. $\bar{e} = 0$

---

## 5. Assumptions (LINE)

| Letter | Assumption | Description |
|--------|-----------|-------------|
| **L** | Linearity | Relationship between $X$ and $Y$ is linear |
| **I** | Independence | Errors are independent |
| **N** | Normality | Errors are normally distributed |
| **E** | Equal variance | $\text{Var}(\varepsilon) = \sigma^2$ constant (homoscedasticity) |

```
  Good (assumptions met):          Bad (assumptions violated):
  
  Residuals │  · ·  ·               Residuals │        ·  ·
            │· · · · · ·                       │    ·  ·
            │ ·  · ·  ·                        │  · ·
  ──────────┼──────────── X        ────────────┼──────────── X
            │· ·  · · ·                        │· ·
            │  · ·  ·                          │·
  
  Random scatter                   Fan shape (heteroscedasticity)
  
  Residuals │  · ·  ·               Residuals │     ·
            │·  · · ·                          │  ·    ·
            │ ·  ·  ·                          │·   ·    ·
  ──────────┼──────────── X        ────────────┼──────·───── X
            │ · · · ·                          │        ·
            │· ·  ·                            │     ·
  
  Random scatter                   Curve (non-linearity)
```

---

## 6. Worked Example

| Student | Hours studied ($X$) | Score ($Y$) |
|:-:|:-:|:-:|
| 1 | 2 | 55 |
| 2 | 3 | 60 |
| 3 | 5 | 70 |
| 4 | 7 | 80 |
| 5 | 8 | 85 |

$n = 5$, $\bar{X} = 5$, $\bar{Y} = 70$

$S_{XY} = \sum(X_i - 5)(Y_i - 70) = (-3)(-15) + (-2)(-10) + (0)(0) + (2)(10) + (3)(15) = 45 + 20 + 0 + 20 + 45 = 130$

$S_{XX} = \sum(X_i - 5)^2 = 9 + 4 + 0 + 4 + 9 = 26$

$$\hat{\beta}_1 = \frac{130}{26} = 5.0$$

$$\hat{\beta}_0 = 70 - 5(5) = 45$$

$$\hat{Y} = 45 + 5X$$

**Interpretation:** Each additional hour of study is associated with a 5-point increase in score, on average.

---

## 7. Prediction

### Point Prediction

For a new $X_0$: $\hat{Y}_0 = \hat{\beta}_0 + \hat{\beta}_1 X_0$

### Prediction vs Extrapolation

```
  Y │                          ·
    │                    ·    ?    ← EXTRAPOLATION
    │              ·   ·         (dangerous!)
    │        ·   ·
    │     · ·                    ← INTERPOLATION
    │  · ·                       (reliable)
    │ ·
    └────────────────────────── X
    │←─── data range ───→│← outside →│
```

**Never extrapolate** far beyond the observed range of $X$.

---

## 8. Real-World Applications

| Field | $X$ (Predictor) | $Y$ (Response) |
|-------|-----------------|----------------|
| Economics | Advertising spend | Revenue |
| Medicine | Dosage | Response |
| Engineering | Temperature | Yield |
| Education | Study hours | Test score |
| Climate | CO₂ levels | Temperature |

---

## Summary Table

| Concept | Formula/Key Point |
|---------|------------------|
| Model | $Y = \beta_0 + \beta_1 X + \varepsilon$ |
| Fitted line | $\hat{Y} = \hat{\beta}_0 + \hat{\beta}_1 X$ |
| Slope | $\hat{\beta}_1 = S_{XY}/S_{XX}$ |
| Intercept | $\hat{\beta}_0 = \bar{Y} - \hat{\beta}_1\bar{X}$ |
| Residual | $e_i = Y_i - \hat{Y}_i$ |
| Assumptions | LINE: Linearity, Independence, Normality, Equal variance |
| Passes through | $(\bar{X}, \bar{Y})$ always |

---

## Quick Revision Questions

1. In the model $Y = 3.2 + 1.5X$: interpret the slope and intercept.

2. A regression line gives $\hat{Y} = 20 - 0.8X$. Predict $Y$ when $X = 10$ and when $X = 50$ (data range: 5-25). Comment.

3. What are the four assumptions of linear regression? How would you check each?

4. If $\sum e_i \neq 0$, what went wrong?

5. Explain why correlation does not imply causation in the context of regression.

6. From data: $\bar{X}=4, \bar{Y}=30, S_{XY}=60, S_{XX}=20$. Find the regression equation and predict $Y$ at $X = 6$.

---

[← Previous: P-Values](../09-Hypothesis-Testing/06-p-values.md) | [Back to README](../README.md) | [Next: Least Squares Method →](02-least-squares-method.md)
