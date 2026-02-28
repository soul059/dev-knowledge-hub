# Chapter 3.2: Clairaut's Equation

[← Previous: Solvable for p, y, x](01-solvable-for-p-y-x.md) | [Back to README](../README.md) | [Next: Lagrange's Equation →](03-lagranges-equation.md)

---

## Overview

**Clairaut's equation** is a beautiful special case where the general solution is a family of straight lines, and a **singular solution** (envelope) also exists. It's one of the most elegant results in ODE theory.

---

## 1. Standard Form

$$\boxed{y = xp + f(p)} \quad \text{where } p = \frac{dy}{dx}$$

### Key Features
- $y$ appears linearly
- $x$ multiplies $p$
- $f(p)$ is any function of $p$ alone
- The equation is solvable for $y$

---

## 2. Solution Method

### Step 1: Differentiate w.r.t. $x$

$$p = p + x\frac{dp}{dx} + f'(p)\frac{dp}{dx}$$

$$0 = [x + f'(p)]\frac{dp}{dx}$$

### Step 2: Two cases arise

**Case A**: $\dfrac{dp}{dx} = 0 \implies p = C$ (constant)

Substitute back: $y = Cx + f(C)$

This is the **general solution** — a family of straight lines!

**Case B**: $x + f'(p) = 0 \implies x = -f'(p)$

Substitute into original: $y = -f'(p) \cdot p + f(p)$

This gives the **singular solution** (envelope) in parametric form: $x = -f'(p)$, $y = f(p) - pf'(p)$.

```
        ┌──────────────────────────────┐
        │  y = xp + f(p)               │
        │         │                    │
        │    Differentiate             │
        │         │                    │
        │  [x + f'(p)] dp/dx = 0      │
        │         │                    │
        │    ┌────┴────┐               │
        │    │         │               │
        │  dp/dx=0   x=-f'(p)          │
        │    │         │               │
        │  p=C       Singular          │
        │    │       Solution          │
        │  y=Cx+f(C)  (envelope)       │
        │  (lines)                     │
        └──────────────────────────────┘
```

---

## 3. Worked Examples

### Example 1: $y = xp + p^2$

Here $f(p) = p^2$.

**General solution**: Replace $p$ by $C$:
$$\boxed{y = Cx + C^2}$$

**Singular solution**: $f'(p) = 2p$, so $x = -2p \implies p = -x/2$

$$y = x(-x/2) + (-x/2)^2 = -x^2/2 + x^2/4 = -x^2/4$$

$$\boxed{y = -\frac{x^2}{4}} \quad \text{(singular solution — parabola)}$$

**Geometric Picture**:
```
   y
   │
   2 │╲    ╱
     │ ╲  ╱  Family of lines y = Cx + C²
   1 │  ╲╱
     │  ╱╲
   0 ├─╱──╲───── x
     │╱    ╲
  -1 │      ╲___   ← Envelope: y = -x²/4
     │          ╲    (parabola touching every line)
  -2 │
    -4 -2  0  2  4
```

**Verification of singular solution**: $y = -x^2/4$, $p = y' = -x/2$

$$xp + p^2 = x(-x/2) + x^2/4 = -x^2/2 + x^2/4 = -x^2/4 = y \quad \checkmark$$

### Example 2: $y = xp + \sqrt{1 + p^2}$

**General solution**: $y = Cx + \sqrt{1 + C^2}$

**Singular solution**: $f'(p) = \dfrac{p}{\sqrt{1+p^2}}$

$x = -\dfrac{p}{\sqrt{1+p^2}}$, so $x^2 = \dfrac{p^2}{1+p^2}$

$1 - x^2 = \dfrac{1}{1+p^2}$, so $\sqrt{1+p^2} = \dfrac{1}{\sqrt{1-x^2}}$

Also $p = -\dfrac{x}{\sqrt{1-x^2}}$

$$y = x \cdot \left(-\frac{x}{\sqrt{1-x^2}}\right) + \frac{1}{\sqrt{1-x^2}} = \frac{-x^2 + 1}{\sqrt{1-x^2}} = \sqrt{1-x^2}$$

$$\boxed{x^2 + y^2 = 1} \quad \text{(circle!)}$$

> The family of tangent lines to a circle has the circle itself as the envelope.

### Example 3: $y = xp + a/p$

**General solution**: $y = Cx + a/C$

**Singular solution**: $f'(p) = -a/p^2$, so $x = a/p^2 \implies p^2 = a/x$

$$y = x \cdot p + \frac{a}{p} = x\sqrt{a/x} + \frac{a}{\sqrt{a/x}} = \sqrt{ax} + \sqrt{ax} = 2\sqrt{ax}$$

$$\boxed{y^2 = 4ax} \quad \text{(parabola)}$$

---

## 4. The Envelope Concept

### What is an Envelope?

> The **envelope** of a one-parameter family of curves is a curve that is **tangent** to every member of the family.

### Finding the Envelope Analytically

For a family $\Phi(x, y, C) = 0$:

1. Write $\Phi(x, y, C) = 0$
2. Differentiate w.r.t. $C$: $\dfrac{\partial \Phi}{\partial C} = 0$
3. Eliminate $C$ between these two equations

### Example: Family $y = Cx + C^2$

$\Phi = y - Cx - C^2 = 0$

$\partial\Phi/\partial C = -x - 2C = 0 \implies C = -x/2$

$y = (-x/2)x + (-x/2)^2 = -x^2/2 + x^2/4 = -x^2/4$ ✓

---

## 5. Why the Singular Solution Matters

| Property | General Solution | Singular Solution |
|----------|-----------------|-------------------|
| Contains $C$? | Yes | No |
| Shape | Family of straight lines | Curve (envelope) |
| Obtained from general? | — | NO (that's why it's "singular") |
| Satisfies the DE? | Yes | Yes |
| Geometric role | Tangent lines | The curve they're tangent to |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Clairaut's form** | $y = xp + f(p)$ |
| **General solution** | $y = Cx + f(C)$ (replace $p$ by $C$) |
| **Singular solution** | Eliminate $p$ from $x = -f'(p)$ and $y = xp + f(p)$ |
| **Geometric meaning** | General: tangent lines; Singular: envelope |
| **Envelope method** | $\Phi = 0$ and $\partial\Phi/\partial C = 0$, eliminate $C$ |
| **Quick test** | If equation has form $y = xp + f(p)$ → Clairaut |

---

## Quick Revision Questions

1. Solve: $y = xp + p^3$ (find both general and singular solutions).

2. What is the geometric relationship between the general and singular solutions of Clairaut's equation?

3. Find the singular solution of $y = xp - p^2$ and identify the curve.

4. Is $y = xp + \ln p$ a Clairaut equation? If so, find the general solution.

5. Show that the singular solution of $y = xp + 1/p$ is $y^2 = 4x$.

6. Why does the singular solution contain **no arbitrary constant**?

---

[← Previous: Solvable for p, y, x](01-solvable-for-p-y-x.md) | [Back to README](../README.md) | [Next: Lagrange's Equation →](03-lagranges-equation.md)
