# Chapter 1: Derivatives and Partial Derivatives

> **Unit 2 — Calculus for ML** | [Next Chapter → Gradient and Gradient Descent](./02-gradient-and-gradient-descent.md)

---

## 1. Overview

Derivatives measure how a function changes as its input changes. In machine learning, derivatives tell us **how the loss changes when we adjust a weight** — the fundamental mechanism behind training every model. This chapter builds from the basic definition through differentiation rules to partial derivatives in multiple dimensions.

---

## 2. The Derivative — Definition

The derivative of f(x) at a point x is the **instantaneous rate of change**:

```
            f(x + h) - f(x)
f'(x) = lim ─────────────────
         h→0        h
```

**Geometric interpretation:** The slope of the tangent line to f(x) at point x.

```
  f(x)
   │        ╱ ← tangent line (slope = f'(x))
   │      ╱·
   │    ╱· ·
   │  ·    ·
   │·      ·
   └───────┼──── x
           x₀
```

---

## 3. Notation Systems

| Notation     | Style     | Written As         | Read As                        |
|-------------|-----------|--------------------|---------------------------------|
| Leibniz      | Fraction  | df/dx, ∂f/∂x      | "d-f d-x"                      |
| Lagrange     | Prime     | f'(x), f''(x)     | "f prime of x"                  |
| Euler        | Operator  | Df, D²f            | "D of f"                        |
| Nabla        | Gradient  | ∇f                 | "del f" or "nabla f"            |

In ML papers you'll encounter all of these — Leibniz for clarity, Lagrange for brevity, Nabla for gradients.

---

## 4. Core Differentiation Rules

### 4.1 Power Rule

```
f(x) = xⁿ   →   f'(x) = n·xⁿ⁻¹
```

**Example:** f(x) = x⁴ → f'(x) = 4x³

### 4.2 Sum Rule

```
(f + g)' = f' + g'
```

### 4.3 Product Rule

```
(f · g)' = f' · g + f · g'
```

**Example:** f(x) = x² · sin(x)
```
f'(x) = 2x · sin(x) + x² · cos(x)
```

### 4.4 Quotient Rule

```
(f / g)' = (f' · g - f · g') / g²
```

### 4.5 Chain Rule (Single Variable)

```
d/dx [f(g(x))] = f'(g(x)) · g'(x)
```

**Example:** f(x) = (3x + 1)⁵
```
f'(x) = 5·(3x + 1)⁴ · 3 = 15·(3x + 1)⁴
```

### 4.6 Common Derivatives Table

| Function f(x) | Derivative f'(x)       |
|---------------|------------------------|
| xⁿ            | n·xⁿ⁻¹                |
| eˣ            | eˣ                     |
| ln(x)         | 1/x                    |
| sin(x)        | cos(x)                 |
| cos(x)        | −sin(x)                |
| σ(x)=1/(1+e⁻ˣ)| σ(x)·(1 − σ(x))      |

> **ML Note:** The sigmoid derivative σ'(x) = σ(x)(1−σ(x)) is used constantly in logistic regression and neural networks.

---

## 5. Higher-Order Derivatives

The second derivative measures the **rate of change of the rate of change** (curvature):

```
f''(x) = d²f/dx²  =  d/dx [f'(x)]
```

**Example:** f(x) = x³ − 6x² + 9x + 1
```
f'(x)  = 3x² − 12x + 9
f''(x) = 6x − 12
```

- f''(x) > 0 → function is **concave up** (local minimum region)
- f''(x) < 0 → function is **concave down** (local maximum region)
- f''(x) = 0 → possible **inflection point**

---

## 6. Partial Derivatives

When f depends on **multiple variables**, a partial derivative measures the change with respect to **one variable while holding others constant**.

```
∂f/∂x = lim  [f(x + h, y) − f(x, y)] / h
         h→0
```

### Step-by-Step Example

**Given:** f(x, y) = 3x²y + 2xy³ − 5y + 7

**Find ∂f/∂x:** Treat y as a constant.
```
∂f/∂x = 6xy + 2y³
```

**Find ∂f/∂y:** Treat x as a constant.
```
∂f/∂y = 3x² + 6xy² − 5
```

### Second-Order Partial Derivatives

```
∂²f/∂x²  = ∂/∂x(∂f/∂x) = 6y

∂²f/∂y²  = ∂/∂y(∂f/∂y) = 12xy

∂²f/∂x∂y = ∂/∂x(∂f/∂y) = 6x + 6y²   (mixed partial)
```

> **Clairaut's Theorem:** If second partial derivatives are continuous, then ∂²f/∂x∂y = ∂²f/∂y∂x.

---

## 7. ML Connection — How Derivatives Drive Learning

In ML, we define a **loss function** L(w) that measures prediction error. Training = finding weights w that **minimize** L(w).

```
  Loss L(w)
   │
   │  ╲          We want to move w in the
   │   ╲         direction that DECREASES L
   │    ╲
   │     ╲___╱   ← minimum (optimal w)
   │
   └──────────── w
         ↑
    dL/dw tells us
    which direction
    to move
```

**Weight update rule:**
```
w_new = w_old − α · (dL/dw)
```
where α is the learning rate. This is **gradient descent** — covered fully in Chapter 2.

### Python Example: Computing Derivatives

```python
import numpy as np

# Numerical derivative
def numerical_derivative(f, x, h=1e-7):
    return (f(x + h) - f(x - h)) / (2 * h)

# Example: derivative of f(x) = x^3 - 2x + 1
f = lambda x: x**3 - 2*x + 1
x0 = 2.0

df = numerical_derivative(f, x0)
print(f"f'({x0}) ≈ {df:.4f}")          # Expected: 3(4) - 2 = 10
# Output: f'(2.0) ≈ 10.0000

# Partial derivative example
def f_multi(x, y):
    return 3*x**2*y + 2*x*y**3

def partial_x(f, x, y, h=1e-7):
    return (f(x + h, y) - f(x - h, y)) / (2 * h)

def partial_y(f, x, y, h=1e-7):
    return (f(x, y + h) - f(x, y - h)) / (2 * h)

print(f"∂f/∂x at (1,2) ≈ {partial_x(f_multi, 1, 2):.4f}")  # 6(1)(2)+2(8)=28
print(f"∂f/∂y at (1,2) ≈ {partial_y(f_multi, 1, 2):.4f}")  # 3(1)+6(1)(4)=27
```

---

## 8. Real-World ML Applications

| Application                  | Role of Derivatives                                 |
|------------------------------|-----------------------------------------------------|
| Linear Regression            | dL/dw gives gradient for weight updates              |
| Logistic Regression          | Sigmoid derivative in gradient computation           |
| Neural Networks              | Chain rule (backprop) computes all weight gradients   |
| Regularization (L2)          | Adds 2λw to gradient, penalizing large weights       |
| Learning Rate Schedulers     | Second derivatives inform adaptive learning rates    |

---

## 9. Summary Table

| Concept                | Key Formula                          | Why It Matters in ML                  |
|------------------------|--------------------------------------|----------------------------------------|
| Derivative             | lim [f(x+h)−f(x)] / h               | Measures rate of change                |
| Power Rule             | d/dx[xⁿ] = nxⁿ⁻¹                   | Fast polynomial differentiation        |
| Product Rule           | (fg)' = f'g + fg'                    | Composite loss terms                   |
| Chain Rule             | d/dx[f(g(x))] = f'(g(x))·g'(x)     | Foundation of backpropagation          |
| Partial Derivative     | ∂f/∂xᵢ (hold others constant)       | Multi-weight optimization              |
| Higher-Order Deriv.    | f''(x) = d²f/dx²                    | Curvature, second-order optimizers     |

---

## 10. Quick Revision Questions

1. **What does the derivative of a function at a point represent geometrically?**
2. **Apply the chain rule:** Find d/dx of f(x) = e^(3x²). *(Answer: 6x·e^(3x²))*
3. **Compute partial derivatives:** Given f(x,y) = x²y − 3xy² + y, find ∂f/∂x and ∂f/∂y.
4. **Why is the sigmoid derivative σ'(x) = σ(x)(1−σ(x)) computationally convenient?**
5. **In gradient descent, why do we subtract the derivative from the current weight?**
6. **What does a positive second derivative f''(x) > 0 tell you about the shape of f at x?**

---

> [Next Chapter → Gradient and Gradient Descent](./02-gradient-and-gradient-descent.md)
