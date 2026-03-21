# 📘 Chapter 4: Gradient Descent for Linear Regression

> **Unit 2.1 – Introduction to Machine Learning / 04 – Linear Regression** | File 4 of 7

---

## 📋 Overview

**Gradient Descent** is the workhorse optimization algorithm of machine learning. It
iteratively adjusts model parameters by moving in the direction of steepest decrease of the
cost function. This chapter covers the gradient descent algorithm, computing partial
derivatives for linear regression, update rules, learning rate effects, convergence
criteria, feature scaling, batch vs. stochastic vs. mini-batch variants, complete Python
implementation from scratch, and convergence visualization.

---

## 1. The Gradient Descent Algorithm

### 1.1 Intuition: The Mountain Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Imagine you are on a mountain in thick fog.                    │
│  You cannot see the valley below.                               │
│                                                                 │
│  Strategy:                                                      │
│    1. Feel the slope beneath your feet                          │
│    2. Take a step in the DOWNHILL direction                     │
│    3. Repeat until the ground is flat (minimum reached)         │
│                                                                 │
│       ╲                                                         │
│        ╲  YOU                                                   │
│         ╲  ↓                                                    │
│          ╲  ↓                                                   │
│           ╲  ↓        ← steps going downhill                   │
│            ╲  ↓                                                 │
│             ╲__↓___                                             │
│                     ← valley (minimum of cost function)        │
│                                                                 │
│  "slope beneath your feet" = GRADIENT                           │
│  "step size"               = LEARNING RATE (α)                  │
│  "downhill direction"      = NEGATIVE gradient                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 The Algorithm

```
  repeat until convergence {
      w := w − α · ∂J/∂w
      b := b − α · ∂J/∂b
  }
```

| Symbol | Name | Role |
|--------|------|------|
| `α` (alpha) | Learning rate | Controls step size (typically 0.001 to 0.1) |
| `∂J/∂w` | Gradient w.r.t. w | Direction and magnitude of steepest ascent |
| `:=` | Assignment | Update the parameter |
| `−` | Minus sign | Move OPPOSITE to gradient (downhill) |

> **Critical:** Both `w` and `b` must be updated **simultaneously** — compute both
> gradients first, THEN update both parameters.

---

## 2. Computing the Partial Derivatives

### 2.1 Recap: Cost Function

```
                  1    m
  J(w, b)  =  ─────  Σ  (wxᵢ + b − yᵢ)²
                2m   i=1
```

### 2.2 Partial Derivative ∂J/∂w

```
  ∂J      ∂     1    m
  ── = ────── [ ───  Σ  (wxᵢ + b − yᵢ)² ]
  ∂w    ∂w     2m   i=1

       1    m
     = ──  Σ  2(wxᵢ + b − yᵢ) · xᵢ          ← chain rule: derivative of (...)² · derivative of inner
       2m  i=1

       1    m
     = ──  Σ  (ŷᵢ − yᵢ) · xᵢ
       m   i=1
```

### 2.3 Partial Derivative ∂J/∂b

```
  ∂J      1    m
  ── =   ──  Σ  (ŷᵢ − yᵢ)
  ∂b      m   i=1
```

> The derivative of `(wxᵢ + b − yᵢ)` with respect to `b` is simply `1`.

### 2.4 Summary of Update Rules

```
┌─────────────────────────────────────────────────────────┐
│  GRADIENT DESCENT UPDATE RULES (Simple Linear Reg)      │
│                                                         │
│                    1   m                                │
│  w := w  −  α · [ ── Σ (ŷᵢ − yᵢ) · xᵢ ]              │
│                    m  i=1                               │
│                                                         │
│                    1   m                                │
│  b := b  −  α · [ ── Σ (ŷᵢ − yᵢ) ]                   │
│                    m  i=1                               │
│                                                         │
│  Both updates use the SAME predictions ŷ                │
│  (computed BEFORE any update in this iteration)         │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Learning Rate (α)

### 3.1 Effect of Learning Rate

```
  Too Small (α = 0.0001)         Just Right (α = 0.01)        Too Large (α = 1.0)

  J │                            J │                           J │
    │╲                             │╲                            │╲     ╱╲
    │ ╲                            │ ╲                           │ ╲   ╱  ╲  ╱
    │  ╲                           │  ╲                          │  ╲ ╱    ╲╱
    │   ╲                          │   ╲                         │   ╳
    │    ╲                         │    ╲____                    │  ╱ ╲
    │     ╲                        │                             │ ╱   ╲
    │      ╲                       │                             │╱     → DIVERGES
    │       ╲____                  │                             │
    └──────────── iter             └──────── iter                └──────── iter
    Takes forever              Converges well               Overshoots & diverges
```

### 3.2 Learning Rate Selection Guide

| α Value | Behavior | Action |
|---------|----------|--------|
| **Way too small** (< 0.0001) | Extremely slow convergence | Increase α |
| **Good range** (0.001 – 0.1) | Smooth, steady decrease in J | Keep it! |
| **Slightly too large** | J oscillates but eventually decreases | Reduce α slightly |
| **Way too large** (> 1) | J increases or oscillates wildly | Reduce α by 10× |

### 3.3 Practical Tip: Try Powers of 10

```
  Try: α ∈ {0.001, 0.003, 0.01, 0.03, 0.1, 0.3, 1.0}

  Plot J vs iterations for each α.
  Pick the largest α that converges smoothly.
```

---

## 4. Convergence

### 4.1 When to Stop?

| Strategy | Implementation |
|----------|---------------|
| **Max iterations** | Stop after N iterations (e.g., 1000) |
| **Cost threshold** | Stop when J < ε (some small tolerance) |
| **Gradient norm** | Stop when `‖∇J‖ < ε` (gradient nearly zero) |
| **Cost change** | Stop when `|J_new − J_old| < ε` (cost barely changing) |

### 4.2 Convergence Check (Recommended)

```python
# Stop when the change in cost is negligibly small
if abs(J_prev - J_current) < 1e-6:
    print(f"Converged at iteration {i}")
    break
```

### 4.3 Cost History Visualization

```
  J (cost)
    │
  50│ ●
    │  ●
  40│   ●
    │    ●
  30│     ●
    │      ●●
  20│        ●●
    │          ●●●
  10│             ●●●●
    │                 ●●●●●●●●●●●  ← converged (flat)
   0│
    └──┬──┬──┬──┬──┬──┬──┬──┬──── iteration
       1  2  3  4  5  6  7  8  ...
```

---

## 5. Feature Scaling

### 5.1 Why Feature Scaling Matters for Gradient Descent

When features have very different scales, the cost function contours become **elongated
ellipses** instead of circles. This causes gradient descent to zigzag and converge slowly.

```
  WITHOUT Scaling                    WITH Scaling
  (x₁: 0-2000, x₂: 0-5)           (x₁: 0-1, x₂: 0-1)

  x₂                                x₂
  │                                  │
  │ ═══════════════════════          │   ╭────╮
  │ ═══════════════════════          │  ╭┤    ├╮
  │ ═══════════════════════          │  │╰─★──╯│
  │ ═══════════════════════          │  ╰─┤  ├─╯
  │ ═══════════════════════          │   ╰────╯
  └────────────────────── x₁         └──────────── x₁

  Elongated → zigzag path            Circular → direct path
  Slow convergence 🐢                Fast convergence 🚀
```

### 5.2 Common Scaling Methods

**Min-Max Normalization (scaling to [0, 1]):**

```
              xᵢ − min(x)
  x_scaled = ──────────────
              max(x) − min(x)
```

**Z-score Standardization (mean=0, std=1):**

```
              xᵢ − μ
  x_scaled = ────────
                σ
```

### 5.3 Comparison

| Method | Range | When to Use |
|--------|-------|-------------|
| **Min-Max** | [0, 1] | Known bounded range, no outliers |
| **Z-score** | ~[−3, 3] | Unknown range, normal-ish distribution |
| **Max Abs** | [−1, 1] | Sparse data (preserves zeros) |

---

## 6. Variants of Gradient Descent

### 6.1 Batch Gradient Descent (BGD)

Uses **all** m training examples to compute the gradient at each step.

```
  for iteration in range(num_iterations):
      gradient_w = (1/m) * Σ_all (ŷᵢ − yᵢ)·xᵢ
      gradient_b = (1/m) * Σ_all (ŷᵢ − yᵢ)
      w = w - α * gradient_w
      b = b - α * gradient_b
```

| Pros | Cons |
|------|------|
| Stable convergence | Slow for large datasets |
| Guaranteed to find minimum (convex J) | Must load all data into memory |
| Smooth cost decrease | Each step is expensive |

### 6.2 Stochastic Gradient Descent (SGD)

Uses **one** random training example per step.

```
  for iteration in range(num_iterations):
      shuffle(training_data)
      for i in range(m):
          gradient_w = (ŷᵢ − yᵢ) · xᵢ
          gradient_b = (ŷᵢ − yᵢ)
          w = w - α * gradient_w
          b = b - α * gradient_b
```

| Pros | Cons |
|------|------|
| Very fast per step | Noisy updates (cost oscillates) |
| Can escape shallow local minima | May never fully converge |
| Online learning capable | Needs learning rate scheduling |

### 6.3 Mini-Batch Gradient Descent

Uses a **batch of B** examples (typically B = 32, 64, 128).

```
  for iteration in range(num_iterations):
      shuffle(training_data)
      for batch in get_batches(data, batch_size=32):
          gradient = compute_gradient(batch)
          w = w - α * gradient
```

| Pros | Cons |
|------|------|
| Balanced speed and stability | Extra hyperparameter (batch size) |
| Vectorized computation (GPU-friendly) | Some noise in updates |
| **Most commonly used in practice** | |

### 6.4 Comparison Summary

```
  Cost over iterations:

  J │ BGD                J │ SGD                J │ Mini-Batch
    │╲                     │╲ ╱╲                  │╲ ╱╲
    │ ╲                    │ ╳  ╲ ╱╲              │ ╳  ╲
    │  ╲                   │╱ ╲  ╳  ╲             │  ╲  ╲
    │   ╲___               │    ╲╱ ╲  ╲___        │   ╲__╲___
    └──────── iter         └──────────── iter      └──────────── iter
    Smooth                 Very noisy              Slightly noisy
```

---

## 7. Batch Gradient Descent — Pseudocode

```
ALGORITHM: Batch Gradient Descent for Linear Regression
─────────────────────────────────────────────────────────
INPUT:
  X[m]     ← feature values (m training examples)
  y[m]     ← target values
  α        ← learning rate
  max_iter ← maximum iterations
  tol      ← convergence tolerance

INITIALIZE:
  w ← 0  (or small random value)
  b ← 0
  J_history ← empty list

FOR iteration = 1 TO max_iter:

    1. COMPUTE PREDICTIONS:
       FOR i = 1 TO m:
           ŷ[i] = w * X[i] + b

    2. COMPUTE COST:
       J = (1/2m) * Σ (ŷ[i] - y[i])²
       APPEND J to J_history

    3. COMPUTE GRADIENTS:
       dw = (1/m) * Σ (ŷ[i] - y[i]) * X[i]
       db = (1/m) * Σ (ŷ[i] - y[i])

    4. UPDATE PARAMETERS:
       w = w - α * dw
       b = b - α * db

    5. CHECK CONVERGENCE:
       IF |J_history[-2] - J_history[-1]| < tol:
           PRINT "Converged at iteration", iteration
           BREAK

OUTPUT: w, b, J_history
```

---

## 8. Complete Python Implementation from Scratch

### 8.1 Simple Linear Regression with Gradient Descent

```python
import numpy as np

class LinearRegressionGD:
    """Simple Linear Regression using Batch Gradient Descent."""

    def __init__(self, learning_rate=0.01, n_iterations=1000, tolerance=1e-6):
        self.lr = learning_rate
        self.n_iter = n_iterations
        self.tol = tolerance
        self.w = None
        self.b = None
        self.cost_history = []

    def fit(self, X, y):
        m = len(y)
        self.w = 0.0
        self.b = 0.0
        self.cost_history = []

        for i in range(self.n_iter):
            # Forward pass: predictions
            y_pred = self.w * X + self.b

            # Compute cost
            cost = np.sum((y_pred - y) ** 2) / (2 * m)
            self.cost_history.append(cost)

            # Compute gradients
            dw = np.sum((y_pred - y) * X) / m
            db = np.sum(y_pred - y) / m

            # Update parameters (simultaneously)
            self.w -= self.lr * dw
            self.b -= self.lr * db

            # Convergence check
            if i > 0 and abs(self.cost_history[-2] - self.cost_history[-1]) < self.tol:
                print(f"Converged at iteration {i}")
                break

        return self

    def predict(self, X):
        return self.w * X + self.b

    def score(self, X, y):
        y_pred = self.predict(X)
        ss_res = np.sum((y - y_pred) ** 2)
        ss_tot = np.sum((y - np.mean(y)) ** 2)
        return 1 - ss_res / ss_tot


# ─── Example Usage ───────────────────────────────────

# Generate data
np.random.seed(42)
X = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], dtype=float)
y = 3 * X + 7 + np.random.randn(10) * 2  # y = 3x + 7 + noise

# Feature scaling (important for GD!)
X_mean, X_std = X.mean(), X.std()
X_scaled = (X - X_mean) / X_std

# Fit model
model = LinearRegressionGD(learning_rate=0.1, n_iterations=500)
model.fit(X_scaled, y)

print(f"\nAfter training:")
print(f"  w (scaled) = {model.w:.4f}")
print(f"  b (scaled) = {model.b:.4f}")
print(f"  R² = {model.score(X_scaled, y):.4f}")

# Convert back to original scale
w_original = model.w / X_std
b_original = model.b - model.w * X_mean / X_std
print(f"\nOriginal scale equation:")
print(f"  ŷ = {w_original:.4f}x + {b_original:.4f}")
print(f"  (True: ŷ ≈ 3x + 7)")
```

### 8.2 Multiple Linear Regression with Gradient Descent

```python
import numpy as np

class MultipleLinearRegressionGD:
    """Multiple Linear Regression using Batch Gradient Descent."""

    def __init__(self, learning_rate=0.01, n_iterations=1000, tolerance=1e-6):
        self.lr = learning_rate
        self.n_iter = n_iterations
        self.tol = tolerance
        self.weights = None
        self.bias = None
        self.cost_history = []

    def fit(self, X, y):
        m, n = X.shape
        self.weights = np.zeros(n)
        self.bias = 0.0
        self.cost_history = []

        for i in range(self.n_iter):
            # Predictions
            y_pred = X @ self.weights + self.bias

            # Cost
            cost = np.sum((y_pred - y) ** 2) / (2 * m)
            self.cost_history.append(cost)

            # Gradients (vectorized)
            errors = y_pred - y
            dw = (1/m) * (X.T @ errors)       # (n,) vector
            db = (1/m) * np.sum(errors)        # scalar

            # Update
            self.weights -= self.lr * dw
            self.bias -= self.lr * db

            # Convergence
            if i > 0 and abs(self.cost_history[-2] - cost) < self.tol:
                print(f"Converged at iteration {i}")
                break

        return self

    def predict(self, X):
        return X @ self.weights + self.bias

    def score(self, X, y):
        y_pred = self.predict(X)
        return 1 - np.sum((y - y_pred)**2) / np.sum((y - np.mean(y))**2)


# ─── Example Usage ──────────────────────────────────────

np.random.seed(42)

# Generate 2-feature dataset
m = 100
X = np.random.randn(m, 2)
true_w = np.array([3.0, -2.0])
true_b = 5.0
y = X @ true_w + true_b + np.random.randn(m) * 0.5

# Fit
model = MultipleLinearRegressionGD(learning_rate=0.1, n_iterations=1000)
model.fit(X, y)

print(f"Learned weights: {model.weights}")
print(f"Learned bias:    {model.bias:.4f}")
print(f"True weights:    {true_w}")
print(f"True bias:       {true_b}")
print(f"R²:              {model.score(X, y):.4f}")
```

### 8.3 Visualizing Convergence

```python
import numpy as np

# After fitting the model (using cost_history from above)
history = model.cost_history

print("\n📉 Cost Convergence:")
print(f"  Initial cost: {history[0]:.4f}")
print(f"  Final cost:   {history[-1]:.4f}")
print(f"  Iterations:   {len(history)}")

# ASCII convergence plot
n_rows = 15
max_cost = max(history[:20]) if len(history) > 20 else max(history)
step = max(1, len(history) // 30)

print(f"\n  Cost vs Iteration:")
for idx in range(0, min(len(history), 30 * step), step):
    bar_len = int((history[idx] / max_cost) * 50) if max_cost > 0 else 0
    bar_len = max(0, min(bar_len, 50))
    print(f"  iter {idx:>4d} │{'█' * bar_len} {history[idx]:.4f}")
```

---

## 9. Gradient Descent vs Normal Equation

| Criteria | Gradient Descent | Normal Equation |
|----------|-----------------|-----------------|
| **Learning rate needed?** | ✅ Yes (must tune α) | ❌ No |
| **Iterations needed?** | ✅ Yes (many iterations) | ❌ No (one computation) |
| **Feature scaling needed?** | ✅ Yes (important) | ❌ No |
| **Computational complexity** | O(kmn) per k iterations | O(n³) for matrix inversion |
| **Works with large n (features)?** | ✅ Yes | ❌ Slow when n > 10,000 |
| **Works with large m (samples)?** | ✅ Yes (mini-batch) | ✅ Yes |
| **Generalizes to other algorithms?** | ✅ Yes (neural nets, etc.) | ❌ Linear regression only |

---

## 10. Real-World Applications

| Scenario | Why Gradient Descent? |
|----------|----------------------|
| **Training neural networks** | No closed-form solution exists; GD is the only option |
| **Large-scale regression** | Millions of features make matrix inversion impossible |
| **Online learning** | SGD can update the model as new data arrives |
| **Logistic regression** | Cost function is non-linear; no closed-form solution |
| **Deep learning** | Adam, RMSProp, etc. are all GD variants |

---

## 11. Summary Table

| Concept | Key Point |
|---------|-----------|
| **Gradient descent** | Iteratively move parameters in the direction of steepest descent |
| **Update rule** | `w := w − α · ∂J/∂w` |
| **∂J/∂w** | `(1/m) Σ(ŷᵢ−yᵢ)·xᵢ` |
| **∂J/∂b** | `(1/m) Σ(ŷᵢ−yᵢ)` |
| **Learning rate α** | Too small → slow; too large → diverge |
| **Feature scaling** | Essential for fast convergence |
| **Convergence** | Check if `|J_new − J_old| < ε` |
| **Batch GD** | All samples per step; smooth but slow |
| **SGD** | One sample per step; fast but noisy |
| **Mini-batch GD** | B samples per step; best of both worlds |
| **Simultaneous update** | Compute ALL gradients before updating ANY parameter |

---

## 12. ✏️ Revision Questions

1. **Write the gradient descent update rules** for simple linear regression.
   Why must w and b be updated simultaneously?

2. **What happens if the learning rate is too large?** Sketch the cost-vs-iteration
   curve for: (a) a good α, (b) an α that's too small, (c) an α that's too large.

3. **Why is feature scaling important** for gradient descent? What problem does it
   solve? Name two common scaling methods.

4. **Compare Batch GD, SGD, and Mini-batch GD.** Create a table showing
   speed, stability, and memory requirements.

5. **Given** `x = [1, 2, 3]`, `y = [2, 4, 5]`, `w = 1`, `b = 0`, `α = 0.1`:
   Perform ONE iteration of gradient descent by hand. Show the new values of w and b.

6. **You run gradient descent** and the cost starts at 150, drops to 50 after 100
   iterations, then barely changes from 50.0001 to 49.9999 over the next 500
   iterations. What should you do? Is this convergence or a problem?

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| [← 03 – Cost Function (MSE)](03-cost-function-mse.md) | [05 – Normal Equation →](05-normal-equation.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
