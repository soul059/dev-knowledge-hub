# Chapter 6: Convex Optimization

> **Unit 6 · Optimization · Module 6 of 7**

---

## 6.1 Overview

Convex optimization is the study of minimizing convex functions over convex sets. It occupies a privileged position in optimization theory because **every local minimum is a global minimum** — there are no traps. Many classical ML models (linear regression, logistic regression, SVM) are convex optimization problems, which is why they have well-understood, reliable solutions.

---

## 6.2 Convex Sets

A set `C ⊆ ℝⁿ` is **convex** if, for any two points `x, y ∈ C`, the line segment between them lies entirely in C:

```
∀ x, y ∈ C,  ∀ λ ∈ [0,1]:   λx + (1−λ)y ∈ C
```

### Visual Examples

```
  Convex Sets:                    Non-Convex Sets:

  ╭──────╮    ╱╲                  ╭──╮  ╭──╮     ╭────╮
  │      │   ╱  ╲                 │  ╰──╯  │     │  ╭─╯
  │  ●───●  ╱ ●──●               │ ●╌╌╌╌● │     │  │
  │      │ ╱    ╲                 │        │     ╰──╯
  ╰──────╯ ╲────╱                 ╰────────╯

  ●───● line stays inside         ●╌╌╌╌● line goes outside!
```

### Common Convex Sets

| Set | Description |
|-----|-------------|
| Hyperplanes | `{x : aᵀx = b}` |
| Half-spaces | `{x : aᵀx ≤ b}` |
| Balls / Ellipsoids | `{x : ‖x − c‖ ≤ r}` |
| Polyhedra | Intersection of half-spaces `{x : Ax ≤ b}` |
| Positive semidefinite cone | `{X : X ≽ 0}` |

**Key property**: Intersection of convex sets is convex.

---

## 6.3 Convex Functions

A function `f: ℝⁿ → ℝ` is **convex** if its domain is a convex set and:

```
f(λx + (1−λ)y) ≤ λf(x) + (1−λ)f(y)    ∀ x,y ∈ dom(f), λ ∈ [0,1]
```

**Geometric interpretation**: The function lies **below** any chord connecting two points.

```
  f(x)
   |        ·  chord  ·
   |       ╱ ·       · ╲
   |      ╱    ·   ·    ╲
   |     ╱      ·╱·      ╲      Convex: f lies below chord
   |    ╱      ╱    ╲      ╲
   |   ╱     ╱        ╲     ╲
   |  ╱    ╱f(x)        ╲    ╲
   +——————————————————————————→ x
          x              y
```

### Tests for Convexity

| Condition | Requirement |
|-----------|-------------|
| **First-order** | `f(y) ≥ f(x) + ∇f(x)ᵀ(y − x)` (tangent lies below) |
| **Second-order** | `∇²f(x) ≽ 0` (Hessian is positive semi-definite) |

### Examples of Convex Functions

| Function | Domain | Why Convex |
|----------|--------|-----------|
| `f(x) = x²` | ℝ | `f''(x) = 2 > 0` |
| `f(x) = eˣ` | ℝ | `f''(x) = eˣ > 0` |
| `f(x) = ‖x‖₂` | ℝⁿ | Norm is always convex |
| `f(x) = max(x₁, ..., xₙ)` | ℝⁿ | Pointwise max of linear functions |
| `f(x) = −log(x)` | ℝ₊₊ | `f''(x) = 1/x² > 0` |
| `f(X) = −log det(X)` | S₊₊ⁿ | Log-det barrier |

### Non-Convex Examples

| Function | Why Not Convex |
|----------|---------------|
| `f(x) = x³` | `f''(x) = 6x` changes sign |
| `f(x) = sin(x)` | Has multiple local minima |
| Neural network loss | Non-linear compositions |

---

## 6.4 Strong Convexity

A function `f` is **μ-strongly convex** (μ > 0) if:

```
f(y) ≥ f(x) + ∇f(x)ᵀ(y − x) + (μ/2)‖y − x‖²
```

**Implications**:
- Unique global minimum
- Faster convergence guarantees
- `f(x) − (μ/2)‖x‖²` is also convex

```
  Convex (flat regions):         Strongly Convex (curved bowl):

  f(x)                          f(x)
   |                              |
   |    ___________               |    ╱         ╲
   |___╱           ╲___           |   ╱    ╱╲     ╲
   |                               |  ╱   ╱  ╲     ╲
   +———————————————————→          + ╱___╱____╲_____╲→
   Has flat regions               Always curves upward at rate ≥ μ
```

**Example**: `f(x) = ‖Ax − b‖² + λ‖x‖²` is λ-strongly convex for λ > 0.

---

## 6.5 Convex Optimization Problem

**Standard form**:

```
minimize    f(x)
subject to  gᵢ(x) ≤ 0,    i = 1, ..., m
            hⱼ(x) = 0,    j = 1, ..., p
```

Where:
- `f(x)` is convex (objective)
- `gᵢ(x)` are convex (inequality constraints)
- `hⱼ(x)` are affine (equality constraints)

### The Global Optimum Guarantee

```
  Non-Convex:                    Convex:

  f(x)                          f(x)
   |  ╱╲    ╱╲                   |
   | ╱  ╲  ╱  ╲                  |  ╲         ╱
   |╱    ╲╱    ╲   Multiple      |   ╲       ╱
   |   L₁  L₂  ╲  local min     |    ╲     ╱     Single
   |             ╲               |     ╲   ╱      global min
   |              ╲              |      ╲_╱
   +——————————————→              +————————*————→
                                       x*

  GD may find L₁ or L₂           GD always finds x*
```

**This is why convexity matters**: any local minimum found by gradient descent is guaranteed to be the global minimum.

---

## 6.6 Convex Optimization in ML

### Linear Regression (Convex ✅)

```
J(θ) = (1/2m) ‖Xθ − y‖²

∇²J = (1/m) XᵀX ≽ 0    (PSD)  →  Convex!
```

With L2 regularization: `J(θ) = (1/2m)‖Xθ − y‖² + λ‖θ‖²` → Strongly convex (λ > 0).

### Logistic Regression (Convex ✅)

```
J(θ) = −(1/m) Σ [yⁱ log σ(θᵀxⁱ) + (1−yⁱ) log(1 − σ(θᵀxⁱ))]

The negative log-likelihood of Bernoulli is convex in θ.
```

### SVM (Convex ✅)

```
minimize   (1/2)‖w‖² + C Σ ξᵢ
s.t.       yⁱ(wᵀxⁱ + b) ≥ 1 − ξᵢ
           ξᵢ ≥ 0
```

Quadratic objective + linear constraints → Convex QP.

### Neural Networks (Non-Convex ❌)

Non-linear activations create a non-convex loss landscape with many local minima and saddle points. However, in practice, SGD often finds good solutions.

---

## 6.7 Gradient Descent Convergence Rates

| Function Class | Rate | Iterations to ε-accuracy |
|---------------|------|--------------------------|
| Convex, L-smooth | O(1/t) | O(L/ε) |
| μ-strongly convex, L-smooth | O(e^(−μt/L)) | O((L/μ) log(1/ε)) |
| Convex + Nesterov | O(1/t²) | O(√(L/ε)) |
| Strongly convex + Nesterov | O(e^(−√(μ/L)·t)) | O(√(L/μ) log(1/ε)) |

### Condition Number

```
κ = L / μ    (ratio of largest to smallest curvature)
```

| κ | Convergence | Example |
|---|------------|---------|
| κ ≈ 1 | Very fast (isotropic) | Well-conditioned data |
| κ ≈ 10² | Moderate | Typical ML problems |
| κ ≈ 10⁶ | Very slow (ill-conditioned) | Poorly scaled features |

**Remedy**: Feature normalization / preconditioning reduces κ.

---

## 6.8 Duality

Every convex optimization problem has a **dual problem** that provides a lower bound on the optimal value.

### Lagrangian

```
L(x, λ, ν) = f(x) + Σᵢ λᵢgᵢ(x) + Σⱼ νⱼhⱼ(x)
```

### Dual Function

```
g(λ, ν) = inf_x L(x, λ, ν)    (minimize over x)
```

### Dual Problem

```
maximize   g(λ, ν)
subject to λ ≥ 0
```

### Key Properties

| Property | Detail |
|----------|--------|
| **Weak duality** | g(λ*, ν*) ≤ f(x*) always holds |
| **Strong duality** | g(λ*, ν*) = f(x*) under Slater's condition |
| **Duality gap** | f(x*) − g(λ*, ν*); zero under strong duality |

---

## 6.9 Step-by-Step Example: Verifying Convexity

**Problem**: Is `f(x₁, x₂) = x₁² + 4x₂² + 2x₁x₂ − 6x₁ − 8x₂` convex?

**Step 1**: Compute the Hessian:

```
∂f/∂x₁ = 2x₁ + 2x₂ − 6
∂f/∂x₂ = 8x₂ + 2x₁ − 8

∂²f/∂x₁² = 2         ∂²f/∂x₁∂x₂ = 2
∂²f/∂x₂∂x₁ = 2       ∂²f/∂x₂² = 8

H = [[2, 2],
     [2, 8]]
```

**Step 2**: Check if H is positive semi-definite:

```
Eigenvalues: det(H − λI) = (2−λ)(8−λ) − 4 = λ² − 10λ + 12 = 0
λ = (10 ± √(100−48)) / 2 = (10 ± √52) / 2
λ₁ ≈ 1.39 > 0,   λ₂ ≈ 8.61 > 0

Both eigenvalues > 0  →  H is positive definite  →  f is strictly convex ✅
```

**Step 3**: Find the minimum by setting ∇f = 0:

```
2x₁ + 2x₂ = 6
2x₁ + 8x₂ = 8

Solving: x₂ = 1/3,  x₁ = 8/3
Minimum at (8/3, 1/3) — guaranteed global by convexity.
```

---

## 6.10 Python: Verifying Convexity

```python
import numpy as np

def check_convexity(H):
    """Check if a Hessian matrix is PSD (convex)."""
    eigenvalues = np.linalg.eigvals(H)
    print(f"Eigenvalues: {eigenvalues}")

    if np.all(eigenvalues > 0):
        print("Positive Definite → Strictly Convex")
    elif np.all(eigenvalues >= 0):
        print("Positive Semi-Definite → Convex")
    else:
        print("Not PSD → Non-Convex")

    return np.all(eigenvalues >= 0)


# Example from Section 6.9
H = np.array([[2, 2],
              [2, 8]])
check_convexity(H)
# Output: Eigenvalues: [1.394  8.606]
#         Positive Definite → Strictly Convex
```

### Solving with scipy

```python
from scipy.optimize import minimize

def f(x):
    return x[0]**2 + 4*x[1]**2 + 2*x[0]*x[1] - 6*x[0] - 8*x[1]

result = minimize(f, x0=[0, 0], method='BFGS')
print(f"Optimal x: {result.x}")       # [2.667, 0.333]
print(f"Optimal f: {result.fun}")      # Minimum value
print(f"Converged: {result.success}")  # True
```

---

## 6.11 Real-World ML Applications

| Application | Convex? | Solver |
|------------|---------|--------|
| **Linear Regression (OLS)** | ✅ Convex | Closed-form or GD |
| **Ridge Regression** | ✅ Strongly convex | Closed-form or GD |
| **Logistic Regression** | ✅ Convex | GD / Newton's method |
| **SVM (dual)** | ✅ Convex QP | SMO, interior-point |
| **Lasso (L1)** | ✅ Convex (non-smooth) | Proximal GD, ADMM |
| **PCA** | ✅ Convex (eigenvalue) | SVD |
| **Neural Networks** | ❌ Non-convex | SGD (finds good local min) |
| **k-Means** | ❌ Non-convex | EM / Lloyd's algorithm |

---

## 6.12 Boyd and Vandenberghe Reference

The standard textbook is *Convex Optimization* by Stephen Boyd and Lieven Vandenberghe (Cambridge University Press, 2004). Available free at: `https://web.stanford.edu/~boyd/cvxbook/`

Key chapters relevant to ML:
- **Ch. 3**: Convex functions
- **Ch. 4**: Convex optimization problems
- **Ch. 5**: Duality
- **Ch. 9**: Unconstrained minimization (gradient methods)
- **Ch. 11**: Interior-point methods

---

## 6.13 Summary Table

| Concept | Key Detail |
|---------|-----------|
| Convex set | Line segment between any two points stays in set |
| Convex function | `f(λx+(1−λ)y) ≤ λf(x)+(1−λ)f(y)` |
| Test (2nd order) | Hessian ∇²f ≽ 0 (PSD) |
| Global optimum | Guaranteed for convex problems |
| Strong convexity | Unique minimum, faster convergence |
| Condition number | κ = L/μ (affects GD speed) |
| GD rate (convex) | O(1/t) |
| GD rate (strongly convex) | O(e^(−t/κ)) — linear convergence |
| Duality | Dual problem gives lower bound; strong duality → gap = 0 |
| ML examples | Linear/logistic regression, SVM, Ridge, Lasso |

---

## 6.14 Quick Revision Questions

1. **What is the defining property of a convex function?**
   → f(λx+(1−λ)y) ≤ λf(x)+(1−λ)f(y) — the function lies below any chord.

2. **How do you check convexity using the Hessian?**
   → The Hessian ∇²f must be positive semi-definite (all eigenvalues ≥ 0).

3. **Why is convexity important in ML?**
   → It guarantees that any local minimum is the global minimum — no risk of getting stuck.

4. **What is the condition number, and why does it matter?**
   → κ = L/μ; large κ means slow convergence (ill-conditioned problem); feature scaling reduces κ.

5. **Name three ML models with convex loss functions.**
   → Linear regression (MSE), logistic regression (cross-entropy), SVM (hinge loss + quadratic regularizer).

6. **What is strong duality?**
   → The optimal values of the primal and dual problems are equal; holds under Slater's condition for convex problems.

---

| [← Previous: Learning Rate Scheduling](05-learning-rate-scheduling.md) | [Next Chapter: Constrained Optimization →](07-constrained-optimization.md) |
|:---|---:|
