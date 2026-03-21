# 📘 Chapter 7: Regularization — Ridge, Lasso & Elastic Net

> **Unit 2.1 – Introduction to Machine Learning / 04 – Linear Regression** | File 7 of 7

---

## 📋 Overview

Regularization is a technique to **prevent overfitting** by adding a penalty term to the cost
function that discourages large weights. This chapter covers the motivation behind
regularization, **Ridge regression** (L2 penalty), **Lasso regression** (L1 penalty),
**Elastic Net** (L1 + L2 combination), their effects on coefficients, shrinkage
visualization, how to choose the regularization strength λ (alpha), cross-validation
strategies, Lasso's feature selection ability, comprehensive comparison, and sklearn
implementations with `RidgeCV` and `LassoCV`.

---

## 1. Why Regularization?

### 1.1 The Overfitting Problem

```
┌─────────────────────────────────────────────────────────────────┐
│  SCENARIO: You fit a high-degree polynomial or use many features│
│                                                                 │
│  Without regularization:                                        │
│  • Model fits the training data PERFECTLY                       │
│  • Weights become HUGE (e.g., w₃ = 50000, w₄ = −80000)         │
│  • Model oscillates wildly between data points                  │
│  • Terrible predictions on new data (high variance)             │
│                                                                 │
│  With regularization:                                           │
│  • Penalty term prevents weights from growing too large         │
│  • Model is smoother, more generalizable                        │
│  • Slight increase in training error                            │
│  • Significant decrease in test error                           │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Visual: Effect of Regularization

```
  Without Regularization          With Regularization
  (Degree 10 poly)                (Degree 10 poly + Ridge)

  y │ ●╲  ╱●╲ ╱●                  y │        ●●●
    │   ╳    ╳  ●                   │      ●     ●
    │  ╱ ╲  ╱ ╲                     │    ●         ●
    │ ╱   ╲╱                        │   ●           ●
    │╱      ●                       │  ●             ●
    │                               │ ●               ●
    └───────── x                    └───────────────── x

  Wiggly, overfits 😱               Smooth, generalizes 😊
  Weights: [-5000, 8000, ...]       Weights: [-0.5, 2.1, ...]
```

### 1.3 The Core Idea

```
  Standard cost:       J(w) = (1/2m) Σ(ŷᵢ − yᵢ)²

  Regularized cost:    J(w) = (1/2m) Σ(ŷᵢ − yᵢ)²  +  λ · Penalty(w)
                               ↑                        ↑
                          Data fidelity              Complexity penalty
                          "Fit the data"             "Keep weights small"
```

The hyperparameter `λ` (lambda, called `alpha` in sklearn) controls the balance:

| λ Value | Effect |
|---------|--------|
| `λ = 0` | No regularization → standard linear regression |
| `λ small` | Slight regularization → minor weight shrinkage |
| `λ optimal` | Right balance → best generalization |
| `λ very large` | Over-regularization → all weights → 0 → underfitting |

---

## 2. Ridge Regression (L2 Regularization)

### 2.1 Cost Function

```
                                                        n
  J_Ridge(w) = (1/2m) Σ(ŷᵢ − yᵢ)²  +  λ  Σ  wⱼ²
                                                       j=1

                MSE term                 L2 penalty
                                     (sum of SQUARED weights)
```

> **Note:** The intercept `b` (or `w₀`) is typically **NOT** regularized.

### 2.2 Effect on Weights

```
  Ridge pushes weights TOWARD zero but never EXACTLY to zero.

  Before Ridge:  w = [15.2,  −23.8,  0.3,  42.1,  −0.01]
  After Ridge:   w = [ 3.1,   −5.2,  0.2,   8.3,  −0.01]
                       ↓       ↓      ↓      ↓       ↓
                    Shrunk  Shrunk  Barely  Shrunk  Barely
                                   changed          changed

  Large weights are shrunk more aggressively.
  Small weights are barely affected.
```

### 2.3 Closed-Form Solution

```
  w*_Ridge = (XᵀX + λI)⁻¹ Xᵀy

  where I is the (n+1) × (n+1) identity matrix
  (with I₀₀ = 0 to exclude the intercept from regularization)
```

> Adding `λI` to `XᵀX` makes it **always invertible** — even when XᵀX is singular!
> This is why Ridge also solves multicollinearity problems.

### 2.4 Geometric Interpretation

```
  Minimize MSE subject to: Σwⱼ² ≤ t   (for some threshold t)

  The L2 constraint region is a CIRCLE (or hypersphere).

  w₂                               w₂
  │       MSE contours              │       Constraint region
  │    ╭────────╮                   │        ╱ ╲
  │  ╭─┤  ╭──╮ ├─╮                 │      ╱     ╲
  │  │ │  │★ │ │ │                 │    ╱    ●    ╲  ← Ridge solution
  │  │ │  ╰──╯ │ │                 │   │           │   where circle meets
  │  ╰─┤       ├─╯                 │    ╲         ╱   lowest contour
  │    ╰────────╯                   │      ╲     ╱
  └───────────────── w₁             │        ╲ ╱
                                    └────────────── w₁

  ★ = OLS solution (no constraint)
  ● = Ridge solution (constrained to circle)
```

---

## 3. Lasso Regression (L1 Regularization)

### 3.1 Cost Function

```
                                                        n
  J_Lasso(w) = (1/2m) Σ(ŷᵢ − yᵢ)²  +  λ  Σ  |wⱼ|
                                                       j=1

                MSE term                 L1 penalty
                                     (sum of ABSOLUTE weights)
```

### 3.2 Key Difference from Ridge: Feature Selection

```
  Lasso can push weights EXACTLY to zero!

  Before Lasso:  w = [15.2,  −23.8,  0.3,  42.1,  −0.01]
  After Lasso:   w = [ 2.5,   −4.1,  0.0,   7.8,   0.0 ]
                                      ↑              ↑
                                   EXACTLY          EXACTLY
                                    ZERO             ZERO

  Features with w = 0 are effectively REMOVED from the model.
  → Lasso performs AUTOMATIC FEATURE SELECTION! 🎯
```

### 3.3 Why Does Lasso Produce Zeros?

```
  The L1 constraint region is a DIAMOND (or cross-polytope).

  w₂                               w₂
  │       MSE contours              │         ╱╲
  │    ╭────────╮                   │       ╱    ╲
  │  ╭─┤  ╭──╮ ├─╮                 │     ╱        ╲
  │  │ │  │★ │ │ │                 │   ●            ╲ ← Lasso solution
  │  │ │  ╰──╯ │ │                 │     ╲          ╱   often at a CORNER
  │  ╰─┤       ├─╯                 │       ╲      ╱    of the diamond
  │    ╰────────╯                   │         ╲  ╱      → w₂ = 0!
  └───────────────── w₁             │          ╲╱
                                    └────────────── w₁

  Diamond corners are on axes → one or more weights = 0.
  Elliptical MSE contours are more likely to first touch a CORNER
  of the diamond than a flat edge → sparsity!
```

### 3.4 No Closed-Form Solution

Unlike Ridge, Lasso has **no analytical solution** because the absolute value `|wⱼ|`
is not differentiable at zero. It must be solved using:

- **Coordinate Descent** (most common, used by sklearn)
- **Subgradient methods**
- **LARS (Least Angle Regression)**

---

## 4. Elastic Net (L1 + L2 Combination)

### 4.1 Cost Function

```
                                                     n              n
  J_EN(w) = (1/2m) Σ(ŷᵢ − yᵢ)²  +  λ [ r Σ |wⱼ| + (1−r) Σ wⱼ² ]
                                                    j=1            j=1

                MSE term               L1 part        L2 part
```

| Parameter | Meaning |
|-----------|---------|
| `λ` (alpha) | Overall regularization strength |
| `r` (l1_ratio) | Mix between L1 and L2 (0 = pure Ridge, 1 = pure Lasso) |

### 4.2 When to Use Elastic Net

```
┌──────────────────────────────────────────────────────────────┐
│  Use Elastic Net when:                                       │
│                                                               │
│  1. You have MANY correlated features                         │
│     → Lasso randomly picks one; Elastic Net keeps both        │
│                                                               │
│  2. n >> m (more features than samples)                       │
│     → Lasso selects at most m features; Elastic Net can       │
│       select more                                             │
│                                                               │
│  3. You want feature selection (L1) + stability (L2)          │
│     → Best of both worlds                                     │
│                                                               │
│  4. As a default starting point for regularized regression    │
│     → l1_ratio=0.5 is a reasonable default                    │
└──────────────────────────────────────────────────────────────┘
```

### 4.3 Constraint Region Shape

```
  L2 (Ridge): Circle      L1 (Lasso): Diamond    Elastic Net: Rounded Diamond

  w₂    ╭──╮               w₂    /╲                w₂   ╭─╲
  │    ╱    ╲               │   /    ╲               │  ╱    ╲
  │  ╱        ╲             │ /        ╲             │╱        ╲
  │ │          │            │/          ╲            ●          │
  │  ╲        ╱             │╲          ╱            │╲        ╱
  │    ╲    ╱               │  ╲      ╱              │  ╲    ╱
  │      ╰──╯               │    ╲  ╱                │    ╰─╱
  └──────── w₁              └──────── w₁             └──────── w₁

  Never zeros             Produces zeros            Sometimes zeros
  Smooth everywhere       Corners on axes           Rounded corners
```

---

## 5. Effect on Coefficients — Shrinkage Visualization

### 5.1 Regularization Path

As `λ` increases, coefficients shrink:

```
  Coefficient
  Value
    │
  +5│──●─────────────                    ← Feature A (important)
    │    ╲
  +3│     ╲─────────────────             ← Feature B (moderate)
    │       ╲
  +1│        ╲───────●                   ← Feature C (weak)
  ──│──────────╲──────────── λ →
  −1│           ╲                        ← Feature D (weak, negative)
    │            ╲────────────
  −3│             ╲──────────────        ← Feature E
    │              ●
  −5│               ──────────────
    │
    └──┬──┬──┬──┬──┬──┬──┬──┬──
      0.001 0.01 0.1  1  10 100 1000
                  λ (log scale)

  At λ=0: OLS solution (no shrinkage)
  As λ↑: Weights shrink toward zero
  Ridge: weights approach 0 but never reach it
  Lasso: weights hit exactly 0 (feature selection!)
```

### 5.2 Ridge vs Lasso Path Comparison

```
  RIDGE PATH                          LASSO PATH

  w │                                 w │
    │──●                                │──●
    │    ╲                              │    ╲
    │     ╲                             │     ╲
    │──●   ╲──────→ 0 (never reaches)   │──●   ╲───→ 0 (REACHES!)
    │    ╲  ╲                           │    ╲  ╲
    │     ╲  ╲                          │     ╲──╲──→ 0 ← ZERO!
    │      ╲  ╲                         │         ╲
    │       ╲──╲─── → 0                │          ╲──→ 0
    └──────────── λ                     └──────────── λ
    All weights stay non-zero           Some weights become EXACTLY zero
```

---

## 6. Choosing λ (Alpha)

### 6.1 Cross-Validation Approach

```
  Step 1: Define a grid of λ values
    λ_grid = [0.001, 0.01, 0.1, 1, 10, 100]

  Step 2: For each λ:
    - Perform k-fold cross-validation
    - Record average validation error

  Step 3: Pick the λ with lowest CV error

  CV Error
    │
    │ ●                                   ● ← too much regularization
    │  ╲                                ╱
    │   ╲                             ╱
    │    ╲                          ╱
    │     ╲        __●_●__        ╱
    │      ╲______╱       ╲_____╱
    │              ↑
    │         Best λ (lowest CV error)
    └──┬──┬──┬──┬──┬──┬──┬──── λ (log scale)
     .001 .01 .1  1  10  100
```

### 6.2 Common Practices

| Strategy | Details |
|----------|---------|
| **RidgeCV / LassoCV** | sklearn classes with built-in cross-validation |
| **GridSearchCV** | General-purpose; works with any estimator |
| **Log-spaced grid** | λ ∈ {10⁻⁴, 10⁻³, ..., 10³, 10⁴} |
| **1-SE rule** | Pick the largest λ within 1 standard error of the minimum CV error |

---

## 7. Feature Selection with Lasso

### 7.1 How It Works

```
  1. Fit Lasso with a range of λ values
  2. As λ increases, less important features get zeroed out FIRST
  3. The features that survive at higher λ values are the MOST important

  Feature importance ranking (by order of zeroing out):

  λ = 0.001:  All 10 features active
  λ = 0.01:   8 features active (2 removed)
  λ = 0.1:    5 features active (5 removed)
  λ = 1.0:    3 features active (7 removed)  ← These 3 are the key features!
  λ = 10:     1 feature active
  λ = 100:    0 features active (all zero → just predicts mean)
```

### 7.2 Lasso for Feature Selection Pipeline

```
  Step 1: LassoCV to find optimal λ
  Step 2: Fit Lasso with optimal λ
  Step 3: Identify features with non-zero coefficients
  Step 4: Retrain a standard model using only those features
```

---

## 8. Comprehensive Comparison

### 8.1 Ridge vs Lasso vs Elastic Net vs OLS

```
┌─────────────────────┬──────────┬──────────┬────────────┬──────────┐
│     Property        │   OLS    │  Ridge   │   Lasso    │Elastic Net│
├─────────────────────┼──────────┼──────────┼────────────┼──────────┤
│ Penalty             │ None     │ λΣwⱼ²   │ λΣ|wⱼ|    │λ[rΣ|wⱼ|  │
│                     │          │          │            │+(1-r)Σwⱼ²]│
│ Penalty type        │ —        │ L2       │ L1         │ L1 + L2  │
│ Constraint shape    │ —        │ Circle   │ Diamond    │ Rounded  │
│                     │          │          │            │ diamond  │
│ Feature selection   │ ❌       │ ❌       │ ✅ Yes     │ ✅ Yes   │
│ Handles collinearity│ ❌       │ ✅ Yes   │ ⚠️ Picks 1 │ ✅ Yes   │
│ Closed-form         │ ✅       │ ✅       │ ❌         │ ❌       │
│ Sparse solution     │ ❌       │ ❌       │ ✅         │ ✅       │
│ n > m possible      │ ❌       │ ✅       │ ⚠️(≤m feat)│ ✅       │
│ Bias introduced     │ None     │ Some     │ Some       │ Some     │
│ Variance reduced    │ None     │ ✅ Yes   │ ✅ Yes     │ ✅ Yes   │
│ sklearn class       │LinearReg │Ridge     │Lasso       │ElasticNet│
│ With CV             │ —        │RidgeCV   │LassoCV     │ElasticNet│
│                     │          │          │            │    CV    │
└─────────────────────┴──────────┴──────────┴────────────┴──────────┘
```

### 8.2 Decision Guide

```
  START
    │
    ▼
  Do you have many features?
    │
    ├── No, few features ──▶ Try OLS first
    │                         │
    │                         ▼
    │                       Overfitting?
    │                         │
    │                         ├── No  → Keep OLS ✅
    │                         └── Yes → Add Ridge ✅
    │
    └── Yes, many features
         │
         ▼
       Are features correlated?
         │
         ├── Yes ──▶ Ridge or Elastic Net ✅
         │           (Lasso randomly picks one of correlated pair)
         │
         └── No  ──▶ Want feature selection?
                       │
                       ├── Yes ──▶ Lasso ✅
                       └── No  ──▶ Ridge ✅
```

---

## 9. Python Implementation

### 9.1 Ridge Regression

```python
import numpy as np
from sklearn.linear_model import Ridge, RidgeCV
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import r2_score

# Generate data with some irrelevant features
np.random.seed(42)
m = 200
n_informative = 5
n_noise = 15
n_total = n_informative + n_noise

X = np.random.randn(m, n_total)
true_w = np.zeros(n_total)
true_w[:n_informative] = [3, -2, 1.5, -1, 0.5]  # only 5 features matter
y = X @ true_w + 5 + np.random.randn(m) * 2

# Split and scale
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# Ridge with manual alpha
ridge = Ridge(alpha=1.0)
ridge.fit(X_train_s, y_train)
print("Ridge (alpha=1.0):")
print(f"  R² (test): {ridge.score(X_test_s, y_test):.4f}")
print(f"  Non-zero coefs: {np.sum(np.abs(ridge.coef_) > 0.01)}")

# RidgeCV: automatic alpha selection
alphas = np.logspace(-4, 4, 50)
ridge_cv = RidgeCV(alphas=alphas, cv=5)
ridge_cv.fit(X_train_s, y_train)
print(f"\nRidgeCV:")
print(f"  Best alpha: {ridge_cv.alpha_:.4f}")
print(f"  R² (test): {ridge_cv.score(X_test_s, y_test):.4f}")
```

### 9.2 Lasso Regression

```python
from sklearn.linear_model import Lasso, LassoCV

# Lasso with manual alpha
lasso = Lasso(alpha=0.1, max_iter=10000)
lasso.fit(X_train_s, y_train)
print("Lasso (alpha=0.1):")
print(f"  R² (test): {lasso.score(X_test_s, y_test):.4f}")
print(f"  Non-zero coefs: {np.sum(lasso.coef_ != 0)} out of {n_total}")
print(f"  Zero coefs: {np.sum(lasso.coef_ == 0)}")

# LassoCV: automatic alpha selection
lasso_cv = LassoCV(alphas=np.logspace(-4, 2, 50), cv=5, max_iter=10000)
lasso_cv.fit(X_train_s, y_train)
print(f"\nLassoCV:")
print(f"  Best alpha: {lasso_cv.alpha_:.4f}")
print(f"  R² (test): {lasso_cv.score(X_test_s, y_test):.4f}")
print(f"  Non-zero coefs: {np.sum(lasso_cv.coef_ != 0)}")

# Show selected features
print(f"\n  Selected features (non-zero coefs):")
for i, coef in enumerate(lasso_cv.coef_):
    if coef != 0:
        marker = " ← informative" if i < n_informative else ""
        print(f"    Feature {i:>2d}: coef = {coef:>8.4f}{marker}")
```

### 9.3 Elastic Net

```python
from sklearn.linear_model import ElasticNet, ElasticNetCV

# ElasticNetCV
enet_cv = ElasticNetCV(
    l1_ratio=[0.1, 0.3, 0.5, 0.7, 0.9, 0.99],  # mix of L1/L2
    alphas=np.logspace(-4, 2, 50),
    cv=5,
    max_iter=10000
)
enet_cv.fit(X_train_s, y_train)

print("ElasticNetCV:")
print(f"  Best alpha: {enet_cv.alpha_:.4f}")
print(f"  Best l1_ratio: {enet_cv.l1_ratio_:.2f}")
print(f"  R² (test): {enet_cv.score(X_test_s, y_test):.4f}")
print(f"  Non-zero coefs: {np.sum(enet_cv.coef_ != 0)}")
```

### 9.4 Complete Comparison Script

```python
import numpy as np
from sklearn.linear_model import (
    LinearRegression, Ridge, RidgeCV,
    Lasso, LassoCV, ElasticNet, ElasticNetCV
)
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import r2_score, mean_squared_error

# Data (same as above)
np.random.seed(42)
m, n_total = 200, 20
X = np.random.randn(m, n_total)
true_w = np.zeros(n_total)
true_w[:5] = [3, -2, 1.5, -1, 0.5]
y = X @ true_w + 5 + np.random.randn(m) * 2

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# Fit all models
models = {
    'OLS':        LinearRegression(),
    'Ridge':      RidgeCV(alphas=np.logspace(-4, 4, 50), cv=5),
    'Lasso':      LassoCV(alphas=np.logspace(-4, 2, 50), cv=5, max_iter=10000),
    'ElasticNet': ElasticNetCV(l1_ratio=[0.1, 0.5, 0.9], cv=5, max_iter=10000),
}

print(f"{'Model':<12} {'R² (train)':<12} {'R² (test)':<12} {'RMSE (test)':<12} {'Non-zero':<10}")
print("-" * 58)

for name, model in models.items():
    model.fit(X_train_s, y_train)
    r2_train = model.score(X_train_s, y_train)
    r2_test = model.score(X_test_s, y_test)
    rmse = np.sqrt(mean_squared_error(y_test, model.predict(X_test_s)))
    n_nonzero = np.sum(np.abs(model.coef_) > 1e-10)

    print(f"{name:<12} {r2_train:<12.4f} {r2_test:<12.4f} {rmse:<12.4f} {n_nonzero:<10}")
```

### 9.5 Ridge from Scratch

```python
import numpy as np

class RidgeRegressionScratch:
    """Ridge Regression using the closed-form solution."""

    def __init__(self, alpha=1.0):
        self.alpha = alpha
        self.weights = None

    def fit(self, X, y):
        m, n = X.shape
        X_b = np.column_stack([np.ones(m), X])
        n_total = X_b.shape[1]

        # Regularization matrix (don't regularize the intercept)
        reg_matrix = self.alpha * np.eye(n_total)
        reg_matrix[0, 0] = 0  # no penalty on intercept

        # Closed-form: w = (XᵀX + λI)⁻¹ Xᵀy
        self.weights = np.linalg.solve(
            X_b.T @ X_b + reg_matrix,
            X_b.T @ y
        )
        return self

    def predict(self, X):
        X_b = np.column_stack([np.ones(X.shape[0]), X])
        return X_b @ self.weights

    def score(self, X, y):
        y_pred = self.predict(X)
        return 1 - np.sum((y - y_pred)**2) / np.sum((y - np.mean(y))**2)

    @property
    def intercept(self):
        return self.weights[0]

    @property
    def coef(self):
        return self.weights[1:]

# Test
ridge_scratch = RidgeRegressionScratch(alpha=1.0)
ridge_scratch.fit(X_train_s, y_train)
print(f"\nRidge from scratch R² (test): {ridge_scratch.score(X_test_s, y_test):.4f}")
```

### 9.6 Regularization Path Visualization

```python
import numpy as np
from sklearn.linear_model import Ridge

alphas = np.logspace(-2, 6, 100)
coefs = []

for a in alphas:
    ridge = Ridge(alpha=a)
    ridge.fit(X_train_s, y_train)
    coefs.append(ridge.coef_)

coefs = np.array(coefs)

# ASCII visualization of top 5 features' paths
print("\nRegularization Path (Ridge):")
print(f"{'alpha':<10}", end="")
for j in range(5):
    print(f"  {'w'+str(j):<8}", end="")
print()
print("-" * 55)

for idx in range(0, len(alphas), 15):
    print(f"{alphas[idx]:<10.4f}", end="")
    for j in range(5):
        print(f"  {coefs[idx, j]:<8.4f}", end="")
    print()
```

---

## 10. Regularization with Polynomial Regression

### 10.1 The Perfect Use Case

```python
from sklearn.preprocessing import PolynomialFeatures, StandardScaler
from sklearn.linear_model import Ridge, Lasso
from sklearn.pipeline import Pipeline

# High-degree polynomial + regularization = controlled complexity
pipe_ridge = Pipeline([
    ('poly', PolynomialFeatures(degree=10, include_bias=False)),
    ('scaler', StandardScaler()),
    ('ridge', Ridge(alpha=1.0))
])

pipe_lasso = Pipeline([
    ('poly', PolynomialFeatures(degree=10, include_bias=False)),
    ('scaler', StandardScaler()),
    ('lasso', Lasso(alpha=0.1, max_iter=10000))
])

# The polynomial creates 10 features from a single x variable.
# Without regularization, degree 10 would overfit badly.
# With regularization, the model stays smooth and generalizable.
```

---

## 11. Real-World Applications

| Application | Method | Why? |
|-------------|--------|------|
| **Gene expression analysis** | Lasso | 20,000+ genes, need feature selection |
| **House price prediction** | Ridge | Correlated features (area, rooms, etc.) |
| **Natural language processing** | Elastic Net | Huge vocabulary, sparse features |
| **Financial modeling** | Ridge | Many correlated market factors |
| **Medical diagnosis** | Lasso | Identify key biomarkers from thousands |
| **Image recognition features** | Elastic Net | Many correlated pixel features |
| **Recommender systems** | Ridge | Regularize user/item factors |

---

## 12. Summary Table

| Concept | Key Point |
|---------|-----------|
| **Why regularize?** | Prevent overfitting by penalizing large weights |
| **Ridge (L2)** | Penalty = `λΣwⱼ²`; shrinks weights but never to zero |
| **Lasso (L1)** | Penalty = `λΣ|wⱼ|`; can zero out weights → feature selection |
| **Elastic Net** | Combines L1 + L2; `λ[r·L1 + (1−r)·L2]` |
| **Ridge solution** | `w* = (XᵀX + λI)⁻¹Xᵀy` (closed-form) |
| **Lasso solution** | No closed-form; solved via coordinate descent |
| **λ too small** | No effect (overfitting persists) |
| **λ too large** | All weights → 0 (underfitting) |
| **Choosing λ** | Cross-validation (RidgeCV, LassoCV) |
| **L2 shape** | Circle → no zeros |
| **L1 shape** | Diamond → zeros at corners |
| **Multicollinearity** | Ridge handles well; Lasso picks one randomly |
| **Feature selection** | Lasso or Elastic Net |

---

## 13. ✏️ Revision Questions

1. **Explain why** regularization helps prevent overfitting. What does the penalty
   term do to the weights? What happens if λ is too large?

2. **Compare the L1 and L2 constraint regions** geometrically. Why does the diamond
   shape of L1 lead to sparse solutions (zero weights)?

3. **You have a dataset** with 50 features, many of which are irrelevant. Which
   regularization method would you choose and why?

4. **Write the Ridge cost function** and derive its closed-form solution. Why does
   adding `λI` to `XᵀX` solve the invertibility problem?

5. **You run LassoCV** and find that the optimal alpha zeros out 15 out of 20 features.
   The remaining 5 features give R² = 0.91 on the test set. What does this tell you
   about the original 20 features?

6. **When would you prefer Elastic Net** over pure Lasso? Describe a scenario involving
   correlated features where Lasso behaves poorly.

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| [← 06 – Polynomial Regression](06-polynomial-regression.md) | — (End of Unit 4: Linear Regression) |

[🔼 Back to Unit 2.1 Overview](../README.md)
