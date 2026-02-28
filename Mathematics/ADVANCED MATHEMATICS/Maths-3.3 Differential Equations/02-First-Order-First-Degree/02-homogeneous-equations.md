# Chapter 2.2: Homogeneous Equations

[← Previous: Variable Separable](01-variable-separable.md) | [Back to README](../README.md) | [Next: Reducible to Homogeneous →](03-reducible-to-homogeneous.md)

---

## Overview

A **homogeneous** first-order ODE is one where $f(x, y)$ can be expressed as a function of the ratio $y/x$ alone. The substitution $v = y/x$ converts it into a separable equation.

> **Note**: "Homogeneous" here refers to a function property, NOT to the linear ODE concept ($g(x) = 0$).

---

## 1. Homogeneous Functions

### Definition

> A function $f(x, y)$ is **homogeneous of degree $n$** if:
> $$f(tx, ty) = t^n f(x, y) \quad \text{for all } t > 0$$

### Examples

| Function | $f(tx, ty)$ | Degree |
|----------|------------|--------|
| $x^2 + y^2$ | $t^2(x^2 + y^2)$ | 2 |
| $xy$ | $t^2 xy$ | 2 |
| $\dfrac{x^2 + y^2}{xy}$ | $\dfrac{t^2(x^2+y^2)}{t^2 xy} = \dfrac{x^2+y^2}{xy}$ | 0 |
| $\dfrac{y}{x} + \sin\dfrac{y}{x}$ | Same (degree 0) | 0 |
| $x + e^{y/x}$ | $t(x + e^{y/x})$... NO | Not homogeneous |

### Key Property for DEs

> A function $f(x, y)$ that is homogeneous of **degree 0** can be written as a function of $v = y/x$ alone.

---

## 2. Homogeneous First-Order ODE

### Recognition

The ODE $\dfrac{dy}{dx} = f(x, y)$ is homogeneous if $f(x, y)$ is homogeneous of degree 0, meaning:

$$\frac{dy}{dx} = F\left(\frac{y}{x}\right)$$

**Equivalent test**: In $\dfrac{dy}{dx} = \dfrac{M(x,y)}{N(x,y)}$, both $M$ and $N$ are homogeneous of the **same degree**.

### Examples of Recognition

| Equation | $M$ degree | $N$ degree | Homogeneous? |
|----------|-----------|-----------|-------------|
| $y' = \dfrac{x^2 + y^2}{xy}$ | 2 | 2 | ✓ |
| $y' = \dfrac{x + y}{x - y}$ | 1 | 1 | ✓ |
| $y' = \dfrac{x^2 + y}{xy}$ | Mixed | Mixed | ✗ |
| $y' = \dfrac{y}{x} + \tan\dfrac{y}{x}$ | — | — | ✓ (degree 0) |

---

## 3. Solution Method

### Substitution

Let $y = vx$ where $v = y/x$. Then:

$$\frac{dy}{dx} = v + x\frac{dv}{dx}$$

### Algorithm

```
Step 1: Verify equation is homogeneous (M, N same degree)
Step 2: Substitute y = vx, dy/dx = v + x(dv/dx)
Step 3: Simplify — equation becomes separable in v and x
Step 4: Separate and integrate
Step 5: Back-substitute v = y/x
```

### Process Diagram

```
Original DE                          Separable DE
dy/dx = f(x,y)    ──y=vx──►    v + x(dv/dx) = F(v)
(homogeneous)                        │
                                     ▼
                               x(dv/dx) = F(v) - v
                                     │
                                     ▼
                              dv/(F(v)-v) = dx/x
                                     │
                                     ▼
                              Integrate both sides
                                     │
                                     ▼
                              Back-substitute v = y/x
```

---

## 4. Worked Examples

### Example 1: $\dfrac{dy}{dx} = \dfrac{x^2 + y^2}{2xy}$

**Step 1** — Check: numerator degree 2, denominator degree 2 ✓

**Step 2** — Let $y = vx$:
$$v + x\frac{dv}{dx} = \frac{x^2 + v^2 x^2}{2x \cdot vx} = \frac{1 + v^2}{2v}$$

**Step 3** — Simplify:
$$x\frac{dv}{dx} = \frac{1 + v^2}{2v} - v = \frac{1 + v^2 - 2v^2}{2v} = \frac{1 - v^2}{2v}$$

**Step 4** — Separate:
$$\frac{2v}{1 - v^2} \, dv = \frac{dx}{x}$$

**Step 5** — Integrate:
$$-\ln|1 - v^2| = \ln|x| + C_0$$

$$\ln\frac{1}{|1-v^2|} = \ln|x| + C_0$$

$$\frac{1}{1 - v^2} = Ax \quad \text{(where } A = \pm e^{C_0}\text{)}$$

**Step 6** — Back-substitute $v = y/x$:
$$\frac{1}{1 - y^2/x^2} = Ax \implies \frac{x^2}{x^2 - y^2} = Ax$$

**Solution**: $x^2 = Ax(x^2 - y^2)$ or $x = A(x^2 - y^2)$

### Example 2: $\dfrac{dy}{dx} = \dfrac{x + y}{x - y}$

**Step 1** — Degree 1 / degree 1 ✓

**Step 2** — Let $y = vx$:
$$v + x\frac{dv}{dx} = \frac{x + vx}{x - vx} = \frac{1 + v}{1 - v}$$

**Step 3**:
$$x\frac{dv}{dx} = \frac{1+v}{1-v} - v = \frac{1 + v - v + v^2}{1-v} = \frac{1 + v^2}{1-v}$$

**Step 4** — Separate:
$$\frac{1-v}{1+v^2} \, dv = \frac{dx}{x}$$

Split: $\dfrac{1}{1+v^2} - \dfrac{v}{1+v^2}$

**Step 5** — Integrate:
$$\arctan v - \frac{1}{2}\ln(1 + v^2) = \ln|x| + C$$

**Step 6** — Back-substitute $v = y/x$:

$$\arctan\frac{y}{x} - \frac{1}{2}\ln\left(1 + \frac{y^2}{x^2}\right) = \ln|x| + C$$

$$\arctan\frac{y}{x} = \frac{1}{2}\ln(x^2 + y^2) + C$$

### Example 3: With IVP

**Solve**: $(x^2 + y^2) \, dx - 2xy \, dy = 0$, $y(1) = 0$

Rewrite: $\dfrac{dy}{dx} = \dfrac{x^2 + y^2}{2xy}$

This is the same as Example 1. Solution: $x = A(x^2 - y^2)$

Apply $y(1) = 0$: $1 = A(1 - 0) \implies A = 1$

**Particular solution**: $x = x^2 - y^2$ → $y^2 = x^2 - x = x(x-1)$

---

## 5. Alternative Substitution: $x = vy$

When the equation is more naturally expressed in $x/y$, use $x = vy$ instead.

**When to use**: If after $y = vx$ the algebra is messy, try $x = vy$.

$$x = vy \implies dx = v \, dy + y \, dv \implies \frac{dx}{dy} = v + y\frac{dv}{dy}$$

### Example: $x \, dy - y \, dx = \sqrt{x^2 + y^2} \, dx$

Rewrite: $\dfrac{dy}{dx} = \dfrac{y + \sqrt{x^2+y^2}}{x}$

Using $y = vx$: $v + x\dfrac{dv}{dx} = v + \sqrt{1 + v^2}$

$$x\frac{dv}{dx} = \sqrt{1+v^2}$$

$$\frac{dv}{\sqrt{1+v^2}} = \frac{dx}{x}$$

$$\sinh^{-1}(v) = \ln|x| + C$$

$$\sinh^{-1}\left(\frac{y}{x}\right) = \ln|x| + C$$

---

## 6. Recognizing Patterns

### Quick Test

Replace every $y$ with $ty$ and every $x$ with $tx$ in $f(x,y)$. If $t$ cancels completely, the equation is homogeneous.

**Example**: $f(x,y) = \dfrac{x^3 + xy^2}{x^2 y}$

$$f(tx, ty) = \frac{t^3 x^3 + t \cdot x \cdot t^2 y^2}{t^2 x^2 \cdot ty} = \frac{t^3(x^3 + xy^2)}{t^3(x^2 y)} = f(x,y) \quad \checkmark$$

Degree 0 — homogeneous ✓

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Homogeneous function** | $f(tx, ty) = t^n f(x, y)$ |
| **Homogeneous ODE** | $dy/dx = F(y/x)$; $M, N$ same degree |
| **Substitution** | $y = vx$, $dy/dx = v + x \, dv/dx$ |
| **Result after substitution** | Separable in $v$ and $x$ |
| **Alternative** | $x = vy$ when more convenient |
| **Quick test** | Replace $(x,y) \to (tx, ty)$; if $t$ cancels → homogeneous |
| **Don't confuse** | Homogeneous function ≠ Homogeneous linear ODE ($g(x)=0$) |

---

## Quick Revision Questions

1. Is $\dfrac{dy}{dx} = \dfrac{x^2 - y^2}{x^2 + y^2}$ homogeneous? If yes, solve it.

2. Verify that $f(x,y) = \dfrac{x + 2y}{3x - y}$ is homogeneous of degree 0.

3. Solve: $x \, dy - y \, dx = x^2 + y^2 \, dy$ ... (hint: rewrite and check)

4. Solve the IVP: $\dfrac{dy}{dx} = \dfrac{y - x}{y + x}$, $y(1) = 1$

5. When would you prefer the substitution $x = vy$ over $y = vx$?

6. Show that $\dfrac{dy}{dx} = \dfrac{y}{x} + \sin\dfrac{y}{x}$ is homogeneous and set up the separated form (no need to integrate).

---

[← Previous: Variable Separable](01-variable-separable.md) | [Back to README](../README.md) | [Next: Reducible to Homogeneous →](03-reducible-to-homogeneous.md)
