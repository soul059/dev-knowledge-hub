# Chapter 7: Convex Functions and Convex Optimization

> **Unit 2 — Calculus for ML** | [← Previous: Optimization Basics](./06-optimization-basics.md) | [Next → Lagrange Multipliers](./08-lagrange-multipliers.md)

---

## 1. Overview

Convexity is the most important structural property in optimization. When a function is **convex**, every local minimum is automatically a global minimum — gradient descent is **guaranteed** to find the best solution. Many classical ML algorithms (linear regression, logistic regression, SVMs) are convex problems by design. Understanding convexity tells us when optimization is "easy" and when it's fundamentally hard.

---

## 2. Convex Sets

A set S ⊆ ℝⁿ is **convex** if for any two points x, y ∈ S, the entire line segment between them also lies in S:

```
∀ x, y ∈ S,  ∀ θ ∈ [0, 1]:    θx + (1−θ)y  ∈  S
```

### Visual Examples

```
    CONVEX SETS:                    NON-CONVEX SETS:

    ╭─────────╮                     ╭───╮
    │         │                     │   │
    │  · · · ·│ ← line stays       │   ╰──╮
    │    inside│   inside           │      │ ← line goes
    ╰─────────╯                     │   ╭──╯   OUTSIDE
                                    ╰───╯

    ╱────────╲                       ☆ (star shape)
   ╱          ╲
  ╱    circle  ╲                   Any set with a "dent"
  ╲            ╱                   is non-convex
   ╲──────────╱
```

**Common convex sets:** ℝⁿ itself, hyperplanes, halfspaces, balls, polyhedra, the positive semidefinite cone.

---

## 3. Convex Functions

### 3.1 Definition

A function f: ℝⁿ → ℝ is **convex** if its domain is a convex set and for all x, y in the domain:

```
f(θx + (1−θ)y)  ≤  θf(x) + (1−θ)f(y)     ∀ θ ∈ [0, 1]
```

**Geometric interpretation:** The function value at any point on the line segment between x and y lies **below or on** the chord connecting f(x) and f(y).

```
  f(x)
   │
   │  ·                 · f(y)          ← chord (line between f(x), f(y))
   │   ·  ·  ·  ·  · ·                 
   │    ·           ·                   ← function curve (below chord)
   │     ·    ·   ·
   │      ·  ·  ·                       For convex functions:
   │       · · ·                        curve ≤ chord EVERYWHERE
   │        ·
   └────────────────── x
        x           y
```

### 3.2 Strictly Convex

f is **strictly convex** if the inequality is strict (< instead of ≤) for x ≠ y. This guarantees a **unique** global minimum.

---

## 4. First-Order Condition for Convexity

If f is differentiable, f is convex if and only if:

```
f(y) ≥ f(x) + ∇f(x)ᵀ(y − x)     ∀ x, y
```

**Meaning:** The function lies **above** every tangent line/plane. The tangent is a global underestimator.

```
  f(x)
   │          · ·
   │        ·     ·
   │      ·         ·            ← f(x) is above the tangent
   │    ·              ·
   │  · · · · · · · · · · ·     ← tangent line at x₀ (global underestimator)
   │·
   └────────────────────── x
              x₀
```

---

## 5. Second-Order Condition (Hessian Test)

If f is twice differentiable, f is convex if and only if:

```
H(x) ≽ 0     (Hessian is positive semidefinite)     for ALL x
```

This means all eigenvalues of H are ≥ 0 everywhere.

| Hessian Condition       | Function Property      |
|------------------------|------------------------|
| H ≽ 0 (PSD) everywhere | Convex                 |
| H ≻ 0 (PD) everywhere  | Strictly convex        |
| H ≼ 0 (NSD) everywhere | Concave                |
| H ≺ 0 (ND) everywhere  | Strictly concave       |

### Example: Testing Convexity

**f(x, y) = x² + 2y²**

```
Hessian: H = ┌ 2  0 ┐
             │ 0  4 │
             └      ┘

Eigenvalues: 2 and 4 (both > 0 for all x, y)
→ H is positive definite everywhere → f is STRICTLY CONVEX ✓
```

**f(x, y) = x² − y²**

```
Hessian: H = ┌ 2   0 ┐
             │ 0  −2 │
             └       ┘

Eigenvalues: 2 and −2 (mixed signs)
→ H is indefinite → f is NEITHER convex NOR concave ✗
```

---

## 6. Key Properties of Convex Functions

### The Golden Property: Local = Global

> **If f is convex and x* is a local minimum, then x* is also a GLOBAL minimum.**

This is why convexity matters so much — gradient descent **cannot** get stuck at a suboptimal point.

### Preservation Rules

| Operation                     | Result                    |
|-------------------------------|---------------------------|
| f + g (both convex)           | Convex ✓                  |
| α·f (α ≥ 0, f convex)        | Convex ✓                  |
| max(f, g) (both convex)      | Convex ✓                  |
| f(Ax + b) (f convex, A, b)   | Convex ✓ (affine comp.)   |
| f · g (both convex)           | NOT necessarily convex ✗  |

---

## 7. Jensen's Inequality

For a convex function f and random variable X:

```
f(E[X])  ≤  E[f(X)]
```

**In words:** The function of the average ≤ the average of the function.

### Visual Interpretation

```
  f(x)
   │           ·  ·             E[f(X)] ← average of function values
   │         ·      ·                      (on the chord)
   │       ·    ─ ─ ─ · ─ ─ 
   │     ·   ·           ·
   │    ·  ·               ·
   │   · ·                         f(E[X]) ← function at the average
   │  ─ · ─ ─ ─ ─ ─ ─ ─ ─ ─      (on the curve, below chord)
   └────────────────────── x
        x₁   E[X]    x₂
```

**ML Applications of Jensen's Inequality:**
- **Evidence Lower Bound (ELBO)** in Variational Autoencoders (VAEs)
- **EM algorithm** convergence proof
- **KL divergence** is non-negative (consequence of Jensen's)
- **Log-likelihood bounds** in probabilistic models

---

## 8. Common Convex Functions in ML

| Function                     | Formula                      | Used In                     | Convex?           |
|------------------------------|------------------------------|-----------------------------|-------------------|
| MSE (Mean Squared Error)     | ½‖y − Xw‖²                  | Linear regression            | ✓ Convex          |
| Log Loss (Cross-Entropy)     | −[y log(p)+(1−y)log(1−p)]   | Logistic regression          | ✓ Convex in w     |
| Hinge Loss                   | max(0, 1 − y·f(x))          | SVM                          | ✓ Convex          |
| L2 Regularization            | λ‖w‖²                       | Ridge regression             | ✓ Strictly convex |
| L1 Regularization            | λ‖w‖₁                       | Lasso                        | ✓ Convex (not strict) |
| Softmax + Cross-Entropy      | −log(eᶻⁱ / Σeᶻʲ)           | Multi-class classification   | ✓ Convex          |
| Neural Network Loss          | Complex composition          | Deep learning                | ✗ Non-convex      |

---

## 9. Step-by-Step Example: Proving MSE is Convex

**Claim:** L(w) = ½‖y − Xw‖² is convex.

**Step 1:** Expand the loss
```
L(w) = ½(y − Xw)ᵀ(y − Xw)
     = ½(yᵀy − 2yᵀXw + wᵀXᵀXw)
```

**Step 2:** Compute the gradient
```
∇L = −Xᵀy + XᵀXw = XᵀX w − Xᵀy
```

**Step 3:** Compute the Hessian
```
H = XᵀX
```

**Step 4:** Check positive semidefiniteness
```
For any vector v:  vᵀ(XᵀX)v = (Xv)ᵀ(Xv) = ‖Xv‖² ≥ 0

Since vᵀHv ≥ 0 for all v, H = XᵀX is positive semidefinite.
```

**Conclusion:** L(w) is convex. ✓

If X has full column rank, then XᵀX is positive **definite**, making L **strictly** convex with a **unique** minimum: w* = (XᵀX)⁻¹Xᵀy.

---

## 10. Python: Convexity Check and Visualization

```python
import numpy as np
import matplotlib.pyplot as plt

# Example 1: Verify MSE is convex
np.random.seed(42)
X = np.random.randn(50, 2)
y = X @ np.array([3, -1]) + 0.5 * np.random.randn(50)

# Hessian of MSE = X^T X
H = X.T @ X
eigenvalues = np.linalg.eigvalsh(H)
print(f"Hessian eigenvalues: {eigenvalues}")
print(f"All non-negative? {all(eigenvalues >= 0)} → Convex!")

# Example 2: Visualize convex vs non-convex
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

x = np.linspace(-3, 3, 200)

# Convex: f(x) = x^2
axes[0].plot(x, x**2, 'b-', linewidth=2)
# Show chord property
x1, x2 = -2, 2.5
for theta in [0.25, 0.5, 0.75]:
    xm = theta * x1 + (1 - theta) * x2
    fm = theta * x1**2 + (1 - theta) * x2**2  # chord value
    axes[0].plot(xm, xm**2, 'go', markersize=8)   # function (below)
    axes[0].plot(xm, fm, 'r^', markersize=8)       # chord (above)
axes[0].plot([x1, x2], [x1**2, x2**2], 'r--', label='chord')
axes[0].set_title('Convex: f(x) = x² (curve ≤ chord)')
axes[0].legend()

# Non-convex: f(x) = x^4 - 3x^2 + 1
axes[1].plot(x, x**4 - 3*x**2 + 1, 'b-', linewidth=2)
x1, x2 = -1.5, 1.5
axes[1].plot([x1, x2], [x1**4-3*x1**2+1, x2**4-3*x2**2+1], 'r--', label='chord')
axes[1].set_title('Non-convex: f(x) = x⁴ − 3x² + 1 (chord violated)')
axes[1].legend()

plt.tight_layout(); plt.show()
```

---

## 11. Why Convexity Matters in ML

```
  Training Guarantee Spectrum:
  
  ──────────────────────────────────────────────────────
  CONVEX                                    NON-CONVEX
  (guaranteed global optimum)    (no guarantee, but works in practice)
  
  ✓ Linear Regression            ✗ Neural Networks
  ✓ Logistic Regression          ✗ Deep Learning
  ✓ SVM (dual)                   ✗ GANs
  ✓ Lasso / Ridge                ✗ Mixture Models (EM)
  ──────────────────────────────────────────────────────
  Easy to optimize               Harder, but more expressive
```

**Key insight:** There's a **trade-off** between model expressiveness and optimization ease. Convex models are easy to optimize but limited; non-convex models (deep networks) are powerful but harder to train.

---

## 12. Real-World ML Applications

| Application                  | Convexity Role                                          |
|------------------------------|----------------------------------------------------------|
| Linear/Logistic Regression   | Convex → guaranteed optimal solution                     |
| SVM                          | Convex QP → unique max-margin solution                   |
| Variational Inference (ELBO) | Jensen's inequality provides tractable lower bound       |
| Regularization Design        | L1/L2 penalties are convex → won't create new local min  |
| Batch Normalization          | Helps make effective loss more convex-like                |
| Curriculum Learning          | Start with convex-like sub-problems, gradually harder     |

---

## 13. Summary Table

| Concept                | Definition / Property                     | ML Significance                     |
|------------------------|-------------------------------------------|-------------------------------------|
| Convex set             | Line segments stay inside                  | Feasible region for constraints     |
| Convex function        | f(θx+(1−θ)y) ≤ θf(x)+(1−θ)f(y)          | Guarantees global optimum           |
| Strictly convex        | Strict inequality for x ≠ y               | Unique global minimum               |
| Hessian test           | H ≽ 0 everywhere → convex                 | Practical convexity check           |
| Jensen's inequality    | f(E[X]) ≤ E[f(X)]                        | ELBO, EM algorithm, KL divergence   |
| Local = Global         | Any local min is global for convex f       | GD always succeeds                  |
| Convex + Convex        | Sum is convex                              | Regularized losses stay convex      |

---

## 14. Quick Revision Questions

1. **State the definition of a convex function using the chord inequality.**
2. **How do you test if a twice-differentiable function is convex?** *(Answer: H ≽ 0 everywhere)*
3. **Why does convexity guarantee that gradient descent finds the global minimum?**
4. **Is the sum of two convex functions always convex? What about the product?**
5. **Give an example of a convex loss function and a non-convex loss function used in ML.**
6. **State Jensen's inequality and name one ML application where it is used.**

---

> [← Previous: Optimization Basics](./06-optimization-basics.md) | [Next → Lagrange Multipliers](./08-lagrange-multipliers.md)
