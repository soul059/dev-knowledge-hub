# 📘 Chapter 4: Cost Function & Log Loss

> **Unit 5: Logistic Regression** | **Section 4 of 7**
> Derive the cost function for logistic regression from first principles. Understand why MSE fails, how log loss (binary cross-entropy) works, its convexity, connection to Maximum Likelihood Estimation, and gradient computation.

---

## 📑 Table of Contents

1. [Why MSE Doesn't Work for Logistic Regression](#1-why-mse-doesnt-work-for-logistic-regression)
2. [Log Loss (Binary Cross-Entropy): The Formula](#2-log-loss-binary-cross-entropy-the-formula)
3. [Intuition Behind Log Loss](#3-intuition-behind-log-loss)
4. [Convexity of Log Loss](#4-convexity-of-log-loss)
5. [Connection to Maximum Likelihood Estimation](#5-connection-to-maximum-likelihood-estimation)
6. [Gradient Derivation](#6-gradient-derivation)
7. [Worked Examples](#7-worked-examples)
8. [Python Implementation](#8-python-implementation)
9. [Summary Table](#9-summary-table)
10. [Revision Questions](#10-revision-questions)

---

## 1. Why MSE Doesn't Work for Logistic Regression

### The Temptation

It's natural to try the MSE cost function that worked for linear regression:

```
    J_MSE(w, b) = (1/m) Σᵢ₌₁ᵐ (yᵢ - ŷᵢ)²
    
    where ŷᵢ = σ(wᵀxᵢ + b)
```

### Problem: Non-Convexity

When ŷ = σ(wᵀx + b) is plugged into MSE, the resulting cost function is **non-convex**:

```
    J_MSE with sigmoid:  (y - σ(wᵀx + b))²
    
    Cost
    J(w)
    │
    │ ╲      ╱╲        ╱
    │  ╲    ╱  ╲      ╱      ← Multiple local minima!
    │   ╲  ╱    ╲    ╱
    │    ╲╱      ╲  ╱
    │  local      ╲╱
    │  min       global
    │             min
    └────────────────────── w
    
    Gradient descent may converge to a LOCAL minimum
    instead of the GLOBAL minimum!
```

### Why the Non-Convexity Occurs

```
The sigmoid σ(z) is nonlinear, so:

    (y - σ(wᵀx + b))²
    
is the square of a nonlinear function of w.

Composition of convex functions is NOT necessarily convex!

    f(w) = σ(w)     ← S-shaped (sigmoid is not convex)
    g(f) = (y-f)²   ← convex in f, BUT...
    g(f(w))          ← NOT convex in w!
```

### Comparison: What We Need

| Property | MSE for Logistic Reg | Log Loss |
|----------|---------------------|----------|
| **Convex?** | ❌ No | ✅ Yes |
| **Unique minimum?** | ❌ Multiple local minima | ✅ One global minimum |
| **Gradient descent reliable?** | ❌ May get stuck | ✅ Always converges |
| **Probabilistic interpretation?** | ❌ No | ✅ MLE connection |
| **Penalizes confident wrong predictions?** | Somewhat | ✅ Severely |

---

## 2. Log Loss (Binary Cross-Entropy): The Formula

### The Cost Function

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  For a SINGLE training example:                                  │
│                                                                  │
│  L(ŷ, y) = -[y · log(ŷ) + (1-y) · log(1-ŷ)]                  │
│                                                                  │
│  For the ENTIRE dataset (m examples):                            │
│                                                                  │
│         1   m                                                    │
│  J = - ─── Σ  [yᵢ · log(ŷᵢ) + (1-yᵢ) · log(1-ŷᵢ)]           │
│         m  i=1                                                   │
│                                                                  │
│  where ŷᵢ = σ(wᵀxᵢ + b) = predicted probability of class 1    │
│        yᵢ = actual label (0 or 1)                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Breaking Down the Two Cases

```
When y = 1 (actual positive):
    L = -log(ŷ)
    
    We want ŷ close to 1
    If ŷ → 1:  L = -log(1) = 0     ← no penalty
    If ŷ → 0:  L = -log(0) → ∞    ← infinite penalty!

When y = 0 (actual negative):
    L = -log(1-ŷ)
    
    We want ŷ close to 0
    If ŷ → 0:  L = -log(1) = 0     ← no penalty
    If ŷ → 1:  L = -log(0) → ∞    ← infinite penalty!
```

### Visual: The Two Components

```
    Loss
    │
  5 ┤
    │
  4 ┤ ╲                         ╱
    │  ╲                       ╱
  3 ┤   ╲                    ╱
    │    ╲                  ╱
  2 ┤     ╲               ╱
    │      ╲             ╱
  1 ┤       ╲╲         ╱╱
    │         ╲╲     ╱╱
    │           ╲╲╱╱╱
  0 ┤─────────────────────────
    └──┬──┬──┬──┬──┬──┬──┬──┬──
       0 .1 .2 .3 .4 .5 .6 .7 .8 .9 1.0
                    ŷ
    
    Left curve:   -log(ŷ)     (loss when y=1)
    Right curve:  -log(1-ŷ)   (loss when y=0)
    
    Key insight: Loss → ∞ as prediction moves AWAY from truth
```

---

## 3. Intuition Behind Log Loss

### The Penalization Logic

```
Scenario Analysis:

┌──────────────────────────────────────────────────────────────┐
│ Actual y │ Predicted ŷ │ Confident? │ Correct? │ Loss       │
│──────────│─────────────│────────────│──────────│────────────│
│    1     │    0.99     │   Very     │   ✓ Yes  │  0.01  ✓  │
│    1     │    0.80     │   Somewhat │   ✓ Yes  │  0.22     │
│    1     │    0.50     │   No       │   Borderline│ 0.69   │
│    1     │    0.20     │   Somewhat │   ✗ No   │  1.61     │
│    1     │    0.01     │   Very     │   ✗ No   │  4.61  ✗  │
│──────────│─────────────│────────────│──────────│────────────│
│    0     │    0.01     │   Very     │   ✓ Yes  │  0.01  ✓  │
│    0     │    0.20     │   Somewhat │   ✓ Yes  │  0.22     │
│    0     │    0.50     │   No       │   Borderline│ 0.69   │
│    0     │    0.80     │   Somewhat │   ✗ No   │  1.61     │
│    0     │    0.99     │   Very     │   ✗ No   │  4.61  ✗  │
└──────────────────────────────────────────────────────────────┘

KEY INSIGHT: The loss grows LOGARITHMICALLY for wrong predictions.
Being CONFIDENTLY WRONG is punished MUCH more harshly than
being uncertain.
```

### The Asymmetric Penalty

```
    Loss
    │
  6 ┤                                        Being 99%
    │                                        confident and
  5 ┤                                        WRONG costs
    │                                        ~100x more
  4 ┤                                        than being
    │                                        right!
  3 ┤
    │
  2 ┤
    │               ← Moderate penalty for
  1 ┤                  uncertain prediction
    │
  0 ┤═══════╗                      ╔════════  ← Near-zero
    └───┬───┬───┬───┬───┬───┬───┬───┬───       cost when
       0.01 0.1 0.2 0.3 0.5 0.7 0.8 0.9 0.99  correct
                        ŷ
    
    This heavy penalty for confident wrong predictions
    forces the model to be CALIBRATED — its probabilities
    must be meaningful.
```

### Why Log (Not Square, Absolute, etc.)?

| Loss Function | Formula (y=1) | Gradient at ŷ→0 | Convex with σ? |
|---|---|---|---|
| **Log Loss** | -log(ŷ) | → -∞ (strong signal) | ✅ Yes |
| **Squared** | (1-ŷ)² | -2(1-ŷ) → -2 (weak) | ❌ No |
| **Absolute** | \|1-ŷ\| | -1 (constant) | Issues at ŷ=1 |
| **Hinge** | max(0, 1-z) | -1 or 0 (SVM-style) | ✅ Yes |

Log loss provides a **strong gradient signal** even when the model is confidently wrong, ensuring the model keeps learning.

---

## 4. Convexity of Log Loss

### What is Convexity?

```
Convex Function:                 Non-Convex Function:

    J                                J
    │                                │
    │ ╲                 ╱            │ ╲      ╱╲       ╱
    │  ╲               ╱             │  ╲    ╱  ╲     ╱
    │   ╲             ╱              │   ╲  ╱    ╲   ╱
    │    ╲           ╱               │    ╲╱      ╲ ╱
    │     ╲         ╱                │   local    global
    │      ╲       ╱                 │   min       min
    │       ╲     ╱                  │
    │        ╲   ╱                   │
    │         ╲ ╱                    │
    │          ●  ← global min       │
    └──────────────── w              └──────────────── w
    
    ANY starting point              Starting here → stuck
    reaches the minimum!            in local minimum!
```

### Proof Sketch: Log Loss is Convex

```
For a single example with y = 1:

    L(w) = -log(σ(wᵀx + b))
    
    The second derivative (Hessian) is:
    
    ∂²L/∂w² = σ(z)(1-σ(z)) · xxᵀ
    
    Since:
    • σ(z)(1-σ(z)) > 0 for all z (always positive)
    • xxᵀ is positive semi-definite (outer product)
    
    → The Hessian is positive semi-definite
    → The function is CONVEX!  ✓
```

### Visualization of Convex Log Loss

```
    J(w)  (Log Loss)
    │
    │ ╲
    │  ╲
    │   ╲
    │    ╲
    │     ╲
    │      ╲
    │       ╲
    │        ╲
    │         ╲     ╱
    │          ╲   ╱
    │           ╲ ╱
    │            ● ← UNIQUE global minimum!
    │
    └──────────────────── w
    
    One bowl shape → gradient descent ALWAYS finds
    the optimal weights!
```

---

## 5. Connection to Maximum Likelihood Estimation

### The Big Picture

Log loss isn't just a convenient function — it's the **negative log-likelihood** of the data under the logistic model. Minimizing log loss = **Maximum Likelihood Estimation (MLE)**.

### Step-by-Step Derivation

```
Step 1: Define the probability model

    P(y=1|x; w, b) = σ(wᵀx + b) = ŷ
    P(y=0|x; w, b) = 1 - σ(wᵀx + b) = 1 - ŷ
    
    Compact form:
    P(y|x; w, b) = ŷ^y · (1-ŷ)^(1-y)


Step 2: Write the likelihood of the ENTIRE dataset

    Assuming i.i.d. (independent and identically distributed):
    
    L(w, b) = ∏ᵢ₌₁ᵐ P(yᵢ|xᵢ; w, b)
    
            = ∏ᵢ₌₁ᵐ ŷᵢ^(yᵢ) · (1-ŷᵢ)^(1-yᵢ)


Step 3: Take the log (log-likelihood)

    Products → sums (much easier to work with):
    
    ℓ(w, b) = log L(w, b)
    
            = Σᵢ₌₁ᵐ [yᵢ · log(ŷᵢ) + (1-yᵢ) · log(1-ŷᵢ)]


Step 4: Maximize log-likelihood = Minimize negative log-likelihood

    Maximize:  ℓ(w, b) = Σ [yᵢ·log(ŷᵢ) + (1-yᵢ)·log(1-ŷᵢ)]
    
    ≡ Minimize: J(w, b) = -(1/m) Σ [yᵢ·log(ŷᵢ) + (1-yᵢ)·log(1-ŷᵢ)]
    
    This is EXACTLY the log loss / binary cross-entropy!

┌──────────────────────────────────────────────────────────────┐
│  Log Loss = Negative Log-Likelihood (scaled by 1/m)         │
│                                                              │
│  Minimizing log loss = Maximum Likelihood Estimation         │
│                                                              │
│  This gives log loss a PRINCIPLED statistical foundation!    │
└──────────────────────────────────────────────────────────────┘
```

### Why MLE Connection Matters

1. **Theoretical justification:** We're not just picking a convenient function — we're finding the parameters that make the observed data most probable.
2. **Asymptotic optimality:** MLE estimators are consistent and efficient (achieve the Cramér-Rao lower bound).
3. **Probabilistic calibration:** The resulting probabilities are well-calibrated (a predicted 70% probability really means ~70% chance of being positive).

---

## 6. Gradient Derivation

### The Gradient of Log Loss

```
    J(w, b) = -(1/m) Σᵢ₌₁ᵐ [yᵢ·log(ŷᵢ) + (1-yᵢ)·log(1-ŷᵢ)]
    
    where ŷᵢ = σ(zᵢ) and zᵢ = wᵀxᵢ + b
```

### Step-by-Step Gradient Derivation

```
We need ∂J/∂wⱼ. Start with a SINGLE example (drop subscript i):

    L = -[y·log(ŷ) + (1-y)·log(1-ŷ)]

Step 1: ∂L/∂ŷ (how loss changes with prediction)

    ∂L/∂ŷ = -[y/ŷ - (1-y)/(1-ŷ)]
           = -[y/ŷ - (1-y)/(1-ŷ)]

Step 2: ∂ŷ/∂z (sigmoid derivative)

    ŷ = σ(z)
    ∂ŷ/∂z = σ(z)·(1-σ(z)) = ŷ·(1-ŷ)

Step 3: ∂z/∂wⱼ (linear part)

    z = w₁x₁ + w₂x₂ + ... + wₙxₙ + b
    ∂z/∂wⱼ = xⱼ

Step 4: Chain rule

    ∂L/∂wⱼ = ∂L/∂ŷ · ∂ŷ/∂z · ∂z/∂wⱼ
    
    = -[y/ŷ - (1-y)/(1-ŷ)] · ŷ·(1-ŷ) · xⱼ
    
    = -[y·(1-ŷ) - (1-y)·ŷ] · xⱼ
    
    = -[y - y·ŷ - ŷ + y·ŷ] · xⱼ
    
    = -[y - ŷ] · xⱼ
    
    = (ŷ - y) · xⱼ
```

### The Beautiful Result

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ∂J/∂wⱼ = (1/m) Σᵢ₌₁ᵐ (ŷᵢ - yᵢ) · xᵢⱼ                  │
│                                                              │
│  ∂J/∂b  = (1/m) Σᵢ₌₁ᵐ (ŷᵢ - yᵢ)                          │
│                                                              │
│  In vector form:                                             │
│                                                              │
│  ∂J/∂w = (1/m) Xᵀ(ŷ - y)                                  │
│  ∂J/∂b = (1/m) Σ(ŷ - y)                                    │
│                                                              │
│  where ŷ = σ(Xw + b)                                       │
│                                                              │
│  THIS IS THE SAME FORM AS LINEAR REGRESSION!                 │
│  (except ŷ = σ(z) instead of ŷ = z)                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Gradient Interpretation

```
    ∂J/∂wⱼ = (1/m) Σ (ŷᵢ - yᵢ) · xᵢⱼ
                      ═══════════   ════
                        "error"    "feature"
    
    The gradient is the AVERAGE of (prediction error × feature value)
    
    If ŷᵢ > yᵢ (over-predicted):  error > 0 → decrease wⱼ for positive xᵢⱼ
    If ŷᵢ < yᵢ (under-predicted): error < 0 → increase wⱼ for positive xᵢⱼ
    
    The update direction is EXACTLY what makes intuitive sense!
```

---

## 7. Worked Examples

### Example 1: Computing Log Loss by Hand

```
Given: Two training examples
    Example 1: x = [1, 2], y = 1, ŷ = 0.9
    Example 2: x = [3, 1], y = 0, ŷ = 0.3

Loss for Example 1 (y=1):
    L₁ = -[1·log(0.9) + 0·log(0.1)]
       = -log(0.9)
       = -(-0.1054)
       = 0.1054

Loss for Example 2 (y=0):
    L₂ = -[0·log(0.3) + 1·log(0.7)]
       = -log(0.7)
       = -(-0.3567)
       = 0.3567

Total cost:
    J = (1/2)(L₁ + L₂)
      = (1/2)(0.1054 + 0.3567)
      = (1/2)(0.4621)
      = 0.2310
```

### Example 2: Effect of Confident Wrong Predictions

```
Scenario A: Uncertain wrong prediction
    y = 1, ŷ = 0.4
    L = -log(0.4) = 0.916

Scenario B: Confident wrong prediction
    y = 1, ŷ = 0.01
    L = -log(0.01) = 4.605

Scenario C: Very confident wrong prediction
    y = 1, ŷ = 0.001
    L = -log(0.001) = 6.908

Scenario A → B: 5× increase in loss (ŷ dropped from 0.4 to 0.01)
Scenario B → C: 1.5× increase (ŷ dropped from 0.01 to 0.001)

The loss grows LOGARITHMICALLY — each order of magnitude
in confidence adds a fixed amount of loss (~2.3).
```

### Example 3: Gradient Computation

```
Given one training example:
    x = [2, 3], y = 1, w = [0.5, -0.3], b = 0.1

Step 1: Compute z
    z = 0.5(2) + (-0.3)(3) + 0.1
      = 1.0 - 0.9 + 0.1
      = 0.2

Step 2: Compute ŷ
    ŷ = σ(0.2) = 1/(1+e^(-0.2)) ≈ 0.5498

Step 3: Compute error
    error = ŷ - y = 0.5498 - 1 = -0.4502

Step 4: Compute gradients
    ∂L/∂w₁ = error · x₁ = (-0.4502)(2) = -0.9004
    ∂L/∂w₂ = error · x₂ = (-0.4502)(3) = -1.3507
    ∂L/∂b  = error      = -0.4502

Step 5: Update (learning rate α = 0.1)
    w₁_new = 0.5 - 0.1(-0.9004)  = 0.5 + 0.0900 = 0.5900
    w₂_new = -0.3 - 0.1(-1.3507) = -0.3 + 0.1351 = -0.1649
    b_new  = 0.1 - 0.1(-0.4502)  = 0.1 + 0.0450 = 0.1450

Interpretation: All weights moved to INCREASE ŷ toward y=1 ✓
```

### Example 4: MLE Interpretation

```
Given dataset:
    x₁ = 1, y₁ = 1 → P(y₁=1|x₁) = ŷ₁ = 0.8
    x₂ = 2, y₂ = 0 → P(y₂=0|x₂) = 1-ŷ₂ = 0.7
    x₃ = 3, y₃ = 1 → P(y₃=1|x₃) = ŷ₃ = 0.9

Likelihood:
    L = P(y₁=1)·P(y₂=0)·P(y₃=1)
      = 0.8 × 0.7 × 0.9
      = 0.504

Log-likelihood:
    ℓ = log(0.8) + log(0.7) + log(0.9)
      = -0.2231 + (-0.3567) + (-0.1054)
      = -0.6852

Negative log-likelihood (our cost):
    J = -ℓ/m = 0.6852/3 = 0.2284

MLE goal: find w, b that MAXIMIZE the likelihood (= MINIMIZE the cost)
```

---

## 8. Python Implementation

### Log Loss from Scratch

```python
import numpy as np

def sigmoid(z):
    """Numerically stable sigmoid."""
    return np.where(
        z >= 0,
        1 / (1 + np.exp(-z)),
        np.exp(z) / (1 + np.exp(z))
    )

def log_loss(y_true, y_pred, epsilon=1e-15):
    """
    Binary cross-entropy / log loss.
    
    Parameters:
    -----------
    y_true : array of shape (m,) — true labels (0 or 1)
    y_pred : array of shape (m,) — predicted probabilities
    epsilon : small value to prevent log(0)
    
    Returns:
    --------
    cost : scalar — average log loss
    """
    # Clip predictions to prevent log(0)
    y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
    
    # Compute log loss
    cost = -(1/len(y_true)) * np.sum(
        y_true * np.log(y_pred) + (1 - y_true) * np.log(1 - y_pred)
    )
    return cost

# ─── Test with known values ───
y_true = np.array([1, 0, 1, 1, 0])
y_pred = np.array([0.9, 0.1, 0.8, 0.7, 0.3])

cost = log_loss(y_true, y_pred)
print(f"Log Loss: {cost:.6f}")

# Compare with sklearn
from sklearn.metrics import log_loss as sklearn_log_loss
cost_sklearn = sklearn_log_loss(y_true, y_pred)
print(f"sklearn Log Loss: {cost_sklearn:.6f}")
print(f"Match: {np.isclose(cost, cost_sklearn)}")
```

### Demonstrating Why MSE Fails (Non-Convexity)

```python
# ─── Show non-convexity of MSE with sigmoid ───

def mse_cost(w, X, y):
    """MSE cost for logistic model (NON-CONVEX!)."""
    z = X @ w
    y_pred = sigmoid(z)
    return np.mean((y - y_pred) ** 2)

def log_loss_cost(w, X, y):
    """Log loss cost for logistic model (CONVEX)."""
    z = X @ w
    y_pred = sigmoid(z)
    y_pred = np.clip(y_pred, 1e-15, 1 - 1e-15)
    return -np.mean(y * np.log(y_pred) + (1-y) * np.log(1-y_pred))

# Create a simple 1D example
np.random.seed(42)
X = np.random.randn(50, 1)
y = (X[:, 0] > 0).astype(float)

# Compute costs for a range of w values
w_range = np.linspace(-10, 10, 200)
mse_costs = [mse_cost(np.array([w]), X, y) for w in w_range]
log_costs = [log_loss_cost(np.array([w]), X, y) for w in w_range]

# Check convexity: second differences should be positive for convex
mse_second_diff = np.diff(mse_costs, n=2)
log_second_diff = np.diff(log_costs, n=2)

print("MSE Cost Function:")
print(f"  Min cost: {min(mse_costs):.4f} at w = {w_range[np.argmin(mse_costs)]:.2f}")
print(f"  All second differences positive (convex)? {np.all(mse_second_diff > -1e-10)}")

print("\nLog Loss Cost Function:")
print(f"  Min cost: {min(log_costs):.4f} at w = {w_range[np.argmin(log_costs)]:.2f}")
print(f"  All second differences positive (convex)? {np.all(log_second_diff > -1e-10)}")
```

### Gradient Computation Implementation

```python
def compute_gradients(X, y, w, b):
    """
    Compute gradients of log loss w.r.t. weights and bias.
    
    Parameters:
    -----------
    X : array of shape (m, n) — features
    y : array of shape (m,)   — labels
    w : array of shape (n,)   — weights
    b : float                 — bias
    
    Returns:
    --------
    dw : array of shape (n,) — gradient w.r.t. weights
    db : float               — gradient w.r.t. bias
    """
    m = len(y)
    
    # Forward pass
    z = X @ w + b
    y_pred = sigmoid(z)
    
    # Gradient computation
    error = y_pred - y          # shape: (m,)
    dw = (1/m) * (X.T @ error)  # shape: (n,)
    db = (1/m) * np.sum(error)  # scalar
    
    return dw, db

# ─── Test gradient computation ───
np.random.seed(42)
X = np.array([[2, 3], [1, -1], [0, 2], [-1, 1]])
y = np.array([1, 0, 1, 0])
w = np.array([0.5, -0.3])
b = 0.1

dw, db = compute_gradients(X, y, w, b)
print(f"Weights: {w}")
print(f"Bias: {b}")
print(f"Gradient dw: {dw}")
print(f"Gradient db: {db}")

# Verify with numerical gradients
def numerical_gradient(X, y, w, b, param='w', idx=0, eps=1e-5):
    """Compute numerical gradient for verification."""
    if param == 'w':
        w_plus = w.copy(); w_plus[idx] += eps
        w_minus = w.copy(); w_minus[idx] -= eps
        cost_plus = log_loss(y, sigmoid(X @ w_plus + b))
        cost_minus = log_loss(y, sigmoid(X @ w_minus + b))
    else:  # bias
        cost_plus = log_loss(y, sigmoid(X @ w + b + eps))
        cost_minus = log_loss(y, sigmoid(X @ w + b - eps))
    return (cost_plus - cost_minus) / (2 * eps)

# Compare analytical vs numerical gradients
print(f"\nGradient verification:")
for j in range(len(w)):
    num_grad = numerical_gradient(X, y, w, b, 'w', j)
    print(f"  dw[{j}]: analytical = {dw[j]:.8f}, numerical = {num_grad:.8f}, "
          f"match = {np.isclose(dw[j], num_grad, atol=1e-5)}")

num_db = numerical_gradient(X, y, w, b, 'b')
print(f"  db:    analytical = {db:.8f}, numerical = {num_db:.8f}, "
      f"match = {np.isclose(db, num_db, atol=1e-5)}")
```

### Log Loss Behavior Analysis

```python
# ─── Analyze how log loss penalizes predictions ───

print("Log Loss for different predictions:")
print(f"{'y':>4} {'ŷ':>6} {'Loss':>10} {'Description'}")
print("-" * 55)

test_cases = [
    (1, 0.99, "Correct, very confident"),
    (1, 0.90, "Correct, confident"),
    (1, 0.70, "Correct, somewhat confident"),
    (1, 0.50, "Uncertain (boundary)"),
    (1, 0.30, "Wrong, somewhat confident"),
    (1, 0.10, "Wrong, confident"),
    (1, 0.01, "Wrong, very confident"),
]

for y_val, y_hat, desc in test_cases:
    loss = -(y_val * np.log(y_hat) + (1-y_val) * np.log(1-y_hat))
    print(f"{y_val:>4d} {y_hat:>6.2f} {loss:>10.4f}   {desc}")

# Show the exponential growth of loss for wrong predictions
print("\nLog loss grows logarithmically:")
for ŷ in [0.5, 0.1, 0.01, 0.001, 0.0001]:
    loss = -np.log(ŷ)
    print(f"  -log({ŷ}) = {loss:.4f}")
```

### Complete Cost Function with sklearn Comparison

```python
from sklearn.datasets import make_classification
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

# ─── Full example ───
X, y = make_classification(n_samples=500, n_features=5, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Train with sklearn
model = LogisticRegression(max_iter=1000, random_state=42)
model.fit(X_train, y_train)

# Get predictions
y_pred_proba = model.predict_proba(X_test)[:, 1]

# Compute log loss manually
manual_loss = log_loss(y_test, y_pred_proba)
sklearn_loss = sklearn_log_loss(y_test, y_pred_proba)

print(f"Manual Log Loss:  {manual_loss:.6f}")
print(f"sklearn Log Loss: {sklearn_loss:.6f}")
print(f"Match: {np.isclose(manual_loss, sklearn_loss, atol=1e-6)}")

# Show cost at different stages of training
print(f"\nCost analysis:")
print(f"  Random prediction (ŷ=0.5): {log_loss(y_test, np.full_like(y_test, 0.5, dtype=float)):.4f}")
print(f"  Trained model prediction:   {manual_loss:.4f}")
print(f"  Perfect prediction (clipped): {log_loss(y_test, np.where(y_test==1, 0.9999, 0.0001)):.4f}")
```

---

## 9. Summary Table

| Concept | Formula / Key Point | Why It Matters |
|---------|-------------------|----------------|
| **MSE for classification** | J = (1/m)Σ(y - σ(z))² | ❌ Non-convex → local minima |
| **Log Loss (single)** | L = -[y·log(ŷ) + (1-y)·log(1-ŷ)] | Penalizes confident wrong predictions |
| **Log Loss (dataset)** | J = -(1/m)Σ[y·log(ŷ) + (1-y)·log(1-ŷ)] | Convex → guaranteed global minimum |
| **When y=1** | L = -log(ŷ) | Want ŷ→1; cost→∞ as ŷ→0 |
| **When y=0** | L = -log(1-ŷ) | Want ŷ→0; cost→∞ as ŷ→1 |
| **Convexity** | Hessian is PSD | Gradient descent always converges |
| **MLE connection** | J = -log-likelihood/m | Principled statistical foundation |
| **Gradient (weights)** | ∂J/∂w = (1/m)Xᵀ(ŷ-y) | Same form as linear regression! |
| **Gradient (bias)** | ∂J/∂b = (1/m)Σ(ŷ-y) | Average of prediction errors |
| **Numerical stability** | Clip ŷ away from 0 and 1 | Prevent log(0) = -∞ |

---

## 10. Revision Questions

### Q1: Log Loss Computation
**Q:** Compute the log loss for a dataset with 3 examples: (y=1, ŷ=0.8), (y=0, ŷ=0.2), (y=1, ŷ=0.6). Show your work.

**A:**
- L₁ = -log(0.8) = 0.2231
- L₂ = -log(1-0.2) = -log(0.8) = 0.2231
- L₃ = -log(0.6) = 0.5108
- J = (1/3)(0.2231 + 0.2231 + 0.5108) = (1/3)(0.9570) = **0.3190**

---

### Q2: Why Not MSE?
**Q:** Explain with a specific example why MSE creates a non-convex cost surface for logistic regression. Draw what the cost function looks like.

**A:** Consider ŷ = σ(w·x) with a single training example (x=1, y=1). MSE = (1-σ(w))². As w→-∞, σ(w)→0, so MSE→1. As w→+∞, σ(w)→1, so MSE→0. Near w=0, σ(w)≈0.5, so MSE≈0.25. The curve looks like: starts at 1, drops to 0 as w increases. But for multiple examples with both classes, the contributions create bumps and local minima because the sigmoid's nonlinearity causes the squared terms to interact non-convexly.

---

### Q3: MLE Derivation
**Q:** Show that minimizing log loss is equivalent to maximizing the likelihood of the observed data. What assumption about the data is required?

**A:** The likelihood is L = ∏ P(yᵢ|xᵢ) = ∏ ŷᵢ^yᵢ · (1-ŷᵢ)^(1-yᵢ). Taking log: ℓ = Σ[yᵢ·log(ŷᵢ) + (1-yᵢ)·log(1-ŷᵢ)]. Maximizing ℓ ≡ minimizing -ℓ/m = log loss. The key assumption is **independence** (i.i.d.) — each example's probability is independent, so the joint probability is the product.

---

### Q4: Gradient Derivation
**Q:** Why is it mathematically elegant that the gradient of log loss for logistic regression has the same form as the gradient of MSE for linear regression? What's different between them?

**A:** Both have gradient ∂J/∂wⱼ = (1/m)Σ(ŷᵢ - yᵢ)·xᵢⱼ. The elegance comes from the fact that the log loss derivative and the sigmoid derivative perfectly cancel, leaving the simple (ŷ-y)·x form. The difference is in what ŷ means: for linear regression, **ŷ = wᵀx + b** (linear); for logistic regression, **ŷ = σ(wᵀx + b)** (nonlinear). So the gradients look the same but behave differently because ŷ is a different function of w.

---

### Q5: Penalty Analysis
**Q:** A model predicts P(y=1) = 0.95 for a sample that is actually y=0. Calculate the log loss for this single sample. How does this compare to predicting P(y=1) = 0.5 for the same sample?

**A:**
- Confident wrong: L = -log(1-0.95) = -log(0.05) = **2.996**
- Uncertain: L = -log(1-0.5) = -log(0.5) = **0.693**
- The confident wrong prediction is penalized 2.996/0.693 ≈ **4.3× more** than the uncertain prediction. This heavy penalty forces the model to not just classify correctly, but to calibrate its confidence appropriately.

---

### Q6: Numerical Stability
**Q:** What happens if a model predicts ŷ = 0 or ŷ = 1 exactly? How is this handled in practice? Why can sigmoid never actually output exactly 0 or 1?

**A:** If ŷ = 0 and y = 1: loss = -log(0) = **+∞** (undefined). If ŷ = 1 and y = 0: loss = -log(1-1) = -log(0) = **+∞**. In practice, predictions are **clipped** to [ε, 1-ε] where ε ≈ 10⁻¹⁵. The sigmoid σ(z) = 1/(1+e⁻ᶻ) can never exactly equal 0 or 1 because e⁻ᶻ is always positive (never zero or infinity in exact math). However, floating-point arithmetic can round to 0 or 1 for extreme z values, hence the clipping.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 3: Decision Boundary](03-decision-boundary.md) | **Chapter 4: Cost Function & Log Loss** | [Chapter 5: Gradient Descent →](05-gradient-descent-logistic.md) |

### Unit 5 Chapters
1. [Classification Problem](01-classification-problem.md)
2. [Sigmoid Function](02-sigmoid-function.md)
3. [Decision Boundary](03-decision-boundary.md)
4. **📍 Cost Function & Log Loss** ← You are here
5. [Gradient Descent for Logistic Regression](05-gradient-descent-logistic.md)
6. [Multi-Class Classification](06-multi-class-classification.md)
7. [Regularization](07-regularization.md)

---

> *"The right cost function doesn't just measure error — it guides the model toward truth with the perfect balance of severity and forgiveness."*

© 2025 AI/ML Study Notes. All rights reserved.
