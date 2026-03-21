# 📘 Chapter 3: Cost Function — Mean Squared Error (MSE)

> **Unit 2.1 – Introduction to Machine Learning / 04 – Linear Regression** | File 3 of 7

---

## 📋 Overview

A **cost function** (also called loss function or objective function) quantifies how wrong
our model's predictions are. For linear regression, the most common cost function is the
**Mean Squared Error (MSE)**. This chapter covers the MSE definition, why we use squared
errors, the convexity property that guarantees a global minimum, contour and surface
visualizations, a comparison with MAE and Huber loss, and the full derivation of
partial derivatives needed for optimization.

---

## 1. Cost Function Definition

### 1.1 The Purpose

```
┌────────────────────────────────────────────────────────────┐
│  WHY DO WE NEED A COST FUNCTION?                          │
│                                                            │
│  Model:    ŷ = wx + b                                      │
│  Problem:  Which values of w and b are BEST?               │
│  Answer:   The ones that MINIMIZE the cost function J(w,b) │
│                                                            │
│  Cost function = a single number that tells us             │
│  "how bad" the current parameters are.                     │
└────────────────────────────────────────────────────────────┘
```

### 1.2 Mean Squared Error (MSE)

```
                   1    m
  J(w, b)  =  ─────  Σ  (ŷᵢ − yᵢ)²
                2m   i=1

            =  ─────  Σ  (wxᵢ + b − yᵢ)²
                2m   i=1
```

| Symbol | Meaning |
|--------|---------|
| `J(w, b)` | Cost — a function of parameters w and b |
| `m` | Number of training examples |
| `ŷᵢ` | Model prediction for sample i: `wxᵢ + b` |
| `yᵢ` | Actual target value for sample i |
| `1/2m` | Average + the ½ is for mathematical convenience (cancels with derivative) |

> **Note:** Some textbooks use `1/m` instead of `1/2m`. The factor of ½ doesn't change
> which parameters minimize J — it just makes the derivative cleaner.

### 1.3 Expanding the Formula

```
  For sample i:

  Error          =  ŷᵢ − yᵢ
  Squared error  =  (ŷᵢ − yᵢ)²
  Sum of SE      =  Σ (ŷᵢ − yᵢ)²
  Mean of SE     =  (1/m) Σ (ŷᵢ − yᵢ)²     ← this is the standard MSE
  Half Mean SE   =  (1/2m) Σ (ŷᵢ − yᵢ)²    ← used in gradient descent derivation
```

---

## 2. Why Squared Errors?

### 2.1 Why Not Just Use the Raw Errors?

```
  Errors:     +3,  −4,  +1,  −2,  +2

  Sum of errors = 3 + (−4) + 1 + (−2) + 2 = 0  ← Misleading! Errors cancel out.
  Sum of |errors| = 3 + 4 + 1 + 2 + 2 = 12     ← Absolute values work, but...
  Sum of errors² = 9 + 16 + 1 + 4 + 4 = 34      ← Squared values also work
```

### 2.2 Advantages of Squaring

| Advantage | Explanation |
|-----------|-------------|
| **No cancellation** | Positive and negative errors don't cancel |
| **Penalizes large errors more** | Error of 4 → cost 16; error of 2 → cost 4. Large errors are 4× worse, not 2× |
| **Differentiable everywhere** | Unlike absolute value, no kink at 0 |
| **Convex function** | Guarantees a single global minimum |
| **Mathematical convenience** | Derivatives are simple, closed-form solution exists |

### 2.3 Squaring Penalty Visualization

```
  Cost
  contribution
    │
  25│         ●                              ● = squared error
  16│                                        ○ = absolute error
    │      ●
   9│
    │   ●
   5│            ○
   4│         ○
   3│      ○
   2│   ○
   1│●○
   0├●○──────────────────────────── |error|
    0  1  2  3  4  5

  Squared error grows MUCH faster → penalizes outliers heavily
```

---

## 3. Convexity of the MSE Cost Function

### 3.1 What Is Convexity?

A function is **convex** if any line segment between two points on the function lies
**above or on** the function. Visually, it looks like a "bowl."

```
  Convex (bowl):              Non-convex (wavy):

  J │                         J │
    │ \         /               │    /\      /
    │  \       /                │   /  \    /
    │   \     /                 │  /    \  /
    │    \   /                  │ /      \/
    │     \_/  ← one minimum   │/  ← multiple minima
    └──────────── w             └──────────── w
    GUARANTEED global min       May get stuck in local min
```

### 3.2 Proof of Convexity for MSE

For simple linear regression, `J(w) = (1/2m) Σ (wxᵢ − yᵢ)²` (ignoring b for simplicity):

```
  Step 1: Expand J(w):
    J(w) = (1/2m) Σ (w²xᵢ² − 2wxᵢyᵢ + yᵢ²)

  Step 2: J(w) is a polynomial in w:
    J(w) = (Σxᵢ²/2m)·w² − (Σxᵢyᵢ/m)·w + (Σyᵢ²/2m)
           ↑               ↑               ↑
           a·w²         +  b·w           +  c

  Step 3: This is a QUADRATIC in w with:
    a = Σxᵢ² / 2m  ≥  0   (sum of squares is always non-negative)

  Step 4: A quadratic with a ≥ 0 is CONVEX. ✅

  Step 5: Second derivative test:
    d²J/dw² = (1/m) Σ xᵢ²  ≥  0   → confirms convexity
```

> **For multiple features:** J(w) = (1/2m)||Xw − y||² is convex because the
> **Hessian matrix** `H = (1/m)XᵀX` is positive semi-definite (all eigenvalues ≥ 0).

### 3.3 Why Convexity Matters

```
┌──────────────────────────────────────────────────────┐
│  Convexity guarantees:                                │
│                                                       │
│  • Every local minimum is also the GLOBAL minimum     │
│  • Gradient descent will ALWAYS converge              │
│    (with appropriate learning rate)                    │
│  • The solution found by optimization is THE BEST     │
│    possible, not just a "good" one                    │
└──────────────────────────────────────────────────────┘
```

---

## 4. Visualizing the Cost Function

### 4.1 1D Cost Curve (Varying w only, b fixed)

```
  J(w)
    │
    │ \                         /
    │  \                       /
    │   \                     /
    │    \                   /
    │     \                 /
    │      \               /
    │       \             /
    │        \           /
    │         \         /
    │          \_______/
    │             ↑
    │          w_optimal
    └──────────────────────────── w
```

### 4.2 Contour Plot (Varying both w and b)

A contour plot shows "slices" of the 3D surface viewed from above. Each contour line
connects points with the **same cost value**.

```
  b
  │
  │        ╭────────────╮
  │      ╭─┤            ├─╮
  │    ╭─┤ │    ╭──╮    │ ├─╮       J = 100
  │    │ │ │    │★ │    │ │ │       J = 50
  │    │ │ │    ╰──╯    │ │ │       J = 10
  │    ╰─┤ │            ├─╯         J = 1
  │      ╰─┤            ├─╯         ★ = minimum
  │        ╰────────────╯
  │
  └──────────────────────────── w

  ★ = optimal (w*, b*) that minimizes J
  Concentric ellipses = contours of equal cost
  Gradient descent follows arrows pointing toward ★
```

### 4.3 3D Surface Description

Imagine the contour plot "popped up" into 3D:

```
  J(w,b)
    │
    │   ╱────────────╲
    │  ╱  ╱────────╲  ╲
    │ ╱  ╱  ╱────╲  ╲  ╲
    │╱  ╱  ╱  ╱╲  ╲  ╲  ╲      ← bowl-shaped surface
    │  ╱  ╱  ╱  ╲  ╲  ╲
    │ ╱  ╱  ╱    ╲  ╲  ╲
    │╱  ╱  ╱______╲  ╲  ╲
    │  ╱  ╱________╲  ╲
    │ ╱  ╱__________╲  ╲
    │╱  ╱____________╲           ← bottom of bowl = minimum
    ├──────────────────── w
   ╱
  b
```

The bowl is smooth and has exactly **one lowest point** (the global minimum).

---

## 5. Computing MSE: Worked Example

### 5.1 Data

| i | xᵢ | yᵢ (actual) | ŷᵢ = 2xᵢ + 1 (predicted) |
|---|-----|-------------|---------------------------|
| 1 | 1 | 3 | 3 |
| 2 | 2 | 5 | 5 |
| 3 | 3 | 6 | 7 |
| 4 | 4 | 8 | 9 |
| 5 | 5 | 12 | 11 |

### 5.2 Step-by-Step MSE Calculation

```
  Step 1: Compute errors (ŷᵢ − yᵢ)
    e₁ = 3 − 3  =  0
    e₂ = 5 − 5  =  0
    e₃ = 7 − 6  =  1
    e₄ = 9 − 8  =  1
    e₅ = 11 − 12 = −1

  Step 2: Square each error
    e₁² = 0
    e₂² = 0
    e₃² = 1
    e₄² = 1
    e₅² = 1

  Step 3: Sum of squared errors
    SSE = 0 + 0 + 1 + 1 + 1 = 3

  Step 4: MSE = SSE / m = 3 / 5 = 0.6

  Step 5: J(w,b) = SSE / 2m = 3 / 10 = 0.3   (with the ½ factor)

  Step 6: RMSE = √MSE = √0.6 ≈ 0.775
```

---

## 6. MSE vs MAE vs Huber Loss — Comparison

### 6.1 Definitions

```
  MSE (Mean Squared Error):
    MSE = (1/m) Σ (ŷᵢ − yᵢ)²

  MAE (Mean Absolute Error):
    MAE = (1/m) Σ |ŷᵢ − yᵢ|

  Huber Loss (delta = δ):
              ⎧ ½(ŷᵢ − yᵢ)²              if |ŷᵢ − yᵢ| ≤ δ
    L_δ(ŷ,y) = ⎨
              ⎩ δ|ŷᵢ − yᵢ| − ½δ²         if |ŷᵢ − yᵢ| > δ
```

### 6.2 Visual Comparison

```
  Loss
    │
    │   MSE               .●
    │                    .●
    │                  .●
    │               ●●       ← Grows quadratically (fast)
    │            ●●
    │         ●●   . MAE
    │      ●●   . .
    │    ●●  . .             ← Grows linearly (slow)
    │  ●  . .
    │●. .     Huber is quadratic near 0,
    ●.          linear far from 0
    └──────────────────── |error|
```

### 6.3 Comparison Table

| Property | MSE | MAE | Huber Loss |
|----------|-----|-----|------------|
| **Formula** | `(ŷ−y)²` | `|ŷ−y|` | Quadratic near 0, linear far |
| **Outlier sensitivity** | 🔴 High | 🟢 Low | 🟡 Medium (tunable via δ) |
| **Differentiable at 0?** | ✅ Yes | ❌ No (kink at 0) | ✅ Yes |
| **Gradient** | `2(ŷ−y)` | `sign(ŷ−y)` | Smooth transition |
| **Convex?** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Closed-form solution?** | ✅ Yes | ❌ No | ❌ No |
| **Best for** | Clean data, no outliers | Data with outliers | Balanced robustness |
| **Common in** | Linear regression | Regression with outliers | Robust regression |

### 6.4 When to Use Each

```
  ┌─────────────────────────────────────────────────────────┐
  │  Choose MSE when:                                       │
  │  • Data is clean (few outliers)                         │
  │  • You want a closed-form solution                      │
  │  • Large errors should be penalized heavily              │
  │                                                          │
  │  Choose MAE when:                                       │
  │  • Data has significant outliers                        │
  │  • You want equal penalty for all error magnitudes      │
  │  • Median prediction is more useful than mean           │
  │                                                          │
  │  Choose Huber when:                                     │
  │  • You want a balance between MSE and MAE               │
  │  • Some outlier robustness + smooth gradients           │
  │  • You can tune δ via cross-validation                  │
  └─────────────────────────────────────────────────────────┘
```

---

## 7. Derivatives of the Cost Function

These derivatives are essential for **gradient descent** (covered in Chapter 4).

### 7.1 Simple Linear Regression (one feature)

```
  J(w, b) = (1/2m) Σ (wxᵢ + b − yᵢ)²
```

**Partial derivative with respect to w:**

```
  ∂J      1    m
  ── = ─────  Σ  2(wxᵢ + b − yᵢ) · xᵢ     ← chain rule
  ∂w    2m   i=1

       1    m
     = ──  Σ  (ŷᵢ − yᵢ) · xᵢ
       m   i=1
```

**Partial derivative with respect to b:**

```
  ∂J      1    m
  ── = ─────  Σ  2(wxᵢ + b − yᵢ) · 1
  ∂b    2m   i=1

       1    m
     = ──  Σ  (ŷᵢ − yᵢ)
       m   i=1
```

### 7.2 Vectorized Form (Multiple Features)

```
  J(w) = (1/2m) (Xw − y)ᵀ(Xw − y)

  Gradient:
    ∇J = (1/m) Xᵀ(Xw − y)

  This is a vector of partial derivatives:
    ∇J = [ ∂J/∂w₀,  ∂J/∂w₁,  ...,  ∂J/∂wₙ ]ᵀ
```

### 7.3 Derivation of the Vector Gradient

```
  Step 1: Expand J(w)
    J(w) = (1/2m)(wᵀXᵀXw − 2wᵀXᵀy + yᵀy)

  Step 2: Differentiate using matrix calculus rules
    ∂(wᵀAw)/∂w = 2Aw        (for symmetric A)
    ∂(wᵀv)/∂w  = v

  Step 3: Apply
    ∂J/∂w = (1/2m)(2XᵀXw − 2Xᵀy)
           = (1/m)(XᵀXw − Xᵀy)
           = (1/m)Xᵀ(Xw − y)      ✅
```

---

## 8. Python Implementation

### 8.1 Computing MSE from Scratch

```python
import numpy as np

def mse(y_true, y_pred):
    """Compute Mean Squared Error."""
    return np.mean((y_pred - y_true) ** 2)

def mse_half(y_true, y_pred):
    """Compute J(w,b) = (1/2m) Σ(ŷ-y)² — used in gradient descent."""
    m = len(y_true)
    return np.sum((y_pred - y_true) ** 2) / (2 * m)

def rmse(y_true, y_pred):
    """Root Mean Squared Error — same units as target."""
    return np.sqrt(mse(y_true, y_pred))

def mae(y_true, y_pred):
    """Mean Absolute Error."""
    return np.mean(np.abs(y_pred - y_true))

# Example
y_true = np.array([3, 5, 6, 8, 12])
y_pred = np.array([3, 5, 7, 9, 11])

print(f"MSE:   {mse(y_true, y_pred):.4f}")      # 0.6000
print(f"J(w):  {mse_half(y_true, y_pred):.4f}")  # 0.3000
print(f"RMSE:  {rmse(y_true, y_pred):.4f}")      # 0.7746
print(f"MAE:   {mae(y_true, y_pred):.4f}")        # 0.6000
```

### 8.2 Visualizing Cost vs. Parameter Value

```python
import numpy as np

# Data
x = np.array([1, 2, 3, 4, 5], dtype=float)
y = np.array([3, 5, 6, 8, 12], dtype=float)

# Compute cost for different values of w (with b fixed at 0)
w_values = np.linspace(-2, 5, 50)
costs = []

for w in w_values:
    y_pred = w * x
    cost = np.mean((y_pred - y) ** 2) / 2
    costs.append(cost)

# Find minimum
min_idx = np.argmin(costs)
print(f"Optimal w ≈ {w_values[min_idx]:.2f}")
print(f"Minimum cost ≈ {costs[min_idx]:.4f}")

# ASCII visualization of cost curve
print("\nCost vs w:")
max_cost = max(costs[:25])  # scale to visible range
for i in range(0, 50, 3):
    bar_len = int(costs[i] / max_cost * 40) if max_cost > 0 else 0
    bar_len = min(bar_len, 40)
    marker = " ◄── minimum" if i == min_idx or i == min_idx - 1 else ""
    print(f"  w={w_values[i]:>5.1f} │{'█' * bar_len}{marker}")
```

### 8.3 Using sklearn Metrics

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
import numpy as np

y_true = np.array([3, 5, 6, 8, 12])
y_pred = np.array([3, 5, 7, 9, 11])

print(f"MSE:  {mean_squared_error(y_true, y_pred):.4f}")
print(f"RMSE: {np.sqrt(mean_squared_error(y_true, y_pred)):.4f}")
print(f"MAE:  {mean_absolute_error(y_true, y_pred):.4f}")
print(f"R²:   {r2_score(y_true, y_pred):.4f}")
```

### 8.4 Computing Gradients

```python
import numpy as np

def compute_gradients(X, y, w, b):
    """Compute ∂J/∂w and ∂J/∂b for simple linear regression."""
    m = len(y)
    y_pred = w * X + b
    errors = y_pred - y

    dJ_dw = (1/m) * np.sum(errors * X)
    dJ_db = (1/m) * np.sum(errors)
    return dJ_dw, dJ_db

# Example: check gradients at w=2, b=1
x = np.array([1, 2, 3, 4, 5], dtype=float)
y = np.array([3, 5, 6, 8, 12], dtype=float)

dw, db = compute_gradients(x, y, w=2, b=1)
print(f"∂J/∂w = {dw:.4f}")
print(f"∂J/∂b = {db:.4f}")
# Negative gradient → w should increase
# Positive gradient → w should decrease
```

---

## 9. Real-World Applications

| Application | Why MSE? |
|-------------|----------|
| **House price prediction** | Large prediction errors (e.g., $100K off) are much worse than small ones |
| **Weather forecasting** | Temperature predictions penalize large deviations heavily |
| **Stock price modeling** | Financial models need smooth optimization landscape |
| **Signal processing** | MSE directly relates to signal-to-noise ratio |
| **Neural networks** | MSE is the default loss for regression tasks |
| **Physics simulations** | Least squares fitting has strong theoretical guarantees |

---

## 10. Summary Table

| Concept | Key Point |
|---------|-----------|
| **MSE** | `(1/m) Σ(ŷ−y)²` — average of squared errors |
| **J(w,b) with ½** | `(1/2m) Σ(ŷ−y)²` — used in gradient descent for cleaner derivatives |
| **Why squared?** | No cancellation, penalizes large errors, differentiable, convex |
| **Convexity** | MSE is quadratic in w → bowl-shaped → guaranteed global minimum |
| **Contour plot** | 2D view: concentric ellipses centered at the optimum |
| **∂J/∂w** | `(1/m) Σ(ŷᵢ−yᵢ)·xᵢ` |
| **∂J/∂b** | `(1/m) Σ(ŷᵢ−yᵢ)` |
| **∇J (vector)** | `(1/m) Xᵀ(Xw−y)` |
| **MSE vs MAE** | MSE penalizes outliers more; MAE is more robust |
| **Huber loss** | Hybrid: quadratic near 0, linear far away |

---

## 11. ✏️ Revision Questions

1. **Compute MSE by hand** for predictions `[2, 4, 6]` and actual values `[3, 3, 7]`.
   Then compute MAE for the same data. Which is larger and why?

2. **Explain why** the MSE cost function is convex for linear regression. What
   mathematical property guarantees this? What would happen if it were non-convex?

3. **Derive ∂J/∂w** from `J(w,b) = (1/2m) Σ(wxᵢ+b−yᵢ)²` step by step using
   the chain rule.

4. **A model gives predictions** `[10, 20, 30]` against true values `[10, 20, 100]`.
   Compute MSE and MAE. Which metric is more affected by the outlier? Why?

5. **When would you choose Huber loss** over MSE? Describe a real-world scenario
   where MSE would give misleading results due to outliers.

6. **Draw (or describe)** the contour plot for `J(w,b)` when the optimal solution is
   `w*=3, b*=1`. What do the ellipses represent? In which direction does gradient
   descent move?

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| [← 02 – Multiple Linear Regression](02-multiple-linear-regression.md) | [04 – Gradient Descent for Linear Regression →](04-gradient-descent-linear-regression.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
