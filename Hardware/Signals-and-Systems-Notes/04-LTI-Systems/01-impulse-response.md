# Chapter 4.1: Impulse Response

## Overview

The **impulse response** $h(t)$ (or $h[n]$) is the output of an LTI system when the input is the unit impulse $\delta(t)$ (or $\delta[n]$). It completely characterizes an LTI system — once $h$ is known, the response to **any** input can be found via convolution.

---

## 4.1.1 Definition

### Continuous-Time

$$x(t) = \delta(t) \implies y(t) = h(t)$$

### Discrete-Time

$$x[n] = \delta[n] \implies y[n] = h[n]$$

```
INPUT:                        OUTPUT:
  δ(t)                         h(t) = impulse response
    │                            │
  1 ┤  ↑                        │╲
    │  │                         │  ╲
  0 ┤──┴──→ t     ──→ [LTI] ──→ │    ╲──→ t
                    System       │
```

---

## 4.1.2 Why the Impulse Response Matters

Since any signal can be decomposed into weighted, shifted impulses:

**CT**: $x(t) = \int_{-\infty}^{\infty} x(\tau)\delta(t-\tau)d\tau$

**DT**: $x[n] = \sum_{k=-\infty}^{\infty} x[k]\delta[n-k]$

By **linearity** and **time-invariance**:

$$y(t) = \int_{-\infty}^{\infty} x(\tau)h(t-\tau)d\tau = x(t) * h(t)$$

$$y[n] = \sum_{k=-\infty}^{\infty} x[k]h[n-k] = x[n] * h[n]$$

```
SIGNAL DECOMPOSITION → LTI SYSTEM → OUTPUT SUPERPOSITION

x[n] = ... + x[-1]δ[n+1] + x[0]δ[n] + x[1]δ[n-1] + ...
                │                │             │
                ▼                ▼             ▼
         x[-1]·h[n+1]    x[0]·h[n]    x[1]·h[n-1]
                │                │             │
                └───────┬────────┴─────────────┘
                        ▼
               y[n] = Σ x[k]·h[n-k]
```

---

## 4.1.3 Finding the Impulse Response

### Method 1: Direct Substitution

Set $x(t) = \delta(t)$ in the system equation and compute $y(t) = h(t)$.

**Example 1**: $y(t) = 3x(t-2)$

$$h(t) = 3\delta(t-2)$$

**Example 2**: $y[n] = x[n] + 2x[n-1]$

$$h[n] = \delta[n] + 2\delta[n-1]$$

```
h[n] = δ[n] + 2δ[n-1]:

  2 ┤     ↑
    │     │
  1 ┤  ↑  │
    │  │  │
  0 ┼──┴──┴──→ n
     0  1
```

### Method 2: Solve the Differential/Difference Equation

For systems described by differential equations, solve with $x(t) = \delta(t)$.

**Example 3**: $\frac{dy}{dt} + 2y(t) = x(t)$, initially at rest.

With $x(t) = \delta(t)$:
- For $t > 0$: $\frac{dy}{dt} + 2y = 0 \implies y = Ae^{-2t}$
- The impulse at $t = 0$ provides discontinuity: $y(0^+) = 1$
- Therefore: $h(t) = e^{-2t}u(t)$

```
h(t) = e⁻²ᵗu(t):

  1 ┤╲
    │ ╲
    │  ╲
    │   ╲╲
  0 ┤─────╲╲─────→ t
    0  0.5  1  1.5  2
```

**Example 4**: $y[n] - 0.5y[n-1] = x[n]$, initially at rest.

With $x[n] = \delta[n]$:
- $y[0] = x[0] + 0.5y[-1] = 1 + 0 = 1$
- $y[1] = x[1] + 0.5y[0] = 0 + 0.5 = 0.5$
- $y[n] = (0.5)^n$ for $n \geq 0$
- $h[n] = (0.5)^n u[n]$

```
h[n] = (0.5)ⁿu[n]:

  1.0 ┤  ↑
      │  │
  0.5 ┤  │  ↑
 0.25 ┤  │  │  ↑
      │  │  │  │  ↑
  0.0 ┼──┴──┴──┴──┴──→ n
       0  1  2  3  4
```

### Method 3: From Transfer Function

$$H(s) = \frac{Y(s)}{X(s)} \implies h(t) = \mathcal{L}^{-1}\{H(s)\}$$

$$H(z) = \frac{Y(z)}{X(z)} \implies h[n] = \mathcal{Z}^{-1}\{H(z)\}$$

---

## 4.1.4 Types of Impulse Responses

### FIR (Finite Impulse Response)

$h[n]$ has **finite duration** — nonzero for only a finite number of samples.

$$h[n] = \sum_{k=0}^{M} b_k \delta[n-k]$$

```
FIR Example: h[n] = {1, 2, 3, 2, 1}

  3 ┤        ↑
  2 ┤     ↑     ↑
  1 ┤  ↑           ↑
  0 ┼──┴──┴──┴──┴──┴──→ n
     0  1  2  3  4
  
  Finite length: N = 5
```

### IIR (Infinite Impulse Response)

$h[n]$ extends to infinity (though it may decay).

$$h[n] = a^n u[n], \quad |a| < 1$$

```
IIR Example: h[n] = (0.8)ⁿu[n]

  1.0 ┤  ↑
  0.8 ┤     ↑
  0.6 ┤        ↑
  0.5 ┤           ↑
  0.4 ┤              ↑
      ┤                 ↑ ...
  0.0 ┼──┴──┴──┴──┴──┴──→ n
       0  1  2  3  4  5
  
  Infinite length but decaying
```

---

## 4.1.5 Properties Determined by $h(t)$

| Property | Condition on $h$ |
|----------|-------------------|
| **Memoryless** | $h(t) = K\delta(t)$ |
| **Causal** | $h(t) = 0$ for $t < 0$ |
| **BIBO Stable** | $\int_{-\infty}^{\infty} \|h(t)\|dt < \infty$ |
| **Invertible** | $\exists h_i(t)$ such that $h(t) * h_i(t) = \delta(t)$ |

```
CAUSAL h(t):              NON-CAUSAL h(t):
    │                          │
    │  ╲                 ╱     │  ╲
    │    ╲             ╱       │    ╲
  ──┼─────╲──→ t    ──┼───────╲──→ t
    0                  -1  0   1
    
  h(t) = 0, t<0         h(t) ≠ 0 for some t<0
```

---

## 4.1.6 Step Response from Impulse Response

The **step response** $s(t)$ is the output when the input is $u(t)$:

$$s(t) = h(t) * u(t) = \int_{-\infty}^{t} h(\tau) d\tau$$

Conversely:

$$h(t) = \frac{ds(t)}{dt}$$

**Example**: If $h(t) = e^{-t}u(t)$:

$$s(t) = \int_{0}^{t} e^{-\tau} d\tau = (1 - e^{-t})u(t)$$

```
Impulse Response:         Step Response:
h(t) = e⁻ᵗu(t)          s(t) = (1-e⁻ᵗ)u(t)
    │                         │
  1 ┤╲                    1 ┤─────────────
    │ ╲                      │      ╱
    │  ╲╲                    │    ╱
    │    ╲╲                  │  ╱
  0 ┤──────╲─→ t          0 ┤╱────────→ t
    0  1  2  3              0  1  2  3

  h = ds/dt                 s = ∫h dt
```

---

## 4.1.7 More Worked Examples

### Example 5: $y(t) = x(t) * x(t)$ — Not LTI!

This system is time-invariant but **not linear** (convolution with itself ≠ convolution with fixed $h$). The impulse response concept applies **only to LTI systems**.

### Example 6: $y[n] = \frac{1}{3}(x[n] + x[n-1] + x[n-2])$

$$h[n] = \frac{1}{3}\delta[n] + \frac{1}{3}\delta[n-1] + \frac{1}{3}\delta[n-2]$$

$$h[n] = \frac{1}{3}\{1, 1, 1\}, \quad n = 0, 1, 2$$

This is a 3-point **moving average filter** (FIR, length 3).

### Example 7: $y(t) = \int_{t-T}^{t} x(\tau)d\tau$

Using $x(t) = \delta(t)$:

$$h(t) = \int_{t-T}^{t} \delta(\tau)d\tau$$

$\delta(\tau)$ contributes only at $\tau = 0$, which is in $[t-T, t]$ when $0 \leq t \leq T$:

$$h(t) = \begin{cases} 1, & 0 \leq t \leq T \\ 0, & \text{otherwise} \end{cases}$$

This is a rectangular pulse of width $T$.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Impulse response | $h(t)$ = output when input is $\delta(t)$ |
| Why it matters | Completely characterizes any LTI system |
| Finding $h$ | Set $x = \delta$, solve for $y$ |
| Output via $h$ | $y(t) = x(t) * h(t)$ |
| FIR | Finite-length $h$ — always stable |
| IIR | Infinite-length $h$ — may or may not be stable |
| Step response | $s(t) = \int_{-\infty}^{t} h(\tau)d\tau$ |
| Inverse relation | $h(t) = ds(t)/dt$ |

---

## Quick Revision Questions

1. **What is the impulse response of $y(t) = 5x(t-3)$?**
   - $h(t) = 5\delta(t-3)$.

2. **Can a non-LTI system have an impulse response?**
   - It can have a response to $\delta(t)$, but it won't characterize the system fully — you can't use convolution.

3. **Is $h[n] = (2)^n u[n]$ representing a stable system?**
   - No. $\sum|(2)^n| = \infty$.

4. **Find the step response if $h[n] = \delta[n] - \delta[n-1]$.**
   - $s[n] = u[n] * (\delta[n] - \delta[n-1]) = u[n] - u[n-1] = \delta[n]$.

5. **What type of system has $h[n] = \{1, -2, 1\}$?**
   - FIR (finite impulse response), length 3. Also causal and stable.

---

[← Previous: Unit 3 — Memory and Memoryless](../03-Systems/06-memory-and-memoryless-systems.md) | [Next: Convolution Integral and Sum →](02-convolution-integral-and-sum.md)
