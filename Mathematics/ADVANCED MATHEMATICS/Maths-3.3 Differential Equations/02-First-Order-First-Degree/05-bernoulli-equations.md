# Chapter 2.5: Bernoulli Equations

[← Previous: Linear Equations](04-linear-equations.md) | [Back to README](../README.md) | [Next: Exact Equations →](06-exact-equations.md)

---

## Overview

The **Bernoulli equation** is a non-linear first-order ODE that can be transformed into a linear equation by a clever substitution. It generalizes the linear equation by allowing a $y^n$ term on the right-hand side.

---

## 1. Standard Form

$$\frac{dy}{dx} + P(x) \, y = Q(x) \, y^n$$

where $n$ is any real number.

### Special Cases

| $n$ | Equation becomes | Type |
|-----|-----------------|------|
| $n = 0$ | $y' + Py = Q$ | Linear (already solved!) |
| $n = 1$ | $y' + Py = Qy \implies y' + (P-Q)y = 0$ | Separable |
| $n \neq 0, 1$ | Genuine Bernoulli | Use substitution below |

---

## 2. Solution Method

### The Key Substitution

Let $v = y^{1-n}$. Then:

$$\frac{dv}{dx} = (1-n) y^{-n} \frac{dy}{dx}$$

### Transformation Steps

```
Step 1: Write in standard Bernoulli form: y' + Py = Qy^n
Step 2: Divide by y^n:  y^(-n)y' + Py^(1-n) = Q
Step 3: Let v = y^(1-n), so dv/dx = (1-n)y^(-n)y'
Step 4: Substitute: (1/(1-n))dv/dx + Pv = Q
Step 5: Simplify: dv/dx + (1-n)Pv = (1-n)Q
Step 6: This is LINEAR in v — solve with I.F.!
Step 7: Back-substitute v = y^(1-n)
```

### Process Diagram

```
Bernoulli Equation          Linear Equation
y' + Py = Qyⁿ              v' + (1-n)Pv = (1-n)Q
(non-linear)                (linear in v)
      │                           │
      │    v = y^(1-n)            │
      ├──────────────────────►    ├── Solve with I.F.
      │                           │
      │    y = v^(1/(1-n))        │
      ◄──────────────────────     │
      │                           │
   Solution in y              Solution in v
```

---

## 3. Worked Examples

### Example 1: $y' + y = y^2$

**Step 1**: $P = 1$, $Q = 1$, $n = 2$

**Step 2**: Divide by $y^2$: $y^{-2}y' + y^{-1} = 1$

**Step 3**: Let $v = y^{1-2} = y^{-1} = 1/y$

$\dfrac{dv}{dx} = -y^{-2}\dfrac{dy}{dx}$, so $y^{-2}y' = -\dfrac{dv}{dx}$

**Step 4**: $-\dfrac{dv}{dx} + v = 1 \implies \dfrac{dv}{dx} - v = -1$

**Step 5**: This is linear! $P_v = -1$, $Q_v = -1$

$\mu = e^{\int -1 \, dx} = e^{-x}$

$\dfrac{d}{dx}(ve^{-x}) = -e^{-x}$

$ve^{-x} = e^{-x} + C$

$v = 1 + Ce^x$

**Step 6**: Back-substitute $v = 1/y$:

$$\frac{1}{y} = 1 + Ce^x$$

$$\boxed{y = \frac{1}{1 + Ce^x}}$$

**Verification**: This is the **logistic function** — connects to population modeling!

```
   y
   1 │─────────────────── (C < 0, certain values)
     │        ___________
     │      _/
     │    _/
   ½ │── /
     │  /    y = 1/(1 + Ce^x)
     │ /     (logistic curve for C > 0)
     │/
   0 ├──────────────────── x
```

### Example 2: $y' + \dfrac{y}{x} = x^2 y^3$

**Step 1**: $P = 1/x$, $Q = x^2$, $n = 3$

**Step 2**: Let $v = y^{1-3} = y^{-2}$

$\dfrac{dv}{dx} = -2y^{-3}y'$

**Step 3**: Divide original by $y^3$:
$$y^{-3}y' + \frac{y^{-2}}{x} = x^2$$

$$-\frac{1}{2}\frac{dv}{dx} + \frac{v}{x} = x^2$$

$$\frac{dv}{dx} - \frac{2v}{x} = -2x^2$$

**Step 4**: Linear in $v$! $P_v = -2/x$

$\mu = e^{\int -2/x \, dx} = e^{-2\ln x} = x^{-2}$

$\dfrac{d}{dx}(vx^{-2}) = -2x^2 \cdot x^{-2} = -2$

$vx^{-2} = -2x + C$

$v = -2x^3 + Cx^2$

**Step 5**: Back-substitute $v = y^{-2}$:

$$\frac{1}{y^2} = Cx^2 - 2x^3$$

$$\boxed{y^2 = \frac{1}{Cx^2 - 2x^3}}$$

### Example 3: IVP with $n = 1/2$

**Solve**: $y' + y = \sqrt{y}$, $y(0) = 4$

**Step 1**: $n = 1/2$, so $v = y^{1-1/2} = y^{1/2} = \sqrt{y}$

$\dfrac{dv}{dx} = \dfrac{1}{2}y^{-1/2}y'$, so $y' = 2\sqrt{y} \cdot v' = 2v \cdot v'$

**Step 2**: Divide by $y^{1/2}$:
$$y^{-1/2}y' + y^{1/2} = 1$$

$$2v' + v = 1$$

$$v' + \frac{v}{2} = \frac{1}{2}$$

**Step 3**: I.F. $= e^{x/2}$

$\dfrac{d}{dx}(ve^{x/2}) = \dfrac{1}{2}e^{x/2}$

$ve^{x/2} = e^{x/2} + C$

$v = 1 + Ce^{-x/2}$

**Step 4**: $\sqrt{y} = 1 + Ce^{-x/2}$

Apply $y(0) = 4$: $\sqrt{4} = 2 = 1 + C \implies C = 1$

$$\boxed{y = (1 + e^{-x/2})^2}$$

---

## 4. Common Values of $n$ and Their Substitutions

| $n$ | $v = y^{1-n}$ | $\dfrac{dv}{dx}$ relation |
|-----|--------------|--------------------------|
| 2 | $v = y^{-1} = 1/y$ | $dv/dx = -y^{-2}y'$ |
| 3 | $v = y^{-2}$ | $dv/dx = -2y^{-3}y'$ |
| $1/2$ | $v = y^{1/2} = \sqrt{y}$ | $dv/dx = \frac{1}{2}y^{-1/2}y'$ |
| $-1$ | $v = y^2$ | $dv/dx = 2yy'$ |
| $1/3$ | $v = y^{2/3}$ | $dv/dx = \frac{2}{3}y^{-1/3}y'$ |

---

## 5. Connection to Other Equations

### Bernoulli → Logistic Equation

When $P$ and $Q$ are constants and $n = 2$:
$$y' = ay - by^2$$

This is the **logistic equation** for population growth with carrying capacity $K = a/b$.

### Bernoulli → Riccati Equation

The **Riccati equation** $y' = P(x) + Q(x)y + R(x)y^2$ includes Bernoulli as a special case (when $P = 0$).

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Standard form** | $y' + P(x)y = Q(x)y^n$ |
| **Substitution** | $v = y^{1-n}$ |
| **Resulting linear DE** | $v' + (1-n)Pv = (1-n)Q$ |
| **Valid for** | $n \neq 0$ and $n \neq 1$ |
| **$n = 0$** | Already linear |
| **$n = 1$** | Separable |
| **Most common $n$** | $n = 2$ (gives $v = 1/y$) |
| **Don't forget** | Back-substitute $v \to y$ at the end |

---

## Quick Revision Questions

1. Solve: $y' - y = -y^3$

2. What substitution converts $y' + 2xy = 6xy^{1/3}$ to a linear equation?

3. Solve the IVP: $y' + y = y^2$, $y(0) = 1/2$

4. Is $y' = y^2 + y + 1$ a Bernoulli equation? Why or why not?

5. For $y' + y/x = y^2 \ln x$, identify $P(x)$, $Q(x)$, and $n$.

6. Show that the Bernoulli substitution $v = y^{1-n}$ always produces a **linear** equation in $v$.

---

[← Previous: Linear Equations](04-linear-equations.md) | [Back to README](../README.md) | [Next: Exact Equations →](06-exact-equations.md)
