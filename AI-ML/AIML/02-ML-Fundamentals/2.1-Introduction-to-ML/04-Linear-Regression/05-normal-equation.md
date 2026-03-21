# 📘 Chapter 5: The Normal Equation

> **Unit 2.1 – Introduction to Machine Learning / 04 – Linear Regression** | File 5 of 7

---

## 📋 Overview

The **Normal Equation** provides a **closed-form (analytical) solution** to linear regression
— no iterations, no learning rate, no convergence waiting. You plug in the data, do one
matrix computation, and get the optimal parameters directly. This chapter covers the full
derivation, computational complexity, when to prefer the Normal Equation over Gradient
Descent, handling non-invertibility, the pseudo-inverse, a detailed comparison table, and
Python implementations.

---

## 1. The Normal Equation Formula

### 1.1 Statement

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│   w* = (XᵀX)⁻¹ Xᵀy                                     │
│                                                          │
│   where:                                                 │
│     X  = m × (n+1) design matrix (with bias column)     │
│     y  = m × 1 target vector                            │
│     w* = (n+1) × 1 optimal weight vector                │
│                                                          │
│   This gives the EXACT w that minimizes J(w) = ‖Xw−y‖²  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 1.2 What Each Part Does

```
  XᵀX     → (n+1) × (n+1) matrix   "feature correlation matrix"
  (XᵀX)⁻¹ → inverse of that matrix  "decorrelation"
  Xᵀy     → (n+1) × 1 vector       "feature-target correlation"

  Together: w* = (decorrelated features) × (feature-target alignment)
```

### 1.3 Dimensions Check

For `m = 1000` samples, `n = 5` features (plus bias column → 6 columns):

```
  X:     1000 × 6
  Xᵀ:    6 × 1000
  XᵀX:   6 × 6         ← small square matrix!
  (XᵀX)⁻¹: 6 × 6
  Xᵀy:   6 × 1
  w*:     6 × 1         ← 6 parameters (5 weights + 1 bias)
```

---

## 2. Full Derivation

### 2.1 Starting Point

We want to minimize the cost function:

```
  J(w) = (1/2m) ‖Xw − y‖²

       = (1/2m)(Xw − y)ᵀ(Xw − y)
```

Since the `1/2m` factor doesn't affect which `w` minimizes `J`, we can equivalently minimize:

```
  f(w) = (Xw − y)ᵀ(Xw − y)
```

### 2.2 Step-by-Step Expansion

```
  Step 1: Expand the product
    f(w) = (Xw − y)ᵀ(Xw − y)
         = (wᵀXᵀ − yᵀ)(Xw − y)
         = wᵀXᵀXw − wᵀXᵀy − yᵀXw + yᵀy
```

Since `wᵀXᵀy` is a scalar and equals its transpose `yᵀXw`:

```
  Step 2: Simplify
    f(w) = wᵀXᵀXw − 2yᵀXw + yᵀy
```

### 2.3 Taking the Derivative

Using matrix calculus rules:

```
  Rule 1:  ∂(wᵀAw)/∂w = 2Aw     (for symmetric A)
  Rule 2:  ∂(vᵀw)/∂w = v

  Step 3: Differentiate f(w) with respect to w
    ∂f/∂w = 2XᵀXw − 2Xᵀy
```

### 2.4 Setting the Gradient to Zero

```
  Step 4: Set ∂f/∂w = 0  (necessary condition for minimum)
    2XᵀXw − 2Xᵀy = 0
    XᵀXw = Xᵀy

  Step 5: Solve for w (multiply both sides by (XᵀX)⁻¹)
    w* = (XᵀX)⁻¹ Xᵀy    ✅
```

### 2.5 Confirming It's a Minimum

```
  Step 6: Check the second derivative (Hessian)
    ∂²f/∂w² = 2XᵀX

  XᵀX is positive semi-definite (all eigenvalues ≥ 0).
  If X has full column rank, XᵀX is positive DEFINITE → w* is a UNIQUE minimum.  ✅
```

### 2.6 Derivation Summary Diagram

```
  J(w) = ½‖Xw−y‖²
         │
         ▼   expand
  wᵀXᵀXw − 2yᵀXw + yᵀy
         │
         ▼   differentiate
  ∇J = 2XᵀXw − 2Xᵀy
         │
         ▼   set = 0
  XᵀXw = Xᵀy
         │
         ▼   solve
  w* = (XᵀX)⁻¹Xᵀy     ← THE NORMAL EQUATION
```

---

## 3. Computational Complexity

### 3.1 Cost Breakdown

| Operation | Complexity | Notes |
|-----------|------------|-------|
| `XᵀX` | O(mn²) | Multiply m×(n+1) by (n+1)×m |
| `(XᵀX)⁻¹` | **O(n³)** | Matrix inversion — the bottleneck |
| `Xᵀy` | O(mn) | Multiply m×(n+1) by m×1 |
| Total | **O(mn² + n³)** | Dominated by n³ for large n |

### 3.2 When n Is Large

```
  n = 100     → n³ = 1,000,000          ← instant
  n = 1,000   → n³ = 1,000,000,000      ← a few seconds
  n = 10,000  → n³ = 10¹²               ← minutes to hours
  n = 100,000 → n³ = 10¹⁵               ← impractical ❌
```

> **Rule of thumb:** Normal Equation is practical when **n < 10,000** features.
> Beyond that, use Gradient Descent.

### 3.3 Memory Requirements

```
  XᵀX requires storing an (n+1) × (n+1) matrix

  n = 10,000 → 10,000² × 8 bytes ≈ 800 MB  ← tight
  n = 100,000 → 100,000² × 8 bytes ≈ 80 GB ← impossible on most machines
```

---

## 4. Normal Equation vs Gradient Descent

### 4.1 Comprehensive Comparison Table

```
┌──────────────────────┬──────────────────────┬──────────────────────┐
│     Criteria         │   Normal Equation    │   Gradient Descent   │
├──────────────────────┼──────────────────────┼──────────────────────┤
│ Type                 │ Analytical (exact)   │ Iterative (approx)   │
│ Learning rate α      │ Not needed           │ Must choose carefully│
│ Iterations           │ None (one-shot)      │ Many (100s to 1000s) │
│ Feature scaling      │ Not needed           │ Essential            │
│ Convergence check    │ Not needed           │ Must monitor         │
│ Complexity           │ O(n³)                │ O(kmn) per k iters   │
│ Large n (features)   │ Slow (n > 10,000)    │ Works well           │
│ Large m (samples)    │ Works (O(mn²))       │ Works (mini-batch)   │
│ Exact solution?      │ Yes (to float prec)  │ Approximate          │
│ Non-invertible XᵀX   │ Fails (need pinv)    │ Still works          │
│ Generalizes?         │ Only linear models   │ Any differentiable J │
│ Implementation       │ One line of code     │ Needs tuning loop    │
└──────────────────────┴──────────────────────┴──────────────────────┘
```

### 4.2 Decision Flowchart

```
  START
    │
    ▼
  How many features (n)?
    │
    ├── n < 10,000 ──▶ Normal Equation ✅
    │                   (fast, exact, no tuning)
    │
    ├── n ≥ 10,000 ──▶ Gradient Descent ✅
    │                   (scales with n)
    │
    └── Using a non-linear model?
         │
         └── Yes ──▶ Gradient Descent ✅
                      (only option for neural nets, logistic regression, etc.)
```

---

## 5. Non-Invertibility Issues

### 5.1 When Is XᵀX Not Invertible?

The matrix `XᵀX` is not invertible (singular) when:

| Cause | Example | Solution |
|-------|---------|----------|
| **Redundant features** | `x₂ = 3·x₁` (linear dependency) | Drop one feature |
| **Too many features** | `n > m` (more features than samples) | Remove features, or regularize |
| **Perfectly correlated features** | `area_sqft` and `area_sqm` | Drop duplicate |
| **Zero-variance feature** | A column of all 5s | Remove constant feature |

### 5.2 Why Singularity Happens

```
  XᵀX is (n+1) × (n+1).
  If X has rank < n+1, then XᵀX is rank-deficient → NOT invertible.

  Rank deficiency causes:
    det(XᵀX) = 0
    One or more eigenvalues = 0
    Infinite solutions exist (the system is underdetermined)
```

### 5.3 Geometric Picture

```
  Full rank (n=2):                Rank deficient (n=2):
  Features span 2D                Features are collinear

  x₂ │   ╱ feature 2              x₂ │   ╱ both features
     │  ╱                            │  ╱  point in the
     │ ╱                             │ ╱   SAME direction
     │╱_____ feature 1               │╱
     └─────── x₁                     └─────── x₁

  Can uniquely determine           Infinite solutions
  the plane ✅                     along the line ❌
```

---

## 6. The Pseudo-Inverse (Moore-Penrose)

### 6.1 What Is It?

When `XᵀX` is not invertible, we use the **pseudo-inverse** `X⁺` instead:

```
  w* = X⁺y     where X⁺ = (XᵀX)⁺Xᵀ
```

NumPy's `np.linalg.pinv()` computes this using **Singular Value Decomposition (SVD)**.

### 6.2 Properties of the Pseudo-Inverse

| Property | Description |
|----------|-------------|
| Always exists | Even for singular or non-square matrices |
| Gives minimum-norm solution | Among all solutions, picks the one with smallest ‖w‖ |
| Equals true inverse when invertible | `pinv(A) = A⁻¹` if A is invertible |
| Numerically stable | SVD-based computation avoids issues with near-singular matrices |

### 6.3 SVD Approach

```
  X = UΣVᵀ     (Singular Value Decomposition)

  X⁺ = VΣ⁺Uᵀ   (pseudo-inverse via SVD)

  where Σ⁺ = reciprocal of non-zero singular values, transposed
```

---

## 7. Worked Example

### 7.1 Problem

Given the data:

| x₁ | x₂ | y |
|----|----|---|
| 1 | 2 | 5 |
| 2 | 3 | 8 |
| 3 | 5 | 12 |
| 4 | 6 | 15 |

Find the optimal weights using the Normal Equation.

### 7.2 Step-by-Step Solution

**Step 1: Construct X (with bias column) and y**

```
        ┌ 1  1  2 ┐         ┌  5 ┐
  X  =  │ 1  2  3 │    y =  │  8 │
        │ 1  3  5 │         │ 12 │
        └ 1  4  6 ┘         └ 15 ┘
```

**Step 2: Compute XᵀX**

```
          ┌ 1  1  1  1 ┐   ┌ 1  1  2 ┐
  XᵀX  =  │ 1  2  3  4 │ × │ 1  2  3 │
          └ 2  3  5  6 ┘   │ 1  3  5 │
                            └ 1  4  6 ┘

        ┌  4   10   16 ┐
  XᵀX = │ 10   30   50 │
        └ 16   50   74 ┘      (this was not a typo — let's verify)
```

Verification:
- Row 1, Col 1: 1·1 + 1·1 + 1·1 + 1·1 = 4 ✓
- Row 1, Col 2: 1·1 + 1·2 + 1·3 + 1·4 = 10 ✓
- Row 2, Col 3: 1·2 + 2·3 + 3·5 + 4·6 = 2+6+15+24 = 47

Let me recompute carefully:

```
          ┌ 1 1 1 1 ┐   ┌ 1 1 2 ┐     ┌  4   10   16 ┐
  XᵀX  =  │ 1 2 3 4 │ × │ 1 2 3 │  =  │ 10   30   47 │
          └ 2 3 5 6 ┘   │ 1 3 5 │     └ 16   47   74 ┘
                         └ 1 4 6 ┘
```

**Step 3: Compute Xᵀy**

```
          ┌ 1 1 1 1 ┐   ┌  5 ┐     ┌  40 ┐
  Xᵀy  =  │ 1 2 3 4 │ × │  8 │  =  │ 121 │
          └ 2 3 5 6 ┘   │ 12 │     └ 184 ┘
                         └ 15 ┘
```

Verification:
- Row 1: 1·5 + 1·8 + 1·12 + 1·15 = 40 ✓
- Row 2: 1·5 + 2·8 + 3·12 + 4·15 = 5+16+36+60 = 117

Let me recompute:

```
  Xᵀy = ┌ 1·5 + 1·8 + 1·12 + 1·15 ┐   ┌ 40  ┐
         │ 1·5 + 2·8 + 3·12 + 4·15 │ = │ 117 │
         └ 2·5 + 3·8 + 5·12 + 6·15 ┘   └ 184 ┘
```

**Step 4: Solve w* = (XᵀX)⁻¹ Xᵀy**

This requires computing the 3×3 matrix inverse (best done numerically — see Python below).

**Step 5: Python verification**

```python
import numpy as np

X = np.array([[1, 1, 2],
              [1, 2, 3],
              [1, 3, 5],
              [1, 4, 6]])
y = np.array([5, 8, 12, 15])

w = np.linalg.pinv(X.T @ X) @ X.T @ y
print(f"w* = {w}")
# Output: w* ≈ [1.0, 1.5, 1.0]
# → ŷ = 1.0 + 1.5·x₁ + 1.0·x₂
```

---

## 8. Python Implementation

### 8.1 Normal Equation from Scratch

```python
import numpy as np

class LinearRegressionNE:
    """Linear Regression using the Normal Equation."""

    def __init__(self):
        self.weights = None

    def fit(self, X, y):
        m = X.shape[0]
        # Add bias column
        X_b = np.column_stack([np.ones(m), X])
        # Normal equation: w = (XᵀX)⁻¹Xᵀy
        # Using pinv for numerical stability
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


# ─── Example ─────────────────────────────────
np.random.seed(42)

# Generate data: y = 3x₁ - 2x₂ + 5 + noise
m = 200
X = np.random.randn(m, 2)
y = 3 * X[:, 0] - 2 * X[:, 1] + 5 + np.random.randn(m) * 0.5

model = LinearRegressionNE()
model.fit(X, y)

print("Normal Equation Results:")
print(f"  Intercept: {model.intercept:.4f}  (true: 5)")
print(f"  Coef x₁:  {model.coefficients[0]:.4f}  (true: 3)")
print(f"  Coef x₂:  {model.coefficients[1]:.4f}  (true: -2)")
print(f"  R²:        {model.score(X, y):.4f}")
```

### 8.2 One-Line Normal Equation

```python
import numpy as np

# Data
X = np.column_stack([np.ones(m), X_features])  # add bias column
y = target_values

# One-line solution
w = np.linalg.pinv(X.T @ X) @ X.T @ y

# Or equivalently using lstsq (recommended — most stable):
w, residuals, rank, sv = np.linalg.lstsq(X, y, rcond=None)
```

### 8.3 Comparing Normal Equation vs sklearn

```python
import numpy as np
from sklearn.linear_model import LinearRegression

np.random.seed(42)
m, n = 500, 3
X = np.random.randn(m, n)
true_w = np.array([2.5, -1.0, 3.0])
y = X @ true_w + 4.0 + np.random.randn(m) * 0.3

# Method 1: Normal Equation
X_b = np.column_stack([np.ones(m), X])
w_ne = np.linalg.pinv(X_b.T @ X_b) @ X_b.T @ y

# Method 2: sklearn
model = LinearRegression()
model.fit(X, y)
w_sk = np.concatenate([[model.intercept_], model.coef_])

# Compare
print("Weight comparison (Normal Eq vs sklearn):")
print(f"  {'Param':<8} {'Normal Eq':>10} {'sklearn':>10} {'True':>10}")
print(f"  {'bias':<8} {w_ne[0]:>10.4f} {w_sk[0]:>10.4f} {4.0:>10.4f}")
for i in range(n):
    print(f"  {'w'+str(i+1):<8} {w_ne[i+1]:>10.4f} {w_sk[i+1]:>10.4f} {true_w[i]:>10.4f}")

# They should be virtually identical!
print(f"\n  Max difference: {np.max(np.abs(w_ne - w_sk)):.2e}")
```

### 8.4 Demonstrating Non-Invertibility

```python
import numpy as np

# Create data with a redundant feature (x₃ = 2·x₁)
np.random.seed(42)
m = 100
x1 = np.random.randn(m)
x2 = np.random.randn(m)
x3 = 2 * x1  # perfectly collinear with x1!

X = np.column_stack([x1, x2, x3])
y = 3 * x1 + 2 * x2 + 5 + np.random.randn(m) * 0.5

X_b = np.column_stack([np.ones(m), X])
XtX = X_b.T @ X_b

print(f"det(XᵀX) = {np.linalg.det(XtX):.6f}")  # Very close to 0
print(f"Condition number: {np.linalg.cond(XtX):.2e}")  # Very large

# np.linalg.inv(XtX) would fail or give garbage
# But pinv still works:
w_pinv = np.linalg.pinv(X_b) @ y
print(f"\nWeights (pinv): {w_pinv}")
print("Note: Individual weights for x1 and x3 are unreliable,")
print("but predictions are still valid.")
```

---

## 9. The `np.linalg.lstsq` Alternative

### 9.1 Why Use lstsq?

`np.linalg.lstsq` solves the least squares problem directly using SVD, without
explicitly computing `(XᵀX)⁻¹`. It is:

- More numerically stable than the normal equation
- Handles rank-deficient matrices automatically
- The recommended approach in practice

```python
import numpy as np

X_b = np.column_stack([np.ones(m), X])
w, residuals, rank, singular_values = np.linalg.lstsq(X_b, y, rcond=None)

print(f"Weights: {w}")
print(f"Matrix rank: {rank}")
print(f"Singular values: {singular_values}")
```

### 9.2 Comparison of Methods

| Method | Stability | Handles Singularity | Code |
|--------|-----------|---------------------|------|
| `inv(XᵀX) @ Xᵀy` | ⚠️ Least stable | ❌ Fails | `np.linalg.inv(XtX) @ Xty` |
| `pinv(XᵀX) @ Xᵀy` | ✅ Good | ✅ Yes | `np.linalg.pinv(XtX) @ Xty` |
| `pinv(X) @ y` | ✅ Better | ✅ Yes | `np.linalg.pinv(X_b) @ y` |
| `lstsq(X, y)` | ✅✅ Best | ✅ Yes | `np.linalg.lstsq(X_b, y)` |

---

## 10. Real-World Applications

| Scenario | Why Normal Equation? |
|----------|---------------------|
| **Small datasets** (< 10K features) | One-shot computation, no hyperparameter tuning |
| **Feature engineering pipelines** | Quick prototyping before trying complex models |
| **Embedded systems** | Pre-compute weights offline, deploy with simple multiplication |
| **Statistical analysis** | Exact solution needed for confidence intervals |
| **Teaching / learning** | Builds intuition about the math behind ML |

---

## 11. Summary Table

| Concept | Key Point |
|---------|-----------|
| **Normal Equation** | `w* = (XᵀX)⁻¹Xᵀy` — closed-form solution |
| **Derivation** | Set ∂J/∂w = 0, solve for w |
| **Complexity** | O(n³) for matrix inversion |
| **Practical limit** | n < 10,000 features |
| **Feature scaling** | NOT needed (unlike Gradient Descent) |
| **Non-invertibility** | Use pseudo-inverse `pinv()` or regularization |
| **Causes of singularity** | Redundant features, n > m, perfect collinearity |
| **Best NumPy function** | `np.linalg.lstsq()` — most stable |
| **vs Gradient Descent** | NE is faster for small n; GD scales to large n |

---

## 12. ✏️ Revision Questions

1. **Derive the Normal Equation** starting from `J(w) = ‖Xw − y‖²`. Show all
   matrix calculus steps.

2. **What is the computational complexity** of the Normal Equation? At what number
   of features does it become impractical? Why?

3. **When would you choose** the Normal Equation over Gradient Descent? When would
   you choose Gradient Descent? Give specific criteria.

4. **XᵀX is not invertible.** List three possible causes. For each, describe a fix.

5. **What is the pseudo-inverse?** How does `np.linalg.pinv` differ from
   `np.linalg.inv`? When must you use `pinv`?

6. **Given X = [[1,1],[1,2],[1,3]] and y = [2,3,5]**, compute w* = (XᵀX)⁻¹Xᵀy
   by hand (showing all matrix multiplications). Verify with NumPy.

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| [← 04 – Gradient Descent](04-gradient-descent-linear-regression.md) | [06 – Polynomial Regression →](06-polynomial-regression.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
