# Chapter 1.4: Standard Signals (Unit Step, Impulse, Ramp, Exponential)

## Overview

Standard (elementary) signals are the fundamental building blocks in signals and systems theory. Every complex signal can be expressed as a combination of these basic signals. They also serve as test inputs to characterize system behavior.

---

## 1.4.1 Unit Step Function

### Continuous-Time Unit Step: $u(t)$

$$u(t) = \begin{cases} 1, & t > 0 \\ 0, & t < 0 \end{cases}$$

> At $t = 0$, the value is undefined (or defined as $1/2$ in some conventions).

```
u(t):
    │
  1 ┤        ┌──────────────────
    │        │
    │        │
  0 ┤────────┤─────────────────→ t
    │        ↑
          t = 0

    "Switches ON at t = 0"
```

### Discrete-Time Unit Step: $u[n]$

$$u[n] = \begin{cases} 1, & n \geq 0 \\ 0, & n < 0 \end{cases}$$

```
u[n]:
    │
  1 ┤        ↑  ↑  ↑  ↑  ↑  ↑
    │        │  │  │  │  │  │
  0 ┼──┼──┼──┼──┼──┼──┼──┼──┼──→ n
    -3 -2 -1  0  1  2  3  4  5
```

### Properties of Unit Step

| Property | Value |
|----------|-------|
| Causal? | Yes |
| Energy | $\infty$ |
| Power | $1/2$ |
| Type | Power signal |
| Bounded? | Yes ($M = 1$) |

### Applications

- **Switching signals on/off**: $x(t) \cdot u(t)$ makes $x(t)$ causal
- **Window functions**: $u(t) - u(t - T)$ creates a rectangular pulse of width $T$
- **Circuit analysis**: Models switch closure at $t = 0$

```
Rectangular pulse using step functions:

p(t) = u(t) - u(t - T)

  1 ┤  ┌──────────┐
    │  │          │
  0 ┤──┘          └──────────→ t
    │  ↑          ↑
       0          T
```

---

## 1.4.2 Unit Impulse Function

### Continuous-Time Unit Impulse (Dirac Delta): $\delta(t)$

$$\delta(t) = \begin{cases} \infty, & t = 0 \\ 0, & t \neq 0 \end{cases}$$

with the constraint:

$$\int_{-\infty}^{\infty} \delta(t) \, dt = 1$$

> $\delta(t)$ is not a function in the ordinary sense — it is a **distribution** (generalized function).

```
δ(t):
    │
    │     ↑ (1)          ← Area = 1 (height is ∞, width is 0)
    │     │
    │     │
  0 ┤─────┤──────────────→ t
    │     ↑
          t = 0

    Arrow notation: ↑(1) means impulse with area/strength = 1
```

### Discrete-Time Unit Impulse (Kronecker Delta): $\delta[n]$

$$\delta[n] = \begin{cases} 1, & n = 0 \\ 0, & n \neq 0 \end{cases}$$

```
δ[n]:
    │
  1 ┤        ↑
    │        │
  0 ┼──┼──┼──┼──┼──┼──┼──→ n
    -3 -2 -1  0  1  2  3

    Exactly 1 at n = 0, zero everywhere else
```

### Key Properties of the Impulse

#### 1. Sifting (Sampling) Property

**Continuous**:
$$\int_{-\infty}^{\infty} x(t) \, \delta(t - t_0) \, dt = x(t_0)$$

**Discrete**:
$$\sum_{n=-\infty}^{\infty} x[n] \, \delta[n - n_0] = x[n_0]$$

> The impulse "sifts out" the value of $x$ at $t = t_0$.

#### 2. Relation to Unit Step

**Continuous**:
$$u(t) = \int_{-\infty}^{t} \delta(\tau) \, d\tau \qquad \delta(t) = \frac{du(t)}{dt}$$

**Discrete**:
$$u[n] = \sum_{k=-\infty}^{n} \delta[k] \qquad \delta[n] = u[n] - u[n-1]$$

#### 3. Scaling Property (CT Only)

$$\delta(at) = \frac{1}{|a|} \delta(t)$$

#### 4. Product Property

$$x(t) \cdot \delta(t - t_0) = x(t_0) \cdot \delta(t - t_0)$$

### Signal Decomposition Using Impulses

Any discrete-time signal can be written as a sum of shifted impulses:

$$x[n] = \sum_{k=-\infty}^{\infty} x[k] \, \delta[n - k]$$

**Example**: $x[n] = \{2, \underline{3}, 1\}$ (values at $n = -1, 0, 1$)

$$x[n] = 2\delta[n+1] + 3\delta[n] + 1\delta[n-1]$$

```
x[n] = 2δ[n+1] + 3δ[n] + δ[n-1]:

  3 ┤     ↑
  2 ┤  ↑
  1 ┤        ↑
  0 ┼──┼──┼──┼──→ n
    -1  0  1
```

---

## 1.4.3 Unit Ramp Function

### Continuous-Time Ramp: $r(t)$

$$r(t) = t \cdot u(t) = \begin{cases} t, & t \geq 0 \\ 0, & t < 0 \end{cases}$$

```
r(t):
    │                    ╱
  3 ┤                  ╱
    │                ╱
  2 ┤              ╱
    │            ╱
  1 ┤          ╱
    │        ╱
  0 ┤───────╱─────────────→ t
    │       ↑
         t = 0      Slope = 1
```

### Discrete-Time Ramp: $r[n]$

$$r[n] = n \cdot u[n] = \begin{cases} n, & n \geq 0 \\ 0, & n < 0 \end{cases}$$

```
r[n]:
    │
  4 ┤                    ↑
  3 ┤              ↑     │
  2 ┤        ↑     │     │
  1 ┤  ↑     │     │     │
  0 ┼──┼──┼──┼─────┼─────┼──→ n
    -1  0  1  2    3     4
```

### Relationships Between Standard Signals

$$r(t) = \int_{-\infty}^{t} u(\tau) \, d\tau \qquad u(t) = \frac{dr(t)}{dt}$$

**Complete chain** (CT):

$$\delta(t) \xrightarrow{\text{integrate}} u(t) \xrightarrow{\text{integrate}} r(t) \xrightarrow{\text{integrate}} \frac{t^2}{2}u(t)$$

$$\frac{t^2}{2}u(t) \xrightarrow{\text{differentiate}} r(t) \xrightarrow{\text{differentiate}} u(t) \xrightarrow{\text{differentiate}} \delta(t)$$

---

## 1.4.4 Exponential Signals

### Real Exponential (CT)

$$x(t) = Ae^{at}$$

| Condition | Behavior | Type |
|-----------|----------|------|
| $a > 0$ | Growing exponential | Unbounded |
| $a < 0$ | Decaying exponential | Energy signal |
| $a = 0$ | Constant ($= A$) | Power signal |

```
Decaying (a < 0):               Growing (a > 0):
    │                                │         ╱
  A ┤╲                             A ┤        ╱
    │ ╲                              │       ╱
    │  ╲                             │      ╱
    │   ╲╲                           │    ╱
    │     ╲╲                         │  ╱
    │       ╲╲╲                      │╱
  0 ┤─────────╲╲╲──→ t           0 ┤────────────→ t
    0   τ   2τ  3τ                   0   τ   2τ  3τ

    τ = 1/|a| (time constant)       Diverges to ∞
    At t = τ: x = A·e⁻¹ ≈ 0.368A
```

### Complex Exponential (CT)

$$x(t) = Ae^{st}, \quad s = \sigma + j\omega$$

$$x(t) = Ae^{\sigma t} e^{j\omega t} = Ae^{\sigma t}[\cos(\omega t) + j\sin(\omega t)]$$

| Component | Role |
|-----------|------|
| $A$ | Amplitude |
| $\sigma$ | Growth/decay rate |
| $\omega$ | Angular frequency |
| $e^{\sigma t}$ | Envelope |
| $e^{j\omega t}$ | Oscillation |

```
Damped sinusoid: x(t) = e^(-σt)·cos(ωt), σ > 0

    │
  1 ┤╲  ╭╲    ╭╲    ╭╲
    │  ╲╱  ╲─╱  ╲─╱  ╲───     ← Decaying envelope
  0 ┤──────────────────────→ t
    │  ╱╲  ╱╲  ╱╲  ╱╲
 -1 ┤╱    ╰╱    ╰╱    ╰───
    │
    Oscillation with exponentially decaying amplitude
```

### Discrete-Time Exponential

$$x[n] = A \cdot r^n \cdot u[n]$$

| Condition | Behavior |
|-----------|----------|
| $|r| < 1$ | Decays to zero |
| $|r| > 1$ | Grows without bound |
| $|r| = 1$ | Constant magnitude |
| $r > 0$ | All same sign |
| $r < 0$ | Alternating sign |

```
|r| < 1, r > 0:         |r| < 1, r < 0:
    │                        │
  1 ┤  ↑                  1 ┤  ↑
    │     ↑                  │        ↑
    │        ↑               │
  0 ┼──┼──┼──┼──┼──→ n   0 ┼──┼──┼──┼──┼──→ n
     0  1  2  3  4           │     ↑        ↑
    │                   -1  ┤
    Decays monotonically     Decays with alternating sign
```

---

## 1.4.5 Sinusoidal Signals

### Continuous-Time

$$x(t) = A\cos(\omega_0 t + \phi)$$

- $A$ = amplitude
- $\omega_0 = 2\pi f_0$ = angular frequency (rad/s)
- $f_0 = 1/T_0$ = frequency (Hz)
- $T_0$ = fundamental period (seconds)
- $\phi$ = initial phase (radians)

```
x(t) = A·cos(ω₀t + φ):
    │
  A ┤     ╭──╮         ╭──╮
    │    ╱    ╲       ╱    ╲
    │   ╱      ╲     ╱      ╲
  0 ┤──╱────────╲───╱────────╲──→ t
    │            ╲ ╱          ╲
    │             ╳
 -A ┤              ╰──╯
    │
    │  |←── T₀ ──→|
    │  f₀ = 1/T₀, ω₀ = 2π/T₀
```

### Discrete-Time

$$x[n] = A\cos(\omega_0 n + \phi)$$

- Periodic only if $\omega_0/(2\pi) = p/q$ (rational)
- Fundamental period: $N = q$ (in lowest terms)

---

## 1.4.6 Signum Function

$$\text{sgn}(t) = \begin{cases} 1, & t > 0 \\ 0, & t = 0 \\ -1, & t < 0 \end{cases}$$

**Relation to step**: $\text{sgn}(t) = 2u(t) - 1$

```
sgn(t):
    │
  1 ┤         ┌─────────────
    │         │
  0 ┤─────────●─────────────→ t
    │         │
 -1 ┤─────────┘
    │       t = 0
```

---

## 1.4.7 Sinc Function

$$\text{sinc}(t) = \frac{\sin(\pi t)}{\pi t}$$

> Note: $\text{sinc}(0) = 1$ (by L'Hôpital's rule)

```
sinc(t):
    │
  1 ┤        ╭╮
    │       ╱  ╲
    │      ╱    ╲
    │     ╱      ╲
  0 ┤─╱──╱────────╲──╲─────→ t
    │╱  ╱            ╲  ╲
    │               ╰╯  ╰╯
    │  -3  -2  -1  0  1  2  3
    │
    Zero crossings at t = ±1, ±2, ±3, ...
    (all non-zero integers)
```

**Application**: Ideal low-pass filter impulse response in frequency domain.

---

## 1.4.8 Rectangular (Gate) Pulse

$$\text{rect}\left(\frac{t}{\tau}\right) = \begin{cases} 1, & |t| < \tau/2 \\ 1/2, & |t| = \tau/2 \\ 0, & |t| > \tau/2 \end{cases}$$

```
rect(t/τ):
    │
  1 ┤  ┌──────────┐
    │  │          │
  0 ┤──┘          └──────→ t
    │  ↑     ↑    ↑
    -τ/2    0    τ/2

    Width = τ, Height = 1, Area = τ
```

---

## 1.4.9 Triangular Pulse

$$\text{tri}\left(\frac{t}{\tau}\right) = \begin{cases} 1 - \frac{|t|}{\tau}, & |t| \leq \tau \\ 0, & |t| > \tau \end{cases}$$

```
tri(t/τ):
    │
  1 ┤       ╱╲
    │      ╱  ╲
    │     ╱    ╲
    │    ╱      ╲
  0 ┤───╱────────╲──────→ t
    │  -τ    0    τ

    Width = 2τ, Height = 1
```

---

## 1.4.10 Relationships Among Standard Signals

```
RELATIONSHIP DIAGRAM:

              differentiate              differentiate
    r(t)  ─────────────────►  u(t)  ─────────────────►  δ(t)
   (Ramp)                    (Step)                   (Impulse)

              integrate                  integrate
    r(t)  ◄─────────────────  u(t)  ◄─────────────────  δ(t)

Discrete-Time:
    δ[n] = u[n] - u[n-1]        (first difference)
    u[n] = Σ δ[k] for k=-∞ to n (running sum)
    r[n] = Σ u[k] for k=-∞ to n (running sum of step)
```

| Signal | Integral of | Derivative of |
|--------|-------------|---------------|
| $\delta(t)$ | — | $u(t)$ |
| $u(t)$ | $\delta(t)$ | $r(t)$ |
| $r(t)$ | $u(t)$ | $\frac{t^2}{2}u(t)$ |

---

## Summary Table

| Signal | CT Definition | DT Definition | Key Property |
|--------|--------------|---------------|--------------|
| Unit Step | $u(t) = 1$ for $t>0$ | $u[n] = 1$ for $n \geq 0$ | Switch ON |
| Unit Impulse | $\delta(t)$: area = 1 | $\delta[n] = 1$ at $n=0$ | Sifting property |
| Ramp | $r(t) = tu(t)$ | $r[n] = nu[n]$ | Linear growth |
| Exponential | $Ae^{at}$ | $Ar^n$ | Growth/decay |
| Sinusoid | $A\cos(\omega_0 t + \phi)$ | $A\cos(\omega_0 n + \phi)$ | Oscillation |
| Signum | $\text{sgn}(t) = 2u(t)-1$ | — | Sign extractor |
| Sinc | $\frac{\sin(\pi t)}{\pi t}$ | — | Ideal LPF |
| Rect pulse | 1 for $\|t\| < \tau/2$ | — | Gating |
| Tri pulse | $1 - \|t\|/\tau$ | — | Window |

---

## Quick Revision Questions

1. **What is the sifting property of the impulse function?**
   - $\int_{-\infty}^{\infty} x(t)\delta(t-t_0)dt = x(t_0)$. The impulse "extracts" the value of $x$ at $t_0$.

2. **Express the unit step $u(t)$ in terms of the impulse $\delta(t)$ and vice versa.**
   - $u(t) = \int_{-\infty}^{t}\delta(\tau)d\tau$ and $\delta(t) = du(t)/dt$.

3. **How do you create a rectangular pulse of width $T$ starting at $t = 0$ using step functions?**
   - $p(t) = u(t) - u(t-T)$.

4. **For the DT exponential $x[n] = (0.5)^n u[n]$, does the signal decay or grow?**
   - Decays, because $|r| = 0.5 < 1$.

5. **What is the value of $\text{sinc}(0)$?**
   - $\text{sinc}(0) = 1$ (by L'Hôpital's rule or the limit definition).

6. **Express $x[n] = \{4, \underline{2}, 5\}$ as a sum of shifted impulses.**
   - $x[n] = 4\delta[n+1] + 2\delta[n] + 5\delta[n-1]$.

---

[← Previous: Basic Operations on Signals](03-basic-operations-on-signals.md) | [Next: Even and Odd Signals →](05-even-and-odd-signals.md)
