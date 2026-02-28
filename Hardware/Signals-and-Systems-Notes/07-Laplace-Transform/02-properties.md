# Chapter 7.2: Properties of Laplace Transform

## Overview

The Laplace Transform properties enable us to find transforms of complex signals from simpler known pairs, solve differential equations algebraically, and analyze systems in the $s$-domain. Each property also specifies how the ROC is affected.

---

## 7.2.1 Property Table

| Property | Time Domain | $s$-Domain | ROC |
|----------|-------------|------------|-----|
| **Linearity** | $ax_1(t) + bx_2(t)$ | $aX_1(s) + bX_2(s)$ | At least $R_1 \cap R_2$ |
| **Time Shifting** | $x(t - t_0)$ | $e^{-st_0} X(s)$ | Same as $R$ |
| **$s$-Domain Shifting** | $e^{s_0 t}x(t)$ | $X(s - s_0)$ | $R + \text{Re}(s_0)$ |
| **Time Scaling** | $x(at)$ | $\frac{1}{|a|}X(s/a)$ | $aR$ |
| **Time Reversal** | $x(-t)$ | $X(-s)$ | $-R$ |
| **Differentiation** | $\frac{d}{dt}x(t)$ | $sX(s)$ | At least $R$ |
| **Integration** | $\int_{-\infty}^{t} x(\tau)d\tau$ | $\frac{1}{s}X(s)$ | At least $R \cap \{\sigma > 0\}$ |
| **Convolution** | $x_1(t) * x_2(t)$ | $X_1(s) \cdot X_2(s)$ | At least $R_1 \cap R_2$ |
| **Multiplication** | $x_1(t) \cdot x_2(t)$ | $\frac{1}{2\pi j}X_1(s) * X_2(s)$ | At least $R_1 + R_2$ |
| **$t$-Domain Mult.** | $t \cdot x(t)$ | $-\frac{d}{ds}X(s)$ | Same as $R$ |
| **Initial Value** | $x(0^+)$ | $\lim_{s\to\infty} sX(s)$ | — |
| **Final Value** | $x(\infty)$ | $\lim_{s\to 0} sX(s)$ | — |

---

## 7.2.2 Linearity

$$\mathcal{L}\{ax_1(t) + bx_2(t)\} = aX_1(s) + bX_2(s)$$

- ROC: at least $R_1 \cap R_2$ (may be larger if poles cancel)

### Example

$$x(t) = 3e^{-2t}u(t) - 5e^{-4t}u(t)$$

$$X(s) = \frac{3}{s+2} - \frac{5}{s+4}, \quad \text{ROC: } \sigma > -2$$

---

## 7.2.3 Time Shifting

$$x(t - t_0) \xleftrightarrow{\mathcal{L}} e^{-st_0}X(s)$$

- Multiplies by a complex exponential in $s$
- ROC unchanged

### Example: Delayed step

$$u(t - 3) \xleftrightarrow{\mathcal{L}} \frac{e^{-3s}}{s}, \quad \text{ROC: } \sigma > 0$$

```
Time shift property:

x(t):             x(t - t₀):
  ┌───            ───┐  ┌───
  │                   │  │
──┘───→ t        ─────┘──┘───→ t
  0                   t₀

s-domain: multiply by e^(-st₀) (delay factor)
```

---

## 7.2.4 $s$-Domain Shifting (Frequency Shifting)

$$e^{s_0 t}x(t) \xleftrightarrow{\mathcal{L}} X(s - s_0)$$

- ROC shifts by $\text{Re}(s_0)$

### Example

$$e^{-3t}\cos(\omega_0 t)u(t) = e^{-3t} \cdot \cos(\omega_0 t)u(t)$$

Using $\cos(\omega_0 t)u(t) \leftrightarrow s/(s^2 + \omega_0^2)$, replace $s$ with $s + 3$:

$$X(s) = \frac{s+3}{(s+3)^2 + \omega_0^2}, \quad \text{ROC: } \sigma > -3$$

---

## 7.2.5 Time Scaling

$$x(at) \xleftrightarrow{\mathcal{L}} \frac{1}{|a|}X\left(\frac{s}{a}\right)$$

- For $a > 0$: time compression → frequency expansion
- For $a = -1$: time reversal → $X(-s)$, ROC reflected

---

## 7.2.6 Differentiation in Time

$$\frac{dx(t)}{dt} \xleftrightarrow{\mathcal{L}} sX(s)$$

### Unilateral LT (with initial conditions)

$$\mathcal{L}\left\{\frac{dx}{dt}\right\} = sX(s) - x(0^-)$$

$$\mathcal{L}\left\{\frac{d^2x}{dt^2}\right\} = s^2 X(s) - sx(0^-) - x'(0^-)$$

### General $n$th-order derivative

$$\mathcal{L}\left\{\frac{d^n x}{dt^n}\right\} = s^nX(s) - s^{n-1}x(0^-) - s^{n-2}x'(0^-) - \cdots - x^{(n-1)}(0^-)$$

> This is the key property that converts differential equations to algebraic equations!

---

## 7.2.7 Integration in Time

$$\int_{0^-}^{t} x(\tau)\,d\tau \xleftrightarrow{\mathcal{L}} \frac{X(s)}{s}$$

### Example

$$\int_0^t e^{-a\tau}d\tau = \frac{1-e^{-at}}{a}$$

Using integration property:

$$\mathcal{L}\left\{\int_0^t e^{-a\tau}d\tau\right\} = \frac{1}{s} \cdot \frac{1}{s+a} = \frac{1}{s(s+a)}$$

---

## 7.2.8 Convolution Property

$$x_1(t) * x_2(t) \xleftrightarrow{\mathcal{L}} X_1(s) \cdot X_2(s)$$

This is the most important property for system analysis:

$$Y(s) = H(s) \cdot X(s)$$

```
CONVOLUTION IN TIME = MULTIPLICATION IN s-DOMAIN:

 x(t) ──→ [ h(t) ] ──→ y(t) = x(t)*h(t)
 
 X(s) ──→ [ H(s) ] ──→ Y(s) = X(s)·H(s)
 
Finding output:
1. Transform x(t) → X(s)
2. Multiply Y(s) = H(s)·X(s)
3. Inverse transform Y(s) → y(t)
```

### Example

$x(t) = e^{-t}u(t)$, $h(t) = e^{-2t}u(t)$

$$Y(s) = \frac{1}{s+1} \cdot \frac{1}{s+2} = \frac{1}{(s+1)(s+2)}$$

Partial fractions:

$$Y(s) = \frac{1}{s+1} - \frac{1}{s+2}$$

$$y(t) = (e^{-t} - e^{-2t})u(t)$$

---

## 7.2.9 Multiplication by $t$ (Differentiation in $s$)

$$t \cdot x(t) \xleftrightarrow{\mathcal{L}} -\frac{dX(s)}{ds}$$

$$t^n x(t) \xleftrightarrow{\mathcal{L}} (-1)^n \frac{d^n X(s)}{ds^n}$$

### Example

$$te^{-at}u(t): \quad -\frac{d}{ds}\left(\frac{1}{s+a}\right) = \frac{1}{(s+a)^2}$$

$$t^2 e^{-at}u(t): \quad \frac{d^2}{ds^2}\left(\frac{1}{s+a}\right) = \frac{2}{(s+a)^3}$$

---

## 7.2.10 Initial and Final Value Theorems

### Initial Value Theorem

$$x(0^+) = \lim_{s \to \infty} sX(s)$$

**Conditions**: $x(t)$ must be causal and contain no impulses at $t = 0$.

### Final Value Theorem

$$\lim_{t \to \infty} x(t) = \lim_{s \to 0} sX(s)$$

**Conditions**: All poles of $sX(s)$ must be in the **left half-plane** (system must be stable).

### Example

$$X(s) = \frac{5(s+2)}{s(s+1)(s+3)}$$

**Initial value**:

$$x(0^+) = \lim_{s\to\infty} \frac{5s(s+2)}{s(s+1)(s+3)} = \lim_{s\to\infty} \frac{5(s+2)}{(s+1)(s+3)} = \frac{5}{1} \cdot \frac{1}{1} = 0$$

Wait — degree of numerator (1) < degree of denominator (2), so:

$$x(0^+) = 0$$

**Final value**: Poles of $sX(s) = \frac{5(s+2)}{(s+1)(s+3)}$ are at $s = -1, -3$ (both in LHP ✓):

$$x(\infty) = \lim_{s\to 0} \frac{5(s+2)}{(s+1)(s+3)} = \frac{5 \cdot 2}{1 \cdot 3} = \frac{10}{3}$$

---

## 7.2.11 Unilateral LT Properties for Solving ODEs

For the unilateral Laplace Transform with initial conditions:

| Operation | $s$-Domain |
|-----------|------------|
| $x(t)$ | $X(s)$ |
| $x'(t)$ | $sX(s) - x(0^-)$ |
| $x''(t)$ | $s^2X(s) - sx(0^-) - x'(0^-)$ |
| $\int_0^t x(\tau)d\tau$ | $X(s)/s$ |

### Solving a Differential Equation

$$\frac{d^2 y}{dt^2} + 5\frac{dy}{dt} + 6y = 2u(t), \quad y(0^-) = 1, \; y'(0^-) = 0$$

**Step 1**: Take Laplace Transform

$$s^2Y(s) - s(1) - 0 + 5[sY(s) - 1] + 6Y(s) = \frac{2}{s}$$

**Step 2**: Solve for $Y(s)$

$$Y(s)(s^2 + 5s + 6) = \frac{2}{s} + s + 5$$

$$Y(s) = \frac{s^2 + 5s + 2}{s(s+2)(s+3)}$$

**Step 3**: Partial fractions

$$Y(s) = \frac{A}{s} + \frac{B}{s+2} + \frac{C}{s+3}$$

$A = \frac{0+0+2}{(2)(3)} = \frac{1}{3}$, $B = \frac{4-10+2}{(-2)(1)} = 2$, $C = \frac{9-15+2}{(-3)(-1)} = -\frac{4}{3}$

**Step 4**: Inverse transform

$$y(t) = \left(\frac{1}{3} + 2e^{-2t} - \frac{4}{3}e^{-3t}\right)u(t)$$

---

## Summary Table

| Property | Transform | ROC Effect |
|----------|-----------|------------|
| Linearity | $aX_1 + bX_2$ | $R_1 \cap R_2$ (at least) |
| Time shift | $e^{-st_0}X(s)$ | Same ROC |
| $s$-shift | $X(s - s_0)$ | ROC shifts by $\text{Re}(s_0)$ |
| Time diff. | $sX(s)$ (bilateral) | At least same |
| Integration | $X(s)/s$ | Adds pole at 0 |
| Convolution | $X_1(s)X_2(s)$ | $R_1 \cap R_2$ |
| $t \cdot x(t)$ | $-dX/ds$ | Same ROC |
| Initial value | $\lim_{s\to\infty} sX(s)$ | Causal signals only |
| Final value | $\lim_{s\to 0} sX(s)$ | Poles of $sX(s)$ in LHP |

---

## Quick Revision Questions

1. **Find $\mathcal{L}\{te^{-3t}u(t)\}$ using properties.**
   - Using $t \cdot x(t) \leftrightarrow -dX/ds$: $-\frac{d}{ds}\frac{1}{s+3} = \frac{1}{(s+3)^2}$, ROC: $\sigma > -3$.

2. **Find $\mathcal{L}\{e^{-2t}\cos(5t)u(t)\}$.**
   - $s$-shift of $\cos 5t$: replace $s \to s+2$: $\frac{s+2}{(s+2)^2+25}$, ROC: $\sigma > -2$.

3. **Use the initial value theorem on $X(s) = \frac{3s+1}{s^2+4s+5}$.**
   - $x(0^+) = \lim_{s\to\infty} \frac{3s^2+s}{s^2+4s+5} = 3$.

4. **How does the differentiation property help solve ODEs?**
   - It converts $d^n y/dt^n$ to $s^n Y(s)$ minus initial condition terms, turning the ODE into an algebraic equation in $Y(s)$.

5. **What happens to the ROC after convolution?**
   - ROC is at least $R_1 \cap R_2$, and can be larger if poles cancel.

6. **Apply the final value theorem to $X(s) = \frac{10}{s(s+5)}$.**
   - $x(\infty) = \lim_{s\to 0} \frac{10}{s+5} = 2$. (Pole of $sX(s)$ at $s=-5$ is in LHP ✓)

---

[← Previous: Definition and ROC](01-definition-and-roc.md) | [Next: Inverse Laplace Transform →](03-inverse-laplace.md)
