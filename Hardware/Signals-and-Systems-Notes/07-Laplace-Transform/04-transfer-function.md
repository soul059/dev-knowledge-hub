# Chapter 7.4: Transfer Function and System Representation

## Overview

The **transfer function** $H(s)$ is the Laplace Transform of the impulse response $h(t)$ and the ratio $Y(s)/X(s)$ for zero initial conditions. It provides a complete algebraic description of an LTI system, enabling analysis through pole-zero plots and frequency response without solving differential equations.

---

## 7.4.1 Definition

$$H(s) = \frac{Y(s)}{X(s)} = \mathcal{L}\{h(t)\}$$

For a system described by:

$$\sum_{k=0}^{N} a_k \frac{d^k y}{dt^k} = \sum_{k=0}^{M} b_k \frac{d^k x}{dt^k}$$

$$H(s) = \frac{b_M s^M + b_{M-1}s^{M-1} + \cdots + b_0}{a_N s^N + a_{N-1}s^{N-1} + \cdots + a_0}$$

```
SYSTEM IN s-DOMAIN:

  X(s) ──→ ┌──────────┐ ──→ Y(s) = H(s)·X(s)
            │  H(s) =  │
            │  Y(s)/X(s)│
            └──────────┘

Time domain:  y(t) = x(t) * h(t)
s-domain:     Y(s) = X(s) · H(s)
```

---

## 7.4.2 Poles and Zeros

### Factored Form

$$H(s) = K\frac{(s-z_1)(s-z_2)\cdots(s-z_M)}{(s-p_1)(s-p_2)\cdots(s-p_N)}$$

| Term | Definition |
|------|-----------|
| **Zeros** ($z_i$) | Values where $H(s) = 0$ |
| **Poles** ($p_i$) | Values where $H(s) \to \infty$ |
| **Gain** ($K$) | Scaling constant |
| **Order** | Degree of denominator $N$ |

### Pole-Zero Plot

Poles marked with $\times$, zeros marked with $\circ$ on the $s$-plane.

```
Example: H(s) = (s+1)/[(s+2)(s+3)]

Pole-Zero Plot:
  jω ↑
     │
     │
     │
  ───×──×──○──────→ σ
    -3  -2  -1
     │
     │
     
  × = poles at s = -2, -3
  ○ = zero at s = -1
```

---

## 7.4.3 Transfer Function from Circuit Analysis

### Example: Series RLC Circuit

```
         R           L
  ╶──┤►►►├──┤⌒⌒⌒├──┬──╴
  +                  │
  Vᵢₙ(s)          ┌─┴─┐ 1/Cs
                   │   │
                   └─┬─┘
  ╶──────────────────┘──╴
  -            Vₒᵤₜ(s)
```

Using voltage divider in $s$-domain:

$$H(s) = \frac{V_{out}(s)}{V_{in}(s)} = \frac{1/Cs}{R + Ls + 1/Cs} = \frac{1}{LCs^2 + RCs + 1}$$

$$= \frac{1/LC}{s^2 + (R/L)s + 1/LC} = \frac{\omega_n^2}{s^2 + 2\zeta\omega_n s + \omega_n^2}$$

where $\omega_n = 1/\sqrt{LC}$ and $\zeta = \frac{R}{2}\sqrt{C/L}$.

### Pole Locations by Damping

| Condition | Poles | Response |
|-----------|-------|----------|
| $\zeta > 1$ (overdamped) | Two real, negative | Sum of decaying exponentials |
| $\zeta = 1$ (critically damped) | Repeated real, $-\omega_n$ | $(A + Bt)e^{-\omega_n t}$ |
| $0 < \zeta < 1$ (underdamped) | Complex conjugate in LHP | Damped sinusoid |
| $\zeta = 0$ (undamped) | Purely imaginary | Sustained oscillation |

```
POLE LOCATIONS VS DAMPING:

  jω ↑
     │   ζ=0
     │   ×
     │  /
     │ / ζ<1     ζ=1    ζ>1
     │×           ×      ×──→ σ
     │ \
     │  \
     │   ×
     │
  ───┼──────────────────→ σ
     │
     
  ζ=0: pure imaginary (oscillatory)
  ζ<1: complex pair (damped oscillation)
  ζ=1: repeated real (critical damping)
  ζ>1: distinct real (overdamped)
```

---

## 7.4.4 System Properties from $H(s)$

### Causality

A system is **causal** if the ROC of $H(s)$ is a **right half-plane** (right of the rightmost pole).

### Stability (BIBO)

A system is **BIBO stable** if the ROC of $H(s)$ includes the **$j\omega$ axis**.

### Causal + Stable

$$\text{All poles of } H(s) \text{ in the open left half-plane: } \text{Re}(p_i) < 0$$

```
STABILITY FROM POLE LOCATIONS:

STABLE (all poles in LHP):      UNSTABLE (pole in RHP):
  jω ↑                           jω ↑
     │                              │
  ───×──×──────→ σ               ───×──────×──→ σ
     │                              │
     │                              │
  All poles left of jω axis      Pole right of jω axis

MARGINALLY STABLE:              UNSTABLE (repeated on jω):
  jω ↑                           jω ↑
     ×                              ×  (double)
     │                              │
  ───┼──×──────→ σ               ───┼──────→ σ
     │                              │
     ×                              ×  (double)
  Simple poles on jω axis        Repeated poles on jω axis
```

---

## 7.4.5 Block Diagram Algebra in $s$-Domain

### Cascade

$$H(s) = H_1(s) \cdot H_2(s)$$

### Parallel

$$H(s) = H_1(s) + H_2(s)$$

### Negative Feedback

$$H(s) = \frac{G(s)}{1 + G(s)F(s)}$$

### Positive Feedback

$$H(s) = \frac{G(s)}{1 - G(s)F(s)}$$

```
NEGATIVE FEEDBACK:

  R(s) ──(+)──→ [ G(s) ] ──┬──→ Y(s)
          ↑-                │
          │                 │
          └── [ F(s) ] ←────┘
          
  Y(s)/R(s) = G(s) / [1 + G(s)F(s)]
```

---

## 7.4.6 Frequency Response from Transfer Function

The **frequency response** is obtained by evaluating $H(s)$ on the $j\omega$ axis:

$$H(j\omega) = H(s)\big|_{s = j\omega}$$

$$|H(j\omega)| = \text{magnitude response}$$

$$\angle H(j\omega) = \text{phase response}$$

### Example: First-Order LPF

$$H(s) = \frac{a}{s + a} \quad \Rightarrow \quad H(j\omega) = \frac{a}{j\omega + a}$$

$$|H(j\omega)| = \frac{a}{\sqrt{\omega^2 + a^2}}, \quad \angle H(j\omega) = -\arctan(\omega/a)$$

---

## 7.4.7 Standard Second-Order System

$$H(s) = \frac{\omega_n^2}{s^2 + 2\zeta\omega_n s + \omega_n^2}$$

### Step Response Characteristics

| Parameter | Formula |
|-----------|---------|
| Rise time | $t_r \approx \frac{1.8}{\omega_n}$ |
| Peak time | $t_p = \frac{\pi}{\omega_d}$ where $\omega_d = \omega_n\sqrt{1-\zeta^2}$ |
| Overshoot | $M_p = e^{-\pi\zeta/\sqrt{1-\zeta^2}} \times 100\%$ |
| Settling time (2%) | $t_s \approx \frac{4}{\zeta\omega_n}$ |

```
STEP RESPONSE OF UNDERDAMPED 2ND-ORDER:

y(t) ↑
     │        Mp (overshoot)
     │       ╱╲
  1.0│ ─ ─ ╱──╲─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ steady state
     │   ╱     ╲    ╱─╲
     │  ╱       ╲──╱   ╲──────────────
     │ ╱                
     │╱
  ───┼────┬───┬──────────┬──────→ t
     0    tᵣ  tₚ         tₛ
     
  tᵣ = rise time    tₚ = peak time
  tₛ = settling time  Mp = peak overshoot
```

---

## 7.4.8 Converting Between Representations

```
EQUIVALENT SYSTEM DESCRIPTIONS:

Differential Equation
    ↕  (take LT / inverse LT)
Transfer Function H(s) = N(s)/D(s)
    ↕  (inverse LT)
Impulse Response h(t)
    ↕  (evaluate at s = jω)
Frequency Response H(jω)
    ↕  (factor N(s), D(s))
Pole-Zero Plot
```

| From | To | Method |
|------|----|--------|
| ODE → $H(s)$ | Replace $d^n/dt^n$ with $s^n$ | Zero initial conditions |
| $H(s)$ → $h(t)$ | Inverse LT via PFE | See Chapter 7.3 |
| $H(s)$ → $H(j\omega)$ | Substitute $s = j\omega$ | If $j\omega \in$ ROC |
| $H(s)$ → poles/zeros | Factor $N(s)$, $D(s)$ | Find roots |
| Poles/zeros → $H(s)$ | Multiply factors + gain | $H(s) = K\frac{\prod(s-z_i)}{\prod(s-p_i)}$ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Transfer function | $H(s) = Y(s)/X(s) = \mathcal{L}\{h(t)\}$ |
| Poles | $D(s) = 0$; determine stability and response shape |
| Zeros | $N(s) = 0$; shape frequency response |
| Causal + Stable | All poles in open LHP |
| Feedback TF | $G/(1+GF)$ (negative), $G/(1-GF)$ (positive) |
| 2nd-order params | $\omega_n$ (natural freq), $\zeta$ (damping ratio) |
| Freq. response | $H(j\omega) = H(s)\|_{s=j\omega}$ |

---

## Quick Revision Questions

1. **Find $H(s)$ for $\frac{d^2y}{dt^2} + 5\frac{dy}{dt} + 6y = 2\frac{dx}{dt} + x$.**
   - $H(s) = \frac{2s+1}{s^2+5s+6}$. Poles at $s=-2,-3$; zero at $s=-1/2$.

2. **Is a system with $H(s) = \frac{1}{s-3}$ BIBO stable (causal)?**
   - No. Causal ROC is $\sigma > 3$, which does not include $j\omega$ axis. Pole at $s = 3$ is in RHP.

3. **What determines the natural response shape of a system?**
   - The pole locations: real poles → exponentials, complex poles → damped sinusoids.

4. **For $H(s) = \frac{10}{(s+1)(s+10)}$, find the DC gain.**
   - $H(0) = 10/(1 \times 10) = 1$.

5. **What is the transfer function of two systems in cascade?**
   - $H(s) = H_1(s) \cdot H_2(s)$.

6. **Given $\omega_n = 10$, $\zeta = 0.5$, find the damped frequency and percent overshoot.**
   - $\omega_d = 10\sqrt{1-0.25} = 8.66$ rad/s. $M_p = e^{-0.5\pi/\sqrt{0.75}} \times 100 \approx 16.3\%$.

---

[← Previous: Inverse Laplace](03-inverse-laplace.md) | [Next: System Analysis →](05-system-analysis.md)
