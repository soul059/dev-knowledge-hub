# Chapter 7: Constrained Optimization

> **Unit 6 · Optimization · Module 7 of 7**

---

## 7.1 Overview

Many real-world optimization problems have **constraints** — limits on resources, physical laws, fairness requirements, or model capacity. Constrained optimization provides the mathematical framework (Lagrangians, KKT conditions) that underpins critical ML methods including SVMs, regularization, and probabilistic inference. This chapter bridges pure optimization theory with practical ML applications.

---

## 7.2 Problem Formulation

### General Constrained Optimization Problem

```
minimize    f(x)                       ← objective function
subject to  gᵢ(x) ≤ 0,  i = 1,...,m   ← inequality constraints
            hⱼ(x) = 0,  j = 1,...,p   ← equality constraints
```

| Component | Symbol | Example |
|-----------|--------|---------|
| Objective | f(x) | Loss function |
| Inequality constraints | gᵢ(x) ≤ 0 | Budget limits, capacity |
| Equality constraints | hⱼ(x) = 0 | Conservation laws, normalization |

### Feasible Region

The **feasible set** is the intersection of all constraint sets:

```
F = {x : gᵢ(x) ≤ 0 ∀i, hⱼ(x) = 0 ∀j}
```

```
  x₂ ↑
     |        ╱ g₁(x) ≤ 0
     |       ╱
     |    ╭─╱──────╮
     |    │╱       │    ← Feasible Region F
     |    ╱  ●x*   │       (intersection of constraints)
     |   ╱│        │
     |  ╱ ╰────────╯
     | ╱      g₂(x) ≤ 0
     +————————————————→ x₁

  ●x* = constrained optimum (may differ from unconstrained!)
```

---

## 7.3 Equality Constraints

### The Lagrangian Approach

For `minimize f(x) subject to h(x) = 0`:

```
L(x, ν) = f(x) + ν · h(x)
```

At the optimum, the gradient of the objective must be **parallel** to the gradient of the constraint:

```
∇f(x*) = −ν · ∇h(x*)
```

### Geometric Intuition

```
  x₂ ↑
     |    ╭─ f = c₁ (contours of f)
     |   ╭┤
     |  ╭┤│    ∇f ↗
     |  │││  ╱
     |  │││╱        h(x) = 0  (constraint curve)
     |  ││╱·····●x*·····
     |  │╱      ↑ ∇h
     |  ╱
     +————————————————→ x₁

  At x*: ∇f is parallel to ∇h (tangent to contour is tangent to constraint)
```

---

## 7.4 Step-by-Step Example: Lagrange Multipliers

**Problem**: Minimize `f(x,y) = x² + y²` subject to `x + y = 4`

**Step 1**: Form the Lagrangian:
```
L(x, y, ν) = x² + y² + ν(x + y − 4)
```

**Step 2**: Set partial derivatives to zero:
```
∂L/∂x = 2x + ν = 0   →  x = −ν/2
∂L/∂y = 2y + ν = 0   →  y = −ν/2
∂L/∂ν = x + y − 4 = 0
```

**Step 3**: Solve the system:
```
x = y = −ν/2
−ν/2 + (−ν/2) = 4  →  −ν = 4  →  ν = −4

x* = 2,  y* = 2,  f(x*, y*) = 8
```

**Verification**: The unconstrained minimum is (0,0) with f=0. The constraint forces us to a higher value of f=8, with the optimal point at (2,2).

---

## 7.5 Inequality Constraints and KKT Conditions

For inequality constraints, we need the **Karush-Kuhn-Tucker (KKT)** conditions, which generalize Lagrange multipliers.

### The Lagrangian (General Form)

```
L(x, λ, ν) = f(x) + Σᵢ λᵢ gᵢ(x) + Σⱼ νⱼ hⱼ(x)
```

### KKT Conditions (Necessary for Optimality)

| Condition | Formula | Meaning |
|-----------|---------|---------|
| **1. Stationarity** | `∇ₓL = 0` | Gradient of Lagrangian is zero |
| **2. Primal feasibility** | `gᵢ(x*) ≤ 0, hⱼ(x*) = 0` | Solution satisfies all constraints |
| **3. Dual feasibility** | `λᵢ ≥ 0` | Multipliers for inequalities are non-negative |
| **4. Complementary slackness** | `λᵢ · gᵢ(x*) = 0` | Either constraint is active OR multiplier is zero |

### Complementary Slackness Explained

```
For each inequality constraint gᵢ(x) ≤ 0:

Case 1: gᵢ(x*) < 0  (constraint NOT active / slack)
         → λᵢ = 0     (constraint doesn't affect solution)

Case 2: gᵢ(x*) = 0  (constraint IS active / binding)
         → λᵢ ≥ 0     (constraint pushes the solution)

In either case: λᵢ · gᵢ(x*) = 0  ✓
```

---

## 7.6 KKT Example

**Problem**: Minimize `f(x) = (x−3)²` subject to `x ≤ 2`

Rewrite constraint: `g(x) = x − 2 ≤ 0`

**Lagrangian**: `L(x, λ) = (x−3)² + λ(x−2)`

**KKT Conditions**:

1. **Stationarity**: `∂L/∂x = 2(x−3) + λ = 0  →  λ = −2(x−3) = 6−2x`
2. **Primal feasibility**: `x ≤ 2`
3. **Dual feasibility**: `λ ≥ 0`
4. **Complementary slackness**: `λ(x−2) = 0`

**Case 1**: λ = 0 → x = 3 → but g(3) = 1 > 0 ❌ (violates primal feasibility)

**Case 2**: x = 2 (constraint active) → λ = 6−2(2) = 2 > 0 ✅

**Solution**: x* = 2, λ* = 2, f(x*) = 1

```
  f(x) = (x-3)²
   |
   |     ╱         ╲
   |    ╱           ╲
   |   ╱      unconstrained
   |  ╱       min at x=3
   | ╱
  1|●─ ─ ─ ─ ─ ─ ─ ─ ╲     ← constrained min at x=2
   |│                   ╲
   +──┼──────────┼──────→ x
      2          3

  The wall at x=2 prevents reaching the unconstrained minimum.
```

---

## 7.7 Connection to SVM Dual Problem

The SVM primal problem is:

```
minimize    (1/2)‖w‖² + C Σᵢ ξᵢ
subject to  yⁱ(wᵀxⁱ + b) ≥ 1 − ξᵢ     ∀i
            ξᵢ ≥ 0                       ∀i
```

Applying the KKT conditions and Lagrangian duality yields the **SVM dual**:

```
maximize    Σᵢ αᵢ − (1/2) Σᵢ Σⱼ αᵢαⱼyⁱyʲ(xⁱ·xʲ)
subject to  0 ≤ αᵢ ≤ C     ∀i
            Σᵢ αᵢyⁱ = 0
```

### KKT in SVM Context

| KKT Condition | SVM Interpretation |
|--------------|-------------------|
| Complementary slackness | `αᵢ[yⁱ(wᵀxⁱ+b) − 1 + ξᵢ] = 0` |
| αᵢ = 0 | Point is correctly classified (not a support vector) |
| 0 < αᵢ < C | Point is exactly on the margin (support vector) |
| αᵢ = C | Point is inside margin or misclassified |

```
  SVM Decision Boundary:

  x₂ ↑      ○         ○
     |         ○               ○ = class +1
     |  ─ ─●─ ─ ─ ─ ─ ─ ─ margin
     |     ╱    support
     |    ╱     vectors (αᵢ > 0)
     |───●─────────────── decision boundary
     |  ╱
     |─●─ ─ ─ ─ ─ ─ ─ ─ margin
     |  ×          ×           × = class −1
     +————————————————————→ x₁
```

---

## 7.8 Regularization as a Constraint

Regularization in ML can be viewed as constrained optimization:

### Equivalence (L2 Regularization ↔ Constraint)

```
Regularized form:               Constrained form:
minimize  L(θ) + λ‖θ‖²    ⟺    minimize  L(θ)
                                  subject to ‖θ‖² ≤ t
```

There exists a one-to-one mapping between λ and t.

### Visual: L1 vs L2 Constraint Regions

```
  θ₂ ↑                       θ₂ ↑
     |    ╱╲                     |   ╭───╮
     |   ╱  ╲                    |  ╱     ╲
     |  ╱ ●  ╲  L1 (Lasso)      | │  ●    │  L2 (Ridge)
     |  ╲    ╱   diamond         | │       │  circle
     |   ╲  ╱                    |  ╲     ╱
     |    ╲╱                     |   ╰───╯
     +————————→ θ₁               +————————→ θ₁

  ● = constrained optimum        ● = constrained optimum
  L1 pushes to corners (sparse)  L2 shrinks uniformly
```

---

## 7.9 Penalty Methods

Convert a constrained problem to an unconstrained one by adding a penalty for constraint violations:

```
minimize  f(x) + ρ · P(x)
```

Where `P(x)` penalizes infeasibility:

| Method | Penalty P(x) | Pros | Cons |
|--------|-------------|------|------|
| Quadratic penalty | `Σ max(0, gᵢ(x))²` | Simple, differentiable | Needs ρ→∞ |
| Exact penalty (L1) | `Σ max(0, gᵢ(x))` | Finite ρ suffices | Non-smooth |
| Log-barrier | `−Σ log(−gᵢ(x))` | Interior-point, smooth | Only for strict interior |

### Example: Quadratic Penalty

```
Original:     minimize  f(x)  s.t.  g(x) ≤ 0
Penalized:    minimize  f(x) + (ρ/2) · max(0, g(x))²

As ρ → ∞, solution of penalized → solution of original
```

---

## 7.10 Augmented Lagrangian Method

Combines the penalty method with Lagrangian multipliers for faster convergence:

```
L_A(x, λ, ρ) = f(x) + Σᵢ λᵢ gᵢ(x) + (ρ/2) Σᵢ max(0, gᵢ(x))²
```

### Algorithm (ADMM-style)

```
Repeat:
  1. x_{k+1} = arg min_x  L_A(x, λ_k, ρ)        ← solve subproblem
  2. λ_{k+1} = λ_k + ρ · g(x_{k+1})               ← update multipliers
  3. Optionally increase ρ
Until convergence
```

**Advantage**: Converges with finite ρ (unlike pure penalty methods).

---

## 7.11 Python Implementation

```python
import numpy as np
from scipy.optimize import minimize

# === Equality constraint with Lagrange multipliers ===
# minimize x² + y² subject to x + y = 4

def objective(xy):
    return xy[0]**2 + xy[1]**2

constraints = [
    {'type': 'eq', 'fun': lambda xy: xy[0] + xy[1] - 4}
]

result = minimize(objective, x0=[0, 0], constraints=constraints, method='SLSQP')
print(f"x* = {result.x}")    # [2.0, 2.0]
print(f"f* = {result.fun}")  # 8.0


# === Inequality constraint (KKT example) ===
# minimize (x-3)² subject to x ≤ 2

def f_kkt(x):
    return (x[0] - 3)**2

constraints_ineq = [
    {'type': 'ineq', 'fun': lambda x: 2 - x[0]}  # x ≤ 2 → 2 - x ≥ 0
]

result = minimize(f_kkt, x0=[0], constraints=constraints_ineq, method='SLSQP')
print(f"x* = {result.x[0]:.4f}")   # 2.0000
print(f"f* = {result.fun:.4f}")     # 1.0000
```

### Penalty Method from Scratch

```python
def penalty_method(f, g, x0, rho_init=1.0, rho_max=1e6, rho_mult=10):
    """
    Quadratic penalty method.
    f: objective function
    g: constraint g(x) ≤ 0
    """
    rho = rho_init
    x = np.array(x0, dtype=float)

    while rho <= rho_max:
        # Penalized objective
        def penalized(x):
            violation = max(0, g(x))
            return f(x) + (rho / 2) * violation**2

        result = minimize(penalized, x, method='Nelder-Mead')
        x = result.x
        rho *= rho_mult
        print(f"ρ={rho:.0f}, x={x}, f={f(x):.4f}, g={g(x):.4f}")

    return x

# minimize (x-3)² subject to x ≤ 2
x_opt = penalty_method(
    f=lambda x: (x[0]-3)**2,
    g=lambda x: x[0] - 2,
    x0=[5.0]
)
```

---

## 7.12 Practical Applications in ML

| Application | Constraint Type | Method |
|------------|----------------|--------|
| **SVM** | Margin constraints | KKT / Dual (SMO) |
| **L1 Regularization (Lasso)** | ‖θ‖₁ ≤ t | Proximal gradient |
| **L2 Regularization (Ridge)** | ‖θ‖₂² ≤ t | Closed-form / GD |
| **Fairness in ML** | Demographic parity constraints | Constrained optimization |
| **Portfolio Optimization** | Budget constraint, risk limits | QP solvers |
| **Max-Entropy Models** | Probability normalization | Lagrange multipliers |
| **Physics-Informed NN** | PDE constraints as penalties | Penalty / augmented Lagrangian |
| **Neural Architecture Search** | FLOPS / latency budget | Constrained search |

---

## 7.13 Key Theorems Summary

| Theorem | Statement |
|---------|----------|
| **Lagrange necessity** | If x* is a local min with equality constraints, ∃ ν such that ∇f + Σ νⱼ∇hⱼ = 0 |
| **KKT necessity** | Under constraint qualification, KKT conditions are necessary for optimality |
| **KKT sufficiency** | For convex problems, KKT conditions are also sufficient |
| **Strong duality** | For convex problems satisfying Slater's condition, primal = dual optimum |
| **Complementary slackness** | At optimum: λᵢgᵢ(x*) = 0 for each inequality |

---

## 7.14 Summary Table

| Concept | Key Detail |
|---------|-----------|
| Equality constraints | Solved via Lagrange multipliers: `∇f = −ν∇h` |
| Inequality constraints | Require KKT conditions |
| KKT (4 conditions) | Stationarity, primal feasibility, dual feasibility, complementary slackness |
| Complementary slackness | Either constraint is active (g=0) or multiplier is zero (λ=0) |
| Lagrangian | `L = f + Σλᵢgᵢ + Σνⱼhⱼ` |
| Duality | Dual problem gives lower bound; equality for convex under Slater |
| Regularization | Equivalent to constrained optimization (penalty ↔ constraint) |
| Penalty method | Add ρ·violation² to objective; increase ρ |
| Augmented Lagrangian | Penalty + multiplier updates; converges with finite ρ |
| SVM connection | KKT gives support vectors; dual enables kernel trick |

---

## 7.15 Quick Revision Questions

1. **What are the four KKT conditions?**
   → Stationarity (∇L=0), primal feasibility, dual feasibility (λ≥0), complementary slackness (λᵢgᵢ=0).

2. **What does complementary slackness mean intuitively?**
   → A constraint either actively affects the solution (binding) or is irrelevant (slack with zero multiplier).

3. **How is L2 regularization related to constrained optimization?**
   → Minimizing L(θ)+λ‖θ‖² is equivalent to minimizing L(θ) subject to ‖θ‖² ≤ t for some t.

4. **What are support vectors in terms of KKT conditions?**
   → Points where αᵢ > 0 (the dual variable is non-zero), meaning the margin constraint is active.

5. **Why is the penalty method inferior to the augmented Lagrangian?**
   → Penalty method needs ρ→∞ (ill-conditioning), while augmented Lagrangian converges with finite ρ.

6. **When are KKT conditions sufficient (not just necessary)?**
   → When the problem is convex (convex f, convex gᵢ, affine hⱼ).

---

| [← Previous: Convex Optimization](06-convex-optimization.md) | [Unit 6 Complete ✅](../README.md) |
|:---|---:|
