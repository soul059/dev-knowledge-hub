# Chapter 6.4: Cauchy-Euler Non-Homogeneous Equations

[← Previous: D-Operator Methods](03-d-operator-methods.md) | [Back to README](../README.md) | [Next: Simple Harmonic Motion →](../07-Applications-Higher-Order/01-simple-harmonic-motion.md)

---

## Overview

Non-homogeneous Cauchy-Euler (equidimensional) equations combine the variable-coefficient structure $x^k y^{(k)}$ with a right-hand side $g(x)$. They are solved by either transforming to constant coefficients via $x = e^t$ or using variation of parameters.

---

## 1. Standard Form

$$\boxed{x^2 y'' + axy' + by = g(x)}$$

More generally, for order $n$:

$$x^n y^{(n)} + a_{n-1}x^{n-1}y^{(n-1)} + \cdots + a_1 xy' + a_0 y = g(x)$$

---

## 2. Method 1: Transformation $x = e^t$

### Substitution Relations

Let $t = \ln x$ (so $x = e^t$) and $\theta = \dfrac{d}{dt}$:

| Expression | In terms of $\theta$ |
|-----------|---------------------|
| $xy'$ | $\theta y$ |
| $x^2 y''$ | $\theta(\theta - 1)y$ |
| $x^3 y'''$ | $\theta(\theta - 1)(\theta - 2)y$ |
| $x^k y^{(k)}$ | $\theta(\theta-1)\cdots(\theta-k+1)y$ |

```
TRANSFORMATION WORKFLOW

  x²y'' + axy' + by = g(x)
           │
           │  x = eᵗ,  θ = d/dt
           ▼
  θ(θ-1)y + aθy + by = g(eᵗ)
           │
           │  Expand
           ▼
  [θ² + (a-1)θ + b]y = G(t)      ← Constant coefficient ODE!
           │
           │  Solve using standard methods
           ▼
  y(t) = y_c(t) + y_p(t)
           │
           │  Back-substitute: t = ln x
           ▼
  y(x) = final answer
```

---

## 3. Method 2: Variation of Parameters

Once $y_c$ is found (using $y = x^m$), apply variation of parameters with the standard form:

$$y'' + \frac{a}{x}y' + \frac{b}{x^2}y = \frac{g(x)}{x^2}$$

---

## 4. Worked Examples

### Example 1: $x^2y'' - 2xy' + 2y = x^3$

**Using transformation**: Let $x = e^t$, $\theta = d/dt$

$$[\theta(\theta-1) - 2\theta + 2]y = e^{3t}$$

$$(\theta^2 - 3\theta + 2)y = e^{3t}$$

$$(\theta - 1)(\theta - 2)y = e^{3t}$$

$y_c(t) = c_1 e^t + c_2 e^{2t}$

$y_p(t) = \dfrac{e^{3t}}{(3-1)(3-2)} = \dfrac{e^{3t}}{2}$

Back-substitute $e^t = x$:

$$\boxed{y = c_1 x + c_2 x^2 + \frac{x^3}{2}}$$

### Example 2: $x^2y'' + xy' - y = \ln x$

$\theta(\theta-1)y + \theta y - y = t$ (since $\ln x = t$)

$(\theta^2 - 1)y = t$

$y_c(t) = c_1 e^t + c_2 e^{-t}$

$y_p = \dfrac{1}{\theta^2 - 1}t = \dfrac{-1}{1 - \theta^2}t = -(1 + \theta^2 + \cdots)t = -t$

(Checking: $y_p = -t$, $y_p'' = 0$, so $0 - (-t) = t$ ✓)

Back-substitute:

$$\boxed{y = c_1 x + \frac{c_2}{x} - \ln x}$$

### Example 3: $x^2y'' - 3xy' + 4y = x^2\ln x$

$[\theta(\theta-1) - 3\theta + 4]y = e^{2t} \cdot t$

$(\theta^2 - 4\theta + 4)y = te^{2t}$

$(\theta - 2)^2 y = te^{2t}$

$y_c = (c_1 + c_2 t)e^{2t}$

For $y_p$: Use exponential shift

$y_p = \dfrac{1}{(\theta - 2)^2}te^{2t} = e^{2t}\dfrac{1}{\theta^2}t = e^{2t}\cdot\dfrac{t^3}{6}$

Wait — $\dfrac{1}{\theta^2}t = \displaystyle\int\int t\,dt\,dt = \int\frac{t^2}{2}\,dt = \frac{t^3}{6}$

Back-substitute:

$$\boxed{y = (c_1 + c_2\ln x)x^2 + \frac{x^2(\ln x)^3}{6}}$$

### Example 4: $x^2y'' + 3xy' + y = \dfrac{1}{x}$ (Using Variation of Parameters)

Indicial equation: $m(m-1) + 3m + 1 = m^2 + 2m + 1 = (m+1)^2 = 0$

$m = -1$ (repeated): $y_c = (c_1 + c_2\ln x)\dfrac{1}{x}$

$y_1 = \dfrac{1}{x}$, $y_2 = \dfrac{\ln x}{x}$

Standard form: $y'' + \dfrac{3}{x}y' + \dfrac{1}{x^2}y = \dfrac{1}{x^3}$

$W = \dfrac{1}{x}\cdot\dfrac{1-\ln x}{x^2} - \dfrac{\ln x}{x}\cdot\dfrac{-1}{x^2} = \dfrac{1 - \ln x + \ln x}{x^3} = \dfrac{1}{x^3}$

$u_1' = \dfrac{-(\ln x/x)(1/x^3)}{1/x^3} = -\dfrac{\ln x}{x}$

$u_1 = -\dfrac{(\ln x)^2}{2}$

$u_2' = \dfrac{(1/x)(1/x^3)}{1/x^3} = \dfrac{1}{x}$

$u_2 = \ln x$

$y_p = -\dfrac{(\ln x)^2}{2}\cdot\dfrac{1}{x} + \ln x \cdot \dfrac{\ln x}{x} = \dfrac{(\ln x)^2}{2x}$

$$\boxed{y = \frac{c_1 + c_2\ln x}{x} + \frac{(\ln x)^2}{2x}}$$

### Example 5: $x^2y'' + xy' - 4y = x^4$

$\theta(\theta-1) + \theta - 4 = \theta^2 - 4 = (\theta - 2)(\theta + 2)$

$(\theta^2 - 4)y = e^{4t}$

$y_p = \dfrac{e^{4t}}{16 - 4} = \dfrac{e^{4t}}{12}$

$$\boxed{y = c_1 x^2 + \frac{c_2}{x^2} + \frac{x^4}{12}}$$

---

## 5. Common Right-Hand Sides After Transformation

| Original $g(x)$ | After $x = e^t$: $G(t)$ |
|-----------------|-------------------------|
| $x^n$ | $e^{nt}$ |
| $\ln x$ | $t$ |
| $x^n \ln x$ | $te^{nt}$ |
| $(\ln x)^2$ | $t^2$ |
| $\sin(\ln x)$ | $\sin t$ |
| $1/x^n$ | $e^{-nt}$ |

---

## 6. Third-Order Example

### $x^3y''' + x^2y'' - 2xy' + 2y = x^3$

$[\theta(\theta-1)(\theta-2) + \theta(\theta-1) - 2\theta + 2]y = e^{3t}$

$[\theta^3 - 3\theta^2 + 2\theta + \theta^2 - \theta - 2\theta + 2]y = e^{3t}$

$[\theta^3 - 2\theta^2 - \theta + 2]y = e^{3t}$

$(\theta - 1)(\theta + 1)(\theta - 2)y = e^{3t}$

$y_c = c_1 e^t + c_2 e^{-t} + c_3 e^{2t} = c_1 x + \dfrac{c_2}{x} + c_3 x^2$

$y_p = \dfrac{e^{3t}}{(3-1)(3+1)(3-2)} = \dfrac{e^{3t}}{8} = \dfrac{x^3}{8}$

$$\boxed{y = c_1 x + \frac{c_2}{x} + c_3 x^2 + \frac{x^3}{8}}$$

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Euler-Cauchy NH** | $x^2y'' + axy' + by = g(x)$ |
| **Transformation** | $x = e^t$, $\theta = d/dt$ |
| **$xy'$** | $\theta y$ |
| **$x^2y''$** | $\theta(\theta-1)y$ |
| **After transform** | Constant-coefficient ODE in $t$ |
| **Back-substitute** | $t = \ln x$, $e^{mt} = x^m$ |
| **Alternative** | Variation of parameters on standard form |

---

## Quick Revision Questions

1. Solve $x^2y'' + xy' - y = x^2$ using the substitution $x = e^t$.

2. Find the particular solution of $x^2y'' - 2xy' + 2y = x^3\ln x$.

3. Solve $x^2y'' + 3xy' + y = \dfrac{1}{(1+x)^2}$ for the homogeneous part, then discuss how to handle the particular solution.

4. Transform $x^3y''' - 3x^2y'' + 6xy' - 6y = x^4$ and solve.

5. Use variation of parameters to solve $x^2y'' - 2xy' + 2y = x\sin(\ln x)$.

6. Show that $x = e^t$ transforms $x^k y^{(k)}$ into $\theta(\theta-1)\cdots(\theta-k+1)y$.

---

[← Previous: D-Operator Methods](03-d-operator-methods.md) | [Back to README](../README.md) | [Next: Simple Harmonic Motion →](../07-Applications-Higher-Order/01-simple-harmonic-motion.md)
