# Chapter 5.3: Reduction of Order

[← Previous: Fundamental Set of Solutions](02-fundamental-set-of-solutions.md) | [Back to README](../README.md) | [Next: Constant Coefficient Homogeneous →](04-constant-coefficient-homogeneous.md)

---

## Overview

If one solution $y_1$ of a second-order linear homogeneous ODE is known, reduction of order finds the second linearly independent solution $y_2$. The technique substitutes $y_2 = v(x)y_1$ and reduces the problem to a first-order ODE.

---

## 1. The Method

### Given

$$y'' + P(x)y' + Q(x)y = 0$$

where $y_1$ is a known (nonzero) solution.

### Assume $y_2 = v(x) \cdot y_1(x)$

Compute:
- $y_2' = v'y_1 + vy_1'$
- $y_2'' = v''y_1 + 2v'y_1' + vy_1''$

Substitute into the ODE:

$$(v''y_1 + 2v'y_1' + vy_1'') + P(v'y_1 + vy_1') + Q(vy_1) = 0$$

$$v''y_1 + 2v'y_1' + v\underbrace{(y_1'' + Py_1' + Qy_1)}_{= 0} + Pv'y_1 = 0$$

$$v''y_1 + v'(2y_1' + Py_1) = 0$$

### Substitution $w = v'$

$$w'y_1 + w(2y_1' + Py_1) = 0$$

$$\frac{w'}{w} = -\frac{2y_1'}{y_1} - P$$

```
REDUCTION OF ORDER — WORKFLOW

  ┌──────────────────────────┐
  │ Known: y₁ (one solution) │
  └─────────┬────────────────┘
            │
            ▼
  ┌──────────────────────────┐
  │ Set y₂ = v(x)·y₁        │
  └─────────┬────────────────┘
            │
            ▼
  ┌──────────────────────────┐
  │ Substitute into ODE      │
  │ Terms with v cancel      │
  │ (since y₁ is a solution) │
  └─────────┬────────────────┘
            │
            ▼
  ┌──────────────────────────┐
  │ Let w = v' → 1st order   │
  │ in w (linear, separable) │
  └─────────┬────────────────┘
            │
            ▼
  ┌──────────────────────────┐
  │ Solve for w, then        │
  │ integrate: v = ∫w dx     │
  └─────────┬────────────────┘
            │
            ▼
  ┌──────────────────────────┐
  │ y₂ = v·y₁                │
  └──────────────────────────┘
```

---

## 2. The Formula

Solving the first-order ODE for $w$:

$$w = \frac{1}{y_1^2}\exp\left(-\int P\,dx\right)$$

Then:

$$\boxed{y_2 = y_1 \int \frac{1}{y_1^2}\exp\left(-\int P\,dx\right)\,dx}$$

> Note: We omit constants of integration since we need only **one** linearly independent solution.

---

## 3. Worked Examples

### Example 1: $y'' - 2y' + y = 0$ with $y_1 = e^x$

Here $P = -2$, $Q = 1$.

$$y_2 = e^x \int \frac{1}{e^{2x}} \exp\left(-\int (-2)\,dx\right)\,dx = e^x \int \frac{e^{2x}}{e^{2x}}\,dx = e^x \int dx = xe^x$$

$$\boxed{y_2 = xe^x}$$

General solution: $y = (c_1 + c_2 x)e^x$

### Example 2: $x^2 y'' - 2xy' + 2y = 0$ with $y_1 = x$

Rewrite in standard form: $y'' - \dfrac{2}{x}y' + \dfrac{2}{x^2}y = 0$, so $P = -2/x$.

$$y_2 = x \int \frac{1}{x^2}\exp\left(-\int \frac{-2}{x}\,dx\right)\,dx = x \int \frac{1}{x^2} \cdot x^2\,dx = x \int dx = x^2$$

$$\boxed{y_2 = x^2}$$

General solution: $y = c_1 x + c_2 x^2$

### Example 3: $y'' + y = 0$ with $y_1 = \cos x$

$P = 0$.

$$y_2 = \cos x \int \frac{1}{\cos^2 x} \cdot 1\,dx = \cos x \int \sec^2 x\,dx = \cos x \cdot \tan x = \sin x$$

$$\boxed{y_2 = \sin x}$$

### Example 4: $(1-x^2)y'' - 2xy' + 2y = 0$ with $y_1 = x$ (Legendre-type)

Standard form: $y'' - \dfrac{2x}{1-x^2}y' + \dfrac{2}{1-x^2}y = 0$

$P = \dfrac{-2x}{1-x^2}$

$$-\int P\,dx = \int \frac{2x}{1-x^2}\,dx = -\ln|1-x^2|$$

$$e^{-\int P\,dx} = \frac{1}{1-x^2}$$

$$y_2 = x\int \frac{1}{x^2} \cdot \frac{1}{1-x^2}\,dx$$

Partial fractions: $\dfrac{1}{x^2(1-x^2)} = \dfrac{1}{x^2} + \dfrac{1}{1-x^2} = \dfrac{1}{x^2} + \dfrac{1/2}{1-x} + \dfrac{1/2}{1+x}$

$$\int = -\frac{1}{x} + \frac{1}{2}\ln\frac{1+x}{1-x}$$

$$\boxed{y_2 = x\left(-\frac{1}{x} + \frac{1}{2}\ln\frac{1+x}{1-x}\right) = -1 + \frac{x}{2}\ln\frac{1+x}{1-x}}$$

### Example 5: Euler-Cauchy $x^2y'' + xy' - y = 0$ with $y_1 = x$

Standard form: $y'' + \dfrac{1}{x}y' - \dfrac{1}{x^2}y = 0$, $P = 1/x$

$$e^{-\int(1/x)\,dx} = e^{-\ln x} = \frac{1}{x}$$

$$y_2 = x\int \frac{1}{x^2} \cdot \frac{1}{x}\,dx = x\int \frac{dx}{x^3} = x \cdot \frac{-1}{2x^2} = \frac{-1}{2x}$$

$$\boxed{y_2 = \frac{1}{x}}$$

General solution: $y = c_1 x + c_2/x$

---

## 4. Verification Using the Wronskian

After finding $y_2$, always verify $W(y_1, y_2) \neq 0$:

| Example | $y_1$ | $y_2$ | $W$ |
|---------|-------|-------|-----|
| Ex. 1 | $e^x$ | $xe^x$ | $e^{2x}$ |
| Ex. 2 | $x$ | $x^2$ | $x^2$ |
| Ex. 3 | $\cos x$ | $\sin x$ | $1$ |
| Ex. 5 | $x$ | $1/x$ | $-2/x$ |

All nonzero (on appropriate intervals) ✓

---

## 5. Connection to Abel's Formula

The formula $y_2 = y_1 \displaystyle\int \frac{e^{-\int P\,dx}}{y_1^2}\,dx$ is actually a consequence of Abel's formula:

$$W = y_1 y_2' - y_2 y_1' = Ce^{-\int P\,dx}$$

Dividing by $y_1^2$:

$$\frac{d}{dx}\left(\frac{y_2}{y_1}\right) = \frac{Ce^{-\int P\,dx}}{y_1^2}$$

Integrating gives the reduction of order formula.

---

## 6. When to Use Reduction of Order

| Situation | Use reduction of order? |
|-----------|------------------------|
| One solution known, need second | ✅ Yes — primary use |
| Constant coefficients | ❌ Use characteristic equation (easier) |
| Variable coefficients, one solution guessed | ✅ Yes |
| Euler-Cauchy equations | ✅ Alternative to standard method |
| No solution known | ❌ Cannot apply |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Assumption** | $y_2 = v(x)y_1(x)$ |
| **Formula** | $y_2 = y_1\displaystyle\int \frac{e^{-\int P\,dx}}{y_1^2}\,dx$ |
| **Reduces to** | First-order linear ODE in $w = v'$ |
| **Requirement** | One solution $y_1$ must be known |
| **Verification** | Check $W(y_1, y_2) \neq 0$ |
| **Connection** | Derived from Abel's formula |

---

## Quick Revision Questions

1. Given $y'' + 4y = 0$ with $y_1 = \cos 2x$, use reduction of order to find $y_2$.

2. For $x^2y'' - 3xy' + 4y = 0$ with $y_1 = x^2$, find the second solution.

3. Show that the reduction of order formula can be derived from Abel's formula.

4. Apply this method to $y'' - 6y' + 9y = 0$ knowing $y_1 = e^{3x}$.

5. For $y'' + y' = 0$ with $y_1 = 1$, find $y_2$ using reduction of order.

6. Why does the method fail if $y_1$ is not a solution of the ODE?

---

[← Previous: Fundamental Set of Solutions](02-fundamental-set-of-solutions.md) | [Back to README](../README.md) | [Next: Constant Coefficient Homogeneous →](04-constant-coefficient-homogeneous.md)
