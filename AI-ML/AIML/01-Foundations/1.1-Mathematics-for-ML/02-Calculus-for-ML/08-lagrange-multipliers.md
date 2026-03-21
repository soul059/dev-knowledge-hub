# Chapter 8: Lagrange Multipliers and Constrained Optimization

> **Unit 2 — Calculus for ML** | [← Previous: Convex Functions](./07-convex-functions.md)

---

## 1. Overview

Many real-world ML problems involve **constraints** — we don't just minimize a function freely; we must satisfy certain conditions. Lagrange multipliers provide an elegant method to solve constrained optimization problems. This chapter covers the method, extends it to inequality constraints (KKT conditions), and connects it directly to SVMs and regularization.

---

## 2. The Constrained Optimization Problem

**Standard form:**
```
minimize   f(x)                  ← objective function
subject to g(x) = 0             ← equality constraint
```

**Example:** Minimize the area of a fence enclosing a rectangular region, given a fixed perimeter.

```
minimize   f(x, y) = −xy         (negative because we want to maximize area)
subject to g(x, y) = 2x + 2y − P = 0   (perimeter constraint)
```

---

## 3. Lagrange Multiplier Method

### 3.1 Key Insight — Geometric Intuition

At the constrained optimum, the gradient of the objective must be **parallel** to the gradient of the constraint:

```
∇f(x*) = λ · ∇g(x*)
```

**Why?** If ∇f had a component along the constraint surface, we could still improve f by moving along the constraint — so it can't be optimal.

```
    y
    │        ╭────────────╮  ← constraint g(x,y) = 0
    │       ╱  ↑ ∇g        ╲
    │      ╱   ↑             ╲
    │     │    · ← optimal    │
    │     │    ↑ ∇f           │     At the optimal point:
    │      ╲                 ╱      ∇f ↑↑ ∇g (parallel!)
    │       ╲               ╱
    │        ╰─────────────╯
    │
    │    ─ ─ ─ ─ ─ ─ ─ ─         ← contour lines of f
    │   ─ ─ ─ ─ ─ ─ ─ ─ ─
    └──────────────────────── x
```

### 3.2 The Lagrangian

Define the **Lagrangian function**:

```
L(x, λ) = f(x) + λ · g(x)
```

where λ is the **Lagrange multiplier** (a new variable).

### 3.3 Solution Method

Set all partial derivatives of L to zero:

```
∂L/∂x₁ = 0
∂L/∂x₂ = 0
  ⋮
∂L/∂xₙ = 0
∂L/∂λ  = 0     ← this recovers the constraint g(x) = 0
```

This gives (n + 1) equations in (n + 1) unknowns (x₁, ..., xₙ, λ).

---

## 4. Step-by-Step Worked Example

**Problem:** Find the point on the line x + y = 4 that is closest to the origin.

**Formulation:**
```
minimize   f(x, y) = x² + y²          (distance² from origin)
subject to g(x, y) = x + y − 4 = 0
```

**Step 1:** Write the Lagrangian
```
L(x, y, λ) = x² + y² + λ(x + y − 4)
```

**Step 2:** Compute partial derivatives and set to zero
```
∂L/∂x = 2x + λ = 0     →  x = −λ/2     ... (i)
∂L/∂y = 2y + λ = 0     →  y = −λ/2     ... (ii)
∂L/∂λ = x + y − 4 = 0                   ... (iii)
```

**Step 3:** Solve the system
```
From (i) and (ii):  x = y = −λ/2
Substitute into (iii): (−λ/2) + (−λ/2) = 4  →  −λ = 4  →  λ = −4
Therefore: x = 2, y = 2
```

**Step 4:** Answer
```
Closest point: (2, 2)
Minimum distance² = 4 + 4 = 8
Minimum distance  = 2√2 ≈ 2.83
```

---

## 5. Multiple Constraints

For multiple equality constraints:

```
minimize   f(x)
subject to g₁(x) = 0
           g₂(x) = 0
           ⋮
           gₖ(x) = 0
```

Lagrangian:
```
L(x, λ₁, ..., λₖ) = f(x) + λ₁g₁(x) + λ₂g₂(x) + ··· + λₖgₖ(x)
```

---

## 6. Inequality Constraints — KKT Conditions

### 6.1 The General Problem

```
minimize   f(x)
subject to gᵢ(x) ≤ 0     i = 1, ..., m    (inequality constraints)
           hⱼ(x) = 0     j = 1, ..., p    (equality constraints)
```

### 6.2 Karush-Kuhn-Tucker (KKT) Conditions

The KKT conditions are **necessary** for optimality (and sufficient when f is convex):

```
1. Stationarity:        ∇f(x*) + Σᵢ μᵢ∇gᵢ(x*) + Σⱼ λⱼ∇hⱼ(x*) = 0
2. Primal feasibility:  gᵢ(x*) ≤ 0,  hⱼ(x*) = 0
3. Dual feasibility:    μᵢ ≥ 0
4. Complementary slackness:  μᵢ · gᵢ(x*) = 0   for all i
```

### 6.3 Complementary Slackness — The Key Condition

```
μᵢ · gᵢ(x*) = 0     means:

Either μᵢ = 0  (multiplier is zero — constraint is NOT active)
OR     gᵢ(x*) = 0  (constraint is tight — active/binding)

Both can be zero simultaneously.
```

**Intuition:** A constraint only "matters" (has nonzero multiplier) when it's active (binding). Inactive constraints don't affect the solution.

```
   ┌──────────────────────────┐
   │ Feasible Region          │
   │                          │
   │     · x_unconstrained    │   If unconstrained optimum is
   │       (inside)           │   inside → constraint inactive (μ = 0)
   │                          │
   └──────────────────────────┘

   ┌──────────────────────────┐
   │ Feasible Region          │
   │                     · x* │   If optimum is on boundary
   │                  (on     │   → constraint active (μ > 0)
   │                boundary) │
   │                          │
   └──────────────────────────┘
```

---

## 7. Connection to SVM (Support Vector Machines)

SVMs are the **canonical** ML application of Lagrange multipliers and KKT conditions.

### 7.1 SVM Primal Problem

```
minimize   ½‖w‖²                              (maximize margin)
subject to yᵢ(wᵀxᵢ + b) ≥ 1   for all i     (correct classification)
```

### 7.2 SVM Lagrangian

```
L(w, b, α) = ½‖w‖² − Σᵢ αᵢ [yᵢ(wᵀxᵢ + b) − 1]
```

where αᵢ ≥ 0 are Lagrange multipliers.

### 7.3 KKT Conditions Give Support Vectors

By **complementary slackness**:
```
αᵢ · [yᵢ(wᵀxᵢ + b) − 1] = 0

→ Either αᵢ = 0  (point is NOT a support vector)
  Or     yᵢ(wᵀxᵢ + b) = 1  (point IS a support vector — lies on margin)
```

```
  ────────────────────────── wᵀx + b = 1  (margin)
          ⊕  ⊕                    ↑ support vectors (αᵢ > 0)
     ⊕                        
  ═══════════════════════════ wᵀx + b = 0  (decision boundary)
     ⊖                        
          ⊖  ⊖                    ↑ support vectors (αᵢ > 0)
  ────────────────────────── wᵀx + b = −1 (margin)
       ⊖    ⊖                    ↑ non-support vectors (αᵢ = 0)
                                    these don't affect the solution!
```

> Only the **support vectors** (points on the margin boundary) determine the decision boundary. This is why SVMs are memory-efficient.

---

## 8. Regularization as Constrained Optimization

### 8.1 L2 Regularization (Ridge)

```
minimize   L(w) + λ‖w‖²        ← unconstrained (Lagrangian form)
```

is **equivalent** to the constrained formulation:

```
minimize   L(w)
subject to ‖w‖² ≤ t            ← constrained form
```

The regularization parameter λ and constraint bound t are inversely related — larger λ means smaller t (tighter constraint on weights).

### 8.2 L1 Regularization (Lasso)

```
minimize   L(w)
subject to ‖w‖₁ ≤ t
```

```
  L2 constraint (circle):        L1 constraint (diamond):

        ╭────╮                         ╱╲
       ╱      ╲                       ╱  ╲
      │   ‖w‖² │                     ╱    ╲
      │  ≤  t  │                    ╱ ‖w‖₁ ╲
       ╲      ╱                     ╲ ≤  t  ╱
        ╰────╯                       ╲    ╱
                                      ╲  ╱
  Contours of L(w)                     ╲╱
  touch the circle →                 Contours touch corners →
  small but non-zero w               SPARSE w (some wᵢ = 0)
```

> This geometric picture explains why **L1 produces sparse solutions** (feature selection) — contour lines are more likely to first touch the diamond at a corner (where some weights are exactly zero).

---

## 9. Python Example

```python
import numpy as np
from scipy.optimize import minimize as scipy_minimize

# Problem: minimize x^2 + y^2 subject to x + y = 4
# Using scipy with equality constraint

def objective(xy):
    return xy[0]**2 + xy[1]**2

def constraint_eq(xy):
    return xy[0] + xy[1] - 4  # must equal 0

# Scipy solver
result = scipy_minimize(
    objective,
    x0=[0, 0],
    method='SLSQP',
    constraints={'type': 'eq', 'fun': constraint_eq}
)
print(f"Optimal point: ({result.x[0]:.4f}, {result.x[1]:.4f})")
print(f"Minimum distance²: {result.fun:.4f}")

# Manual Lagrange multiplier solution
# From our derivation: x = y = 2, lambda = -4
print(f"\nAnalytical solution: (2, 2)")
print(f"Lagrange multiplier λ = -4")
print(f"λ interpretation: increasing the constraint bound by ε")
print(f"  changes the optimal value by approximately λ·ε")

# SVM-like example: 2D linearly separable data
from sklearn.svm import SVC

X = np.array([[1, 2], [2, 3], [3, 3], [2, 1], [3, 2], [4, 2]])
y = np.array([1, 1, 1, -1, -1, -1])

svm = SVC(kernel='linear', C=1e6)  # hard margin (large C)
svm.fit(X, y)

print(f"\nSVM Results:")
print(f"Support vectors:\n{svm.support_vectors_}")
print(f"Number of support vectors: {len(svm.support_vectors_)}")
print(f"Dual coefficients (αᵢ·yᵢ): {svm.dual_coef_}")
print(f"Weight vector w: {svm.coef_}")
print(f"Bias b: {svm.intercept_}")
```

---

## 10. Lagrange Multiplier Interpretation

The multiplier λ has a concrete meaning:

```
λ = −df*/dc
```

where f* is the optimal objective value and c is the constraint bound. In other words, λ tells us **how much the optimal value would change if we relaxed the constraint slightly**.

| λ Value    | Meaning                                             |
|------------|-----------------------------------------------------|
| λ = 0     | Constraint is not active; removing it won't help     |
| λ > 0     | Constraint is binding; relaxing it improves objective|
| Large |λ|  | Constraint is "expensive" — strongly limiting        |

---

## 11. Real-World ML Applications

| Application                   | Constrained Optimization Role                          |
|-------------------------------|--------------------------------------------------------|
| SVM                           | Maximize margin subject to correct classification       |
| Regularization (L1/L2)        | Minimize loss subject to weight norm constraint         |
| Fairness Constraints          | Minimize loss s.t. demographic parity                   |
| Resource-Constrained Models   | Minimize error s.t. model size ≤ budget                 |
| Maximum Entropy Models        | Maximize entropy s.t. feature expectation constraints   |
| Variational Inference         | Optimize ELBO (constrained to distribution family)      |
| Robust Optimization           | Minimize worst-case loss over uncertainty set            |

---

## 12. Summary Table

| Concept                  | Key Idea                                             | Formula / Condition               |
|--------------------------|------------------------------------------------------|-----------------------------------|
| Lagrangian               | Combine objective + constraints                      | L = f(x) + λg(x)                 |
| Lagrange multiplier (λ)  | Shadow price of constraint                           | ∇f = λ∇g at optimum              |
| KKT conditions           | Extend to inequality constraints                     | Stationarity + complementarity   |
| Complementary slackness  | μᵢgᵢ = 0 → constraint active or multiplier zero     | Key to identifying active set     |
| SVM connection           | Support vectors have αᵢ > 0                         | Only margin points matter         |
| L2 regularization        | ‖w‖² ≤ t ↔ L(w) + λ‖w‖²                            | Equivalent formulations           |
| L1 sparsity              | Diamond constraint → solutions at corners            | Automatic feature selection       |

---

## 13. Quick Revision Questions

1. **What is the geometric condition for optimality in Lagrange multiplier problems?** *(Answer: ∇f parallel to ∇g)*
2. **Write the Lagrangian for: minimize x²+y² subject to x+2y = 5.**
3. **What does complementary slackness mean in the KKT conditions?**
4. **In SVM, what are support vectors, and how do Lagrange multipliers identify them?**
5. **Explain why L1 regularization leads to sparse solutions using the constraint geometry.**
6. **How is the Lagrange multiplier λ interpreted economically?** *(Answer: marginal cost of the constraint)*

---

> [← Previous: Convex Functions](./07-convex-functions.md)
