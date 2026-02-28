# Chapter 7.4: Adams-Bashforth Methods (Multi-step)

[← Previous: Runge-Kutta](03-runge-kutta.md) | [Next: Milne's Method →](05-milnes-method.md)

---

## Overview

**Adams-Bashforth (AB) methods** are **explicit multi-step** methods that use previously computed solution values to advance the solution. Unlike single-step Runge-Kutta, they are very economical — only **one new $f$-evaluation per step** — but they require starting values from another method.

---

## 1. General Form

$$y_{n+1} = y_n + h \sum_{j=0}^{s-1} b_j f_{n-j}$$

The method uses $f$ values at $x_n, x_{n-1}, \ldots, x_{n-s+1}$ (past points only → **explicit**).

---

## 2. Common Adams-Bashforth Formulas

### 1-step (AB1 = Euler)
$$y_{n+1} = y_n + hf_n$$

### 2-step (AB2)
$$\boxed{y_{n+1} = y_n + \frac{h}{2}(3f_n - f_{n-1})}$$

### 3-step (AB3)
$$y_{n+1} = y_n + \frac{h}{12}(23f_n - 16f_{n-1} + 5f_{n-2})$$

### 4-step (AB4)
$$\boxed{y_{n+1} = y_n + \frac{h}{24}(55f_n - 59f_{n-1} + 37f_{n-2} - 9f_{n-3})}$$

---

## 3. Derivation (AB2)

Integrate $y' = f(x,y)$ from $x_n$ to $x_{n+1}$:

$$y_{n+1} - y_n = \int_{x_n}^{x_{n+1}} f(x, y(x))\,dx$$

Fit a polynomial through $(x_{n-1}, f_{n-1})$ and $(x_n, f_n)$:

$$p(x) = f_n + \frac{f_n - f_{n-1}}{h}(x - x_n)$$

Integrate:

$$\int_{x_n}^{x_{n+1}} p(x)\,dx = hf_n + \frac{h}{2}(f_n - f_{n-1}) = \frac{h}{2}(3f_n - f_{n-1})$$

---

## 4. Error Analysis

| Method | Order | Local Error | Coefficients |
|--------|-------|-------------|-------------|
| AB1 | 1 | $O(h^2)$ | $1$ |
| AB2 | 2 | $\frac{5h^3}{12}y'''$ | $3/2, -1/2$ |
| AB3 | 3 | $\frac{3h^4}{8}y^{(4)}$ | $23/12, -16/12, 5/12$ |
| AB4 | 4 | $\frac{251h^5}{720}y^{(5)}$ | $55/24, -59/24, 37/24, -9/24$ |

---

## 5. Worked Example (AB2)

Solve $y' = y$, $y(0) = 1$ with $h = 0.1$. Starting values from exact: $y_0 = 1, y_1 = e^{0.1} = 1.10517$.

$f_0 = y_0 = 1, \quad f_1 = y_1 = 1.10517$

$$y_2 = y_1 + \frac{0.1}{2}(3f_1 - f_0) = 1.10517 + 0.05(3(1.10517) - 1)$$

$$= 1.10517 + 0.05(3.31551 - 1) = 1.10517 + 0.05(2.31551)$$

$$= 1.10517 + 0.11578 = 1.22095$$

**Exact:** $e^{0.2} = 1.22140$. Error: $0.00045$.

---

## 6. Adams-Moulton (Implicit Companion)

The **Adams-Moulton (AM)** formulas are the **implicit** counterpart:

### AM2 (Trapezoidal)
$$y_{n+1} = y_n + \frac{h}{2}(f_{n+1} + f_n)$$

### AM3
$$y_{n+1} = y_n + \frac{h}{12}(5f_{n+1} + 8f_n - f_{n-1})$$

### AM4
$$y_{n+1} = y_n + \frac{h}{24}(9f_{n+1} + 19f_n - 5f_{n-1} + f_{n-2})$$

These are more accurate but require solving an implicit equation at each step.

---

## 7. Predictor-Corrector: Adams-Bashforth-Moulton

Use AB as **predictor** and AM as **corrector**:

1. **Predict** with AB4: $y^*_{n+1} = y_n + \frac{h}{24}(55f_n - 59f_{n-1} + 37f_{n-2} - 9f_{n-3})$
2. **Evaluate**: $f^*_{n+1} = f(x_{n+1}, y^*_{n+1})$
3. **Correct** with AM4: $y_{n+1} = y_n + \frac{h}{24}(9f^*_{n+1} + 19f_n - 5f_{n-1} + f_{n-2})$

This gives 4th-order accuracy with only **2 $f$-evaluations** per step!

---

## 8. Starting the Method

Multi-step methods need initial values ($y_0, y_1, \ldots, y_{s-1}$). These are obtained from:
- **RK4** (most common — compute first few steps with RK4, then switch to AB)
- Taylor series expansion
- Exact solution (in exams)

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Explicit multi-step |
| **AB4 formula** | $y_n + \frac{h}{24}(55f_n - 59f_{n-1} + 37f_{n-2} - 9f_{n-3})$ |
| **AB4 order** | 4 |
| **$f$-evals per step** | 1 (explicit AB), 2 (predictor-corrector) |
| **Starting values** | Needed (use RK4) |
| **Implicit version** | Adams-Moulton (more accurate, needs iteration) |

---

## Quick Revision Questions

1. **Write the Adams-Bashforth 2-step and 4-step formulas.**

2. **Why are multi-step methods more efficient than Runge-Kutta per step?**

3. **How are starting values obtained for the Adams-Bashforth method?**

4. **Explain the Adams-Bashforth-Moulton predictor-corrector scheme.**

5. **What is the difference between Adams-Bashforth (explicit) and Adams-Moulton (implicit)?**

6. **Apply AB2 to $y' = -y$, $y(0)=1$ for two steps with $h=0.1$ (use RK for starting value).**

---

[← Previous: Runge-Kutta](03-runge-kutta.md) | [Next: Milne's Method →](05-milnes-method.md)
