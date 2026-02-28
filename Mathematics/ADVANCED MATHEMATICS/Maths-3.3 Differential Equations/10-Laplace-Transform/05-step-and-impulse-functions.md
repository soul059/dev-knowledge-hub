# Chapter 10.5: Step and Impulse Functions

[← Previous: Convolution Theorem](04-convolution-theorem.md) | [Back to README](../README.md)

---

## Overview

Real-world systems often experience sudden switches, shocks, and discontinuous inputs. The **Heaviside step function** models sudden on/off changes, while the **Dirac delta function** models instantaneous impulses. Together with the Laplace transform, they make solving such problems elegant and systematic.

---

## 1. Heaviside Step Function $u(t - a)$

$$u(t - a) = \begin{cases} 0, & t < a \\ 1, & t \geq a \end{cases}$$

```
  HEAVISIDE STEP FUNCTION u(t - a)

   1 │              ┌──────────────
     │              │
     │              │
   0 │──────────────┘
     └──────────────┼──────────── t
                    a
```

### Laplace Transform

$$\boxed{\mathcal{L}\{u(t-a)\} = \frac{e^{-as}}{s}, \quad a \geq 0}$$

### Building Piecewise Functions

Any piecewise function can be written using step functions:

$$f(t) = \begin{cases} f_1(t), & 0 \leq t < a \\ f_2(t), & a \leq t < b \\ f_3(t), & t \geq b \end{cases}$$

$$= f_1(t) + [f_2(t) - f_1(t)]\,u(t-a) + [f_3(t) - f_2(t)]\,u(t-b)$$

### Example: Rectangular Pulse

$$\text{Pulse}(t) = u(t-a) - u(t-b) = \begin{cases} 1, & a \leq t < b \\ 0, & \text{otherwise} \end{cases}$$

```
  RECTANGULAR PULSE

   1 │     ┌──────────┐
     │     │          │
     │     │          │
   0 │─────┘          └───── t
         a            b
```

$$\mathcal{L}\{\text{Pulse}\} = \frac{e^{-as} - e^{-bs}}{s}$$

---

## 2. Second Shifting Theorem (t-shift)

$$\boxed{\mathcal{L}\{f(t-a)\,u(t-a)\} = e^{-as}F(s)}$$

$$\boxed{\mathcal{L}^{-1}\{e^{-as}F(s)\} = f(t-a)\,u(t-a)}$$

```
  SECOND SHIFT THEOREM — VISUAL

  f(t)                      f(t-a)·u(t-a)
  │  ╱╲                     │       ╱╲
  │ ╱  ╲                    │      ╱  ╲
  │╱    ╲                   │     ╱    ╲
  ────────╲── t             │────╱──────╲── t
  0                         0   a
  
  Original                  Delayed by a, zero before a
  
  ℒ{f(t)} = F(s)           ℒ{f(t-a)u(t-a)} = e^{-as}F(s)
```

### Example

$$\mathcal{L}\{(t-3)^2 u(t-3)\} = e^{-3s} \cdot \frac{2}{s^3}$$

$$\mathcal{L}^{-1}\left\{\frac{e^{-2s}}{s^2+1}\right\} = \sin(t-2)\,u(t-2)$$

---

## 3. Dirac Delta Function $\delta(t - a)$

$$\delta(t-a) = \begin{cases} 0, & t \neq a \\ \text{"}\infty\text{"}, & t = a \end{cases}, \quad \int_{-\infty}^{\infty}\delta(t-a)\,dt = 1$$

```
  DIRAC DELTA δ(t - a)

     │
     │         ↑ (∞)
     │         │
     │         │
   0 │─────────┼────────── t
               a

  Zero everywhere except at t = a
  Total area = 1
  Models an instantaneous impulse
```

### Key Property (Sifting)

$$\int_0^{\infty} f(t)\,\delta(t-a)\,dt = f(a), \quad a > 0$$

### Laplace Transform

$$\boxed{\mathcal{L}\{\delta(t-a)\} = e^{-as}}$$

Special case: $\mathcal{L}\{\delta(t)\} = 1$

### Relation to Step Function

$$\delta(t-a) = \frac{d}{dt}u(t-a) \quad \text{(distributional derivative)}$$

$$u(t-a) = \int_{-\infty}^t \delta(\tau - a)\,d\tau$$

---

## 4. Solving ODEs with Discontinuous Forcing

### Example 1: Switched-On Input

$$y' + 2y = u(t-3), \quad y(0) = 0$$

$sY + 2Y = \dfrac{e^{-3s}}{s}$

$$Y = \frac{e^{-3s}}{s(s+2)} = e^{-3s}\left[\frac{1/2}{s} - \frac{1/2}{s+2}\right]$$

$$\boxed{y(t) = \frac{1}{2}\left(1 - e^{-2(t-3)}\right)u(t-3)}$$

```
  SOLUTION y(t)

     │
  0.5│                    ───────────
     │                 ╱
     │                ╱
   0 │────────────────
     └──────────────┼────────────── t
                    3
```

### Example 2: Impulse Response

$$y'' + 4y = \delta(t - \pi), \quad y(0) = 0, \quad y'(0) = 0$$

$s^2Y + 4Y = e^{-\pi s}$

$$Y = \frac{e^{-\pi s}}{s^2 + 4}$$

$$\boxed{y(t) = \frac{1}{2}\sin 2(t-\pi)\,u(t-\pi) = -\frac{1}{2}\sin 2t\,\cdot\,u(t-\pi)}$$

(Using $\sin 2(t-\pi) = \sin(2t - 2\pi) = \sin 2t$; but with the delay: actually $= \frac{1}{2}\sin 2(t - \pi) \cdot u(t-\pi)$.)

### Example 3: Piecewise Forcing

$$y'' + y = \begin{cases} t, & 0 \leq t < 1 \\ 0, & t \geq 1 \end{cases}, \quad y(0) = y'(0) = 0$$

Rewrite: $g(t) = t - t\,u(t-1) = t - [(t-1)+1]\,u(t-1) = t - (t-1)u(t-1) - u(t-1)$

$$G(s) = \frac{1}{s^2} - \frac{e^{-s}}{s^2} - \frac{e^{-s}}{s}$$

$Y = G(s) \cdot \dfrac{1}{s^2 + 1}$

Applying partial fractions to $\dfrac{1}{s^2(s^2+1)} = \dfrac{1}{s^2} - \dfrac{1}{s^2+1}$:

$$y(t) = (t - \sin t) - [(t-1) - \sin(t-1)]u(t-1) - [1 - \cos(t-1)]u(t-1)$$

---

## 5. Physical Interpretations

| Function | Physical Meaning | Example |
|----------|-----------------|---------|
| $u(t)$ | Switch turned on at $t = 0$ | Voltage source activated |
| $u(t-a)$ | Switch turned on at $t = a$ | Delayed relay |
| $u(t-a) - u(t-b)$ | On from $a$ to $b$ | Timed pulse |
| $\delta(t)$ | Unit impulse at $t = 0$ | Hammer blow, voltage spike |
| $\delta(t-a)$ | Unit impulse at $t = a$ | Delayed hammer blow |
| $A\delta(t-a)$ | Impulse of magnitude $A$ at $t = a$ | Collision |

---

## 6. Summary of Transforms

| Function | $\mathcal{L}$ |
|----------|---------------|
| $u(t-a)$ | $\dfrac{e^{-as}}{s}$ |
| $f(t-a)u(t-a)$ | $e^{-as}F(s)$ |
| $\delta(t-a)$ | $e^{-as}$ |
| $\delta(t)$ | $1$ |
| $t^n u(t)$ | $\dfrac{n!}{s^{n+1}}$ |
| $f(t)u(t-a)$ | $e^{-as}\mathcal{L}\{f(t+a)\}$ |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Heaviside** $u(t-a)$ | Step: 0 for $t<a$, 1 for $t \geq a$ |
| **Dirac** $\delta(t-a)$ | Impulse: zero everywhere, area 1 |
| **Second shift** | $\mathcal{L}\{f(t-a)u(t-a)\} = e^{-as}F(s)$ |
| **$\delta$ transform** | $\mathcal{L}\{\delta(t-a)\} = e^{-as}$ |
| **Sifting property** | $\int f(t)\delta(t-a)\,dt = f(a)$ |
| **Piecewise inputs** | Build from step functions |
| **Impulse response** | $h = \mathcal{L}^{-1}\{H(s)\}$, system's DNA |

---

## Quick Revision Questions

1. Write $f(t) = \begin{cases} 0, & t < 2 \\ 3, & 2 \leq t < 5 \\ -1, & t \geq 5 \end{cases}$ using Heaviside functions and find its Laplace transform.

2. Solve $y' + y = \delta(t - 1)$, $y(0) = 0$.

3. Find $\mathcal{L}^{-1}\left\{\dfrac{se^{-\pi s}}{s^2 + 4}\right\}$.

4. A spring-mass system $y'' + 9y = 5\delta(t - 2)$, $y(0) = 0$, $y'(0) = 0$ receives a hammer blow at $t = 2$. Find $y(t)$.

5. Express the **ramp function** $r(t-a) = (t-a)u(t-a)$ and find its Laplace transform.

6. Show that $\displaystyle\int_0^{\infty} e^{-3t}\delta(t-2)\,dt = e^{-6}$ using the sifting property.

---

[← Previous: Convolution Theorem](04-convolution-theorem.md) | [Back to README](../README.md)
