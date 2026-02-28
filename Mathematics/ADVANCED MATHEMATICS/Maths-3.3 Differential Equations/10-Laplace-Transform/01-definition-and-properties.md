# Chapter 10.1: Definition and Properties of the Laplace Transform

[← Previous: Legendre Polynomials](../09-Series-Solutions/04-legendre-polynomials.md) | [Back to README](../README.md) | [Next: Inverse Laplace Transform →](02-inverse-laplace-transform.md)

---

## Overview

The **Laplace transform** converts a function of time $f(t)$ into a function of a complex variable $s$, turning differential equations into algebraic equations. This powerful operational technique is indispensable in engineering, control theory, and signal processing.

---

## 1. Definition

$$\boxed{\mathcal{L}\{f(t)\} = F(s) = \int_0^{\infty} e^{-st} f(t)\,dt}$$

- Domain: $f(t)$ defined for $t \geq 0$
- The integral converges for $s$ greater than some value $s_0$ (abscissa of convergence)
- Notation: $f(t) \xleftrightarrow{\mathcal{L}} F(s)$

```
  LAPLACE TRANSFORM CONCEPT

  Time Domain               Frequency Domain
  ┌─────────────┐    ℒ     ┌─────────────┐
  │   f(t)      │ ──────→  │    F(s)     │
  │  (hard ODE) │          │ (easy alg.) │
  └─────────────┘          └─────────────┘
       ↑                         │
       │      ℒ⁻¹                │
       │  ←─────────────────────┘
       │                         
  solve algebraically, then invert
```

---

## 2. Table of Standard Transforms

| $f(t)$ | $F(s) = \mathcal{L}\{f(t)\}$ | Condition |
|--------|------------------------------|-----------|
| $1$ | $\dfrac{1}{s}$ | $s > 0$ |
| $t$ | $\dfrac{1}{s^2}$ | $s > 0$ |
| $t^n$ | $\dfrac{n!}{s^{n+1}}$ | $s > 0$ |
| $e^{at}$ | $\dfrac{1}{s-a}$ | $s > a$ |
| $\sin(\omega t)$ | $\dfrac{\omega}{s^2 + \omega^2}$ | $s > 0$ |
| $\cos(\omega t)$ | $\dfrac{s}{s^2 + \omega^2}$ | $s > 0$ |
| $\sinh(at)$ | $\dfrac{a}{s^2 - a^2}$ | $s > |a|$ |
| $\cosh(at)$ | $\dfrac{s}{s^2 - a^2}$ | $s > |a|$ |
| $t^n e^{at}$ | $\dfrac{n!}{(s-a)^{n+1}}$ | $s > a$ |
| $e^{at}\sin(\omega t)$ | $\dfrac{\omega}{(s-a)^2 + \omega^2}$ | $s > a$ |
| $e^{at}\cos(\omega t)$ | $\dfrac{s-a}{(s-a)^2 + \omega^2}$ | $s > a$ |

---

## 3. Properties (Operational Rules)

### 3.1 Linearity

$$\mathcal{L}\{af(t) + bg(t)\} = aF(s) + bG(s)$$

### 3.2 First Shifting Theorem (s-shift)

$$\boxed{\mathcal{L}\{e^{at}f(t)\} = F(s-a)}$$

"Multiplying by $e^{at}$ in $t$-domain = shifting $s \to s-a$ in $s$-domain"

### 3.3 Second Shifting Theorem (t-shift)

$$\boxed{\mathcal{L}\{f(t-a)\,u(t-a)\} = e^{-as}F(s)}, \quad a > 0$$

where $u(t-a)$ is the Heaviside step function (see Chapter 10.5).

### 3.4 Derivatives

$$\boxed{\mathcal{L}\{f'(t)\} = sF(s) - f(0)}$$
$$\mathcal{L}\{f''(t)\} = s^2 F(s) - sf(0) - f'(0)$$
$$\mathcal{L}\{f^{(n)}(t)\} = s^n F(s) - s^{n-1}f(0) - \cdots - f^{(n-1)}(0)$$

This is **the key property** for solving ODEs — derivatives become algebraic.

### 3.5 Integration

$$\mathcal{L}\left\{\int_0^t f(\tau)\,d\tau\right\} = \frac{F(s)}{s}$$

### 3.6 Multiplication by $t$

$$\mathcal{L}\{t\,f(t)\} = -F'(s)$$

$$\mathcal{L}\{t^n f(t)\} = (-1)^n F^{(n)}(s)$$

### 3.7 Division by $t$

$$\mathcal{L}\left\{\frac{f(t)}{t}\right\} = \int_s^{\infty} F(\sigma)\,d\sigma$$

(valid if the limit $\lim_{t \to 0^+} f(t)/t$ exists)

### 3.8 Scaling

$$\mathcal{L}\{f(at)\} = \frac{1}{a}F\left(\frac{s}{a}\right), \quad a > 0$$

---

## 4. Derivation Examples

### Example 1: $\mathcal{L}\{e^{at}\}$

$$\int_0^{\infty} e^{-st} e^{at}\,dt = \int_0^{\infty} e^{-(s-a)t}\,dt = \left[\frac{e^{-(s-a)t}}{-(s-a)}\right]_0^{\infty} = \frac{1}{s-a}, \quad s > a$$

### Example 2: $\mathcal{L}\{t^2 e^{3t}\}$

Use first shift: $\mathcal{L}\{t^2\} = \frac{2}{s^3}$

$$\mathcal{L}\{t^2 e^{3t}\} = \frac{2}{(s-3)^3}$$

### Example 3: $\mathcal{L}\{t\sin 2t\}$

$$\mathcal{L}\{\sin 2t\} = \frac{2}{s^2 + 4}$$

$$\mathcal{L}\{t\sin 2t\} = -\frac{d}{ds}\left(\frac{2}{s^2+4}\right) = \frac{4s}{(s^2+4)^2}$$

### Example 4: $\mathcal{L}\{e^{-2t}\cos 3t\}$

$$\mathcal{L}\{\cos 3t\} = \frac{s}{s^2 + 9}$$

First shift ($a = -2$): replace $s$ with $s+2$:

$$\mathcal{L}\{e^{-2t}\cos 3t\} = \frac{s+2}{(s+2)^2 + 9}$$

---

## 5. Existence Conditions

$\mathcal{L}\{f(t)\}$ exists if:
1. $f(t)$ is **piecewise continuous** on $[0, \infty)$
2. $f(t)$ is of **exponential order**: $|f(t)| \leq Me^{\alpha t}$ for some $M, \alpha$

Functions like $e^{t^2}$ do NOT have a Laplace transform (grows too fast).

---

## 6. Properties Summary

```
  PROPERTY MAP

  ┌──────────────────────────────┐
  │  t-domain  →  s-domain      │
  ├──────────────────────────────┤
  │  f'(t)     →  sF(s) - f(0)  │
  │  e^at f(t) →  F(s-a)        │
  │  t·f(t)    →  -F'(s)        │
  │  f(t-a)u(t-a) → e^{-as}F(s)│
  │  ∫₀ᵗf(τ)dτ →  F(s)/s       │
  │  f(at)     →  (1/a)F(s/a)   │
  └──────────────────────────────┘
```

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Definition** | $F(s) = \int_0^\infty e^{-st}f(t)\,dt$ |
| **Linearity** | $\mathcal{L}\{af+bg\} = aF + bG$ |
| **Derivative rule** | $\mathcal{L}\{f'\} = sF - f(0)$ |
| **First shift** | $\mathcal{L}\{e^{at}f\} = F(s-a)$ |
| **Second shift** | $\mathcal{L}\{f(t-a)u(t-a)\} = e^{-as}F$ |
| **Multiply by $t$** | $\mathcal{L}\{tf\} = -F'(s)$ |
| **Existence** | Piecewise continuous + exponential order |

---

## Quick Revision Questions

1. Find $\mathcal{L}\{3t^2 + 5e^{-4t} - 2\sin 3t\}$ using linearity and the table.

2. Compute $\mathcal{L}\{t^3 e^{2t}\}$ using the first shifting theorem.

3. Derive $\mathcal{L}\{\cos \omega t\}$ from the definition.

4. Find $\mathcal{L}\{e^{-t}\sin 2t \cdot t\}$ using a combination of shifting and the $t$-multiplication rule.

5. Does $\mathcal{L}\{e^{t^2}\}$ exist? Justify.

6. If $F(s) = \dfrac{2}{(s+1)^3}$, what is $f(t)$?

---

[← Previous: Legendre Polynomials](../09-Series-Solutions/04-legendre-polynomials.md) | [Back to README](../README.md) | [Next: Inverse Laplace Transform →](02-inverse-laplace-transform.md)
