# Chapter 6: Optimization Basics

> **Unit 2 — Calculus for ML** | [← Previous: Taylor Series](./05-taylor-series.md) | [Next → Convex Functions](./07-convex-functions.md)

---

## 1. Overview

**Optimization** is the heart of machine learning — every ML model is trained by minimizing (or maximizing) an objective function. This chapter covers the fundamental concepts: critical points, local vs global optima, saddle points, convex vs non-convex landscapes, and the major classes of optimization methods used in practice.

---

## 2. The Optimization Problem

Standard form:

```
minimize   f(x)          ← objective function (loss, cost, error)
  x ∈ ℝⁿ

subject to:               ← optional constraints
  gᵢ(x) ≤ 0              (inequality constraints)
  hⱼ(x) = 0              (equality constraints)
```

In **unconstrained ML** (most deep learning), there are no explicit constraints — we simply minimize the loss function over all possible weights.

---

## 3. Critical Points

A **critical point** (stationary point) is where the gradient is zero:

```
∇f(x*) = 0
```

But not every critical point is a minimum! There are three types:

```
  LOCAL MINIMUM          LOCAL MAXIMUM           SADDLE POINT
                                    
    ╲     ╱                ╱     ╲               ╲ maximum ╱
     ╲   ╱                ╱       ╲               ╲  dir  ╱
      ╲_╱                ╱─────────╲               ╲    ╱
                                                    ╳
   ∇f = 0               ∇f = 0                  ╱    ╲
   f'' > 0               f'' < 0               ╱  min  ╲
                                               ╱   dir   ╲
                                              ∇f = 0
                                              mixed curvature
```

---

## 4. Necessary and Sufficient Conditions

### First-Order Necessary Condition (FONC)

If x* is a local minimum of f, then:
```
∇f(x*) = 0
```
(The gradient must vanish — but this is **necessary, not sufficient**.)

### Second-Order Sufficient Condition (SOSC)

If ∇f(x*) = 0 AND the Hessian H(x*) is **positive definite**, then x* is a **strict local minimum**.

| Condition                    | Conclusion            |
|------------------------------|-----------------------|
| ∇f = 0, H positive definite | Local minimum ✓       |
| ∇f = 0, H negative definite | Local maximum ✓       |
| ∇f = 0, H indefinite        | Saddle point ✓        |
| ∇f = 0, H semidefinite      | Inconclusive ⚠️       |

---

## 5. Local vs Global Optima

```
  f(x)
   │
   │  ╱╲         ╱╲
   │ ╱  ╲       ╱  ╲
   │╱    ╲     ╱    ╲         ╱
   │      ╲   ╱      ╲       ╱
   │       ╲_╱        ╲     ╱
   │        ↑          ╲___╱
   │     local          ↑
   │     minimum      GLOBAL
   │                  MINIMUM
   └────────────────────────── x
```

- **Local minimum:** f(x*) ≤ f(x) for all x **near** x*
- **Global minimum:** f(x*) ≤ f(x) for **all** x in the domain
- Every global minimum is a local minimum, but not vice versa!

> **ML Challenge:** Neural network loss surfaces have many local minima and saddle points. Gradient descent can get stuck at **any** of them.

---

## 6. Saddle Points — The Real Enemy

In high-dimensional optimization (deep learning), **saddle points** are far more common than local minima.

**Why?** At a critical point, each eigenvalue of the Hessian is independently +/−. With n dimensions:
- P(local minimum) = P(all n eigenvalues > 0) = (1/2)ⁿ → vanishingly small
- Saddle points dominate as n grows

```
  3D Saddle Point (horse saddle):

       ╲ ╱ ╱ ╱          Direction A: curves UP (minimum direction)
        ╲╱ ╱ ╱
    ─────╳─────          Direction B: curves DOWN (maximum direction)
        ╱╲ ╲ ╲
       ╱ ╲ ╲ ╲          The gradient is zero at the center ╳
                         but it's neither a min nor a max!
```

---

## 7. The ML Loss Surface

A typical deep learning loss surface is **highly non-convex**:

```
  Loss L(w)
   │
   │ ╱╲     ╱╲
   │╱  ╲   ╱  ╲    ╱╲        ╱╲
   │    ╲_╱    ╲  ╱  ╲      ╱  ╲
   │     ↑      ╲╱    ╲____╱    ╲__
   │   local     ↑      ↑           ↑
   │   min     saddle  global     flat
   │                   min       region
   └──────────────────────────────── w
   
   Features of neural network loss surfaces:
   ✓ Many local minima (but often similar loss values!)
   ✓ Many saddle points (especially in high dimensions)
   ✓ Flat regions (plateaus where gradient ≈ 0)
   ✓ Sharp vs flat minima (flat = better generalization)
```

---

## 8. Classification of Optimization Methods

### 8.1 Gradient-Based Methods (most common in ML)

| Method               | Order | Uses              | Convergence    | Cost/Step     |
|----------------------|-------|-------------------|----------------|---------------|
| Gradient Descent     | 1st   | ∇f                | Linear         | O(n)          |
| Momentum (SGD+M)     | 1st   | ∇f + history      | Faster linear  | O(n)          |
| Adam / RMSprop       | 1st   | ∇f + adaptive lr  | Fast           | O(n)          |
| Newton's Method      | 2nd   | ∇f, H             | Quadratic      | O(n³)         |
| L-BFGS               | Quasi-2nd | ∇f, approx H  | Super-linear   | O(n·m)        |

### 8.2 Derivative-Free Methods

| Method               | Strategy            | When to Use                          |
|----------------------|--------------------|--------------------------------------|
| Grid Search          | Exhaustive          | Hyperparameter tuning (small space)   |
| Random Search        | Random sampling     | Hyperparameter tuning (medium space)  |
| Bayesian Optimization| Surrogate model     | Expensive-to-evaluate functions       |
| Evolutionary/Genetic | Population-based    | Non-differentiable objectives         |
| Nelder-Mead          | Simplex             | Low-dimensional, no gradient          |

---

## 9. Convex vs Non-Convex Optimization

```
  CONVEX                                NON-CONVEX
  (easy — one minimum)                  (hard — many minima)

     ╲         ╱                         ╱╲    ╱╲
      ╲       ╱                         ╱  ╲  ╱  ╲
       ╲     ╱                         ╱    ╲╱    ╲
        ╲___╱                         ╱      ↑     ╲___
          ↑                          ╱    local
       global                               min
       min                         ANY gradient descent
   GUARANTEED to find it!          finds A local min (maybe not global)
```

| Property             | Convex                          | Non-Convex                     |
|----------------------|---------------------------------|--------------------------------|
| Local min = Global?  | YES — always                     | NO — can have many local min   |
| GD converges to min? | YES (with proper learning rate)  | Converges to *a* critical pt   |
| Examples             | Linear/logistic regression       | Neural networks                |
| Difficulty           | "Solved" (polynomial time)       | NP-hard in general             |

> **Good news:** Empirically, local minima in deep networks often have similar loss values, so finding **any** local minimum tends to work well in practice.

---

## 10. Step-by-Step Example: Finding Optima

**Problem:** Find and classify all critical points of f(x, y) = x² + y² − 2x − 4y + 5.

**Step 1:** Compute gradient and set to zero
```
∂f/∂x = 2x − 2 = 0  →  x = 1
∂f/∂y = 2y − 4 = 0  →  y = 2
```
Critical point: (1, 2)

**Step 2:** Compute Hessian
```
H = ┌ 2  0 ┐
    │ 0  2 │
    └      ┘
```

**Step 3:** Check eigenvalues
```
λ₁ = 2 > 0,  λ₂ = 2 > 0  →  H is positive definite
```

**Step 4:** Classify → **Local (and global) minimum** at (1, 2), f(1,2) = 0.

---

## 11. Python: Visualizing a Loss Surface

```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

def loss(x, y):
    """A non-convex loss surface with multiple minima."""
    return (x**2 + y - 11)**2 + (x + y**2 - 7)**2  # Himmelblau's function

# Create grid
x = np.linspace(-5, 5, 200)
y = np.linspace(-5, 5, 200)
X, Y = np.meshgrid(x, y)
Z = loss(X, Y)

# 2D Contour plot
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Contour plot
axes[0].contour(X, Y, Z, levels=50, cmap='viridis')
axes[0].set_title("Himmelblau's Function (Contour)")
axes[0].set_xlabel('x'); axes[0].set_ylabel('y')

# Mark the 4 known minima
minima = [(3, 2), (-2.805, 3.131), (-3.779, -3.283), (3.584, -1.848)]
for m in minima:
    axes[0].plot(*m, 'r*', markersize=15)

# Log-scale contour for better visibility
axes[1].contourf(X, Y, np.log(Z + 1), levels=30, cmap='hot')
axes[1].set_title("Log Loss Surface")
axes[1].set_xlabel('x'); axes[1].set_ylabel('y')

plt.tight_layout(); plt.show()
```

---

## 12. Real-World ML Applications

| Application                   | Optimization Role                                    |
|-------------------------------|------------------------------------------------------|
| Neural Network Training       | Minimize cross-entropy loss (non-convex)              |
| Linear Regression             | Minimize MSE (convex — closed-form or GD)             |
| SVM                           | Convex quadratic program                              |
| Hyperparameter Tuning         | Bayesian optimization or random search                |
| Reinforcement Learning        | Policy gradient (non-convex, stochastic)              |
| Neural Architecture Search    | Combinatorial optimization + gradient methods         |

---

## 13. Summary Table

| Concept              | Key Idea                                        | Relevance to ML                      |
|----------------------|-------------------------------------------------|--------------------------------------|
| Critical point       | ∇f = 0                                          | Candidates for optima                |
| Local minimum        | Lowest point in a neighborhood                   | What GD converges to                 |
| Global minimum       | Lowest point overall                             | What we ideally want                 |
| Saddle point         | Min in some dirs, max in others                  | Dominant in high-dim (deep learning) |
| FONC                 | ∇f(x*) = 0 (necessary)                          | Finding candidates                   |
| SOSC                 | H positive definite (sufficient)                 | Confirming minima                    |
| Convex optimization  | Local min = global min                           | Linear/logistic regression           |
| Non-convex           | Multiple local minima                            | Neural networks                      |

---

## 14. Quick Revision Questions

1. **What are the three types of critical points?**
2. **Why are saddle points more common than local minima in high-dimensional spaces?**
3. **State the second-order sufficient condition for a local minimum.**
4. **Is the loss surface of a deep neural network convex or non-convex?**
5. **Name two derivative-free optimization methods and when you'd use them.**
6. **Why might a "flat" local minimum generalize better than a "sharp" one?**

---

> [← Previous: Taylor Series](./05-taylor-series.md) | [Next → Convex Functions](./07-convex-functions.md)
