# Chapter 4.3: Properties of LTI Systems

## Overview

LTI systems enjoy a rich set of algebraic properties that make analysis tractable. Convolution is commutative, associative, and distributive, and every system property (causality, stability, invertibility) can be determined from the impulse response alone.

---

## 4.3.1 Algebraic Properties of Convolution

### Commutativity

$$x(t) * h(t) = h(t) * x(t)$$

> The order in which signals are convolved does not matter.

```
x * h  =  h * x

┌────┐   ┌────┐     ┌────┐   ┌────┐
│ x  │──→│ h  │──→  │ h  │──→│ x  │──→  Same output
└────┘   └────┘     └────┘   └────┘
```

**Proof**: Substituting $\sigma = t - \tau$ in the integral:
$$\int_{-\infty}^{\infty} x(\tau)h(t-\tau)d\tau = \int_{-\infty}^{\infty} h(\sigma)x(t-\sigma)d\sigma$$

### Associativity

$$(x * h_1) * h_2 = x * (h_1 * h_2)$$

> Cascading the system order does not matter.

```
Method 1:                    Method 2:
┌────┐   ┌────┐   ┌────┐    ┌────┐   ┌──────────┐
│ x  │──→│ h₁ │──→│ h₂ │    │ x  │──→│ h₁ * h₂  │
└────┘   └────┘   └────┘    └────┘   └──────────┘
    Same result               Combine first, convolve once
```

### Distributivity

$$x * (h_1 + h_2) = x * h_1 + x * h_2$$

> Parallel systems can be combined by adding impulse responses.

```
         ┌────┐
    ┌───→│ h₁ │───┐              ┌──────────┐
    │    └────┘   │              │           │
x ──┤             ⊕──→ y   =  x─→│ h₁ + h₂  │──→ y
    │    ┌────┐   │              │           │
    └───→│ h₂ │───┘              └──────────┘
         └────┘
```

---

## 4.3.2 Identity and Null Elements

### Identity Element

$$x(t) * \delta(t) = x(t)$$

> The impulse $\delta$ is the identity for convolution.

The "pass-through" system has $h(t) = \delta(t)$ — output equals input.

### Zero/Null Element

$$x(t) * 0 = 0$$

> Convolving with zero gives zero output.

---

## 4.3.3 Derivative and Integral Properties

### Derivative Property

$$\frac{d}{dt}[x(t) * h(t)] = \frac{dx}{dt} * h(t) = x(t) * \frac{dh}{dt}$$

> Differentiation can be applied to **either** signal in the convolution.

**Useful technique**: To convolve complex signals, differentiate first to simplify, convolve, then integrate.

### Integral Property

$$\int_{-\infty}^{t} [x * h](\lambda) d\lambda = \left[\int_{-\infty}^{t} x(\lambda)d\lambda \right] * h(t) = x(t) * \left[\int_{-\infty}^{t} h(\lambda)d\lambda \right]$$

---

## 4.3.4 System Properties via Impulse Response

### Causality

An LTI system is **causal** if and only if:

$$h(t) = 0 \quad \text{for } t < 0$$

```
CAUSAL h(t):                NON-CAUSAL h(t):
    │                           │
    │╲                     ╱    │╲
    │ ╲                   ╱     │ ╲
  ──┼──╲──→ t           ──┼─────╲──→ t
    0                    -2  0
    h = 0 for t < 0       h ≠ 0 for some t < 0
```

### BIBO Stability

An LTI system is **BIBO stable** if and only if:

$$\int_{-\infty}^{\infty} |h(t)| \, dt < \infty \quad \text{(CT)}$$

$$\sum_{n=-\infty}^{\infty} |h[n]| < \infty \quad \text{(DT)}$$

### Invertibility

An LTI system is **invertible** if there exists $h_i(t)$ such that:

$$h(t) * h_i(t) = \delta(t)$$

$h_i(t)$ is the impulse response of the **inverse system**.

```
┌────┐   ┌─────┐
│ h  │──→│ h_i │──→  δ(t)   (identity)
└────┘   └─────┘

System followed by its inverse recovers the input:
x ──→ [h] ──→ [h_i] ──→ x
```

**Example**: Accumulator $h[n] = u[n]$ is invertible.
- Inverse: first-difference $h_i[n] = \delta[n] - \delta[n-1]$
- Check: $u[n] * (\delta[n] - \delta[n-1]) = u[n] - u[n-1] = \delta[n]$ ✓

### Memoryless

An LTI system is **memoryless** if and only if:

$$h(t) = K\delta(t) \quad \text{for some constant } K$$

---

## 4.3.5 Causal + Stable LTI

For a **causal LTI** system with $h(t) = 0$ for $t < 0$:

Stability reduces to:

$$\int_{0}^{\infty} |h(t)| dt < \infty$$

| $h(t)$ | Causal? | Stable? |
|---------|---------|---------|
| $e^{-3t}u(t)$ | ✓ | ✓ ($\int = 1/3$) |
| $e^{3t}u(t)$ | ✓ | ✗ ($\int = \infty$) |
| $e^{-3|t|}$ | ✗ | ✓ ($\int = 2/3$) |
| $e^{3t}u(-t)$ | ✗ | ✓ ($\int = 1/3$) |
| $(0.8)^n u[n]$ | ✓ | ✓ ($\sum = 5$) |
| $u[n]$ | ✓ | ✗ ($\sum = \infty$) |

---

## 4.3.6 Properties Summary — Convolution Algebra

| Property | CT Formula | DT Formula |
|----------|-----------|-----------|
| **Commutative** | $x * h = h * x$ | $x * h = h * x$ |
| **Associative** | $(x * h_1)*h_2 = x*(h_1*h_2)$ | same |
| **Distributive** | $x*(h_1+h_2) = x*h_1 + x*h_2$ | same |
| **Identity** | $x * \delta = x$ | $x * \delta = x$ |
| **Shift** | $x * \delta(t-t_0) = x(t-t_0)$ | $x * \delta[n-n_0] = x[n-n_0]$ |
| **Derivative** | $(x*h)' = x'*h = x*h'$ | N/A |
| **Scaling** | $ax * h = a(x*h)$ | $ax * h = a(x*h)$ |
| **Width** | Duration$(y)$ = Duration$(x)$ + Duration$(h)$ | Length$(y)$ = Length$(x)$ + Length$(h) - 1$ |
| **Area** | Area$(y)$ = Area$(x) \times$ Area$(h)$ | Sum$(y)$ = Sum$(x) \times$ Sum$(h)$ |

---

## 4.3.7 Eigenfunctions of LTI Systems

Complex exponentials are **eigenfunctions** of all LTI systems:

$$x(t) = e^{st} \implies y(t) = H(s) \cdot e^{st}$$

where $H(s) = \int_{-\infty}^{\infty} h(\tau) e^{-s\tau} d\tau$ is the **transfer function** (Laplace transform of $h$).

For DT:

$$x[n] = z^n \implies y[n] = H(z) \cdot z^n$$

where $H(z) = \sum_{n=-\infty}^{\infty} h[n] z^{-n}$ is the **Z-transform** of $h$.

> The system scales the eigenfunction by $H(s)$ or $H(z)$ — no change in functional form.

```
EIGENFUNCTION PROPERTY:

e^(st) ──→ [LTI: h(t)] ──→ H(s)·e^(st)

  Input shape = Output shape
  Only the AMPLITUDE and PHASE change
  
  This is WHY frequency-domain analysis works!
```

---

## 4.3.8 Worked Examples

### Example 1: Area Rule Verification

$x = \{1, 1, 1\}$ and $h = \{1, 2\}$.

- $y = x * h = \{1, 3, 3, 2\}$
- Area$(x) = 3$, Area$(h) = 3$, Area$(y) = 9 = 3 \times 3$ ✓

### Example 2: Using Derivative Property

Find $y(t) = r(t) * r(t)$ where $r(t) = tu(t)$ (ramp).

Direct convolution is complex. Instead:

$$r'(t) = u(t), \quad r''(t) = \delta(t)$$

$$y''(t) = r''(t) * r(t) = \delta(t) * r(t) = r(t) = tu(t)$$

Integrate twice:

$$y'(t) = \frac{t^2}{2}u(t), \quad y(t) = \frac{t^3}{6}u(t)$$

$$\boxed{r(t) * r(t) = \frac{t^3}{6}u(t)}$$

### Example 3: Cascade Simplification

System: $x \to h_1(t) = e^{-t}u(t) \to h_2(t) = 2\delta(t-1) \to y$

Using associativity: $h_{eq} = h_1 * h_2 = e^{-t}u(t) * 2\delta(t-1) = 2e^{-(t-1)}u(t-1)$

One convolution instead of two sequential ones.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Commutativity | $x*h = h*x$ — flip either signal |
| Associativity | Cascade order doesn't matter |
| Distributivity | Parallel sum of impulse responses |
| Identity | $\delta$ is the identity element |
| Eigenfunction | Complex exponentials pass through unchanged in shape |
| Causality | $h(t) = 0, t < 0$ |
| Stability | $\int\|h\| < \infty$ |
| Memoryless | $h = K\delta$ |
| Invertibility | $h * h_i = \delta$ |

---

## Quick Revision Questions

1. **What is the identity element for convolution?**
   - The Dirac delta $\delta(t)$ (CT) or $\delta[n]$ (DT).

2. **Can two unstable LTI systems in cascade be stable?**
   - Yes. Example: $h_1 = u(t)$ (integrator, unstable) cascaded with $h_2 = \delta(t) - \delta(t-\epsilon)/\epsilon$ (differentiator) gives approximately $\delta(t)$.

3. **State the eigenfunction property for CT-LTI systems.**
   - If $x(t) = e^{st}$, then $y(t) = H(s)e^{st}$ where $H(s) = \mathcal{L}\{h(t)\}$.

4. **An LTI system has $h(t) = \delta(t) + \delta(t+1)$. Is it causal?**
   - No. $h(-1) = \delta(0) = 1 \neq 0$, so $h(t) \neq 0$ for $t < 0$.

5. **If $x = \{2, 3\}$ and $h = \{1, 1\}$, verify the area rule.**
   - $y = \{2, 5, 3\}$. Area$(x) = 5$, Area$(h) = 2$, Area$(y) = 10 = 5 \times 2$ ✓.

---

[← Previous: Convolution Integral and Sum](02-convolution-integral-and-sum.md) | [Next: System Interconnection →](04-system-interconnection.md)
