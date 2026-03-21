# Chapter 4: Jacobian and Hessian Matrices

> **Unit 2 — Calculus for ML** | [← Previous: Chain Rule](./03-chain-rule.md) | [Next → Taylor Series](./05-taylor-series.md)

---

## 1. Overview

The **Jacobian** and **Hessian** are matrix extensions of the derivative. The Jacobian captures first-order information for vector-valued functions, while the Hessian captures second-order (curvature) information for scalar functions. Together they enable advanced optimization methods like Newton's method and inform us about the geometry of loss surfaces.

---

## 2. The Jacobian Matrix

### 2.1 Definition

For a function **f**: ℝⁿ → ℝᵐ that maps n inputs to m outputs:

```
f(x₁, ..., xₙ) = [f₁(x), f₂(x), ..., fₘ(x)]ᵀ
```

The **Jacobian** is the m × n matrix of all first-order partial derivatives:

```
         ┌ ∂f₁/∂x₁   ∂f₁/∂x₂  ···  ∂f₁/∂xₙ ┐
         │ ∂f₂/∂x₁   ∂f₂/∂x₂  ···  ∂f₂/∂xₙ │
J(x) =   │    ⋮          ⋮      ···     ⋮     │
         │ ∂fₘ/∂x₁   ∂fₘ/∂x₂  ···  ∂fₘ/∂xₙ │
         └                                     ┘
```

**Row i** = gradient of the i-th output component fᵢ.

### 2.2 Special Cases

| Function Type      | Input → Output   | Jacobian Shape | Reduces To     |
|--------------------|------------------|----------------|----------------|
| f: ℝ → ℝ          | scalar → scalar  | 1 × 1          | Ordinary derivative |
| f: ℝⁿ → ℝ         | vector → scalar  | 1 × n          | Gradient (row)   |
| f: ℝ → ℝᵐ         | scalar → vector  | m × 1          | Column of derivatives |
| f: ℝⁿ → ℝᵐ        | vector → vector  | m × n          | Full Jacobian    |

---

## 3. Jacobian — Worked Example

**Given:** f: ℝ² → ℝ² defined as:
```
f₁(x, y) = x²y
f₂(x, y) = 5x + sin(y)
```

**Compute the Jacobian:**
```
J = ┌ ∂f₁/∂x   ∂f₁/∂y ┐   =   ┌ 2xy     x²    ┐
    │ ∂f₂/∂x   ∂f₂/∂y │       │  5    cos(y)  │
    └                   ┘       └               ┘
```

**Evaluate at (1, π/2):**
```
J(1, π/2) = ┌ 2(1)(π/2)    1²    ┐   =   ┌ π     1  ┐
            │    5       cos(π/2) │       │ 5     0  │
            └                     ┘       └         ┘
```

### Jacobian in ML

In neural networks, when a layer transforms a vector **z** to **a** = σ(**z**), the Jacobian of this transformation appears in backpropagation:

```
∂L/∂z = (∂a/∂z)ᵀ · (∂L/∂a) = Jᵀ · (∂L/∂a)
```

For element-wise activations (ReLU, sigmoid), this Jacobian is **diagonal**.

---

## 4. The Hessian Matrix

### 4.1 Definition

For a scalar function f: ℝⁿ → ℝ, the **Hessian** is the n × n matrix of second-order partial derivatives:

```
          ┌ ∂²f/∂x₁²      ∂²f/∂x₁∂x₂  ···  ∂²f/∂x₁∂xₙ ┐
          │ ∂²f/∂x₂∂x₁   ∂²f/∂x₂²     ···  ∂²f/∂x₂∂xₙ │
H(x) =    │      ⋮             ⋮        ···       ⋮       │
          │ ∂²f/∂xₙ∂x₁   ∂²f/∂xₙ∂x₂  ···  ∂²f/∂xₙ²    │
          └                                                 ┘
```

**Key properties:**
- H is **symmetric** (by Clairaut's theorem): H = Hᵀ
- H captures **curvature** information
- H is the **Jacobian of the gradient**: H = J(∇f)

### 4.2 Hessian — Worked Example

**Given:** f(x, y) = x³ − 2xy + y²

**Gradient:**
```
∇f = [3x² − 2y,  −2x + 2y]
```

**Hessian:**
```
H = ┌ ∂²f/∂x²     ∂²f/∂x∂y ┐   =   ┌ 6x   −2 ┐
    │ ∂²f/∂y∂x    ∂²f/∂y²   │       │ −2    2  │
    └                        ┘       └          ┘
```

**At point (1, 1):**
```
H(1,1) = ┌ 6   −2 ┐
          │ −2    2 │
          └         ┘
```

**Eigenvalues:** λ² − 8λ + 8 = 0 → λ₁ ≈ 6.83, λ₂ ≈ 1.17 (both positive → local minimum)

---

## 5. Second-Order Information and Curvature

The Hessian tells us about the **shape** of the function near a critical point:

```
  All eigenvalues > 0          Some > 0, some < 0        All eigenvalues < 0
  ────────────────            ─────────────────          ────────────────
    Local Minimum                Saddle Point              Local Maximum
                                 
      ╲     ╱                    ╲   ╱                      ╱     ╲
       ╲   ╱                      ╲ ╱                      ╱       ╲
        ╲_╱                        ╳                      ╱─────────╲
     (bowl up)                (horse saddle)              (bowl down)
```

| Eigenvalue Condition          | Classification       | Hessian Property           |
|-------------------------------|----------------------|----------------------------|
| All λᵢ > 0                   | Local minimum        | Positive definite          |
| All λᵢ < 0                   | Local maximum        | Negative definite          |
| Mixed signs                  | Saddle point         | Indefinite                 |
| Some λᵢ = 0                  | Inconclusive         | Positive/negative semidefinite |

---

## 6. Newton's Method

Newton's method uses the Hessian to take **curvature-informed** steps, converging faster than gradient descent:

```
w(t+1) = w(t) − H⁻¹ · ∇f(w(t))
```

### Comparison: Gradient Descent vs Newton's Method

```
  Gradient Descent (1st order):        Newton's Method (2nd order):
  
   · · · ·                                · · ·
    · · ·                                   ·
     · · ·                                   ·
      ·___                                   ·___
  Many small steps,                     Fewer, smarter steps,
  can zig-zag                           accounts for curvature
```

| Feature              | Gradient Descent       | Newton's Method          |
|----------------------|------------------------|--------------------------|
| Order                | First (gradient only)   | Second (gradient + Hessian) |
| Step computation     | α · ∇f                | H⁻¹ · ∇f                |
| Convergence rate     | Linear                 | Quadratic                |
| Cost per step        | O(n)                   | O(n³) — matrix inversion |
| Practical for large n| Yes                    | No (n > 10⁶ params)     |

> **In practice**, quasi-Newton methods (L-BFGS) approximate H⁻¹ without computing it fully.

---

## 7. Python Example

```python
import numpy as np

def f(x, y):
    return x**3 - 2*x*y + y**2

def gradient(x, y):
    return np.array([3*x**2 - 2*y, -2*x + 2*y])

def hessian(x, y):
    return np.array([[6*x, -2],
                     [-2,   2]])

# Analyze critical point
# Set gradient = 0: 3x² − 2y = 0 and −2x + 2y = 0 → y = x → 3x² − 2x = 0
# Solutions: (0, 0) and (2/3, 2/3)

for point in [(0, 0), (2/3, 2/3)]:
    x, y = point
    H = hessian(x, y)
    eigenvalues = np.linalg.eigvalsh(H)
    
    print(f"\nPoint {point}:")
    print(f"  Gradient: {gradient(x, y)}")
    print(f"  Hessian:\n{H}")
    print(f"  Eigenvalues: {eigenvalues}")
    
    if all(eigenvalues > 0):
        print(f"  Classification: LOCAL MINIMUM")
    elif all(eigenvalues < 0):
        print(f"  Classification: LOCAL MAXIMUM")
    else:
        print(f"  Classification: SADDLE POINT")

# Newton's method step from (2, 2)
x0, y0 = 2.0, 2.0
g = gradient(x0, y0)
H = hessian(x0, y0)
step = np.linalg.solve(H, g)  # H⁻¹ · g (more stable than inverting)
x_new = np.array([x0, y0]) - step
print(f"\nNewton step from (2, 2): → ({x_new[0]:.4f}, {x_new[1]:.4f})")
```

---

## 8. Jacobian Determinant

The **determinant of the Jacobian** measures how a transformation scales area/volume:

```
|det(J)| > 1  →  region expands
|det(J)| = 1  →  volume preserved
|det(J)| < 1  →  region contracts
|det(J)| = 0  →  dimension collapse (singular)
```

**ML application:** In **normalizing flows** (generative models), the log Jacobian determinant appears in the change-of-variables formula for computing probability densities:

```
log p(x) = log p(z) + log |det(∂z/∂x)|
```

---

## 9. Real-World ML Applications

| Application                | Matrix Used  | Purpose                                           |
|----------------------------|--------------|----------------------------------------------------|
| Backpropagation            | Jacobian     | Chain rule across layers (vector-to-vector)         |
| Newton's / L-BFGS          | Hessian      | Second-order optimization, faster convergence       |
| Fisher Information Matrix  | ~Hessian     | Natural gradient descent, confidence estimation     |
| Normalizing Flows          | Jacobian det | Probability density transformation                  |
| Saddle Point Detection     | Hessian      | Eigenvalue analysis of critical points              |
| Batch Normalization        | Jacobian     | Gradient flow through normalization layers          |

---

## 10. Summary Table

| Concept               | Definition                          | Size   | Key Use in ML                    |
|-----------------------|--------------------------------------|--------|----------------------------------|
| Jacobian J            | Matrix of 1st partial derivatives    | m × n  | Backprop, normalizing flows      |
| Hessian H             | Matrix of 2nd partial derivatives    | n × n  | Curvature, optimization          |
| H positive definite   | All eigenvalues > 0                  | —      | Confirms local minimum           |
| H indefinite          | Mixed sign eigenvalues               | —      | Indicates saddle point           |
| Newton's method       | w ← w − H⁻¹∇f                      | —      | Quadratic convergence            |
| Jacobian determinant  | det(J)                               | scalar | Volume scaling, normalizing flows|

---

## 11. Quick Revision Questions

1. **What is the shape of the Jacobian for a function f: ℝ³ → ℝ⁵?** *(Answer: 5 × 3)*
2. **Why is the Hessian matrix always symmetric?**
3. **How do the eigenvalues of the Hessian classify a critical point?**
4. **What is the main computational bottleneck of Newton's method?**
5. **In what type of generative model does the Jacobian determinant appear explicitly?**
6. **For element-wise activation functions (like ReLU), what special structure does the Jacobian have?** *(Answer: diagonal)*

---

> [← Previous: Chain Rule](./03-chain-rule.md) | [Next → Taylor Series](./05-taylor-series.md)
