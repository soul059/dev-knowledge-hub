# Chapter 5: Taylor Series and Approximations

> **Unit 2 — Calculus for ML** | [← Previous: Jacobian & Hessian](./04-jacobian-and-hessian.md) | [Next → Optimization Basics](./06-optimization-basics.md)

---

## 1. Overview

Taylor series let us **approximate complex functions** using polynomials. In machine learning, Taylor series justify why gradient descent works (first-order approximation), motivate Newton's method (second-order approximation), and help us understand and approximate activation functions. This chapter covers the theory, common expansions, and direct ML connections.

---

## 2. Taylor Series Definition

The Taylor series of f(x) centered at point a is:

```
f(x) = f(a) + f'(a)(x−a) + f''(a)(x−a)²/2! + f'''(a)(x−a)³/3! + ···

     = Σ [f⁽ⁿ⁾(a) / n!] · (x − a)ⁿ     for n = 0, 1, 2, ...
```

When a = 0, this is called the **Maclaurin series**:

```
f(x) = f(0) + f'(0)·x + f''(0)·x²/2! + f'''(0)·x³/3! + ···
```

### Intuition

Each term adds more detail to the approximation:

```
  f(x)
   │      ╭─·──╮
   │    ╱·      ╲         ── actual function
   │  ·╱         ╲        -- 1st order (line)
   │·╱            ╲       ·· 2nd order (parabola)
   ├──────────────────
   │     a
   │
   0th order: f(a)                     → just the value (horizontal line)
   1st order: f(a) + f'(a)(x−a)       → tangent line
   2nd order: + f''(a)(x−a)²/2!       → parabolic fit
   3rd order: + f'''(a)(x−a)³/3!      → cubic fit (even better)
```

---

## 3. First-Order Approximation (Linear)

```
f(x) ≈ f(a) + f'(a)(x − a)
```

This is just the **tangent line** at x = a.

**Multivariable version:**
```
f(x) ≈ f(a) + ∇f(a)ᵀ · (x − a)
```

> **ML Connection:** Gradient descent assumes the loss function is approximately linear near the current point. It steps in the direction that decreases this linear approximation.

---

## 4. Second-Order Approximation (Quadratic)

```
f(x) ≈ f(a) + f'(a)(x−a) + ½ f''(a)(x−a)²
```

**Multivariable version:**
```
f(x) ≈ f(a) + ∇f(a)ᵀ(x−a) + ½(x−a)ᵀ H(a) (x−a)
```

where H is the **Hessian matrix**.

> **ML Connection:** Newton's method minimizes this quadratic approximation exactly at each step → faster convergence than gradient descent.

---

## 5. Common Taylor/Maclaurin Expansions

### 5.1 Exponential Function

```
eˣ = 1 + x + x²/2! + x³/3! + x⁴/4! + ···

   = Σ xⁿ/n!     for all x
```

### 5.2 Sine Function

```
sin(x) = x − x³/3! + x⁵/5! − x⁷/7! + ···

       = Σ (−1)ⁿ x²ⁿ⁺¹/(2n+1)!
```

### 5.3 Cosine Function

```
cos(x) = 1 − x²/2! + x⁴/4! − x⁶/6! + ···

       = Σ (−1)ⁿ x²ⁿ/(2n)!
```

### 5.4 Geometric Series

```
1/(1−x) = 1 + x + x² + x³ + ···     for |x| < 1
```

### 5.5 Natural Logarithm

```
ln(1+x) = x − x²/2 + x³/3 − x⁴/4 + ···     for |x| ≤ 1
```

### Summary of Expansions

| Function     | Maclaurin Series                        | Radius of Convergence |
|-------------|------------------------------------------|----------------------|
| eˣ           | 1 + x + x²/2! + x³/3! + ···           | ∞                    |
| sin(x)       | x − x³/3! + x⁵/5! − ···               | ∞                    |
| cos(x)       | 1 − x²/2! + x⁴/4! − ···               | ∞                    |
| 1/(1−x)      | 1 + x + x² + x³ + ···                  | |x| < 1             |
| ln(1+x)      | x − x²/2 + x³/3 − ···                  | |x| ≤ 1             |

---

## 6. Step-by-Step Example

**Problem:** Find the Taylor series of f(x) = eˣ centered at a = 2, up to 3rd order.

**Step 1:** Compute derivatives at a = 2
```
f(x)   = eˣ     →  f(2)   = e²
f'(x)  = eˣ     →  f'(2)  = e²
f''(x) = eˣ     →  f''(2) = e²
f'''(x)= eˣ     →  f'''(2)= e²
```

**Step 2:** Write the Taylor polynomial
```
T₃(x) = e² + e²(x−2) + e²(x−2)²/2! + e²(x−2)³/3!

       = e² [1 + (x−2) + (x−2)²/2 + (x−2)³/6]
```

**Step 3:** Verify at x = 2.1
```
T₃(2.1) = e² [1 + 0.1 + 0.005 + 0.000167] = e² × 1.105167
         = 7.389 × 1.105167 ≈ 8.166

Exact: e^2.1 ≈ 8.166    ✓ (excellent approximation near a)
```

---

## 7. Taylor Series in ML

### 7.1 Justifying Gradient Descent

Gradient descent is justified by the **first-order Taylor approximation**:

```
L(w − α∇L) ≈ L(w) − α ‖∇L‖²
                       ↑
                   always ≤ 0, so loss decreases!
```

As long as α is small enough, the linear approximation is valid, and the loss decreases.

### 7.2 Justifying Newton's Method

The second-order approximation gives a **quadratic model** of the loss:

```
L(w + Δw) ≈ L(w) + ∇Lᵀ Δw + ½ Δwᵀ H Δw
```

Setting the derivative to 0: ∇L + H·Δw = 0 → **Δw = −H⁻¹∇L** (Newton step).

### 7.3 Approximating Activation Functions

**Sigmoid near x = 0:**
```
σ(x) ≈ 0.5 + 0.25x − 0.0208x³ + ···    (1st order: σ(x) ≈ 0.5 + 0.25x)
```

**Softplus near x = 0:**
```
log(1 + eˣ) ≈ log(2) + x/2 + x²/8 − ···
```

These polynomial approximations are used in **efficient hardware implementations** of activation functions.

---

## 8. Python Demonstration

```python
import numpy as np
import matplotlib.pyplot as plt

def taylor_exp(x, a, order):
    """Taylor approximation of e^x centered at a."""
    result = 0
    for n in range(order + 1):
        # f^(n)(a) = e^a for all n, for f(x) = e^x
        result += (np.exp(a) * (x - a)**n) / np.math.factorial(n)
    return result

x = np.linspace(-1, 4, 200)
a = 2  # Center of expansion

plt.figure(figsize=(10, 6))
plt.plot(x, np.exp(x), 'k-', linewidth=2, label='e^x (exact)')

for order in [0, 1, 2, 3, 5]:
    y_approx = taylor_exp(x, a, order)
    plt.plot(x, y_approx, '--', label=f'Order {order}')

plt.ylim(-5, 30)
plt.axvline(x=a, color='gray', linestyle=':', alpha=0.5)
plt.xlabel('x'); plt.ylabel('f(x)')
plt.title(f'Taylor Approximations of e^x centered at a = {a}')
plt.legend(); plt.grid(True, alpha=0.3)
plt.show()

# Sigmoid approximation
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

x = np.linspace(-3, 3, 100)
sig_exact = sigmoid(x)
sig_linear = 0.5 + 0.25 * x       # 1st order Taylor at 0
sig_cubic = 0.5 + 0.25*x - (1/48)*x**3  # 3rd order

print("Sigmoid approximation errors at x=0.5:")
print(f"  Exact:    {sigmoid(0.5):.6f}")
print(f"  Linear:   {0.5 + 0.25*0.5:.6f}  (error: {abs(sigmoid(0.5) - 0.625):.6f})")
print(f"  Cubic:    {0.5 + 0.25*0.5 - (1/48)*0.5**3:.6f}")
```

---

## 9. Multivariate Taylor Expansion — Complete Form

For f: ℝⁿ → ℝ, expanded around point **a**:

```
f(x) ≈ f(a)                              ← 0th order (constant)
     + ∇f(a)ᵀ (x − a)                    ← 1st order (linear, uses gradient)
     + ½ (x − a)ᵀ H(a) (x − a)          ← 2nd order (quadratic, uses Hessian)
     + higher order terms ...
```

| Order | Uses                | Approximation Type | ML Application         |
|-------|---------------------|--------------------|-----------------------|
| 0     | f(a)                | Constant           | Baseline prediction    |
| 1     | Gradient ∇f         | Linear (plane)     | Gradient descent       |
| 2     | Hessian H           | Quadratic (bowl)   | Newton's method        |
| 3+    | Higher derivatives  | Polynomial         | Rarely used in ML      |

---

## 10. Real-World ML Applications

| Application                   | Taylor Series Role                                    |
|-------------------------------|-------------------------------------------------------|
| Gradient Descent              | Justified by 1st-order approximation                  |
| Newton's Method / L-BFGS      | Based on 2nd-order approximation                      |
| Activation Approximation      | Polynomial approx. for hardware efficiency             |
| Loss Surface Analysis         | Local quadratic model reveals curvature               |
| Natural Gradient Descent      | Uses Fisher information (related to 2nd-order approx) |
| GELU Activation               | Defined using Taylor expansion of Gaussian CDF        |

---

## 11. Summary Table

| Concept                 | Formula                                         | ML Relevance                       |
|-------------------------|--------------------------------------------------|------------------------------------|
| Taylor series           | Σ f⁽ⁿ⁾(a)(x−a)ⁿ / n!                           | Foundation of approximation theory |
| Maclaurin series        | Taylor series at a = 0                            | Common function expansions         |
| 1st-order approx.       | f(a) + f'(a)(x−a)                                | Gradient descent justification     |
| 2nd-order approx.       | + ½f''(a)(x−a)²                                  | Newton's method justification      |
| Multivariate 2nd-order  | f(a) + ∇fᵀΔx + ½ΔxᵀHΔx                         | Quadratic loss model               |
| Convergence radius      | Range where series converges                      | Validity of approximation          |

---

## 12. Quick Revision Questions

1. **Write the Maclaurin series for eˣ up to the x³ term.**
2. **What order Taylor approximation does gradient descent implicitly use?** *(Answer: 1st order)*
3. **How does Newton's method improve upon gradient descent, from a Taylor series perspective?**
4. **Why does the Taylor approximation become less accurate far from the expansion point?**
5. **Approximate sigmoid(0.1) using the first-order Taylor expansion at 0.** *(Answer: 0.5 + 0.25(0.1) = 0.525; exact ≈ 0.525)*
6. **What is the radius of convergence of the geometric series 1/(1−x)?**

---

> [← Previous: Jacobian & Hessian](./04-jacobian-and-hessian.md) | [Next → Optimization Basics](./06-optimization-basics.md)
