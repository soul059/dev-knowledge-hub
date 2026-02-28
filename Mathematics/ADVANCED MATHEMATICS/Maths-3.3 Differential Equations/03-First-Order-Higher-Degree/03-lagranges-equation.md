# Chapter 3.3: Lagrange's Equation

[← Previous: Clairaut's Equation](02-clairauts-equation.md) | [Back to README](../README.md) | [Next: Singular Solutions →](04-singular-solutions.md)

---

## Overview

**Lagrange's equation** (also called d'Alembert's equation) generalizes Clairaut's equation. While Clairaut has $y = xp + f(p)$, Lagrange allows the coefficient of $x$ to be any function of $p$.

---

## 1. Standard Form

$$\boxed{y = x \, g(p) + f(p)} \quad \text{where } p = \frac{dy}{dx}$$

- **Clairaut**: $g(p) = p$ (special case)
- **Lagrange**: $g(p)$ is any function of $p$, with $g(p) \neq p$

---

## 2. Solution Method

### Step 1: Differentiate w.r.t. $x$

$$p = g(p) + x \, g'(p) \frac{dp}{dx} + f'(p) \frac{dp}{dx}$$

$$p - g(p) = [x \, g'(p) + f'(p)] \frac{dp}{dx}$$

### Step 2: Rearrange (treat $p$ as independent variable)

$$\frac{dx}{dp} = \frac{x \, g'(p) + f'(p)}{p - g(p)}$$

$$\frac{dx}{dp} - \frac{g'(p)}{p - g(p)} \cdot x = \frac{f'(p)}{p - g(p)}$$

This is a **linear first-order ODE** in $x$ as a function of $p$!

### Step 3: Solve Using Integrating Factor

$$\mu(p) = e^{-\int \frac{g'(p)}{p - g(p)} \, dp}$$

### Step 4: Solution in Parametric Form

$$x = x(p, C) \quad \text{and} \quad y = x \, g(p) + f(p)$$

with $p$ as the parameter.

### Process

```
y = x·g(p) + f(p)
        │
   Differentiate w.r.t. x
        │
   dx/dp - [g'(p)/(p-g(p))]·x = f'(p)/(p-g(p))
        │
   LINEAR in x (as function of p)
        │
   Solve with I.F.  →  x = h(p, C)
        │
   Parametric solution:
   x = h(p, C),  y = x·g(p) + f(p)
```

---

## 3. Worked Examples

### Example 1: $y = 2xp + p^3$

Here $g(p) = 2p$, $f(p) = p^3$.

**Step 1**: Differentiate:
$$p = 2p + 2x\frac{dp}{dx} + 3p^2\frac{dp}{dx}$$

$$-p = (2x + 3p^2)\frac{dp}{dx}$$

**Step 2**: Invert:
$$\frac{dx}{dp} = \frac{-(2x + 3p^2)}{p} = \frac{-2x}{p} - 3p$$

$$\frac{dx}{dp} + \frac{2}{p} x = -3p$$

**Step 3**: I.F. $= e^{\int 2/p \, dp} = p^2$

$$\frac{d}{dp}(xp^2) = -3p^3$$

$$xp^2 = -\frac{3p^4}{4} + C$$

$$x = -\frac{3p^2}{4} + \frac{C}{p^2}$$

**Step 4**: Parametric solution:
$$x = -\frac{3p^2}{4} + \frac{C}{p^2}$$
$$y = 2xp + p^3 = 2p\left(-\frac{3p^2}{4} + \frac{C}{p^2}\right) + p^3 = -\frac{3p^3}{2} + \frac{2C}{p} + p^3 = -\frac{p^3}{2} + \frac{2C}{p}$$

### Example 2: $y = xp^2 + p$

Here $g(p) = p^2$, $f(p) = p$.

$$\frac{dx}{dp} - \frac{2p}{p - p^2} x = \frac{1}{p - p^2}$$

$$\frac{dx}{dp} - \frac{2p}{p(1-p)} x = \frac{1}{p(1-p)}$$

$$\frac{dx}{dp} - \frac{2}{1-p} x = \frac{1}{p(1-p)}$$

I.F. $= e^{\int \frac{-2}{1-p} dp} = e^{2\ln|1-p|} = (1-p)^2$

$$\frac{d}{dp}[x(1-p)^2] = \frac{(1-p)^2}{p(1-p)} = \frac{1-p}{p} = \frac{1}{p} - 1$$

$$x(1-p)^2 = \ln|p| - p + C$$

$$x = \frac{\ln|p| - p + C}{(1-p)^2}$$

**Parametric solution**: $x$ as above, $y = xp^2 + p$.

---

## 4. Special Case: Reduces to Clairaut

When $g(p) = p$, the equation $y = xp + f(p)$ is Clairaut's equation.

In this case, $p - g(p) = p - p = 0$, and the linear equation method has a singularity. The solution method simplifies to the direct Clairaut approach:
- General solution: $y = Cx + f(C)$
- Singular solution: from the envelope

---

## 5. Comparison: Clairaut vs Lagrange

| Feature | Clairaut | Lagrange |
|---------|----------|----------|
| Form | $y = xp + f(p)$ | $y = xg(p) + f(p)$ |
| $g(p)$ | $= p$ | $ \neq p$ (general) |
| General solution | Explicit: $y = Cx + f(C)$ | Parametric in $p$ |
| Solution type | Family of lines | Parametric curves |
| Singular solution | Envelope (always exists) | May or may not exist |
| Method | Direct substitution $p = C$ | Linear ODE for $x(p)$ |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Standard form** | $y = xg(p) + f(p)$, $g(p) \neq p$ |
| **Method** | Differentiate → get linear ODE in $x$ as function of $p$ |
| **Resulting ODE** | $\dfrac{dx}{dp} - \dfrac{g'(p)}{p - g(p)} x = \dfrac{f'(p)}{p - g(p)}$ |
| **Solution form** | Parametric: $x = x(p, C)$, $y = xg(p) + f(p)$ |
| **Special case** | $g(p) = p$ → Clairaut's equation |
| **Key insight** | Swapping independent variable from $x$ to $p$ |

---

## Quick Revision Questions

1. Is $y = 3xp + p^2$ a Clairaut or Lagrange equation? Solve it.

2. Set up the linear ODE in $x(p)$ for $y = xp^3 + p^2$.

3. What happens in the Lagrange method when $g(p) = p$?

4. Solve $y = 2xp - p^2$ in parametric form.

5. Why are Lagrange equation solutions typically given in parametric form rather than explicit form?

6. For $y = x(1+p) + p^2$, identify $g(p)$ and $f(p)$, and set up the linear ODE.

---

[← Previous: Clairaut's Equation](02-clairauts-equation.md) | [Back to README](../README.md) | [Next: Singular Solutions →](04-singular-solutions.md)
