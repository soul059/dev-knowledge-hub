# Chapter 2.4: Linear First-Order Equations

[← Previous: Reducible to Homogeneous](03-reducible-to-homogeneous.md) | [Back to README](../README.md) | [Next: Bernoulli Equations →](05-bernoulli-equations.md)

---

## Overview

The **linear first-order ODE** is one of the most important types, with a guaranteed solution formula using an **integrating factor**. This method always works — no guesswork needed.

---

## 1. Standard Form

$$\frac{dy}{dx} + P(x) \, y = Q(x)$$

where $P(x)$ and $Q(x)$ are functions of $x$ only (or constants).

### Key Properties

- The equation is **linear in $y$** (and linear in $y'$)
- Degree is always 1
- $Q(x) = 0$: homogeneous; $Q(x) \neq 0$: non-homogeneous

### Recognition

| Equation | Standard Form | $P(x)$ | $Q(x)$ |
|----------|--------------|---------|---------|
| $y' + 2y = e^x$ | Already standard | $2$ | $e^x$ |
| $xy' + y = x^2$ | $y' + \dfrac{y}{x} = x$ | $\dfrac{1}{x}$ | $x$ |
| $y' = y + \sin x$ | $y' - y = \sin x$ | $-1$ | $\sin x$ |
| $\cos x \cdot y' + y \sin x = 1$ | $y' + y\tan x = \sec x$ | $\tan x$ | $\sec x$ |

---

## 2. The Integrating Factor Method

### Core Idea

Multiply the entire equation by a function $\mu(x)$ that makes the left side a **perfect derivative**:

$$\mu(x)[y' + P(x)y] = \frac{d}{dx}[\mu(x) \cdot y]$$

### Derivation

For $\mu(y' + Py) = \mu y' + \mu P y$ to equal $(\mu y)' = \mu y' + \mu' y$:

$$\mu P y = \mu' y \implies \mu' = \mu P \implies \frac{d\mu}{\mu} = P \, dx$$

$$\ln \mu = \int P(x) \, dx \implies \boxed{\mu(x) = e^{\int P(x) \, dx}}$$

### Complete Solution Formula

$$\boxed{y \cdot e^{\int P \, dx} = \int Q(x) \cdot e^{\int P \, dx} \, dx + C}$$

or equivalently:

$$y = e^{-\int P \, dx} \left[\int Q \cdot e^{\int P \, dx} \, dx + C\right]$$

### Algorithm

```
Step 1: Write in standard form: y' + P(x)y = Q(x)
Step 2: Identify P(x) and Q(x)
Step 3: Compute I.F. = μ = e^{∫P(x)dx}
Step 4: Multiply both sides by μ
Step 5: Left side becomes d/dx[μy]
Step 6: Integrate both sides: μy = ∫μQ dx + C
Step 7: Solve for y
```

---

## 3. Worked Examples

### Example 1: Constant Coefficients

**Solve**: $y' + 3y = 6$

**Step 1**: $P = 3$, $Q = 6$

**Step 2**: $\mu = e^{\int 3 \, dx} = e^{3x}$

**Step 3**: Multiply: $e^{3x}y' + 3e^{3x}y = 6e^{3x}$

**Step 4**: $\dfrac{d}{dx}(ye^{3x}) = 6e^{3x}$

**Step 5**: Integrate: $ye^{3x} = 2e^{3x} + C$

**Solution**: $y = 2 + Ce^{-3x}$

```
   y
   │
   │  __________ y = 2 (steady state)
   2 ─•─────────────────
   │ /              
   │/  C > 0: approaches from below
   ├──────────────────── x
   │\  C < 0: approaches from above
   │ \___________
```

### Example 2: Variable Coefficient

**Solve**: $y' + \dfrac{y}{x} = x^2$, $x > 0$

**Step 1**: $P = 1/x$, $Q = x^2$

**Step 2**: $\mu = e^{\int dx/x} = e^{\ln x} = x$

**Step 3**: Multiply by $x$: $xy' + y = x^3$

**Step 4**: $\dfrac{d}{dx}(xy) = x^3$

**Step 5**: Integrate: $xy = \dfrac{x^4}{4} + C$

**Solution**: $y = \dfrac{x^3}{4} + \dfrac{C}{x}$

### Example 3: With Trigonometry

**Solve**: $y' + y \tan x = \sec x$

**Step 1**: $P = \tan x$, $Q = \sec x$

**Step 2**: $\mu = e^{\int \tan x \, dx} = e^{-\ln|\cos x|} = \dfrac{1}{\cos x} = \sec x$

**Step 3**: Multiply: $\sec x \cdot y' + y \sec x \tan x = \sec^2 x$

**Step 4**: $\dfrac{d}{dx}(y \sec x) = \sec^2 x$

**Step 5**: Integrate: $y \sec x = \tan x + C$

**Solution**: $y = \sin x + C \cos x$

### Example 4: IVP

**Solve**: $y' - 2y = 4$, $y(0) = 1$

**Step 1**: $P = -2$, $Q = 4$

**Step 2**: $\mu = e^{-2x}$

**Step 3**: $\dfrac{d}{dx}(ye^{-2x}) = 4e^{-2x}$

**Step 4**: $ye^{-2x} = -2e^{-2x} + C$

**General**: $y = -2 + Ce^{2x}$

**Apply IC**: $1 = -2 + C \implies C = 3$

**Particular solution**: $y = -2 + 3e^{2x}$

---

## 4. Linear Equation in $x$ (Reversed Roles)

Sometimes it's easier to consider $x$ as a function of $y$:

$$\frac{dx}{dy} + P(y) \, x = Q(y)$$

The I.F. is then $\mu(y) = e^{\int P(y) \, dy}$.

### Example: $(y^2 + 1) \, dx = (xy + 2y) \, dy$

Rewrite: $\dfrac{dx}{dy} = \dfrac{xy + 2y}{y^2 + 1} = \dfrac{y}{y^2+1} x + \dfrac{2y}{y^2+1}$

Standard form: $\dfrac{dx}{dy} - \dfrac{y}{y^2+1} x = \dfrac{2y}{y^2+1}$

$P(y) = -\dfrac{y}{y^2+1}$, $Q(y) = \dfrac{2y}{y^2+1}$

$\mu = e^{\int -y/(y^2+1) \, dy} = e^{-\frac{1}{2}\ln(y^2+1)} = \dfrac{1}{\sqrt{y^2+1}}$

Continue with formula...

---

## 5. Structure of the Solution

The general solution $y = y_h + y_p$ has two parts:

$$y = \underbrace{Ce^{-\int P \, dx}}_{\text{homogeneous solution}} + \underbrace{e^{-\int P \, dx} \int Q \cdot e^{\int P \, dx} \, dx}_{\text{particular solution}}$$

```
General Solution = Homogeneous + Particular
                        │              │
                   Ce^(-∫P dx)    Depends on Q(x)
                        │              │
                  Transient part   Steady-state
                  (decays if        (persists)
                   ∫P > 0)
```

This decomposition principle carries over to higher-order linear ODEs.

---

## 6. Common Integrating Factors

| $P(x)$ | $\int P \, dx$ | $\mu = e^{\int P \, dx}$ |
|---------|----------------|--------------------------|
| $a$ (constant) | $ax$ | $e^{ax}$ |
| $1/x$ | $\ln x$ | $x$ |
| $n/x$ | $n\ln x$ | $x^n$ |
| $-1/x$ | $-\ln x$ | $1/x$ |
| $\tan x$ | $-\ln|\cos x|$ | $\sec x$ |
| $-\tan x$ | $\ln|\cos x|$ | $\cos x$ |
| $\cot x$ | $\ln|\sin x|$ | $\sin x$ |
| $2x$ | $x^2$ | $e^{x^2}$ |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Standard form** | $y' + P(x)y = Q(x)$ |
| **Integrating factor** | $\mu = e^{\int P(x) \, dx}$ |
| **Solution formula** | $y \cdot \mu = \int Q \cdot \mu \, dx + C$ |
| **Always works** | Yes — no restrictions on $P$ or $Q$ |
| **Reversed form** | $dx/dy + P(y)x = Q(y)$ when $x = f(y)$ easier |
| **Solution structure** | $y = y_h + y_p$ (homogeneous + particular) |
| **Key skill** | Correctly identifying $P(x)$ from the standard form |

---

## Quick Revision Questions

1. Solve: $y' + y = e^{-x}$

2. Find the integrating factor for $xy' - 2y = x^3$ (first write in standard form).

3. Solve the IVP: $y' + y \cot x = 2\cos x$, $y(\pi/2) = 1$

4. Why is the equation $yy' + x = 0$ NOT a linear first-order ODE?

5. Solve: $\dfrac{dx}{dy} + x = e^y$ (linear in $x$)

6. For $y' + 3y = 12$, find the steady-state value (as $x \to \infty$) without solving completely.

---

[← Previous: Reducible to Homogeneous](03-reducible-to-homogeneous.md) | [Back to README](../README.md) | [Next: Bernoulli Equations →](05-bernoulli-equations.md)
