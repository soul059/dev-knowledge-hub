# 📈 Support Vector Regression (SVR)

> **Unit 9, Chapter 6 — ε-Insensitive Tube, SVR Formulation & Nonlinear Regression**

---

## 📋 Table of Contents

1. [Chapter Overview](#1-chapter-overview)
2. [From Classification to Regression](#2-from-classification-to-regression)
3. [The ε-Insensitive Tube](#3-the-ε-insensitive-tube)
4. [ε-SVR Formulation](#4-ε-svr-formulation)
5. [Points Inside the Tube Have Zero Loss](#5-points-inside-the-tube-have-zero-loss)
6. [C and ε Parameters](#6-c-and-ε-parameters)
7. [Kernel SVR — Nonlinear Regression](#7-kernel-svr--nonlinear-regression)
8. [Comparison with Linear Regression](#8-comparison-with-linear-regression)
9. [Python Implementation — sklearn SVR](#9-python-implementation--sklearn-svr)
10. [Applications](#10-applications)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. Chapter Overview

While SVM is most famous for classification, its principles extend beautifully to **regression** problems. Support Vector Regression (SVR) finds a function that:

1. Has at most ε deviation from the actual targets for all training data
2. Is as **flat** as possible (small ‖w‖)

```
SVM Classification:                    SVR Regression:
├── Find a hyperplane that              ├── Find a function that
│   SEPARATES classes                   │   FITS the data
├── Maximize MARGIN between classes     ├── Keep predictions within ε-TUBE
├── Points outside margin → zero loss   ├── Points inside tube → zero loss
└── Support vectors ON the margin       └── Support vectors ON or OUTSIDE tube
```

---

## 2. From Classification to Regression

### The Key Insight

In classification, we want the hyperplane to be **far from** data points (maximize margin). In regression, we want the regression function to be **close to** data points (minimize error), but we tolerate errors up to ε.

```
CLASSIFICATION (SVM):                   REGRESSION (SVR):

    + + +    ┃    - - -                 ●
    + + +    ┃    - - -                    ●    ●
    + + +    ┃    - - -                 ●═══●══════●═══●   ← f(x) ± ε tube
    + + +    ┃    - - -                        ●
    + + +    ┃    - - -                 ●
             ┃
    MARGIN = empty region                TUBE = tolerance region
    (no points inside)                   (points inside → no penalty)
```

### SVR Model

The regression function has the same form as the SVM decision function:

```
f(x) = w · x + b

Goal: Find w and b such that f(x) approximates y for all training points
      within tolerance ε, while keeping w as "flat" as possible.
```

---

## 3. The ε-Insensitive Tube

### Definition

The **ε-insensitive tube** (or ε-tube) is a band of width 2ε centered around the regression function f(x):

```
f(x) - ε  ≤  y  ≤  f(x) + ε
```

Any training point whose target falls **inside** the tube incurs **zero loss**.

### Visualization

```
    y ↑
      │                 ●  (ξ* = outside ABOVE tube)
      │                ╱
      │         ●     ╱
      │        ╱     ╱ ● (inside tube → NO penalty)
      │  ═════╱═══ε═╱════════════════  ← f(x) + ε  (upper boundary)
      │      ╱   ●╱   ●   ●
      │     ╱───────────────────────  ← f(x) = w·x + b
      │    ╱  ●   ╱  ●   ●
      │  ═╱══════╱═══ε══════════════  ← f(x) - ε  (lower boundary)
      │  ╱    ● ╱
      │ ╱      ╱
      │╱  ●   ╱  (ξ = outside BELOW tube)
      │      ╱
      └───────────────────────────→ x

    Width of tube = 2ε

    ● inside tube:  |yᵢ - f(xᵢ)| ≤ ε → ZERO loss, NOT a support vector
    ● on boundary:  |yᵢ - f(xᵢ)| = ε → ZERO loss, IS a support vector
    ● outside tube: |yᵢ - f(xᵢ)| > ε → PENALIZED, IS a support vector
```

### ε-Insensitive Loss Function

```
              ┌ 0                    if |y - f(x)| ≤ ε
L_ε(y, f(x)) = │
              └ |y - f(x)| - ε      if |y - f(x)| > ε

Equivalently:  L_ε = max(0, |y - f(x)| - ε)
```

### Loss Function Plot

```
Loss
  ↑
  │                 ╱
3 │                ╱
  │               ╱
2 │              ╱                ╱
  │             ╱                ╱
1 │            ╱                ╱
  │           ╱                ╱
0 │──────────╱────────────────╱──────
  │                                  
  │  -4  -3  -2  -1  0  1  2  3  4   → y - f(x)
  │              │←─ε─→│
  │              │  =0  │
  │              │ zone │
  │
  │  Zero loss for |y - f(x)| ≤ ε
  │  Linear loss for |y - f(x)| > ε
```

### Comparison of Regression Loss Functions

| Loss Function | Formula | Properties |
|---------------|---------|-----------|
| **MSE (L2)** | (y - f(x))² | Smooth, penalizes all errors, sensitive to outliers |
| **MAE (L1)** | \|y - f(x)\| | Robust, penalizes all errors linearly |
| **ε-insensitive** | max(0, \|y-f(x)\|-ε) | Ignores small errors, robust, sparse |
| **Huber** | Quadratic for small, linear for large | Smooth, robust |

---

## 4. ε-SVR Formulation

### Primal Problem

```
              1                    N
minimize    ─── ‖w‖²  +  C · Σ  (ξᵢ + ξᵢ*)
  w,b,ξ,ξ*   2                  i=1

subject to:
    yᵢ - (w · xᵢ + b) ≤ ε + ξᵢ       for all i  (upper slack)
    (w · xᵢ + b) - yᵢ ≤ ε + ξᵢ*      for all i  (lower slack)
    ξᵢ ≥ 0, ξᵢ* ≥ 0                    for all i
```

### Understanding the Two Sets of Slack Variables

```
ξᵢ  = slack for points ABOVE the tube (y > f(x) + ε)
ξᵢ* = slack for points BELOW the tube (y < f(x) - ε)

For each point, at most ONE of ξᵢ and ξᵢ* can be non-zero.

      ● (ξᵢ = |yᵢ-f(xᵢ)| - ε)
      │↕ ξᵢ
  ════╪══════════════════  f(x) + ε
      │
  ────┼──────────────────  f(x)
      │
  ════╪══════════════════  f(x) - ε
      │↕ ξⱼ*
      ● (ξⱼ* = |yⱼ-f(xⱼ)| - ε)
```

### Objective Function Breakdown

```
              1                    N
Objective = ─── ‖w‖²    +    C · Σ  (ξᵢ + ξᵢ*)
             2                   i=1
            ─────────         ──────────────────
          Flatness            Sum of violations
          (regularization)    (empirical loss)
          
    "Keep the function      "Keep errors
     smooth/flat"             within tube"
```

### Dual Problem

Introducing Lagrange multipliers αᵢ, αᵢ* ≥ 0:

```
                N                           N   N
maximize  - ½ Σ Σ (αᵢ-αᵢ*)(αⱼ-αⱼ*)(xᵢ·xⱼ) - ε Σ (αᵢ+αᵢ*) + Σ yᵢ(αᵢ-αᵢ*)
              i j                             i=1              i=1

subject to:
    Σᵢ (αᵢ - αᵢ*) = 0
    0 ≤ αᵢ ≤ C        for all i
    0 ≤ αᵢ* ≤ C       for all i
```

### Recovering the Solution

```
w = Σᵢ (αᵢ - αᵢ*) xᵢ

f(x) = Σᵢ (αᵢ - αᵢ*)(xᵢ · x) + b

Support vectors: points where αᵢ > 0 or αᵢ* > 0
(i.e., points ON or OUTSIDE the ε-tube)
```

---

## 5. Points Inside the Tube Have Zero Loss

### The Sparsity Property

Just like in SVM classification, **most training points don't contribute** to the model:

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Point INSIDE tube: |yᵢ - f(xᵢ)| ≤ ε                     │
│  ├── ξᵢ = 0, ξᵢ* = 0  (no violation)                     │
│  ├── αᵢ = 0, αᵢ* = 0  (no contribution to w)             │
│  └── NOT a support vector                                  │
│                                                            │
│  Point ON tube boundary: |yᵢ - f(xᵢ)| = ε                │
│  ├── ξᵢ = 0 or ξᵢ* = 0  (no violation, but active)       │
│  ├── 0 < αᵢ < C or 0 < αᵢ* < C                           │
│  └── IS a support vector (free SV)                        │
│                                                            │
│  Point OUTSIDE tube: |yᵢ - f(xᵢ)| > ε                    │
│  ├── ξᵢ > 0 or ξᵢ* > 0  (violation!)                     │
│  ├── αᵢ = C or αᵢ* = C  (bounded SV)                     │
│  └── IS a support vector (bounded SV)                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Why This Matters

```
Example: 1000 data points, ε = 0.5

After training SVR:
├── 900 points inside tube → αᵢ = αᵢ* = 0 (ignored!)
├── 60 points on tube boundary → support vectors
├── 40 points outside tube → support vectors (penalized)
└── Model depends only on 100 support vectors!

Prediction: f(x) = Σ_{100 SVs} (αᵢ-αᵢ*)(xᵢ·x) + b
(Much faster than using all 1000 points)
```

---

## 6. C and ε Parameters

### ε (Epsilon) — Tube Width

```
ε controls the width of the insensitive tube:

SMALL ε (e.g., ε = 0.01):          LARGE ε (e.g., ε = 2.0):

      ● ●                                  ●
     ●  ●  ●                              ● ● ●
  ══●══●══●══●══●══  (narrow tube)     ═══════════════  (wide tube)
     ●  ●  ●                              ● ●
      ●                                    ●   ●

  ├── Many points outside tube          ├── Most points inside tube
  ├── Many support vectors              ├── Few support vectors  
  ├── More complex model                ├── Simpler model
  ├── Lower bias, higher variance       ├── Higher bias, lower variance
  └── Fits data more closely            └── Smoother approximation
```

### C — Regularization Parameter

```
C controls how much we penalize tube violations:

SMALL C (e.g., C = 0.01):              LARGE C (e.g., C = 1000):

  Tolerate large violations             Penalize violations heavily
  ├── Flatter function (small ‖w‖)      ├── More wiggly function
  ├── May miss data patterns            ├── Follows every data point
  ├── Underfitting tendency              ├── Overfitting tendency
  └── More robust to outliers            └── Sensitive to outliers
```

### Combined Effect of C and ε

```
         ε small                     ε large
       ┌───────────────────┬───────────────────┐
       │                   │                   │
C      │  Tight fit,       │  Smooth fit,      │
small  │  but regularized  │  very simple      │
       │  (balanced)       │  (underfitting)   │
       │                   │                   │
       ├───────────────────┼───────────────────┤
       │                   │                   │
C      │  Very tight fit   │  Moderate fit,    │
large  │  (overfitting!)   │  allows some      │
       │  Follows noise    │  flexibility      │
       │                   │                   │
       └───────────────────┴───────────────────┘
```

### Parameter Selection Guidelines

```
For ε:
├── ε ≈ noise level of data (if known)
├── Larger ε for noisier data
├── Typical range: [0.01, 1.0]
└── Can use cross-validation

For C:
├── Default: C = 1.0
├── Increase if underfitting
├── Decrease if overfitting
├── Typical range: [0.01, 1000] on log scale
└── Use GridSearchCV
```

---

## 7. Kernel SVR — Nonlinear Regression

### The Power of Kernelized SVR

Just like kernel SVM for classification, we can replace dot products with kernels for **nonlinear regression**:

```
Linear SVR:     f(x) = w · x + b
                     = Σᵢ (αᵢ-αᵢ*)(xᵢ · x) + b

Kernel SVR:     f(x) = Σᵢ (αᵢ-αᵢ*) K(xᵢ, x) + b
                                     ──────────
                                     Kernel function!
```

### Nonlinear Regression Examples

```
LINEAR SVR (kernel='linear'):

    y ↑
      │        ●  ●
      │     ●  ───────●── f(x)    Linear fit
      │   ● ──────────── ●
      │ ──●──●
      │●──
      └──────────────────→ x


RBF SVR (kernel='rbf'):

    y ↑
      │        ●  ●
      │     ● ╱──╲──●    Nonlinear fit
      │   ●──╱    ╲──●
      │ ╱──●●      ╲
      │╱              ╲
      └──────────────────→ x


POLYNOMIAL SVR (kernel='poly', degree=3):

    y ↑
      │        ●  ●
      │     ● ╱──╲──●    Cubic fit
      │   ●─╱    ╲──●
      │ ╱─●●      ╲
      │╱            ╲
      └──────────────────→ x
```

### When to Use Kernel SVR

```
Use LINEAR SVR when:
├── Relationship is approximately linear
├── Fast training needed
└── Many features

Use RBF SVR when:
├── Relationship is clearly nonlinear
├── Don't know the functional form
└── Have moderate-sized dataset

Use POLYNOMIAL SVR when:
├── Know the polynomial degree of relationship
├── Want interpretable feature interactions
└── Degree ≤ 3 usually sufficient
```

---

## 8. Comparison with Linear Regression

### Key Differences

| Aspect | Linear Regression (OLS) | SVR |
|--------|------------------------|-----|
| **Loss function** | Squared error (L2) | ε-insensitive |
| **All points contribute?** | Yes (all influence fit) | No (only SVs) |
| **Outlier sensitivity** | High (squared penalty) | Low (linear penalty beyond ε) |
| **Sparsity** | Not sparse | Sparse (few SVs) |
| **Regularization** | Optional (Ridge, Lasso) | Built-in (‖w‖²) |
| **Nonlinear extension** | Polynomial features | Kernel trick |
| **Closed-form solution?** | Yes (Normal equation) | No (QP solver needed) |
| **Probabilistic?** | Yes (Gaussian errors) | No (deterministic tube) |

### Loss Function Comparison

```
Loss
  ↑
  │
  │     ╱         ╲
  │    ╱           ╲        Squared Loss (L2)
  │   ╱    ╱───╲    ╲       for Linear Regression
  │  ╱    ╱     ╲    ╲
  │ ╱    ╱       ╲    ╲
  │╱    ╱  ╱──╲   ╲    ╲    ε-Insensitive Loss
  │    ╱  ╱    ╲   ╲        for SVR
  │   ╱──╱      ╲──╲
  │──╱────────────╲──
  │──────────────────────→ y - f(x)
  │     -ε    0    ε
  │
  │  Key difference: ε-insensitive has FLAT bottom
  │  (zero penalty for small errors)
```

### When SVR is Better Than OLS

```
SVR wins when:
├── Data has outliers (ε-insensitive loss is robust)
├── Want sparse model (fewer SVs than N)
├── Nonlinear relationships (kernel trick)
├── Want regularization built-in
└── Noise level is approximately known (set ε ≈ noise)

OLS wins when:
├── Want closed-form solution (fast, no iteration)
├── Want statistical inference (confidence intervals, p-values)
├── All errors should be penalized (no tolerance zone)
├── Very large datasets (OLS scales better)
└── Need probabilistic predictions (prediction intervals)
```

---

## 9. Python Implementation — sklearn SVR

### Basic SVR Example

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVR
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score

# ── Generate nonlinear data ──
np.random.seed(42)
X = np.sort(5 * np.random.rand(200, 1), axis=0)
y = np.sin(X).ravel() + 0.3 * np.random.randn(200)

# ── Scale features (important for SVR!) ──
scaler_X = StandardScaler()
scaler_y = StandardScaler()
X_scaled = scaler_X.fit_transform(X)
y_scaled = scaler_y.fit_transform(y.reshape(-1, 1)).ravel()

# ── Train SVR models with different kernels ──
models = {
    'Linear SVR': SVR(kernel='linear', C=1.0, epsilon=0.1),
    'Poly SVR (d=3)': SVR(kernel='poly', degree=3, C=1.0, epsilon=0.1, coef0=1),
    'RBF SVR': SVR(kernel='rbf', C=10.0, gamma='scale', epsilon=0.1),
}

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

for ax, (name, model) in zip(axes, models.items()):
    model.fit(X_scaled, y_scaled)
    
    # Predictions
    X_test = np.linspace(X_scaled.min(), X_scaled.max(), 300).reshape(-1, 1)
    y_pred = model.predict(X_test)
    
    # Plot
    ax.scatter(X_scaled, y_scaled, c='gray', s=15, alpha=0.5, label='Data')
    ax.plot(X_test, y_pred, 'r-', linewidth=2, label='SVR fit')
    ax.fill_between(X_test.ravel(),
                    y_pred - 0.1,  # ε-tube
                    y_pred + 0.1,
                    alpha=0.2, color='red', label='ε-tube')
    
    # Highlight support vectors
    sv_idx = model.support_
    ax.scatter(X_scaled[sv_idx], y_scaled[sv_idx], c='green', s=50,
               edgecolors='k', zorder=5, label=f'SVs ({len(sv_idx)})')
    
    rmse = np.sqrt(mean_squared_error(y_scaled, model.predict(X_scaled)))
    ax.set_title(f'{name}\nRMSE: {rmse:.4f}, SVs: {len(sv_idx)}/{len(X)}')
    ax.legend(fontsize=8)
    ax.grid(True, alpha=0.3)

plt.suptitle('SVR with Different Kernels', fontsize=14)
plt.tight_layout()
plt.savefig('svr_comparison.png', dpi=150, bbox_inches='tight')
plt.show()
```

### Effect of ε Parameter

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVR
from sklearn.preprocessing import StandardScaler

# Generate data
np.random.seed(42)
X = np.sort(5 * np.random.rand(100, 1), axis=0)
y = np.sin(X).ravel() + 0.2 * np.random.randn(100)

scaler = StandardScaler()
X_sc = scaler.fit_transform(X)

epsilons = [0.01, 0.1, 0.5, 1.0]
fig, axes = plt.subplots(1, 4, figsize=(20, 4))

for ax, eps in zip(axes, epsilons):
    svr = SVR(kernel='rbf', C=10, epsilon=eps, gamma='scale')
    svr.fit(X_sc, y)
    
    X_test = np.linspace(X_sc.min()-0.5, X_sc.max()+0.5, 200).reshape(-1, 1)
    y_pred = svr.predict(X_test)
    
    ax.scatter(X_sc, y, c='gray', s=15, alpha=0.5)
    ax.plot(X_test, y_pred, 'r-', linewidth=2)
    ax.fill_between(X_test.ravel(), y_pred-eps, y_pred+eps,
                    alpha=0.2, color='red')
    
    n_sv = len(svr.support_)
    ax.set_title(f'ε = {eps}\nSVs: {n_sv}/{len(X)} ({100*n_sv/len(X):.0f}%)')
    ax.grid(True, alpha=0.3)

plt.suptitle('Effect of ε on SVR', fontsize=14)
plt.tight_layout()
plt.savefig('epsilon_effect.png', dpi=150, bbox_inches='tight')
plt.show()
```

### GridSearch for Optimal SVR Parameters

```python
from sklearn.svm import SVR
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.datasets import fetch_california_housing
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

# Load dataset
data = fetch_california_housing()
X, y = data.data, data.target

# Use subset for speed
X_train, X_test, y_train, y_test = train_test_split(
    X[:2000], y[:2000], test_size=0.2, random_state=42
)

# Pipeline
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('svr', SVR())
])

# Parameter grid
param_grid = {
    'svr__kernel': ['rbf'],
    'svr__C': [0.1, 1, 10, 100],
    'svr__epsilon': [0.01, 0.1, 0.5],
    'svr__gamma': ['scale', 0.01, 0.1],
}

grid = GridSearchCV(pipe, param_grid, cv=5, scoring='neg_mean_squared_error',
                    n_jobs=-1, verbose=0)
grid.fit(X_train, y_train)

print(f"Best params: {grid.best_params_}")
print(f"Best CV MSE: {-grid.best_score_:.4f}")

# Test performance
y_pred = grid.predict(X_test)
print(f"\nTest MSE:  {mean_squared_error(y_test, y_pred):.4f}")
print(f"Test RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")
print(f"Test R²:   {r2_score(y_test, y_pred):.4f}")

# Support vectors
best_svr = grid.best_estimator_.named_steps['svr']
print(f"\nSupport vectors: {len(best_svr.support_)} / {len(X_train)} "
      f"({100*len(best_svr.support_)/len(X_train):.1f}%)")
```

### Comparing SVR with Other Regressors

```python
import numpy as np
from sklearn.svm import SVR
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.ensemble import RandomForestRegressor
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score
from sklearn.datasets import fetch_california_housing

X, y = fetch_california_housing(return_X_y=True)
X, y = X[:2000], y[:2000]  # Subset for speed

models = {
    'Linear Regression': Pipeline([
        ('scaler', StandardScaler()),
        ('model', LinearRegression())
    ]),
    'Ridge Regression': Pipeline([
        ('scaler', StandardScaler()),
        ('model', Ridge(alpha=1.0))
    ]),
    'SVR (Linear)': Pipeline([
        ('scaler', StandardScaler()),
        ('model', SVR(kernel='linear', C=1.0))
    ]),
    'SVR (RBF)': Pipeline([
        ('scaler', StandardScaler()),
        ('model', SVR(kernel='rbf', C=10.0, gamma='scale'))
    ]),
    'Random Forest': RandomForestRegressor(n_estimators=100, random_state=42),
}

print(f"{'Model':<22} | {'CV R²':>10} | {'CV RMSE':>10}")
print("-" * 50)

for name, model in models.items():
    r2_scores = cross_val_score(model, X, y, cv=5, scoring='r2')
    mse_scores = -cross_val_score(model, X, y, cv=5, scoring='neg_mean_squared_error')
    rmse = np.sqrt(mse_scores.mean())
    print(f"{name:<22} | {r2_scores.mean():>10.4f} | {rmse:>10.4f}")
```

---

## 10. Applications

### Where SVR Excels

| Application | Why SVR | Kernel |
|-------------|---------|--------|
| **Financial forecasting** | Robust to outliers (market crashes) | RBF |
| **Time series prediction** | Handles nonlinear trends | RBF, Poly |
| **Drug dosage prediction** | Small datasets, noisy data | RBF |
| **Load forecasting** | Complex nonlinear patterns | RBF |
| **Environmental modeling** | Sparse data, noisy sensors | RBF |
| **Quality control** | Tolerance-based predictions | Linear, RBF |

### SVR Advantages and Disadvantages

```
✅ ADVANTAGES:
├── Robust to outliers (ε-insensitive loss)
├── Sparse model (only SVs matter)
├── Built-in regularization (‖w‖²)
├── Kernel trick for nonlinear regression
├── Good generalization (margin theory)
└── Works well on small/medium datasets

❌ DISADVANTAGES:
├── Slow for large datasets (O(N²) to O(N³))
├── Sensitive to feature scaling (must standardize)
├── Multiple hyperparameters (C, ε, γ)
├── No probabilistic output (no prediction intervals)
├── Doesn't handle missing values
└── Memory-intensive (stores support vectors)
```

### Practical Tips

```
1. ALWAYS scale features (StandardScaler)
2. Start with RBF kernel (most general)
3. Tune in order: C first, then ε, then γ
4. Use cross-validation for all parameters
5. Monitor support vector count (too many → overfitting)
6. Set ε ≈ expected noise level if known
7. For large data (>10,000): consider LinearSVR or SGDRegressor
```

---

## 11. Summary Table

| Concept | Key Point |
|---------|-----------|
| **SVR goal** | Fit data within ε-tube while keeping function flat |
| **ε-tube** | Band of width 2ε where predictions incur zero loss |
| **ε-insensitive loss** | max(0, \|y-f(x)\| - ε) — ignores small errors |
| **Primal objective** | min ½‖w‖² + CΣ(ξᵢ+ξᵢ*) |
| **Slack variables** | ξᵢ (above tube), ξᵢ* (below tube) |
| **Support vectors** | Points ON or OUTSIDE the ε-tube |
| **Inside tube** | Zero loss, αᵢ = αᵢ* = 0 (not SVs) |
| **C parameter** | Penalty for violations (large C → tighter fit) |
| **ε parameter** | Tube width (large ε → fewer SVs, simpler model) |
| **Kernel SVR** | f(x) = Σ(αᵢ-αᵢ*)K(xᵢ,x) + b — nonlinear regression |
| **vs Linear Reg** | SVR: robust, sparse, nonlinear; OLS: fast, probabilistic |
| **Key requirement** | Feature scaling is essential! |

---

## 12. Revision Questions

### Q1: ε-Tube Concept
**Explain what the ε-insensitive tube is and why points inside it are treated differently from points outside it.**

<details>
<summary>Answer</summary>

The ε-insensitive tube is a band of width 2ε centered around the SVR regression function f(x). It defines a **tolerance zone** for predictions:

- **Points inside** the tube (|yᵢ - f(xᵢ)| ≤ ε): Incur **zero loss**. The prediction is considered "good enough" — these points don't need to be fit more precisely. They have αᵢ = αᵢ* = 0 and are NOT support vectors.

- **Points outside** the tube (|yᵢ - f(xᵢ)| > ε): Incur **linear loss** proportional to their distance beyond ε. These points become support vectors and influence the model.

**Why this design?**
1. **Sparsity**: Only support vectors matter → efficient model
2. **Robustness**: Small errors are tolerated (noise insensitivity)
3. **Generalization**: ε acts as a regularizer, preventing overfitting to noise
4. **Practical**: In many applications, errors below a threshold are acceptable

</details>

### Q2: Parameter Trade-offs
**How do C and ε interact? What happens when both C and ε are very small?**

<details>
<summary>Answer</summary>

**C controls** how much the model penalizes deviations beyond the tube:
- Large C: High penalty → model tries to keep all points inside the tube
- Small C: Low penalty → model allows large deviations, focusing on flatness

**ε controls** the tube width:
- Large ε: Wide tube → many points inside → fewer SVs → simpler model
- Small ε: Narrow tube → many points outside → more SVs → complex model

**When both C and ε are very small**:
- Small ε: Narrow tube → many points are "outside" and become candidates for penalization
- Small C: But the penalty for being outside is very low → violations are cheaply tolerated
- Net effect: The model heavily prioritizes **flatness** (small ‖w‖) over fitting the data
- Result: An overly smooth/flat function that **underfits** the data
- The model essentially says: "I want a narrow tube, but I don't really care if points miss it"

The interaction creates a balance where C/ε ratio matters more than individual values.

</details>

### Q3: SVR vs Linear Regression
**When would you prefer SVR over ordinary least squares regression? Give a specific scenario.**

<details>
<summary>Answer</summary>

**Prefer SVR when**: You have a **small dataset with outliers and a nonlinear relationship**.

**Specific scenario**: Predicting house prices in a neighborhood with 200 houses.
- Some houses have abnormally high/low prices (outliers from unique features)
- Price vs. size relationship is nonlinear (diminishing returns for very large houses)
- Data is limited (only 200 samples)

**Why SVR wins here**:
1. **Outlier robustness**: ε-insensitive loss doesn't quadratically penalize outlier prices, so a few extreme prices won't distort the model
2. **Nonlinear capability**: RBF kernel SVR can capture the nonlinear price-size relationship without manually engineering polynomial features
3. **Built-in regularization**: ‖w‖² term prevents overfitting on small dataset
4. **Sparse representation**: Model only stores ~50 support vectors, not all 200 points

OLS would be distorted by outliers, would miss the nonlinear pattern (unless features are manually added), and would use all 200 points equally.

</details>

### Q4: Support Vectors in SVR
**After training an SVR model, you find 80% of training points are support vectors. What does this indicate and what would you adjust?**

<details>
<summary>Answer</summary>

**80% support vectors is very high** — it indicates:
1. The ε-tube captures very few points (tube might be too narrow)
2. The model is likely **overfitting** — it depends on almost every training point
3. The model loses the sparsity advantage of SVR
4. Prediction will be slow (must compute 80% of kernel evaluations)

**What to adjust**:

1. **Increase ε**: Widen the tube so more points fall inside → fewer SVs
   ```python
   # Try: ε = 0.1 → ε = 0.5 → ε = 1.0
   ```

2. **Decrease C**: Reduce the penalty for being outside the tube → model prioritizes flatness → wider effective margin
   ```python
   # Try: C = 100 → C = 1 → C = 0.1
   ```

3. **Decrease γ** (if using RBF): Smoother decision function → fewer points needed
   ```python
   # Try: gamma = 1 → gamma = 0.1 → gamma = 0.01
   ```

4. **Check if data is too noisy**: If noise level is high, ε should match it
5. **Consider if SVR is appropriate**: With 80% SVs, a simpler model (Ridge) might be equally good and faster

</details>

### Q5: Kernel SVR
**Write a Python snippet that trains an RBF SVR on a sinusoidal dataset, tunes C and ε with GridSearchCV, and plots the best fit with the ε-tube.**

<details>
<summary>Answer</summary>

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVR
from sklearn.model_selection import GridSearchCV
from sklearn.preprocessing import StandardScaler

# Generate sinusoidal data
np.random.seed(42)
X = np.sort(np.random.uniform(0, 2*np.pi, 100)).reshape(-1, 1)
y = np.sin(X).ravel() + 0.2 * np.random.randn(100)

# Scale
sc_X = StandardScaler()
X_sc = sc_X.fit_transform(X)

# Grid search
param_grid = {
    'C': [0.1, 1, 10, 100],
    'epsilon': [0.01, 0.1, 0.3, 0.5],
    'gamma': ['scale', 0.1, 1],
}

grid = GridSearchCV(SVR(kernel='rbf'), param_grid, cv=5,
                    scoring='neg_mean_squared_error', n_jobs=-1)
grid.fit(X_sc, y)

best = grid.best_estimator_
print(f"Best params: {grid.best_params_}")

# Plot
X_plot = np.linspace(X_sc.min()-0.5, X_sc.max()+0.5, 300).reshape(-1, 1)
y_plot = best.predict(X_plot)
eps = grid.best_params_['epsilon']

plt.figure(figsize=(10, 6))
plt.scatter(X_sc, y, c='gray', s=20, alpha=0.6, label='Data')
plt.plot(X_plot, y_plot, 'r-', linewidth=2, label='SVR fit')
plt.fill_between(X_plot.ravel(), y_plot-eps, y_plot+eps,
                 alpha=0.2, color='red', label=f'ε-tube (ε={eps})')

sv_idx = best.support_
plt.scatter(X_sc[sv_idx], y[sv_idx], c='green', s=60,
            edgecolors='k', zorder=5,
            label=f'Support Vectors ({len(sv_idx)})')

plt.legend()
plt.title(f'RBF SVR | C={grid.best_params_["C"]}, '
          f'ε={eps}, γ={grid.best_params_["gamma"]}')
plt.xlabel('x (scaled)')
plt.ylabel('y')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

</details>

### Q6: Mathematical
**Derive the ε-insensitive loss from the SVR constraints. Show that ξᵢ = max(0, yᵢ - f(xᵢ) - ε) and ξᵢ* = max(0, f(xᵢ) - yᵢ - ε).**

<details>
<summary>Answer</summary>

From the SVR constraints:

**Constraint 1**: yᵢ - (w·xᵢ + b) ≤ ε + ξᵢ, with ξᵢ ≥ 0
→ ξᵢ ≥ yᵢ - f(xᵢ) - ε

Since we minimize Σξᵢ, at optimality ξᵢ is as small as possible:
→ ξᵢ = max(0, yᵢ - f(xᵢ) - ε)

This is nonzero only when yᵢ > f(xᵢ) + ε (point above tube).

**Constraint 2**: (w·xᵢ + b) - yᵢ ≤ ε + ξᵢ*, with ξᵢ* ≥ 0
→ ξᵢ* ≥ f(xᵢ) - yᵢ - ε

At optimality:
→ ξᵢ* = max(0, f(xᵢ) - yᵢ - ε)

This is nonzero only when yᵢ < f(xᵢ) - ε (point below tube).

**Total loss for point i**:
ξᵢ + ξᵢ* = max(0, yᵢ - f(xᵢ) - ε) + max(0, f(xᵢ) - yᵢ - ε)
           = max(0, |yᵢ - f(xᵢ)| - ε)

This is exactly the **ε-insensitive loss**! ✓

Note: At most one of ξᵢ, ξᵢ* can be nonzero (a point can't be simultaneously above AND below the tube).

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Common Kernels](./05-common-kernels.md) | [Unit 9: SVM](../) | [Multi-class SVM →](./07-multi-class-svm.md) |

---

*📝 Part of the AIML Study Notes — Unit 9: Support Vector Machines*
*🔖 Chapter 6 of 7 — SVM Regression (SVR)*
