# Chapter 10.3: Solving ODEs with Laplace Transform

[← Previous: Inverse Laplace Transform](02-inverse-laplace-transform.md) | [Back to README](../README.md) | [Next: Convolution Theorem →](04-convolution-theorem.md)

---

## Overview

The Laplace transform's greatest power is solving **initial value problems** (IVPs) directly — initial conditions are built into the process, not applied after finding the general solution. This chapter demonstrates the complete workflow.

---

## 1. The Method

```
  SOLVING ODEs WITH LAPLACE TRANSFORM

  ┌──────────────────────┐
  │ ODE + initial conds  │
  │ ay'' + by' + cy = g(t)│
  │ y(0) = y₀, y'(0)=y₁ │
  └──────────┬───────────┘
             │ Step 1: Take ℒ
             ▼
  ┌──────────────────────┐
  │ Algebraic equation   │
  │ in Y(s)              │
  │ (polynomial in s)    │
  └──────────┬───────────┘
             │ Step 2: Solve for Y(s)
             ▼
  ┌──────────────────────┐
  │ Y(s) = rational      │
  │ function of s        │
  └──────────┬───────────┘
             │ Step 3: Partial fractions
             │         + ℒ⁻¹
             ▼
  ┌──────────────────────┐
  │ y(t) = solution!     │
  │ (ICs already applied)│
  └──────────────────────┘
```

### Key Transform Rules for Derivatives

$$\mathcal{L}\{y'\} = sY(s) - y(0)$$
$$\mathcal{L}\{y''\} = s^2Y(s) - sy(0) - y'(0)$$
$$\mathcal{L}\{y'''\} = s^3Y(s) - s^2y(0) - sy'(0) - y''(0)$$

---

## 2. Worked Examples

### Example 1: First-Order IVP

$$y' + 3y = 6, \quad y(0) = 2$$

**Step 1**: $sY - 2 + 3Y = \dfrac{6}{s}$

**Step 2**: $(s+3)Y = \dfrac{6}{s} + 2 = \dfrac{6 + 2s}{s}$

$$Y = \frac{2s + 6}{s(s+3)} = \frac{2(s+3)}{s(s+3)} = \frac{2}{s}$$

$$\boxed{y(t) = 2}$$

(Equilibrium — the initial condition matches the steady state!)

### Example 2: Second-Order with Constant Forcing

$$y'' + 4y = 8, \quad y(0) = 0, \quad y'(0) = 0$$

**Step 1**: $s^2Y + 4Y = \dfrac{8}{s}$

**Step 2**: $Y = \dfrac{8}{s(s^2+4)}$

**Step 3**: Partial fractions:

$$\frac{8}{s(s^2+4)} = \frac{A}{s} + \frac{Bs + C}{s^2+4}$$

$s=0$: $A = 2$. Comparing $s^2$: $A + B = 0 \implies B = -2$. Constants: $4A + C = 0 \implies C = 0$ (wait: $4A = 8$ and the constant term on right = $4A + C$, so from numerator matching: $C = 0$).

$$Y = \frac{2}{s} - \frac{2s}{s^2+4}$$

$$\boxed{y(t) = 2 - 2\cos 2t}$$

### Example 3: Damped Oscillator

$$y'' + 2y' + 5y = 0, \quad y(0) = 1, \quad y'(0) = -1$$

$s^2Y - s + 1 + 2(sY - 1) + 5Y = 0$

$(s^2 + 2s + 5)Y = s + 1$

$$Y = \frac{s+1}{s^2+2s+5} = \frac{s+1}{(s+1)^2+4} = \frac{(s+1)}{(s+1)^2 + 2^2}$$

$$\boxed{y(t) = e^{-t}\cos 2t}$$

### Example 4: Resonance

$$y'' + 9y = \cos 3t, \quad y(0) = 0, \quad y'(0) = 0$$

$s^2Y + 9Y = \dfrac{s}{s^2+9}$

$$Y = \frac{s}{(s^2+9)^2}$$

Using the formula $\mathcal{L}^{-1}\left\{\dfrac{s}{(s^2+\omega^2)^2}\right\} = \dfrac{t\sin\omega t}{2\omega}$:

$$\boxed{y(t) = \frac{t\sin 3t}{6}}$$

The growing amplitude confirms **resonance**.

### Example 5: Exponential Forcing

$$y'' - 3y' + 2y = e^{3t}, \quad y(0) = 1, \quad y'(0) = 0$$

$s^2Y - s + 0 - 3(sY - 1) + 2Y = \dfrac{1}{s-3}$

$(s^2 - 3s + 2)Y = s - 3 + \dfrac{1}{s-3} = \dfrac{(s-3)^2 + 1}{s-3}$

$(s-1)(s-2)Y = \dfrac{s^2 - 6s + 10}{s-3}$

$$Y = \frac{s^2 - 6s + 10}{(s-1)(s-2)(s-3)}$$

Partial fractions:

$s = 1$: $\dfrac{5}{(-1)(-2)} = \dfrac{5}{2}$

$s = 2$: $\dfrac{2}{(1)(-1)} = -2$

$s = 3$: $\dfrac{1}{(2)(1)} = \dfrac{1}{2}$

$$\boxed{y(t) = \frac{5}{2}e^t - 2e^{2t} + \frac{1}{2}e^{3t}}$$

### Example 6: Third-Order ODE

$$y''' - y = 0, \quad y(0) = 0, \quad y'(0) = 1, \quad y''(0) = 0$$

$s^3Y - 0 - s - 0 - Y = 0$

$$Y = \frac{s}{s^3 - 1} = \frac{s}{(s-1)(s^2+s+1)}$$

Partial fractions: $\dfrac{s}{(s-1)(s^2+s+1)} = \dfrac{A}{s-1} + \dfrac{Bs+C}{s^2+s+1}$

$s = 1$: $A = 1/3$

Comparing $s^2$: $A + B = 0 \implies B = -1/3$

Constants: $A + C = 0 \implies C = -1/3$

$$Y = \frac{1/3}{s-1} - \frac{s/3 + 1/3}{s^2+s+1}$$

Complete the square: $s^2+s+1 = (s+\frac{1}{2})^2 + \frac{3}{4}$

$$y(t) = \frac{1}{3}e^t - \frac{1}{3}e^{-t/2}\cos\frac{\sqrt{3}}{2}t - \frac{1}{\sqrt{3}}e^{-t/2}\sin\frac{\sqrt{3}}{2}t$$

---

## 3. Advantages of the Laplace Method

| Advantage | Explanation |
|-----------|------------|
| ICs built in | No need to find general solution first |
| Handles discontinuous forcing | Heaviside/Dirac inputs (see Ch. 10.5) |
| Systematic | Reduces to algebra + table lookup |
| Works for systems | Transform each equation |
| No guessing required | Unlike undetermined coefficients |

---

## 4. Transfer Function

For a linear system with zero ICs:

$$a y'' + b y' + c y = g(t)$$

$$Y(s) = \frac{G(s)}{as^2 + bs + c} = H(s) \cdot G(s)$$

where $H(s) = \dfrac{1}{as^2 + bs + c}$ is the **transfer function**.

```
  TRANSFER FUNCTION VIEW

  Input G(s) ──→ ┌─────────┐ ──→ Output Y(s)
                  │  H(s)   │
                  │= 1/P(s) │
                  └─────────┘
  
  P(s) = characteristic polynomial
```

---

## Summary Table

| Step | Action |
|------|--------|
| 1 | Take $\mathcal{L}$ of entire ODE |
| 2 | Substitute $\mathcal{L}\{y'\} = sY - y(0)$, etc. |
| 3 | Solve algebraically for $Y(s)$ |
| 4 | Partial fractions to decompose $Y(s)$ |
| 5 | Invert: $y(t) = \mathcal{L}^{-1}\{Y(s)\}$ |

---

## Quick Revision Questions

1. Solve $y' - 2y = e^{3t}$, $y(0) = 1$ using Laplace transforms.

2. Solve $y'' + y = \sin t$, $y(0) = 0$, $y'(0) = 0$. What physical phenomenon does this illustrate?

3. Use Laplace transforms to solve the system: $x' = 2x - y$, $y' = x + 2y$, with $x(0) = 1$, $y(0) = 0$.

4. Find the transfer function for $y'' + 4y' + 3y = g(t)$.

5. Solve $y'' + 6y' + 9y = 0$, $y(0) = 2$, $y'(0) = -3$. (Repeated roots case.)

6. Compare solving $y'' + y = e^t$, $y(0) = 0$, $y'(0) = 0$ by (a) undetermined coefficients and (b) Laplace transform. Which is faster?

---

[← Previous: Inverse Laplace Transform](02-inverse-laplace-transform.md) | [Back to README](../README.md) | [Next: Convolution Theorem →](04-convolution-theorem.md)
