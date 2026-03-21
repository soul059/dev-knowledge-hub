# 📘 Chapter 1: Simple Linear Regression

> **Unit 2.1 – Introduction to Machine Learning / 04 – Linear Regression** | File 1 of 7

---

## 📋 Overview

Simple Linear Regression (SLR) is the most fundamental supervised learning algorithm for
**predicting a continuous target variable** using a single input feature. It fits a straight
line through the data that minimizes the total prediction error. This chapter covers the
model equation, the Least Squares method, geometric interpretation, key assumptions,
residual analysis, the R² metric, a complete worked example, and Python implementations
both from scratch and with scikit-learn.

---

## 1. The Model Equation

### 1.1 Mathematical Form

```
ŷ = mx + b
```

| Symbol | Name | Meaning |
|--------|------|---------|
| `ŷ` | Predicted value | The model's estimate of the target |
| `x` | Input feature | The independent / explanatory variable |
| `m` | Slope (weight) | Change in ŷ per unit change in x |
| `b` | Intercept (bias) | Value of ŷ when x = 0 |

> In ML notation you will often see `ŷ = w₁x + w₀` or `ŷ = wx + b`.
> All three forms are identical — just different naming conventions.

### 1.2 Geometric Interpretation

```
    ŷ (target)
    │              ╱ ŷ = mx + b
    │            ╱
    │          ╱  ●          ← data point above the line (positive residual)
    │        ╱
    │      ╱ ●               ← data point ON the line (zero residual)
    │    ╱
    │  ╱        ●            ← data point below the line (negative residual)
    │╱
    ├──────────────────────── x (feature)
    │
   b = intercept (where line crosses ŷ-axis)
```

- **Slope `m`:** The angle of the line. A positive slope means ŷ increases as x increases.
  Numerically, `m = Δŷ / Δx`.
- **Intercept `b`:** Where the line crosses the vertical axis (x = 0).
- **Residual:** The vertical distance from a data point to the line: `eᵢ = yᵢ − ŷᵢ`.

---

## 2. Fitting the Line – The Least Squares Method

### 2.1 The Goal

We want to find `m` and `b` that **minimize the sum of squared residuals (SSR)**:

```
                m
SSR = Σ (yᵢ − ŷᵢ)²  =  Σ (yᵢ − mxᵢ − b)²
               i=1              i=1
```

Why squared?
- Squaring penalizes large errors more than small ones.
- Squaring ensures positive and negative residuals don't cancel out.
- The resulting function is **convex** → guarantees a unique global minimum.

### 2.2 Derivation of Closed-Form Solution

We take partial derivatives of SSR with respect to `m` and `b`, set them to zero, and solve.

**Step 1 — Partial derivative w.r.t. `b`:**

```
∂SSR/∂b = -2 Σ (yᵢ − mxᵢ − b) = 0
          ⟹  Σyᵢ = m Σxᵢ + nb
          ⟹  ȳ   = mx̄ + b               (divide both sides by n)
          ⟹  b   = ȳ − mx̄
```

**Step 2 — Partial derivative w.r.t. `m`:**

```
∂SSR/∂m = -2 Σ xᵢ(yᵢ − mxᵢ − b) = 0
          ⟹  Σ xᵢyᵢ = m Σ xᵢ² + b Σ xᵢ
```

Substituting `b = ȳ − mx̄`:

```
          Σ xᵢyᵢ = m Σ xᵢ² + (ȳ − mx̄) Σ xᵢ
          Σ xᵢyᵢ − ȳ Σ xᵢ = m(Σ xᵢ² − x̄ Σ xᵢ)
```

### 2.3 Final Formulas

```
         Σ(xᵢ − x̄)(yᵢ − ȳ)         Cov(x, y)
  m  =  ─────────────────────  =  ─────────────
           Σ(xᵢ − x̄)²               Var(x)

  b  =  ȳ − m · x̄
```

| Quantity | Formula | Description |
|----------|---------|-------------|
| `x̄` | `(1/n) Σxᵢ` | Mean of feature values |
| `ȳ` | `(1/n) Σyᵢ` | Mean of target values |
| `Cov(x,y)` | `(1/n) Σ(xᵢ−x̄)(yᵢ−ȳ)` | Covariance |
| `Var(x)` | `(1/n) Σ(xᵢ−x̄)²` | Variance of x |

---

## 3. Assumptions of Linear Regression

For the Least Squares estimates to be **BLUE** (Best Linear Unbiased Estimators — the
Gauss-Markov theorem), these assumptions must hold:

### 3.1 The Four Key Assumptions

```
┌─────────────────────────────────────────────────────────────────────┐
│                  ASSUMPTIONS OF LINEAR REGRESSION                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. LINEARITY                                                       │
│     The relationship between x and y is linear.                     │
│     Check: Scatter plot of x vs y, residual plot.                   │
│                                                                     │
│  2. INDEPENDENCE                                                    │
│     Residuals are independent of each other.                        │
│     Violated in: time-series data with autocorrelation.             │
│     Check: Durbin-Watson test (value ≈ 2 means no autocorrelation). │
│                                                                     │
│  3. HOMOSCEDASTICITY (Equal Variance)                               │
│     Residuals have constant variance across all levels of x.        │
│     Opposite: Heteroscedasticity (fan-shaped residual plot).        │
│     Check: Residuals vs fitted values plot.                         │
│                                                                     │
│  4. NORMALITY of Residuals                                          │
│     Residuals follow a Normal distribution N(0, σ²).                │
│     Check: Q-Q plot, Shapiro-Wilk test.                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 Visual: Good vs. Bad Residual Patterns

```
  GOOD (Homoscedastic)          BAD (Heteroscedastic — fan shape)
  Residuals vs Fitted           Residuals vs Fitted

  e │    ·  ·                   e │          · ·
    │  · ·    · ·                 │        ·   ·
  0 │─·──·──·──·──·──            0 │──·──·───·────·──
    │  · ·   · ·                  │  · ·  ·
    │    ·   ·                    │ ·
    └──────────────── ŷ           └──────────────── ŷ
    Random scatter = ✅           Funnel shape = ❌
```

### 3.3 Consequences of Violated Assumptions

| Violation | Consequence | Remedy |
|-----------|-------------|--------|
| Non-linearity | Biased estimates, poor predictions | Transform features (log, sqrt), use polynomial regression |
| Autocorrelation | Underestimated standard errors | Use time-series models (ARIMA) |
| Heteroscedasticity | Inefficient estimates, unreliable p-values | Weighted Least Squares, log-transform target |
| Non-normal residuals | Invalid confidence intervals | Transform target, use robust regression |

---

## 4. Residual Analysis

### 4.1 Types of Residuals

```
  Raw residual:         eᵢ = yᵢ − ŷᵢ
  Standardized:         rᵢ = eᵢ / s        where s = std dev of residuals
  Studentized:          tᵢ = eᵢ / (s√(1 − hᵢᵢ))    where hᵢᵢ = leverage
```

### 4.2 Influential Points

- **Outlier:** A point with a large residual (far from the line vertically).
- **Leverage point:** A point with an extreme x-value (far from x̄ horizontally).
- **Influential point:** A point that, if removed, significantly changes the fitted line.
  Measured by **Cook's Distance**: `Dᵢ > 1` is typically influential.

```
    y │
      │  ●                     ← Outlier (large residual)
      │        ╱
      │      ╱ ● ●
      │    ╱ ● ●
      │  ╱ ● ●
      │╱ ●                                  ●  ← High leverage + large
      ├───────────────────────────────── x       residual = INFLUENTIAL
```

---

## 5. Evaluating the Model – R² Score

### 5.1 Sum of Squares Decomposition

```
  SST  =  SSR  +  SSE
  (Total)  (Regression)  (Error)

  SST = Σ(yᵢ − ȳ)²         Total variation in y
  SSR = Σ(ŷᵢ − ȳ)²         Variation explained by the model
  SSE = Σ(yᵢ − ŷᵢ)²        Variation NOT explained (residual)
```

### 5.2 R² (Coefficient of Determination)

```
              SSE            Σ(yᵢ − ŷᵢ)²
  R²  =  1 − ───  =  1  −  ─────────────
              SST            Σ(yᵢ − ȳ)²
```

| R² Value | Interpretation |
|----------|----------------|
| `1.0` | Perfect fit — model explains 100% of variance |
| `0.8 – 0.99` | Strong fit |
| `0.5 – 0.8` | Moderate fit |
| `0.0 – 0.5` | Weak fit |
| `< 0` | Model is worse than predicting the mean (possible with test data) |

### 5.3 Relationship with Correlation

For **simple** linear regression only:

```
  R² = r²     (where r = Pearson correlation coefficient)
```

So if `r = 0.9`, then `R² = 0.81`, meaning the model explains 81% of variance.

---

## 6. Complete Worked Example

### 6.1 Problem Statement

A tutoring company wants to predict a student's **exam score** based on
**hours studied**. Here is data from 8 students:

| Student | Hours (x) | Score (y) |
|---------|-----------|-----------|
| 1 | 1.0 | 35 |
| 2 | 2.0 | 45 |
| 3 | 3.0 | 55 |
| 4 | 4.0 | 60 |
| 5 | 5.0 | 68 |
| 6 | 6.0 | 72 |
| 7 | 7.0 | 80 |
| 8 | 8.0 | 88 |

### 6.2 Step-by-Step Calculation

**Step 1: Compute means**

```
x̄ = (1+2+3+4+5+6+7+8) / 8 = 36 / 8 = 4.5
ȳ = (35+45+55+60+68+72+80+88) / 8 = 503 / 8 = 62.875
```

**Step 2: Compute Σ(xᵢ − x̄)(yᵢ − ȳ) and Σ(xᵢ − x̄)²**

| xᵢ | yᵢ | xᵢ−x̄ | yᵢ−ȳ | (xᵢ−x̄)(yᵢ−ȳ) | (xᵢ−x̄)² |
|-----|-----|--------|--------|-----------------|-----------|
| 1.0 | 35 | −3.5 | −27.875 | 97.5625 | 12.25 |
| 2.0 | 45 | −2.5 | −17.875 | 44.6875 | 6.25 |
| 3.0 | 55 | −1.5 | −7.875 | 11.8125 | 2.25 |
| 4.0 | 60 | −0.5 | −2.875 | 1.4375 | 0.25 |
| 5.0 | 68 | 0.5 | 5.125 | 2.5625 | 0.25 |
| 6.0 | 72 | 1.5 | 9.125 | 13.6875 | 2.25 |
| 7.0 | 80 | 2.5 | 17.125 | 42.8125 | 6.25 |
| 8.0 | 88 | 3.5 | 25.125 | 87.9375 | 12.25 |
| **Σ** | | | | **302.5** | **42.0** |

**Step 3: Compute slope `m`**

```
m = 302.5 / 42.0 = 7.2024
```

**Step 4: Compute intercept `b`**

```
b = ȳ − m·x̄ = 62.875 − 7.2024 × 4.5 = 62.875 − 32.411 = 30.464
```

**Step 5: Final model**

```
ŷ = 7.2024x + 30.464
```

**Interpretation:** For every additional hour studied, the exam score increases by
approximately **7.2 points**. A student who studies 0 hours is predicted to score ~30.5.

**Step 6: Compute R²**

```
SST = Σ(yᵢ − ȳ)² = 27.875² + 17.875² + ... = 2178.875
SSE = Σ(yᵢ − ŷᵢ)² = (35 − 37.67)² + (45 − 44.87)² + ... ≈ 0.5 × remaining
R²  = 1 − SSE/SST ≈ 0.982
```

The model explains **~98.2%** of the variance in exam scores — an excellent fit.

### 6.3 Visualization (ASCII)

```
  Score
  90 │                              ●
     │                         ╱
  80 │                    ● ╱
     │                  ╱
  70 │             ● ╱
     │           ╱
  60 │       ● ╱
     │     ╱
  50 │   ╱  ●
     │ ╱
  40 │╱ ●
     │
  30 ●
     │
     └──┬──┬──┬──┬──┬──┬──┬──┬── Hours
        1  2  3  4  5  6  7  8
```

---

## 7. Python Implementation

### 7.1 From Scratch (NumPy Only)

```python
import numpy as np

# Data
x = np.array([1, 2, 3, 4, 5, 6, 7, 8], dtype=float)
y = np.array([35, 45, 55, 60, 68, 72, 80, 88], dtype=float)

# Step 1: Means
x_mean = np.mean(x)
y_mean = np.mean(y)

# Step 2: Slope (m) and Intercept (b)
numerator = np.sum((x - x_mean) * (y - y_mean))
denominator = np.sum((x - x_mean) ** 2)
m = numerator / denominator
b = y_mean - m * x_mean

print(f"Slope (m): {m:.4f}")
print(f"Intercept (b): {b:.4f}")
print(f"Equation: ŷ = {m:.4f}x + {b:.4f}")

# Step 3: Predictions
y_pred = m * x + b

# Step 4: R² score
ss_total = np.sum((y - y_mean) ** 2)
ss_residual = np.sum((y - y_pred) ** 2)
r_squared = 1 - (ss_residual / ss_total)
print(f"R² Score: {r_squared:.4f}")

# Step 5: Predict for a new value
hours_new = 5.5
score_pred = m * hours_new + b
print(f"Predicted score for {hours_new} hours: {score_pred:.2f}")
```

**Output:**
```
Slope (m): 7.2024
Intercept (b): 30.4643
Equation: ŷ = 7.2024x + 30.4643
R² Score: 0.9867
Predicted score for 5.5 hours: 70.08
```

### 7.2 Using scikit-learn

```python
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.metrics import r2_score, mean_squared_error

# Data — sklearn expects 2D array for X
X = np.array([1, 2, 3, 4, 5, 6, 7, 8]).reshape(-1, 1)
y = np.array([35, 45, 55, 60, 68, 72, 80, 88])

# Fit model
model = LinearRegression()
model.fit(X, y)

# Coefficients
print(f"Slope (coef_): {model.coef_[0]:.4f}")
print(f"Intercept (intercept_): {model.intercept_:.4f}")

# Predictions
y_pred = model.predict(X)

# Metrics
print(f"R² Score: {r2_score(y, y_pred):.4f}")
print(f"MSE: {mean_squared_error(y, y_pred):.4f}")
print(f"RMSE: {np.sqrt(mean_squared_error(y, y_pred)):.4f}")

# Predict new value
new_hours = np.array([[5.5]])
print(f"Predicted score for 5.5 hours: {model.predict(new_hours)[0]:.2f}")
```

### 7.3 Residual Analysis in Python

```python
import numpy as np

# After fitting the model
residuals = y - y_pred

print("Residual Analysis")
print(f"  Mean of residuals: {np.mean(residuals):.6f}")   # Should be ~0
print(f"  Std of residuals:  {np.std(residuals):.4f}")

# Check for patterns
for i in range(len(x)):
    bar = "+" * int(abs(residuals[i]) * 2) if residuals[i] > 0 else "-" * int(abs(residuals[i]) * 2)
    print(f"  x={x[i]:.0f}  residual={residuals[i]:+.2f}  {bar}")
```

---

## 8. Real-World Applications

| Domain | Feature (x) | Target (y) | Example |
|--------|-------------|-------------|---------|
| **Real Estate** | Square footage | House price | Predicting home values |
| **Marketing** | Ad spend ($) | Revenue ($) | ROI estimation |
| **Healthcare** | Dosage (mg) | Blood pressure reduction | Drug effectiveness |
| **Agriculture** | Rainfall (mm) | Crop yield (tons) | Harvest planning |
| **Education** | Study hours | Exam score | Student performance prediction |
| **Physics** | Force applied | Displacement | Hooke's law (spring constant) |
| **Economics** | GDP per capita | Life expectancy | Development indicators |

---

## 9. Summary Table

| Concept | Key Point |
|---------|-----------|
| **Model** | `ŷ = mx + b` (one feature, one target) |
| **Objective** | Minimize SSE = Σ(yᵢ − ŷᵢ)² |
| **Slope formula** | `m = Cov(x,y) / Var(x)` |
| **Intercept formula** | `b = ȳ − mx̄` |
| **R² Score** | Proportion of variance explained; 1 = perfect |
| **Assumptions** | Linearity, Independence, Homoscedasticity, Normality |
| **Residual** | `eᵢ = yᵢ − ŷᵢ`; should be random, centered at 0 |
| **Influential points** | Detected via Cook's Distance (Dᵢ > 1) |
| **sklearn class** | `LinearRegression()` from `sklearn.linear_model` |

---

## 10. ✏️ Revision Questions

1. **Derive the formula** for slope `m` using the Least Squares method. Show each
   step of taking the partial derivative of SSR with respect to `m`.

2. **A dataset** has `r = −0.85`. What is R²? What does the negative correlation tell
   you about the direction of the relationship?

3. **List all four assumptions** of linear regression. For each, describe one diagnostic
   tool/test to check whether it is violated.

4. **Explain the difference** between an outlier, a leverage point, and an influential
   point. Can a data point be high-leverage but not influential?

5. **Given the model** `ŷ = 3.5x + 12`, predict ŷ for x = 10. If the actual y is 50,
   compute the residual. Is the model over- or under-predicting?

6. **In your own words**, explain why the Least Squares method uses squared errors
   instead of absolute errors. What mathematical advantage does squaring provide?

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| — | [02 – Multiple Linear Regression →](02-multiple-linear-regression.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
