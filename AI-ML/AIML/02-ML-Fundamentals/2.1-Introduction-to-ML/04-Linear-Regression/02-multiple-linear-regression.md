# 📘 Chapter 2: Multiple Linear Regression

> **Unit 2.1 – Introduction to Machine Learning / 04 – Linear Regression** | File 2 of 7

---

## 📋 Overview

Multiple Linear Regression (MLR) extends simple linear regression to **multiple input
features**. Instead of fitting a line in 2D, we fit a **hyperplane** in n-dimensional space.
This chapter covers the matrix formulation, coefficient interpretation, the multicollinearity
problem, Variance Inflation Factor (VIF), adjusted R², feature significance testing, and
a complete Python example with real-world dataset interpretation.

---

## 1. The Model Equation

### 1.1 Scalar Form

```
ŷ = w₁x₁ + w₂x₂ + w₃x₃ + ... + wₙxₙ + b
```

| Symbol | Name | Meaning |
|--------|------|---------|
| `ŷ` | Predicted value | Model output |
| `x₁..xₙ` | Features | n independent variables |
| `w₁..wₙ` | Weights (coefficients) | Effect of each feature on ŷ |
| `b` | Bias (intercept) | Value of ŷ when all features are 0 |

### 1.2 Geometric Interpretation

```
  Simple LR (1 feature):         Multiple LR (2 features):
  Fit a LINE in 2D               Fit a PLANE in 3D

  y │    ╱                        y  ╱│
    │  ╱                            ╱ │
    │╱                             ╱  │
    └───── x                      ╱ ──┤
                                 ╱    │
                                ╱─────┘──── x₂
                               x₁

  With 3+ features → hyperplane in n+1 dimensions (not visualizable)
```

---

## 2. Matrix Formulation

### 2.1 Design Matrix

We encode all data into matrices for efficient computation.

Given `m` samples and `n` features:

```
        ┌  1   x₁₁  x₁₂  ...  x₁ₙ  ┐       ┌ y₁ ┐       ┌ b  ┐
        │  1   x₂₁  x₂₂  ...  x₂ₙ  │       │ y₂ │       │ w₁ │
  X  =  │  1   x₃₁  x₃₂  ...  x₃ₙ  │  y =  │ y₃ │  w =  │ w₂ │
        │  ⋮    ⋮     ⋮    ⋱    ⋮   │       │ ⋮  │       │ ⋮  │
        └  1   xₘ₁  xₘ₂  ...  xₘₙ  ┘       └ yₘ ┘       └ wₙ ┘

  X: (m × (n+1))     y: (m × 1)     w: ((n+1) × 1)
```

> The column of 1s absorbs the intercept `b` into the weight vector `w`.

### 2.2 Compact Matrix Equation

```
  ŷ = Xw

  where:
    X  is the  m × (n+1)  design matrix (with bias column)
    w  is the  (n+1) × 1  parameter vector
    ŷ  is the  m × 1      prediction vector
```

### 2.3 Normal Equation (Closed-Form Solution)

```
  w* = (XᵀX)⁻¹ Xᵀy
```

This gives the optimal weight vector that minimizes the sum of squared errors.
(Detailed derivation in Chapter 5.)

---

## 3. Interpreting Coefficients

### 3.1 Meaning of Each Weight

```
  ŷ = 50000 + 8500·x₁ + 3200·x₂ − 1200·x₃
       ↑       ↑          ↑          ↑
       base    sq_ft      bedrooms   age_years
```

**Interpretation (holding all other features constant):**

| Coefficient | Value | Interpretation |
|-------------|-------|----------------|
| `b` (intercept) | 50,000 | Base price when all features = 0 |
| `w₁` (sq_ft) | +8,500 | Each extra sq ft adds $8,500 to price |
| `w₂` (bedrooms) | +3,200 | Each extra bedroom adds $3,200 |
| `w₃` (age_years) | −1,200 | Each extra year of age reduces price by $1,200 |

> ⚠️ **Caution:** Coefficients are only directly comparable if features are on the
> same scale. Use **standardized coefficients** (z-score scaled features) for fair
> comparison of feature importance.

### 3.2 Standardized Coefficients

```
  Standardized weight:  w_std_j = wⱼ × (σ_xj / σ_y)

  where σ_xj = std dev of feature j
        σ_y  = std dev of target
```

A larger |w_std_j| means feature j has a **stronger effect** on the target.

---

## 4. Multicollinearity

### 4.1 What Is It?

Multicollinearity occurs when two or more features are **highly correlated** with each other.

```
  Example:   x₁ = house area (sq ft)
             x₂ = number of rooms

  These are correlated: bigger houses tend to have more rooms.
  The model can't reliably separate their individual effects.
```

### 4.2 Problems Caused

```
┌───────────────────────────────────────────────────────┐
│  CONSEQUENCES OF MULTICOLLINEARITY                    │
├───────────────────────────────────────────────────────┤
│                                                       │
│  1. Unstable coefficients                             │
│     → Small data changes cause large weight swings    │
│                                                       │
│  2. Inflated standard errors                          │
│     → Coefficients appear statistically insignificant │
│       even when the feature matters                   │
│                                                       │
│  3. Sign flips                                        │
│     → A coefficient may become negative when the      │
│       true relationship is positive                   │
│                                                       │
│  4. XᵀX becomes nearly singular                       │
│     → Inversion is numerically unstable               │
│                                                       │
│  NOTE: Predictions may still be accurate!             │
│  Multicollinearity hurts INTERPRETATION, not          │
│  necessarily prediction accuracy.                     │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### 4.3 Detection: Variance Inflation Factor (VIF)

VIF measures how much the variance of a coefficient is inflated due to collinearity.

```
                    1
  VIF_j  =  ─────────────
              1 − R²_j

  where R²_j = R² from regressing feature j on all OTHER features
```

| VIF Value | Interpretation |
|-----------|----------------|
| 1 | No collinearity |
| 1 – 5 | Moderate (usually acceptable) |
| 5 – 10 | High (investigate further) |
| > 10 | Severe (take action) |

### 4.4 Solutions

| Strategy | How |
|----------|-----|
| **Drop a feature** | Remove one of the correlated pair |
| **Combine features** | PCA or create a composite feature |
| **Regularization** | Ridge regression handles collinearity well |
| **Increase data** | More samples can stabilize estimates |

---

## 5. Assumptions of Multiple Linear Regression

All four assumptions from simple LR apply, plus one more:

| # | Assumption | Description |
|---|-----------|-------------|
| 1 | **Linearity** | y is a linear function of each feature |
| 2 | **Independence** | Residuals are independent |
| 3 | **Homoscedasticity** | Constant variance of residuals |
| 4 | **Normality** | Residuals are normally distributed |
| 5 | **No multicollinearity** | Features are not highly correlated with each other |

---

## 6. Adjusted R²

### 6.1 The Problem with R²

Adding **any** feature to the model will never decrease R² — even a random noise feature.
This makes R² misleading when comparing models with different numbers of features.

### 6.2 Adjusted R² Formula

```
                         (1 − R²)(m − 1)
  Adjusted R²  =  1  −  ─────────────────
                           m − n − 1

  where:
    m = number of samples
    n = number of features
    R² = regular R-squared
```

### 6.3 Key Differences

| Metric | R² | Adjusted R² |
|--------|-----|------------|
| Always increases with more features? | ✅ Yes | ❌ No — penalizes useless features |
| Can be negative? | Only on test data | ✅ Yes — if model is poor |
| Use for model comparison? | ❌ Not reliable | ✅ Preferred |
| Accounts for complexity? | ❌ No | ✅ Yes |

### 6.4 Example

```
  Model A: 3 features, R² = 0.85, Adjusted R² = 0.83
  Model B: 7 features, R² = 0.87, Adjusted R² = 0.80

  → Model A is better despite lower R², because Model B's
    extra features are not contributing meaningful information.
```

---

## 7. Feature Significance Testing

### 7.1 t-test for Individual Coefficients

For each coefficient `wⱼ`, we test:

```
  H₀: wⱼ = 0   (feature j has no effect on y)
  H₁: wⱼ ≠ 0   (feature j has a significant effect)

  t-statistic = wⱼ / SE(wⱼ)
  p-value     = P(|t| > |t_observed|)
```

| p-value | Conclusion |
|---------|------------|
| < 0.01 | Highly significant *** |
| 0.01 – 0.05 | Significant ** |
| 0.05 – 0.10 | Marginally significant * |
| > 0.10 | Not significant (consider dropping) |

### 7.2 F-test for Overall Model

```
  H₀: All coefficients = 0 (model is useless)
  H₁: At least one coefficient ≠ 0

  F = (SSR / n) / (SSE / (m − n − 1))
```

A large F-statistic (low p-value) means the model as a whole is significant.

---

## 8. Complete Python Example

### 8.1 Dataset: Predicting House Prices

```python
import numpy as np
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import r2_score, mean_squared_error
from sklearn.preprocessing import StandardScaler

# Create a synthetic housing dataset
np.random.seed(42)
m = 200  # samples

sq_ft      = np.random.uniform(800, 3500, m)
bedrooms   = np.random.randint(1, 6, m)
age_years  = np.random.uniform(0, 50, m)
dist_city  = np.random.uniform(1, 30, m)

# True relationship (with some noise)
price = (50000
         + 85 * sq_ft
         + 5000 * bedrooms
         - 1000 * age_years
         - 2000 * dist_city
         + np.random.normal(0, 15000, m))

# Create DataFrame
df = pd.DataFrame({
    'sq_ft': sq_ft,
    'bedrooms': bedrooms,
    'age_years': age_years,
    'dist_city_km': dist_city,
    'price': price
})

print(df.describe().round(2))
```

### 8.2 Train-Test Split and Model Fitting

```python
# Features and target
X = df[['sq_ft', 'bedrooms', 'age_years', 'dist_city_km']]
y = df['price']

# Split 80/20
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Fit model
model = LinearRegression()
model.fit(X_train, y_train)

# Results
print("Coefficients:")
for name, coef in zip(X.columns, model.coef_):
    print(f"  {name:>15s}: {coef:>10.2f}")
print(f"  {'intercept':>15s}: {model.intercept_:>10.2f}")

# Predictions
y_pred = model.predict(X_test)
print(f"\nR² (test):   {r2_score(y_test, y_pred):.4f}")
print(f"RMSE (test): {np.sqrt(mean_squared_error(y_test, y_pred)):.2f}")
```

**Expected Output (approximate):**
```
Coefficients:
          sq_ft:      85.23
       bedrooms:    5124.67
      age_years:   -1012.45
   dist_city_km:   -2034.89
      intercept:   49832.15

R² (test):   0.9512
RMSE (test): 15234.67
```

### 8.3 Checking VIF for Multicollinearity

```python
from sklearn.linear_model import LinearRegression

def compute_vif(X):
    """Compute VIF for each feature in DataFrame X."""
    vif_data = []
    for i, col in enumerate(X.columns):
        # Regress feature i on all other features
        other_cols = [c for c in X.columns if c != col]
        reg = LinearRegression()
        reg.fit(X[other_cols], X[col])
        r_squared = reg.score(X[other_cols], X[col])
        vif = 1 / (1 - r_squared) if r_squared < 1 else float('inf')
        vif_data.append({'Feature': col, 'VIF': round(vif, 2)})
    return pd.DataFrame(vif_data)

vif_df = compute_vif(X_train)
print("\nVariance Inflation Factors:")
print(vif_df.to_string(index=False))
```

### 8.4 Standardized Coefficients for Feature Importance

```python
# Scale features to z-scores
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# Fit on scaled data
model_scaled = LinearRegression()
model_scaled.fit(X_train_scaled, y_train)

print("\nStandardized Coefficients (feature importance):")
for name, coef in sorted(
    zip(X.columns, model_scaled.coef_),
    key=lambda x: abs(x[1]),
    reverse=True
):
    print(f"  {name:>15s}: {coef:>10.2f}")
```

### 8.5 Using statsmodels for Detailed Statistics

```python
import statsmodels.api as sm

# Add intercept column
X_train_sm = sm.add_constant(X_train)

# Fit OLS model
ols_model = sm.OLS(y_train, X_train_sm).fit()

# Print full summary (includes p-values, t-stats, confidence intervals)
print(ols_model.summary())
```

**The summary includes:**
- Coefficients with standard errors, t-statistics, and p-values
- R², Adjusted R², F-statistic
- Confidence intervals for each coefficient
- Durbin-Watson statistic (tests autocorrelation)

---

## 9. From-Scratch Implementation (NumPy)

```python
import numpy as np

class MultipleLinearRegression:
    """Multiple Linear Regression using Normal Equation."""

    def __init__(self):
        self.weights = None

    def fit(self, X, y):
        m = X.shape[0]
        # Add bias column of 1s
        X_b = np.column_stack([np.ones(m), X])
        # Normal equation: w = (XᵀX)⁻¹ Xᵀy
        self.weights = np.linalg.pinv(X_b.T @ X_b) @ X_b.T @ y
        return self

    def predict(self, X):
        m = X.shape[0]
        X_b = np.column_stack([np.ones(m), X])
        return X_b @ self.weights

    def score(self, X, y):
        y_pred = self.predict(X)
        ss_res = np.sum((y - y_pred) ** 2)
        ss_tot = np.sum((y - np.mean(y)) ** 2)
        return 1 - ss_res / ss_tot

    @property
    def intercept(self):
        return self.weights[0]

    @property
    def coefficients(self):
        return self.weights[1:]

# Usage
model_scratch = MultipleLinearRegression()
model_scratch.fit(X_train.values, y_train.values)
print(f"R² (scratch): {model_scratch.score(X_test.values, y_test.values):.4f}")
print(f"Intercept: {model_scratch.intercept:.2f}")
print(f"Coefficients: {model_scratch.coefficients}")
```

---

## 10. Real-World Applications

| Domain | Features | Target | Notes |
|--------|----------|--------|-------|
| **Real Estate** | Area, rooms, location, age | Price | Classic MLR application |
| **Salary Prediction** | Experience, education, location | Salary | HR analytics |
| **Advertising** | TV, radio, newspaper spend | Sales | Budget allocation |
| **Medical** | Age, BMI, blood pressure, glucose | Insurance cost | Risk modeling |
| **Manufacturing** | Temperature, pressure, humidity | Defect rate | Quality control |
| **Finance** | Market cap, P/E ratio, debt ratio | Stock return | Factor models |

---

## 11. Summary Table

| Concept | Key Point |
|---------|-----------|
| **Model** | `ŷ = Xw + b` (n features, one target) |
| **Matrix form** | `ŷ = Xw` with bias column in X |
| **Normal equation** | `w* = (XᵀX)⁻¹Xᵀy` |
| **Coefficient interpretation** | Effect of feature j on y, holding others constant |
| **Multicollinearity** | Correlated features → unstable coefficients |
| **VIF** | `1/(1−R²ⱼ)` — values > 10 indicate severe collinearity |
| **Adjusted R²** | Penalizes adding useless features |
| **Feature significance** | t-test per coefficient; F-test for overall model |
| **Standardized coefficients** | Scale-independent measure of feature importance |

---

## 12. ✏️ Revision Questions

1. **Write the matrix formulation** of multiple linear regression. What are the
   dimensions of X, w, and ŷ for a dataset with 500 samples and 8 features?

2. **A model has coefficients** `sq_ft: +120` and `bedrooms: +15000`. Can you
   conclude that bedrooms is more important? Why or why not?

3. **Explain VIF.** If feature A has VIF = 12, what does that mean? What action
   would you take?

4. **Model A** has R² = 0.92 with 3 features. **Model B** has R² = 0.93 with 10
   features. Which model would you prefer and why? Which metric would you use?

5. **What is the difference** between R² and Adjusted R²? In what scenario can
   Adjusted R² decrease when adding a new feature?

6. **Using statsmodels**, you find that the p-value for `age_years` is 0.45.
   What does this mean? Should you keep or drop this feature?

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| [← 01 – Simple Linear Regression](01-simple-linear-regression.md) | [03 – Cost Function (MSE) →](03-cost-function-mse.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
