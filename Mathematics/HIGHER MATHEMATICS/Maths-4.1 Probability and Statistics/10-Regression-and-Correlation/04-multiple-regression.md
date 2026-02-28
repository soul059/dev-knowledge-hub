# Chapter 10.4: Multiple Regression

[← Previous: Correlation Coefficient](03-correlation-coefficient.md) | [Back to README](../README.md)

---

## Overview

**Multiple regression** extends simple linear regression to model the relationship between a response variable $Y$ and **two or more** predictor variables $X_1, X_2, \ldots, X_k$. It is one of the most widely used statistical techniques in practice.

---

## 1. The Multiple Regression Model

$$\boxed{Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \cdots + \beta_k X_k + \varepsilon}$$

where $\varepsilon \sim N(0, \sigma^2)$ independently.

```
  Simple (1 predictor)          Multiple (2 predictors)
  
  Y │     ·                     Y │  ·     ·
    │   · ╱ ·                     │ · ╱  ╱ ·
    │  ·╱· ·                      │  ╱· ╱ ·
    │ ╱· ·                        │ ╱ ·╱·          ← a PLANE
    │╱· ·                         │╱ ╱·
    └──────── X                   └──────── X₁
                                 ╱
                                X₂
  
  Line: Y = β₀ + β₁X           Plane: Y = β₀ + β₁X₁ + β₂X₂
```

### Interpretation of Coefficients

$\beta_j$ = expected change in $Y$ for a **one-unit increase** in $X_j$, **holding all other predictors constant** (ceteris paribus).

---

## 2. Matrix Formulation

For $n$ observations and $k$ predictors:

$$\mathbf{Y} = \mathbf{X}\boldsymbol{\beta} + \boldsymbol{\varepsilon}$$

$$\begin{pmatrix} Y_1 \\ Y_2 \\ \vdots \\ Y_n \end{pmatrix} = \begin{pmatrix} 1 & X_{11} & X_{12} & \cdots & X_{1k} \\ 1 & X_{21} & X_{22} & \cdots & X_{2k} \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 1 & X_{n1} & X_{n2} & \cdots & X_{nk} \end{pmatrix} \begin{pmatrix} \beta_0 \\ \beta_1 \\ \vdots \\ \beta_k \end{pmatrix} + \begin{pmatrix} \varepsilon_1 \\ \varepsilon_2 \\ \vdots \\ \varepsilon_n \end{pmatrix}$$

### Least Squares Solution

$$\boxed{\hat{\boldsymbol{\beta}} = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{Y}}$$

---

## 3. Worked Example (Two Predictors)

Predict exam score ($Y$) from study hours ($X_1$) and sleep hours ($X_2$):

| Student | $X_1$ (Study) | $X_2$ (Sleep) | $Y$ (Score) |
|:-------:|:-----:|:-----:|:-----:|
| 1 | 3 | 7 | 65 |
| 2 | 5 | 6 | 72 |
| 3 | 2 | 8 | 60 |
| 4 | 7 | 5 | 80 |
| 5 | 4 | 7 | 68 |
| 6 | 6 | 6 | 75 |

Suppose least squares yields: $\hat{Y} = 30 + 5.5X_1 + 2.8X_2$

**Interpretation:**
- Each additional study hour → score increases by **5.5** (holding sleep constant)
- Each additional sleep hour → score increases by **2.8** (holding study constant)
- Intercept: predicted score with 0 study and 0 sleep = 30 (extrapolation, not meaningful)

---

## 4. Coefficient of Determination ($R^2$ and Adjusted $R^2$)

### $R^2$

$$R^2 = 1 - \frac{SSE}{SST}$$

**Problem:** $R^2$ **always increases** when adding predictors, even useless ones.

### Adjusted $R^2$

$$\boxed{R^2_{\text{adj}} = 1 - \frac{SSE/(n-k-1)}{SST/(n-1)} = 1 - (1-R^2)\frac{n-1}{n-k-1}}$$

```
  R² vs Adjusted R² as predictors are added
  
  1.0 │        ·────────── R²  (always increases)
      │      ·
      │    ·   ·──·────── R²_adj (can decrease)
  0.5 │   ·      ·
      │  ·        ·
      │ ·
  0.0 │·
      └──────────────── Number of predictors
       1  2  3  4  5
  
  R²_adj PENALIZES adding useless predictors
```

| Metric | Use |
|--------|-----|
| $R^2$ | Proportion of variance explained (descriptive) |
| $R^2_{\text{adj}}$ | Model comparison with different $k$ (inferential) |

---

## 5. Tests of Significance

### Overall F-Test

$$H_0: \beta_1 = \beta_2 = \cdots = \beta_k = 0 \quad (\text{model is useless})$$

$$\boxed{F = \frac{MSR}{MSE} = \frac{SSR/k}{SSE/(n-k-1)} \sim F(k, n-k-1)}$$

### Individual t-Tests

$$H_0: \beta_j = 0 \quad (\text{predictor } X_j \text{ is not significant})$$

$$T_j = \frac{\hat{\beta}_j}{\text{SE}(\hat{\beta}_j)} \sim t(n-k-1)$$

### ANOVA Table for Multiple Regression

| Source | SS | df | MS | F |
|--------|-----|------|------|------|
| Regression | $SSR$ | $k$ | $MSR = SSR/k$ | $MSR/MSE$ |
| Error | $SSE$ | $n-k-1$ | $MSE = SSE/(n-k-1)$ | |
| Total | $SST$ | $n-1$ | | |

---

## 6. Multicollinearity

When predictors are highly correlated with each other.

```
  ┌─────────────────────────────────────┐
  │  Multicollinearity: X₁ ≈ f(X₂)     │
  │                                     │
  │  Effects:                           │
  │  • SE(β̂ⱼ) inflated → wider CIs     │
  │  • Coefficients unstable            │
  │  • Individual t-tests misleading    │
  │  • R² may still be high            │
  │  • Signs of β̂ⱼ may flip            │
  └─────────────────────────────────────┘
```

### Variance Inflation Factor (VIF)

$$\boxed{VIF_j = \frac{1}{1 - R_j^2}}$$

where $R_j^2$ = $R^2$ from regressing $X_j$ on all other predictors.

| VIF | Assessment |
|-----|-----------|
| 1 | No collinearity |
| 1–5 | Moderate |
| 5–10 | High (concerning) |
| > 10 | Severe (take action) |

### Remedies

1. Remove one of the correlated predictors
2. Combine predictors (e.g., PCA)
3. Ridge regression or LASSO
4. Center the data

---

## 7. Model Building Strategies

### Variable Selection

| Method | How It Works |
|--------|-------------|
| **Forward Selection** | Start empty, add best predictor at each step |
| **Backward Elimination** | Start full, remove worst predictor at each step |
| **Stepwise** | Combine forward and backward |
| **Best Subsets** | Evaluate all $2^k$ possible models |

### Model Selection Criteria

| Criterion | Formula | Goal |
|-----------|---------|------|
| $R^2_{\text{adj}}$ | $1 - (1-R^2)\frac{n-1}{n-k-1}$ | Maximize |
| $AIC$ | $n\ln(SSE/n) + 2(k+1)$ | Minimize |
| $BIC$ | $n\ln(SSE/n) + (k+1)\ln(n)$ | Minimize |
| $C_p$ (Mallows) | $SSE_p/MSE_{\text{full}} - n + 2p$ | Close to $p$ |

---

## 8. Assumptions and Diagnostics

### Assumptions (same as simple regression, extended)

1. **Linearity**: $E[Y|\mathbf{X}]$ is linear in $X_j$
2. **Independence**: Observations independent
3. **Normality**: $\varepsilon \sim N(0, \sigma^2)$
4. **Equal variance** (homoscedasticity): $\text{Var}(\varepsilon)$ constant
5. **No multicollinearity**: Predictors not too correlated

### Diagnostic Plots

```
  Residual vs Fitted                  Normal Q-Q Plot
  
  eᵢ │  ·  ·    ·                   Ordered  │         ·
     │·  ·  ·  · ·                  residuals│      · ·
  0  │─·──·──·──·──                          │   · ·
     │  ·  · ·  · ·                          │ ·· 
     │ · ·    ·                              │·
     └──────────── Ŷᵢ                        └─────────
                                              Theoretical quantiles
  Good: random scatter                Good: points on diagonal
```

---

## 9. Dummy Variables

Use **indicator variables** (0/1) for categorical predictors.

For a categorical variable with $c$ categories, use $c - 1$ dummy variables.

### Example: Season Effect on Sales

| Season | $D_1$ (Summer) | $D_2$ (Fall) | $D_3$ (Winter) |
|--------|:-:|:-:|:-:|
| Spring (baseline) | 0 | 0 | 0 |
| Summer | 1 | 0 | 0 |
| Fall | 0 | 1 | 0 |
| Winter | 0 | 0 | 1 |

$$Y = \beta_0 + \beta_1 D_1 + \beta_2 D_2 + \beta_3 D_3 + \varepsilon$$

$\beta_j$ = difference in mean $Y$ between category $j$ and the baseline.

---

## 10. Simple vs Multiple Regression Comparison

| Feature | Simple | Multiple |
|---------|--------|----------|
| Predictors | 1 ($X$) | $k$ ($X_1, \ldots, X_k$) |
| Geometry | Line | Hyperplane |
| $R^2$ | $r^2$ | Not simply related to individual $r$'s |
| df (error) | $n-2$ | $n-k-1$ |
| Coefficients | Effect of $X$ alone | Effect of $X_j$ **controlling for others** |
| Overall test | $t$-test for $\beta_1$ | $F$-test for all $\beta_j$ |
| Key concern | Assumptions | Assumptions + multicollinearity |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Model | $Y = \beta_0 + \beta_1X_1 + \cdots + \beta_kX_k + \varepsilon$ |
| Matrix form | $\hat{\boldsymbol{\beta}} = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{Y}$ |
| $R^2_{\text{adj}}$ | Penalizes adding useless predictors |
| $F$-test | Tests overall model significance ($H_0$: all $\beta_j = 0$) |
| $t$-test | Tests individual predictor ($H_0$: $\beta_j = 0$) |
| VIF | Detects multicollinearity; $VIF > 10$ is severe |
| Dummy variables | Encode categorical predictors with $c-1$ indicators |
| Model selection | Use $R^2_{\text{adj}}$, AIC, BIC to compare models |

---

## Quick Revision Questions

1. Write the multiple regression model with 3 predictors. How many parameters does it have?

2. If $R^2 = 0.80$, $n = 50$, $k = 4$, compute $R^2_{\text{adj}}$.

3. Explain what "$\beta_2 = 3.5$" means in a model with predictors $X_1$ and $X_2$.

4. Given $SSR = 400$, $SSE = 100$, $k = 3$, $n = 25$: construct the ANOVA table and test overall significance.

5. What is multicollinearity? How does it affect regression results and how would you detect it?

6. How many dummy variables are needed for a categorical variable with 5 levels? Why not 5?

---

[← Previous: Correlation Coefficient](03-correlation-coefficient.md) | [Back to README](../README.md)
