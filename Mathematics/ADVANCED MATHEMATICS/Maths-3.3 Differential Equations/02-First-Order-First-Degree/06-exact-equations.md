# Chapter 2.6: Exact Equations

[← Previous: Bernoulli Equations](05-bernoulli-equations.md) | [Back to README](../README.md) | [Next: Integrating Factors →](07-integrating-factors.md)

---

## Overview

An **exact differential equation** is one where the left-hand side is already the **total differential** of some function $F(x, y)$. Recognizing and solving exact equations is both elegant and powerful.

---

## 1. Total Differential Review

For a function $F(x, y)$, the total differential is:

$$dF = \frac{\partial F}{\partial x} dx + \frac{\partial F}{\partial y} dy$$

If an ODE can be written as $dF = 0$, then the solution is simply $F(x, y) = C$.

---

## 2. Definition of Exactness

### Standard Form

$$M(x, y) \, dx + N(x, y) \, dy = 0$$

### Exactness Condition

> This equation is **exact** if there exists a function $F(x, y)$ such that:
> $$\frac{\partial F}{\partial x} = M \quad \text{and} \quad \frac{\partial F}{\partial y} = N$$

### Test for Exactness

$$\boxed{\frac{\partial M}{\partial y} = \frac{\partial N}{\partial x}}$$

This follows from equality of mixed partials: $\dfrac{\partial^2 F}{\partial y \partial x} = \dfrac{\partial^2 F}{\partial x \partial y}$.

### Decision Diagram

```
Given: M dx + N dy = 0
            │
            ▼
    Compute ∂M/∂y and ∂N/∂x
            │
     ┌──────┴──────┐
     │  Equal?     │
  ┌──┴──┐       ┌──┴──┐
  │ YES │       │ NO  │
  └──┬──┘       └──┬──┘
     │              │
     ▼              ▼
  EXACT!        Not exact.
  Find F(x,y)  Try integrating
  s.t. F=C     factor (Ch 2.7)
```

---

## 3. Solution Method

### Algorithm

```
Step 1: Verify ∂M/∂y = ∂N/∂x (exact)
Step 2: Integrate M w.r.t. x:  F = ∫M dx + g(y)
        (g(y) is the "constant" of integration — a function of y)
Step 3: Differentiate F w.r.t. y and set equal to N:
        ∂F/∂y = ∂/∂y[∫M dx] + g'(y) = N
Step 4: Solve for g'(y) and integrate to find g(y)
Step 5: Solution: F(x, y) = C
```

### Why $g(y)$?

When integrating $M$ with respect to $x$, any function of $y$ alone acts as a "constant." We determine $g(y)$ by matching the $N$ condition.

---

## 4. Worked Examples

### Example 1: $(2xy + 3) \, dx + (x^2 + 4y) \, dy = 0$

**Step 1** — Test:
$$M = 2xy + 3, \quad N = x^2 + 4y$$
$$\frac{\partial M}{\partial y} = 2x, \quad \frac{\partial N}{\partial x} = 2x \quad \checkmark \text{ Exact!}$$

**Step 2** — Integrate $M$ w.r.t. $x$:
$$F = \int (2xy + 3) \, dx = x^2 y + 3x + g(y)$$

**Step 3** — Differentiate w.r.t. $y$:
$$\frac{\partial F}{\partial y} = x^2 + g'(y)$$

Set equal to $N$: $x^2 + g'(y) = x^2 + 4y$

$$g'(y) = 4y \implies g(y) = 2y^2$$

**Step 4** — Solution:
$$\boxed{x^2 y + 3x + 2y^2 = C}$$

**Verification**: $dF = (2xy + 3)dx + (x^2 + 4y)dy = Mdx + Ndy$ ✓

### Example 2: $(ye^{xy} + 2x) \, dx + (xe^{xy} + 2y) \, dy = 0$

**Step 1** — Test:
$$\frac{\partial M}{\partial y} = e^{xy} + xye^{xy}, \quad \frac{\partial N}{\partial x} = e^{xy} + xye^{xy} \quad \checkmark$$

**Step 2** — Integrate $M$ w.r.t. $x$:
$$F = \int (ye^{xy} + 2x) \, dx = e^{xy} + x^2 + g(y)$$

**Step 3**:
$$\frac{\partial F}{\partial y} = xe^{xy} + g'(y) = xe^{xy} + 2y$$

$$g'(y) = 2y \implies g(y) = y^2$$

**Solution**: $\boxed{e^{xy} + x^2 + y^2 = C}$

### Example 3: Starting from $N$ instead

**Solve**: $(3x^2 + y\cos x) \, dx + (\sin x + 4y^3) \, dy = 0$

**Verify**: $\partial M/\partial y = \cos x$, $\partial N/\partial x = \cos x$ ✓

**Alternative**: Integrate $N$ w.r.t. $y$ first:
$$F = \int (\sin x + 4y^3) \, dy = y\sin x + y^4 + h(x)$$

Differentiate w.r.t. $x$:
$$\frac{\partial F}{\partial x} = y\cos x + h'(x) = 3x^2 + y\cos x$$

$$h'(x) = 3x^2 \implies h(x) = x^3$$

**Solution**: $\boxed{y\sin x + y^4 + x^3 = C}$

### Example 4: IVP

**Solve**: $(2x + y) \, dx + (x + 6y) \, dy = 0$, $y(1) = 2$

**Verify**: $\partial M/\partial y = 1 = \partial N/\partial x$ ✓

$F = \int (2x + y) \, dx = x^2 + xy + g(y)$

$\partial F/\partial y = x + g'(y) = x + 6y$

$g'(y) = 6y \implies g(y) = 3y^2$

General: $x^2 + xy + 3y^2 = C$

Apply $y(1) = 2$: $1 + 2 + 12 = 15$

**Solution**: $\boxed{x^2 + xy + 3y^2 = 15}$

---

## 5. Common Exact Differential Forms

Recognizing these patterns can save significant time:

| Expression | Is the differential of |
|------------|----------------------|
| $x \, dy + y \, dx$ | $d(xy)$ |
| $\dfrac{x \, dy - y \, dx}{x^2}$ | $d\left(\dfrac{y}{x}\right)$ |
| $\dfrac{y \, dx - x \, dy}{y^2}$ | $d\left(\dfrac{x}{y}\right)$ |
| $\dfrac{x \, dy - y \, dx}{x^2 + y^2}$ | $d\left(\arctan\dfrac{y}{x}\right)$ |
| $\dfrac{x \, dx + y \, dy}{x^2 + y^2}$ | $\dfrac{1}{2}d\left(\ln(x^2 + y^2)\right)$ |
| $2(x \, dx + y \, dy)$ | $d(x^2 + y^2)$ |
| $e^x(f'(x) + f(x)) \, dx$ | $d(e^x f(x))$ |

### Recognition Example

**Solve**: $y \, dx + x \, dy + 2x \, dx = 0$

Recognize: $y \, dx + x \, dy = d(xy)$ and $2x \, dx = d(x^2)$

$$d(xy) + d(x^2) = 0 \implies d(xy + x^2) = 0$$

**Solution**: $xy + x^2 = C$

---

## 6. Geometric Interpretation

The solution $F(x, y) = C$ represents **level curves** of the surface $z = F(x, y)$.

```
   y
   │
   │    ╲  ╭──╮  ╱
   │     ╲ │  │ ╱      Level curves F(x,y) = C
   │      ╲│  │╱       for different values of C
   │       │  │
   │      ╱│  │╲       Each curve is a solution
   │     ╱ │  │ ╲      of the exact DE
   │    ╱  ╰──╯  ╲
   ├──────────────── x
   │
   │  At any point, the slope dy/dx = -M/N
   │  is perpendicular to ∇F = (M, N)
```

The direction field of $dy/dx = -M/N$ is always **tangent** to the level curves.

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Standard form** | $M \, dx + N \, dy = 0$ |
| **Exactness test** | $\partial M/\partial y = \partial N/\partial x$ |
| **Solution method** | Find $F$ such that $F_x = M$, $F_y = N$ |
| **Step 1** | Integrate $M$ w.r.t. $x$ (add $g(y)$) |
| **Step 2** | Differentiate w.r.t. $y$, match with $N$ to find $g(y)$ |
| **Alternative** | Start by integrating $N$ w.r.t. $y$ (add $h(x)$) |
| **Solution** | $F(x, y) = C$ (implicit) |
| **Key patterns** | $d(xy) = x\,dy + y\,dx$, $d(y/x) = (x\,dy - y\,dx)/x^2$ |

---

## Quick Revision Questions

1. Test for exactness: $(3x^2 y + y^2) \, dx + (x^3 + 2xy + 1) \, dy = 0$

2. Solve: $(2xy - 3) \, dx + (x^2 + 4y) \, dy = 0$

3. Which of the following is the total differential of $x^2y^3$?
   - (a) $2xy^3 \, dx + 3x^2y^2 \, dy$
   - (b) $x^2 \, dx + y^3 \, dy$
   - (c) $3x^2y^2 \, dx + 2xy^3 \, dy$

4. Verify that $(y + e^x) \, dx + (x + \cos y) \, dy = 0$ is exact and solve it.

5. Rewrite $\dfrac{x \, dy - y \, dx}{x^2}$ as an exact differential and identify the function.

6. Solve the IVP: $(y \cos x + \sin y) \, dx + (\sin x + x \cos y) \, dy = 0$, $y(0) = \pi$

---

[← Previous: Bernoulli Equations](05-bernoulli-equations.md) | [Back to README](../README.md) | [Next: Integrating Factors →](07-integrating-factors.md)
