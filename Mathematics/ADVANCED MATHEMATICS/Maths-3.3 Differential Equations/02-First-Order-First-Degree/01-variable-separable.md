# Chapter 2.1: Variable Separable Method

[← Previous: Initial Value Problems](../01-Introduction/05-initial-value-problems.md) | [Back to README](../README.md) | [Next: Homogeneous Equations →](02-homogeneous-equations.md)

---

## Overview

The **variable separable** method is the simplest and most intuitive technique for solving first-order ODEs. If we can rearrange the equation so that all $y$-terms are on one side and all $x$-terms on the other, we can integrate both sides independently.

---

## 1. Standard Form

### Definition

> A first-order ODE is **separable** if it can be written as:
> $$\frac{dy}{dx} = g(x) \cdot h(y)$$
> or equivalently:
> $$M(x) \, dx + N(y) \, dy = 0$$

### Solution Method

$$\frac{dy}{h(y)} = g(x) \, dx$$

$$\int \frac{dy}{h(y)} = \int g(x) \, dx + C$$

### Decision Flowchart

```
Given: dy/dx = f(x, y)
         │
         ▼
Can f(x,y) be written as g(x)·h(y)?
         │
    ┌────┴────┐
   YES       NO
    │         │
    ▼         ▼
Separate   Try another
variables   method
    │       (homogeneous,
    ▼        linear, etc.)
Integrate
both sides
    │
    ▼
Solution!
```

---

## 2. Identifying Separable Equations

| Equation | Separable? | Reason |
|----------|-----------|--------|
| $y' = x^2 y$ | ✓ | $g(x) = x^2$, $h(y) = y$ |
| $y' = x + y$ | ✗ | Cannot factor as $g(x) \cdot h(y)$ |
| $y' = e^{x+y} = e^x \cdot e^y$ | ✓ | Product of functions of single variables |
| $y' = \dfrac{x^2}{1+y^2}$ | ✓ | $g(x) = x^2$, $h(y) = \dfrac{1}{1+y^2}$ |
| $y' = xy + x = x(y+1)$ | ✓ | Factor out $x$ |
| $y' = x^2 + y^2$ | ✗ | Sum, not product |

---

## 3. Worked Examples

### Example 1: Basic Separation

**Solve**: $\dfrac{dy}{dx} = \dfrac{x}{y}$

**Step 1** — Separate: $y \, dy = x \, dx$

**Step 2** — Integrate: $\displaystyle\int y \, dy = \int x \, dx$

$$\frac{y^2}{2} = \frac{x^2}{2} + C_1$$

**Step 3** — Simplify: $y^2 - x^2 = C$ (where $C = 2C_1$)

**Solution**: $y^2 = x^2 + C$ (family of hyperbolas)

```
   y
   │  ╲     ╱
   │   ╲   ╱     Family of hyperbolas
   │    ╲ ╱      y² - x² = C
   │     ╳
   │    ╱ ╲      C > 0: opens up/down
   │   ╱   ╲     C < 0: opens left/right
   │  ╱     ╲    C = 0: y = ±x (asymptotes)
   ├──────────── x
```

### Example 2: Exponential Separation

**Solve**: $\dfrac{dy}{dx} = e^{x-y} = e^x \cdot e^{-y}$

**Step 1** — Separate: $e^y \, dy = e^x \, dx$

**Step 2** — Integrate: $e^y = e^x + C$

**Solution**: $y = \ln(e^x + C)$

### Example 3: Trigonometric

**Solve**: $\cos y \, \dfrac{dy}{dx} = \sin x$

**Step 1** — Separate: $\cos y \, dy = \sin x \, dx$

**Step 2** — Integrate: $\sin y = -\cos x + C$

**Solution**: $\sin y + \cos x = C$

### Example 4: With Initial Condition

**IVP**: $y' = 2xy$, $y(0) = 3$

**Step 1** — Separate: $\dfrac{dy}{y} = 2x \, dx$

**Step 2** — Integrate: $\ln|y| = x^2 + C_0$

**Step 3** — General solution: $y = Ce^{x^2}$ (where $C = \pm e^{C_0}$)

**Step 4** — Apply IC: $3 = Ce^0 = C$

**Particular solution**: $y = 3e^{x^2}$

```
   y
   │         /
   │        /   y = 3e^(x²)
   │       /    Grows rapidly
   │      /
   │    _/
   3 ──•        (0, 3)
   │  /
   │ /
   ├──────────── x
  -2  -1  0  1  2
```

### Example 5: Requires Factoring

**Solve**: $\dfrac{dy}{dx} = 1 + x + y + xy$

**Key step** — Factor: $1 + x + y + xy = (1+x)(1+y)$

**Separate**: $\dfrac{dy}{1+y} = (1+x) \, dx$

**Integrate**: $\ln|1+y| = x + \dfrac{x^2}{2} + C$

**Solution**: $1 + y = Ae^{x + x^2/2}$

---

## 4. Common Integrals Reference

| Integral | Result |
|----------|--------|
| $\int \dfrac{dy}{y} $ | $\ln\|y\| + C$ |
| $\int \dfrac{dy}{y^2}$ | $-\dfrac{1}{y} + C$ |
| $\int \dfrac{dy}{1+y^2}$ | $\arctan y + C$ |
| $\int \dfrac{dy}{\sqrt{1-y^2}}$ | $\arcsin y + C$ |
| $\int e^y \, dy$ | $e^y + C$ |
| $\int \dfrac{dy}{ay+b}$ | $\dfrac{1}{a}\ln\|ay+b\| + C$ |

---

## 5. Special Cases

### Case 1: Lost Solutions

When separating $\dfrac{dy}{dx} = y \cdot g(x)$, we divide by $y$, which is valid only if $y \neq 0$.

**Always check**: Is $y = 0$ a solution?

**Example**: $y' = y \sin x$

- Separating: $\dfrac{dy}{y} = \sin x \, dx$ → $\ln|y| = -\cos x + C$ → $y = Ae^{-\cos x}$
- **But** $y = 0$ is also a solution (check: $0' = 0 \cdot \sin x = 0$ ✓)
- $y = 0$ IS included in general solution when $A = 0$, so no solution is lost here.

### Case 2: Implicit Solutions

Sometimes the integral cannot be evaluated in closed form. The implicit relation is the answer.

**Example**: $y' = \dfrac{x}{e^y + y}$

Separate: $(e^y + y) \, dy = x \, dx$

Integrate: $e^y + \dfrac{y^2}{2} = \dfrac{x^2}{2} + C$

This is the **implicit solution** — cannot solve for $y$ explicitly, and that's fine.

---

## 6. Applications

### Population Growth (Malthusian Model)

$$\frac{dP}{dt} = kP, \quad P(0) = P_0$$

Separate: $\dfrac{dP}{P} = k \, dt$

Integrate: $\ln P = kt + C_0$

Solution: $P = P_0 e^{kt}$

```
   P
   │            /
   │           / k > 0 (growth)
   │          /
   │        _/
   │      _/
P₀ │── ──•
   │      \_
   │        \_ k < 0 (decay)
   │          \
   ├───────────── t
```

### Radioactive Decay

$$\frac{dN}{dt} = -\lambda N$$

Solution: $N = N_0 e^{-\lambda t}$

**Half-life**: $t_{1/2} = \dfrac{\ln 2}{\lambda}$

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Separable form** | $\dfrac{dy}{dx} = g(x) \cdot h(y)$ |
| **Method** | Separate → Integrate both sides |
| **Key step** | Identify the factored form $g(x) \cdot h(y)$ |
| **Watch for** | Division by zero (lost solutions) |
| **Implicit solutions** | Acceptable when explicit form impossible |
| **Common trick** | Factor expressions: $1+x+y+xy = (1+x)(1+y)$ |
| **Applications** | Growth, decay, simple rate problems |

---

## Quick Revision Questions

1. Solve: $\dfrac{dy}{dx} = \dfrac{y^2}{x^2}$

2. Solve the IVP: $y' = y \tan x$, $y(0) = 1$

3. Is $y' = x^2 + y^2$ separable? Why or why not?

4. Solve: $\dfrac{dy}{dx} = e^{2x+3y}$ and write the solution explicitly.

5. A substance decays at a rate proportional to its mass. If 50g remain after 10 hours and 25g remain after 20 hours, what was the initial mass?

6. Solve: $(1 + e^x) y \, dy = e^x \, dx$

---

[← Previous: Initial Value Problems](../01-Introduction/05-initial-value-problems.md) | [Back to README](../README.md) | [Next: Homogeneous Equations →](02-homogeneous-equations.md)
