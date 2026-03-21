# 📘 Chapter 6: Polynomial Regression

> **Unit 2.1 – Introduction to Machine Learning / 04 – Linear Regression** | File 6 of 7

---

## 📋 Overview

Real-world data is often **nonlinear** — a straight line simply won't fit. **Polynomial
Regression** extends linear regression by adding polynomial features (x², x³, etc.) to
capture curves in the data. Despite using nonlinear features, it remains a **linear model**
(linear in the weights). This chapter covers polynomial feature engineering, degree
selection, overfitting risks, the bias-variance tradeoff, sklearn's `PolynomialFeatures`,
pipeline construction, and model complexity curves.

---

## 1. Why Polynomial Regression?

### 1.1 The Limitation of Linear Regression

```
  Case 1: Linear data              Case 2: Nonlinear data
  (Linear regression works ✅)      (Linear regression fails ❌)

  y │      ╱ ●                     y │          ● ●
    │    ╱ ●                         │        ●     ●
    │  ╱ ●                           │      ●         ●
    │╱ ●                             │    ●
    └───────── x                     │  ●               ●
                                     │●
                                     └──────────────────── x
  Fit: ŷ = wx + b ✅                 Fit: ŷ = wx + b ❌ (misses the curve)
                                     Need: ŷ = w₂x² + w₁x + b ✅
```

### 1.2 The Idea

Instead of fitting `ŷ = w₁x + b`, create new features from powers of x:

```
  Original feature:      x
  Polynomial features:   x, x², x³, ..., xᵈ

  Model (degree d):
    ŷ = wᵈxᵈ + wᵈ₋₁xᵈ⁻¹ + ... + w₂x² + w₁x + b
```

> **Key insight:** The model is **nonlinear in x** but **linear in the weights**.
> We can still use Least Squares, Normal Equation, or Gradient Descent to find the weights!

---

## 2. Polynomial Features

### 2.1 Single Feature

For one input feature `x` with degree `d`:

```
  Degree 1:  [x]                           → 1 feature
  Degree 2:  [x, x²]                       → 2 features
  Degree 3:  [x, x², x³]                   → 3 features
  Degree d:  [x, x², x³, ..., xᵈ]          → d features
```

### 2.2 Multiple Features (degree 2 example)

For two input features `x₁, x₂` with degree 2:

```
  Original:     [x₁, x₂]                   → 2 features

  Polynomial:   [x₁, x₂, x₁², x₁x₂, x₂²]  → 5 features
                              ↑
                        interaction term!
```

### 2.3 Feature Explosion

The number of polynomial features grows **combinatorially**:

```
  Number of poly features = C(n + d, d) − 1

  where n = original features, d = degree
```

| Original Features (n) | Degree (d) | Polynomial Features | Growth |
|------------------------|------------|---------------------|--------|
| 1 | 2 | 2 | Manageable |
| 1 | 5 | 5 | Manageable |
| 2 | 2 | 5 | Manageable |
| 2 | 3 | 9 | OK |
| 5 | 2 | 20 | Getting large |
| 5 | 3 | 55 | Large |
| 10 | 3 | 285 | Very large |
| 10 | 5 | 3,002 | Explosion! 💥 |

> ⚠️ High degree + many features = **feature explosion** → overfitting risk!

---

## 3. Model Equation

### 3.1 Degree 2 (Quadratic)

```
  ŷ = w₂x² + w₁x + b

  Shape: Parabola (opens up if w₂ > 0, down if w₂ < 0)

  y │       ●
    │     ●   ●
    │    ●     ●
    │   ●       ●
    │  ●         ●           ← Quadratic fit
    │ ●           ●
    │●             ●
    └──────────────── x
```

### 3.2 Degree 3 (Cubic)

```
  ŷ = w₃x³ + w₂x² + w₁x + b

  Shape: S-curve (one inflection point)

  y │         ●●●
    │       ●     ●
    │      ●       ●
    │     ●
    │   ●                    ← Cubic fit
    │  ●
    │●●
    └──────────────── x
```

### 3.3 General Form

```
  Given feature vector for sample i:

  φ(xᵢ) = [1, xᵢ, xᵢ², xᵢ³, ..., xᵢᵈ]

  Model:
    ŷᵢ = φ(xᵢ)ᵀw = w₀ + w₁xᵢ + w₂xᵢ² + ... + wᵈxᵢᵈ

  In matrix form (for all m samples):
    Φ = [φ(x₁)ᵀ, φ(x₂)ᵀ, ..., φ(xₘ)ᵀ]ᵀ    (m × (d+1) matrix)
    ŷ = Φw

  Normal equation:
    w* = (ΦᵀΦ)⁻¹Φᵀy
```

---

## 4. Worked Example

### 4.1 Problem

Fit a degree-2 polynomial to this data:

| x | y |
|---|---|
| 1 | 1 |
| 2 | 4 |
| 3 | 9 |
| 4 | 15 |
| 5 | 25 |

### 4.2 Step-by-Step

**Step 1: Create polynomial feature matrix**

```
        ┌ 1   1   1  ┐
        │ 1   2   4  │
  Φ  =  │ 1   3   9  │     (columns: 1, x, x²)
        │ 1   4  16  │
        └ 1   5  25  ┘
```

**Step 2: Compute ΦᵀΦ**

```
         ┌  5   15    55 ┐
  ΦᵀΦ =  │ 15   55   225 │
         └ 55  225   979 ┘
```

**Step 3: Compute Φᵀy**

```
         ┌  54  ┐
  Φᵀy =  │ 218 │
         └ 934 ┘
```

**Step 4: Solve w* = (ΦᵀΦ)⁻¹Φᵀy** (using NumPy)

```python
import numpy as np

x = np.array([1, 2, 3, 4, 5])
y = np.array([1, 4, 9, 15, 25])

Phi = np.column_stack([np.ones(5), x, x**2])
w = np.linalg.lstsq(Phi, y, rcond=None)[0]
print(f"w₀ (intercept) = {w[0]:.4f}")
print(f"w₁ (x)         = {w[1]:.4f}")
print(f"w₂ (x²)        = {w[2]:.4f}")
# Output: ŷ ≈ 0.2857 − 0.2857x + 1.0x²
```

---

## 5. Degree Selection and Overfitting

### 5.1 The Problem: Underfitting vs. Overfitting

```
  Degree 1 (Underfitting)     Degree 3 (Good Fit)         Degree 15 (Overfitting)

  y │     ╱                   y │       ●●●                y │ ●╲  ╱●  ╱●╲
    │   ╱  ● ●                  │     ●     ●                │   ╳  ╳    ╳ ●
    │ ╱  ●                      │   ●        ●               │  ╱ ╲╱ ╲  ╱
    │╱ ●                        │  ●          ●              │ ╱       ╲╱
    │                           │ ●                          │╱
    └───────── x                └───────────── x             └───────────── x

  Too simple                  Just right                   Too complex
  High BIAS                   Balanced                     High VARIANCE
  Poor train + test           Good train + test            Great train, poor test
```

### 5.2 Bias-Variance Tradeoff

```
  Error
    │
    │╲  Total Error
    │ ╲         ╱
    │  ╲       ╱
    │   ╲     ╱
    │    ╲   ╱  ← Variance (increases with complexity)
    │     ╲_╱
    │      ↑      ← Sweet spot (optimal degree)
    │   Bias
    │   (decreases with complexity)
    └────────────────────── Model Complexity (degree)

  Low degree  → High bias, low variance   (underfitting)
  High degree → Low bias, high variance   (overfitting)
  Optimal     → Balanced bias and variance (good generalization)
```

### 5.3 Bias-Variance Decomposition

```
  Expected Error = Bias² + Variance + Irreducible Noise

  Bias²:      Error from oversimplified model assumptions
              (model can't capture the true pattern)

  Variance:   Error from model's sensitivity to training data
              (model changes drastically with different training sets)

  Noise:      Error inherent in the data (can never be eliminated)
```

| Component | High Bias | High Variance |
|-----------|-----------|---------------|
| Symptom | Underfitting | Overfitting |
| Training error | High | Very low |
| Test error | High | Very high |
| Gap (test − train) | Small | Large |
| Fix | Increase complexity, add features | Decrease complexity, add data, regularize |

### 5.4 How to Choose the Right Degree

| Method | Description |
|--------|-------------|
| **Cross-validation** | Try degrees 1–10, pick the one with lowest CV error |
| **Learning curves** | Plot train/test error vs. degree |
| **Information criteria** | AIC, BIC penalize model complexity |
| **Domain knowledge** | Physics suggests quadratic? Use degree 2. |
| **Regularization** | Use high degree + Ridge/Lasso to control complexity |

---

## 6. Model Complexity Curves

### 6.1 Training vs. Validation Error

```
  Error
    │
    │                                    ╱ Test Error
    │╲                                 ╱
    │  ╲                             ╱
    │    ╲                         ╱
    │     ╲        _______________╱
    │      ╲______╱
    │       ↑ ← Sweet spot
    │        ╲
    │         ╲
    │          ╲___________________  Training Error
    │
    └──┬──┬──┬──┬──┬──┬──┬──┬──── Polynomial Degree
       1  2  3  4  5  6  7  8

  The gap between curves = overfitting
  Both curves high = underfitting
  Sweet spot ≈ degree 3-4 in this example
```

### 6.2 How to Read the Curves

| Pattern | Diagnosis | Action |
|---------|-----------|--------|
| Both errors high, close together | **Underfitting** (high bias) | Increase degree |
| Train error low, test error high | **Overfitting** (high variance) | Decrease degree or regularize |
| Both errors low, close together | **Good fit** | Keep this degree! |

---

## 7. Python Implementation

### 7.1 Manual Polynomial Feature Creation

```python
import numpy as np

def create_polynomial_features(X, degree):
    """Create polynomial features for a single-feature array.

    X: shape (m,) or (m, 1)
    Returns: shape (m, degree) — columns are [x, x², ..., xᵈ]
    """
    X = X.ravel()
    return np.column_stack([X ** d for d in range(1, degree + 1)])

# Example
x = np.array([1, 2, 3, 4, 5])
X_poly = create_polynomial_features(x, degree=3)
print("Polynomial features (degree 3):")
print(X_poly)
# [[  1   1   1]
#  [  2   4   8]
#  [  3   9  27]
#  [  4  16  64]
#  [  5  25 125]]
```

### 7.2 Using sklearn's PolynomialFeatures

```python
import numpy as np
from sklearn.preprocessing import PolynomialFeatures

x = np.array([1, 2, 3, 4, 5]).reshape(-1, 1)

# Degree 3, include bias column
poly = PolynomialFeatures(degree=3, include_bias=True)
X_poly = poly.fit_transform(x)
print(f"Feature names: {poly.get_feature_names_out()}")
print(f"Shape: {X_poly.shape}")
print(X_poly)
# Feature names: ['1', 'x0', 'x0^2', 'x0^3']
# Shape: (5, 4)

# Without bias (use with LinearRegression which adds its own)
poly_no_bias = PolynomialFeatures(degree=3, include_bias=False)
X_poly_nb = poly_no_bias.fit_transform(x)
print(f"\nWithout bias: {poly_no_bias.get_feature_names_out()}")
```

### 7.3 Complete Pipeline Example

```python
import numpy as np
from sklearn.preprocessing import PolynomialFeatures, StandardScaler
from sklearn.linear_model import LinearRegression
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score

# Generate nonlinear data
np.random.seed(42)
m = 100
X = np.sort(np.random.uniform(-3, 3, m)).reshape(-1, 1)
y = 0.5 * X.ravel()**3 - 2 * X.ravel()**2 + X.ravel() + 3 + np.random.randn(m) * 2

# Create pipeline: PolynomialFeatures → Scale → LinearRegression
def create_poly_pipeline(degree):
    return Pipeline([
        ('poly', PolynomialFeatures(degree=degree, include_bias=False)),
        ('scaler', StandardScaler()),
        ('regressor', LinearRegression())
    ])

# Compare different degrees using cross-validation
print(f"{'Degree':<8} {'Mean CV R²':<15} {'Std CV R²':<12}")
print("-" * 35)

best_degree = 1
best_score = -np.inf

for d in range(1, 11):
    pipe = create_poly_pipeline(d)
    scores = cross_val_score(pipe, X, y, cv=5, scoring='r2')
    mean_score = scores.mean()
    std_score = scores.std()

    marker = ""
    if mean_score > best_score:
        best_score = mean_score
        best_degree = d
        marker = " ◄── best"

    print(f"  {d:<6} {mean_score:<15.4f} {std_score:<12.4f}{marker}")

print(f"\nOptimal degree: {best_degree} (CV R² = {best_score:.4f})")

# Fit final model with best degree
final_pipe = create_poly_pipeline(best_degree)
final_pipe.fit(X, y)
y_pred = final_pipe.predict(X)
```

### 7.4 Visualizing Underfitting vs. Overfitting

```python
import numpy as np
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

# Generate data
np.random.seed(42)
m = 30
X = np.sort(np.random.uniform(0, 5, m)).reshape(-1, 1)
y_true = np.sin(X.ravel()) * 5 + X.ravel() * 2
y = y_true + np.random.randn(m) * 1.5

# Split manually
train_idx = np.random.choice(m, size=int(0.7*m), replace=False)
test_idx = np.setdiff1d(np.arange(m), train_idx)

X_train, y_train = X[train_idx], y[train_idx]
X_test, y_test = X[test_idx], y[test_idx]

# Try different degrees
print(f"{'Degree':<8} {'Train MSE':<12} {'Test MSE':<12} {'Status'}")
print("-" * 50)

for d in [1, 3, 5, 10, 20]:
    poly = PolynomialFeatures(d, include_bias=False)
    X_train_p = poly.fit_transform(X_train)
    X_test_p = poly.transform(X_test)

    model = LinearRegression()
    model.fit(X_train_p, y_train)

    train_mse = mean_squared_error(y_train, model.predict(X_train_p))
    test_mse = mean_squared_error(y_test, model.predict(X_test_p))

    if train_mse > 5 and test_mse > 5:
        status = "Underfitting"
    elif test_mse > 3 * train_mse:
        status = "Overfitting ⚠️"
    else:
        status = "Good fit ✅"

    print(f"  {d:<6} {train_mse:<12.4f} {test_mse:<12.4f} {status}")
```

### 7.5 Learning Curves (Training Size vs. Error)

```python
import numpy as np
from sklearn.model_selection import learning_curve
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.pipeline import Pipeline

pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=3, include_bias=False)),
    ('reg', LinearRegression())
])

train_sizes, train_scores, val_scores = learning_curve(
    pipe, X, y,
    train_sizes=np.linspace(0.2, 1.0, 8),
    cv=5,
    scoring='neg_mean_squared_error'
)

train_mse = -train_scores.mean(axis=1)
val_mse = -val_scores.mean(axis=1)

print("\nLearning Curve (Degree 3):")
print(f"{'Train Size':<12} {'Train MSE':<12} {'Val MSE':<12}")
print("-" * 36)
for size, t_mse, v_mse in zip(train_sizes, train_mse, val_mse):
    print(f"  {size:<10} {t_mse:<12.4f} {v_mse:<12.4f}")
```

---

## 8. Important Considerations

### 8.1 Feature Scaling Is Critical

Polynomial features create values on wildly different scales:

```
  x = 100
  x² = 10,000
  x³ = 1,000,000

  Without scaling, the model struggles to balance weights across features.
  ALWAYS use StandardScaler() or similar when doing polynomial regression.
```

### 8.2 Polynomial Regression Is Still "Linear"

```
  "Linear" in regression refers to linearity in WEIGHTS, not in features.

  ŷ = w₂x² + w₁x + b

  Nonlinear in x? Yes ✅ (x² is a curve)
  Linear in w?    Yes ✅ (ŷ is linear combination of w₁, w₂, b)

  This means all linear regression tools (Normal Equation, OLS) still apply!
```

### 8.3 Extrapolation Warning

```
  Training range: x ∈ [0, 10]

  Prediction at x=11:  Might be OK
  Prediction at x=50:  DANGEROUS ⚠️

  y │              ●●●●●●●●      ← training data
    │           ●●          ●●
    │        ●●               ●●
    │      ●                    ●
    │    ●                       ●
    │  ●                          ●
    │ ●                            ╲╲╲ → explodes in extrapolation!
    └──────────────────────────────────── x
                    ↑               ↑
                  x=10            x=20
                 (safe)        (dangerous)

  High-degree polynomials become EXTREMELY unreliable outside the training range.
```

---

## 9. Real-World Applications

| Domain | Why Polynomial? | Typical Degree |
|--------|-----------------|----------------|
| **Physics** | Many phenomena are quadratic (projectile motion, energy) | 2 |
| **Economics** | Diminishing returns, growth curves | 2–3 |
| **Biology** | Population growth, dose-response curves | 2–4 |
| **Engineering** | Stress-strain relationships | 2–3 |
| **Climate science** | Temperature trends over decades | 2–3 |
| **Marketing** | Saturation effects (ad spend vs. sales) | 2 |

---

## 10. Summary Table

| Concept | Key Point |
|---------|-----------|
| **Polynomial regression** | Extends linear regression with polynomial features (x², x³, ...) |
| **Still linear?** | Yes — linear in weights, nonlinear in features |
| **Feature creation** | `PolynomialFeatures(degree=d)` in sklearn |
| **Feature explosion** | Number of features grows combinatorially with degree |
| **Underfitting** | Degree too low → can't capture the pattern |
| **Overfitting** | Degree too high → fits noise, poor generalization |
| **Bias-variance** | Low degree = high bias; high degree = high variance |
| **Degree selection** | Use cross-validation to find optimal degree |
| **Feature scaling** | Essential — polynomial features have wildly different scales |
| **Pipeline** | `PolynomialFeatures → StandardScaler → LinearRegression` |
| **Extrapolation** | High-degree polynomials are unreliable outside training range |

---

## 11. ✏️ Revision Questions

1. **Why is polynomial regression** considered a "linear" model despite fitting curves?
   What exactly is "linear" about it?

2. **You have 5 features** and want degree 3 polynomial features. Approximately how
   many features will you end up with? Why is this a concern?

3. **Sketch (or describe)** what happens when you fit degree 1, degree 4, and degree
   20 polynomials to a sinusoidal dataset with 30 points.

4. **Explain the bias-variance tradeoff** in the context of polynomial degree.
   What happens to bias and variance as degree increases?

5. **Why is feature scaling** especially important for polynomial regression?
   What happens to the values x, x², x³ when x = 1000?

6. **Write a sklearn Pipeline** that performs polynomial regression of degree 4 with
   feature scaling. Include cross-validation to evaluate the model.

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| [← 05 – Normal Equation](05-normal-equation.md) | [07 – Regularization →](07-regularization.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
