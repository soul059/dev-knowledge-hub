# Chapter 6.1: Continuous-Time Fourier Transform (CTFT)

## Overview

The CTFT decomposes **any** (aperiodic) signal into a continuous spectrum of complex exponentials. While the Fourier Series gives discrete spectral lines for periodic signals, the Fourier Transform provides a continuous spectral density $X(j\omega)$ for aperiodic signals.

---

## 6.1.1 Definition — The Transform Pair

### Forward Transform (Analysis)

$$X(j\omega) = \int_{-\infty}^{\infty} x(t) \, e^{-j\omega t} \, dt$$

### Inverse Transform (Synthesis)

$$x(t) = \frac{1}{2\pi} \int_{-\infty}^{\infty} X(j\omega) \, e^{j\omega t} \, d\omega$$

> $X(j\omega)$ is the **spectral density** — a complex-valued function of frequency.

```
TIME DOMAIN                      FREQUENCY DOMAIN
                    CTFT
x(t) ──────────────────────→ X(jω)
                   ←──────
                   Inverse
  
  Signal              Spectrum
  (amplitude vs t)    (density vs ω)
```

---

## 6.1.2 Derivation from Fourier Series

As the period $T \to \infty$:
- Harmonics $n\omega_0$ become infinitely close → continuous spectrum
- Coefficients $c_n$ become infinitesimally small
- Sum $\sum c_n e^{jn\omega_0 t}$ → integral $\frac{1}{2\pi}\int X(j\omega)e^{j\omega t}d\omega$

The connection: $T c_n \to X(j\omega)$ and $\omega_0 \to d\omega$ and $n\omega_0 \to \omega$.

```
Fourier Series (periodic):       Fourier Transform (aperiodic):

|cₙ| (discrete lines)           |X(jω)| (continuous envelope)
  │  ↑  ↑  ↑  ↑  ↑              │  ╱╲
  │  │  │  │  │  │              │ ╱  ╲
  │  │  │  │  │  │              │╱    ╲╲
  ┼──┴──┴──┴──┴──┴──→ nω₀      ┼───────╲╲──→ ω
  
  As T → ∞, lines merge        Continuous spectral density
  into a continuous curve
```

---

## 6.1.3 Existence — Dirichlet Conditions

$X(j\omega)$ exists if:

1. $x(t)$ is absolutely integrable: $\int_{-\infty}^{\infty}|x(t)|dt < \infty$
2. Finite number of maxima, minima, and discontinuities in any finite interval

> Energy signals ($E < \infty$) always have a Fourier Transform. Power signals (like periodic signals) require generalized functions ($\delta$).

---

## 6.1.4 Common Transform Pairs

### Pair 1: Rectangular Pulse

$$x(t) = \begin{cases} A, & |t| < \tau/2 \\ 0, & |t| > \tau/2 \end{cases}$$

$$X(j\omega) = A\tau \, \text{sinc}\left(\frac{\omega\tau}{2\pi}\right) = A\tau \frac{\sin(\omega\tau/2)}{\omega\tau/2}$$

```
x(t):                         |X(jω)|:
  A ┤  ┌───────┐               Aτ ┤   ╱╲
    │  │       │                   │  ╱  ╲
  0 ┤──┘       └──→ t          0 ┤─╱╲╱──╲╱╲─→ ω
   -τ/2  0  τ/2               -4π/τ  0  4π/τ
   
   Narrow pulse → Wide spectrum (and vice versa)
```

### Pair 2: Decaying Exponential

$$x(t) = e^{-at}u(t), \quad a > 0$$

$$X(j\omega) = \frac{1}{a + j\omega}$$

$$|X(j\omega)| = \frac{1}{\sqrt{a^2 + \omega^2}}, \quad \angle X(j\omega) = -\arctan\left(\frac{\omega}{a}\right)$$

```
x(t):                      |X(jω)|:
  1 ┤╲                     1/a ┤   ╱╲
    │ ╲                         │  ╱  ╲
    │  ╲╲                       │ ╱    ╲╲
  0 ┤────╲╲──→ t            0  ┤╱───────╲╲──→ ω
    0                          -a   0    a
    Decay rate a                3dB BW = 2a
```

### Pair 3: Double-Sided Exponential

$$x(t) = e^{-a|t|}, \quad a > 0$$

$$X(j\omega) = \frac{2a}{a^2 + \omega^2}$$

### Pair 4: Impulse

$$x(t) = \delta(t) \implies X(j\omega) = 1$$

> The impulse has a **flat** (uniform) spectrum — all frequencies with equal weight.

### Pair 5: Constant

$$x(t) = 1 \implies X(j\omega) = 2\pi\delta(\omega)$$

> A constant signal has all its "energy" at $\omega = 0$ (DC).

### Pair 6: Complex Exponential

$$x(t) = e^{j\omega_0 t} \implies X(j\omega) = 2\pi\delta(\omega - \omega_0)$$

### Pair 7: Cosine

$$x(t) = \cos(\omega_0 t) \implies X(j\omega) = \pi[\delta(\omega - \omega_0) + \delta(\omega + \omega_0)]$$

```
cos(ω₀t):                     X(jω):
    ╱╲   ╱╲   ╱╲                 ↑     ↑
   ╱  ╲ ╱  ╲ ╱  ╲               │     │
  ╱────╳────╳────╲──→ t      ──┼──┬──┼──→ ω
                               -ω₀  0  ω₀
  Periodic signal               Two spectral lines
                                (impulses at ±ω₀)
```

### Pair 8: Signum Function

$$x(t) = \text{sgn}(t) \implies X(j\omega) = \frac{2}{j\omega}$$

### Pair 9: Unit Step

$$x(t) = u(t) \implies X(j\omega) = \pi\delta(\omega) + \frac{1}{j\omega}$$

---

## 6.1.5 Transform Pair Summary Table

| $x(t)$ | $X(j\omega)$ |
|---------|---------------|
| $\delta(t)$ | $1$ |
| $1$ | $2\pi\delta(\omega)$ |
| $u(t)$ | $\pi\delta(\omega) + 1/(j\omega)$ |
| $e^{-at}u(t)$, $a>0$ | $1/(a+j\omega)$ |
| $e^{-a\|t\|}$, $a>0$ | $2a/(a^2+\omega^2)$ |
| $te^{-at}u(t)$ | $1/(a+j\omega)^2$ |
| $\text{rect}(t/\tau)$ | $\tau\text{sinc}(\omega\tau/2\pi)$ |
| $\text{sinc}(Wt/\pi)$ | $(\pi/W)\text{rect}(\omega/2W)$ |
| $e^{j\omega_0 t}$ | $2\pi\delta(\omega-\omega_0)$ |
| $\cos\omega_0 t$ | $\pi[\delta(\omega-\omega_0)+\delta(\omega+\omega_0)]$ |
| $\sin\omega_0 t$ | $\frac{\pi}{j}[\delta(\omega-\omega_0)-\delta(\omega+\omega_0)]$ |
| $\text{sgn}(t)$ | $2/(j\omega)$ |

---

## 6.1.6 Key Observations

### Time-Bandwidth Product

For signals of effective duration $\Delta t$ and bandwidth $\Delta\omega$:

$$\Delta t \cdot \Delta\omega \geq 2\pi \quad \text{(uncertainty principle)}$$

> You **cannot** have a signal that is simultaneously narrow in time AND narrow in frequency.

```
Narrow pulse (Δt small):     Wide pulse (Δt large):
  │ ┌┐                        │  ┌──────────┐
  ┼─┘└───→ t                  ┼──┘          └──→ t
  
  │ ╱────╲                     │  ╱╲
  ┼╱──────╲──→ ω               ┼─╱──╲──→ ω
  Wide spectrum (Δω large)     Narrow spectrum (Δω small)
```

### Duality Observation

If $x(t) \leftrightarrow X(j\omega)$, then $X(jt) \leftrightarrow 2\pi x(-\omega)$.

Example: $\text{rect} \leftrightarrow \text{sinc}$ and $\text{sinc} \leftrightarrow \text{rect}$ (duality!).

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| CTFT definition | $X(j\omega) = \int x(t)e^{-j\omega t}dt$ |
| Inverse | $x(t) = \frac{1}{2\pi}\int X(j\omega)e^{j\omega t}d\omega$ |
| Existence | Absolutely integrable or use generalized functions |
| $\delta(t) \leftrightarrow 1$ | Impulse has flat spectrum |
| $1 \leftrightarrow 2\pi\delta(\omega)$ | Constant is pure DC |
| Rect ↔ sinc | Fundamental duality pair |
| Uncertainty | $\Delta t \cdot \Delta\omega \geq 2\pi$ |
| Periodic signals | Spectrum contains impulses in $\omega$ |

---

## Quick Revision Questions

1. **What is the CTFT of $\delta(t-3)$?**
   - $e^{-j3\omega}$ (time shift → phase rotation).

2. **What signal has a flat spectrum $X(j\omega) = 1$?**
   - $\delta(t)$ — the unit impulse.

3. **If $x(t)$ is narrower in time, what happens to $X(j\omega)$?**
   - It becomes wider (broader bandwidth).

4. **What is $X(j\omega)$ for $x(t) = e^{-3t}u(t)$?**
   - $1/(3+j\omega)$.

5. **Why does $\cos(\omega_0 t)$ have impulses in its spectrum?**
   - It's periodic (infinite energy) — the transform requires $\delta$ functions.

6. **State the duality property.**
   - If $x(t) \leftrightarrow X(j\omega)$, then $X(jt) \leftrightarrow 2\pi x(-\omega)$.

---

[← Previous: Unit 5 — Parseval's Theorem](../05-Fourier-Series/05-parsevals-theorem.md) | [Next: CTFT Properties →](02-ctft-properties.md)
