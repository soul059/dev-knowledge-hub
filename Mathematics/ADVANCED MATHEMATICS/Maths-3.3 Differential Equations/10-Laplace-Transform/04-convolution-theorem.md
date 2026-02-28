# Chapter 10.4: Convolution Theorem

[← Previous: Solving ODEs with Laplace](03-solving-odes-with-laplace.md) | [Back to README](../README.md) | [Next: Step and Impulse Functions →](05-step-and-impulse-functions.md)

---

## Overview

The **convolution theorem** provides a way to find the inverse Laplace transform of a product $F(s)\cdot G(s)$ without partial fractions. It also gives a beautiful interpretation: the output of a linear system is the convolution of the input with the system's impulse response.

---

## 1. Convolution Definition

$$\boxed{(f * g)(t) = \int_0^t f(\tau)\,g(t - \tau)\,d\tau}$$

This integral "slides" $g$ over $f$ and accumulates the product — it captures the system's memory of past inputs.

```
  CONVOLUTION — VISUAL INTERPRETATION

  f(τ)          g(t-τ)          f*g at time t
  │              │
  │ ╱╲           │    ╱╲        = ∫ product
  │╱  ╲          │   ╱  ╲         over τ
  │    ╲         │  ╱    ╲
  ──────╲───→τ   │╱───────╲→τ
  0     a        0    t-a   t

  g is flipped and shifted → multiply pointwise → integrate
```

---

## 2. Properties of Convolution

| Property | Formula |
|----------|---------|
| Commutativity | $f * g = g * f$ |
| Associativity | $(f * g) * h = f * (g * h)$ |
| Distributivity | $f * (g + h) = f * g + f * h$ |
| Identity | $f * \delta = f$ (where $\delta$ is the Dirac delta) |
| Zero | $f * 0 = 0$ |
| NOT multiplicative | $f * g \neq fg$ in general |

---

## 3. The Convolution Theorem

$$\boxed{\mathcal{L}\{f * g\} = F(s) \cdot G(s)}$$

Equivalently:

$$\mathcal{L}^{-1}\{F(s) \cdot G(s)\} = (f * g)(t) = \int_0^t f(\tau)\,g(t-\tau)\,d\tau$$

```
  CONVOLUTION THEOREM

  t-domain               s-domain
  ┌─────────────┐        ┌─────────────┐
  │  f(t) * g(t) │  ←→   │  F(s) · G(s) │
  │ (convolution)│        │  (product)   │
  └─────────────┘        └─────────────┘

  Convolution in time = Multiplication in s
```

---

## 4. Worked Examples

### Example 1: Basic Convolution

Find $\mathcal{L}^{-1}\left\{\dfrac{1}{s(s-1)}\right\}$ using convolution.

$F(s) = \dfrac{1}{s}$, $G(s) = \dfrac{1}{s-1}$ → $f(t) = 1$, $g(t) = e^t$

$$(f * g)(t) = \int_0^t 1 \cdot e^{t-\tau}\,d\tau = e^t \int_0^t e^{-\tau}\,d\tau = e^t[-e^{-\tau}]_0^t = e^t(1 - e^{-t}) = e^t - 1$$

Verify: $\dfrac{1}{s(s-1)} = \dfrac{-1}{s} + \dfrac{1}{s-1}$ → $-1 + e^t = e^t - 1$ ✓

### Example 2: Repeated Factor

Find $\mathcal{L}^{-1}\left\{\dfrac{1}{(s+1)^2}\right\}$.

$F = G = \dfrac{1}{s+1}$ → $f = g = e^{-t}$

$$(f * g)(t) = \int_0^t e^{-\tau} e^{-(t-\tau)}\,d\tau = e^{-t}\int_0^t d\tau = te^{-t}$$

### Example 3: With Trigonometric Functions

Find $\mathcal{L}^{-1}\left\{\dfrac{1}{s^2(s^2+1)}\right\}$.

$F = \dfrac{1}{s^2}$ → $f = t$, $G = \dfrac{1}{s^2+1}$ → $g = \sin t$

$$(f * g)(t) = \int_0^t \tau \sin(t-\tau)\,d\tau$$

Integration by parts:

$$= [-\tau\cos(t-\tau)]_0^t - \int_0^t [-\cos(t-\tau)]\,d\tau$$

$$= -t\cos 0 + [\sin(t-\tau)]_0^t = -t + 0 - (- \sin t) = t - \sin t$$

Verify by partial fractions: $\dfrac{1}{s^2} - \dfrac{1}{s^2+1}$ → $t - \sin t$ ✓

### Example 4: Solving an Integral Equation

Solve: $y(t) = t + \displaystyle\int_0^t y(\tau)\sin(t-\tau)\,d\tau$

This is $y = t + y * \sin t$.

Take $\mathcal{L}$: $Y = \dfrac{1}{s^2} + Y \cdot \dfrac{1}{s^2+1}$

$$Y\left(1 - \frac{1}{s^2+1}\right) = \frac{1}{s^2}$$

$$Y \cdot \frac{s^2}{s^2+1} = \frac{1}{s^2}$$

$$Y = \frac{s^2+1}{s^4} = \frac{1}{s^2} + \frac{1}{s^4}$$

$$\boxed{y(t) = t + \frac{t^3}{6}}$$

### Example 5: Solving an ODE

Solve $y'' + y = \sin t$, $y(0) = 0$, $y'(0) = 0$.

$$Y = \frac{1}{(s^2+1)^2}$$

Using convolution: $f = g = \sin t$

$$(f*g)(t) = \int_0^t \sin\tau \sin(t-\tau)\,d\tau$$

Using product-to-sum: $\sin\tau\sin(t-\tau) = \frac{1}{2}[\cos(2\tau - t) - \cos t]$

$$= \frac{1}{2}\left[\frac{\sin(2\tau-t)}{2}\right]_0^t - \frac{t\cos t}{2} = \frac{1}{2}\left[\frac{\sin t}{2} - \frac{-\sin t}{2}\right] - \frac{t\cos t}{2}$$

$$\boxed{y(t) = \frac{\sin t - t\cos t}{2}}$$

---

## 5. Application: Transfer Functions and Impulse Response

For a system $H(s) = \dfrac{1}{P(s)}$:

- **Impulse response**: $h(t) = \mathcal{L}^{-1}\{H(s)\}$
- **Output for any input** $g(t)$: $y(t) = h * g = \displaystyle\int_0^t h(\tau)\,g(t-\tau)\,d\tau$

```
  SYSTEM INTERPRETATION

  g(t) ──→ ┌──────────┐ ──→ y(t) = h * g
  Input     │   H(s)   │     Output
            │ = ℒ{h(t)}│
            └──────────┘
            Impulse
            Response h(t)

  Everything flows through convolution!
```

---

## 6. Integral Equations

The convolution theorem converts **Volterra integral equations** to algebraic:

| Equation Type | Form | Laplace Result |
|--------------|------|---------------|
| Volterra (2nd kind) | $y = f + \lambda\,k * y$ | $Y = \dfrac{F}{1 - \lambda K}$ |
| Abel's equation | $f(t) = \displaystyle\int_0^t \dfrac{y(\tau)}{\sqrt{t-\tau}}\,d\tau$ | $F = Y \cdot \dfrac{\sqrt{\pi}}{s^{1/2}}$ |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Convolution** | $(f*g)(t) = \int_0^t f(\tau)g(t-\tau)\,d\tau$ |
| **Theorem** | $\mathcal{L}\{f*g\} = F \cdot G$ |
| **Commutative** | $f * g = g * f$ |
| **Identity** | $f * \delta = f$ |
| **Impulse response** | $h = \mathcal{L}^{-1}\{H(s)\}$ |
| **System output** | $y = h * g$ (input convolved with impulse response) |
| **Integral equations** | Transform to algebraic via convolution theorem |

---

## Quick Revision Questions

1. Evaluate $e^t * e^{2t}$ directly and verify using the convolution theorem.

2. Find $\mathcal{L}^{-1}\left\{\dfrac{1}{s(s^2+4)}\right\}$ using convolution.

3. Solve the integral equation $y(t) = 1 + \displaystyle\int_0^t y(\tau)(t-\tau)\,d\tau$.

4. Show that $f * g = g * f$ by substituting $u = t - \tau$ in the integral.

5. For a system with $H(s) = \dfrac{1}{s+3}$, find the output when the input is $g(t) = e^{-t}$.

6. Use convolution to solve $y'' + 4y = g(t)$, $y(0) = y'(0) = 0$ in terms of an arbitrary $g(t)$.

---

[← Previous: Solving ODEs with Laplace](03-solving-odes-with-laplace.md) | [Back to README](../README.md) | [Next: Step and Impulse Functions →](05-step-and-impulse-functions.md)
