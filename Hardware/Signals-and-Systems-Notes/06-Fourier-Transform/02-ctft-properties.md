# Chapter 6.2: CTFT Properties

## Overview

The properties of the Fourier Transform allow signal manipulation in the frequency domain without recomputing the integral. These properties form the foundation for system analysis, filtering, and modulation.

---

## 6.2.1 Complete Properties Table

| Property | Time Domain | Frequency Domain |
|----------|-------------|------------------|
| **Linearity** | $ax(t)+by(t)$ | $aX(j\omega)+bY(j\omega)$ |
| **Time shift** | $x(t-t_0)$ | $X(j\omega)e^{-j\omega t_0}$ |
| **Freq shift** | $x(t)e^{j\omega_0 t}$ | $X(j(\omega-\omega_0))$ |
| **Time scaling** | $x(at)$ | $\frac{1}{\|a\|}X\left(\frac{j\omega}{a}\right)$ |
| **Time reversal** | $x(-t)$ | $X(-j\omega) = X^*(j\omega)$ (real $x$) |
| **Conjugation** | $x^*(t)$ | $X^*(-j\omega)$ |
| **Differentiation (t)** | $dx/dt$ | $j\omega X(j\omega)$ |
| **Integration** | $\int_{-\infty}^{t}x(\tau)d\tau$ | $\frac{X(j\omega)}{j\omega}+\pi X(0)\delta(\omega)$ |
| **Differentiation (ω)** | $(-jt)x(t)$ | $dX/d\omega$ |
| **Convolution** | $x(t)*h(t)$ | $X(j\omega)\cdot H(j\omega)$ |
| **Multiplication** | $x(t)\cdot y(t)$ | $\frac{1}{2\pi}X(j\omega)*Y(j\omega)$ |
| **Duality** | $X(jt)$ | $2\pi x(-\omega)$ |
| **Parseval's** | $\int\|x\|^2 dt$ | $\frac{1}{2\pi}\int\|X\|^2 d\omega$ |

---

## 6.2.2 Key Properties — Detailed

### Linearity

$$ax(t) + by(t) \xleftrightarrow{\mathcal{F}} aX(j\omega) + bY(j\omega)$$

Superposition holds — the FT of a sum is the sum of FTs.

### Time Shifting

$$x(t - t_0) \xleftrightarrow{\mathcal{F}} X(j\omega) \cdot e^{-j\omega t_0}$$

> A time delay adds a **linear phase** $-\omega t_0$ to the spectrum. **Magnitude unchanged!**

```
TIME SHIFT EFFECT:

|X(jω)|: unchanged        ∠X(jω): linear phase added
  │  ╱╲                     │ ╱
  │ ╱  ╲                    │╱         slope = -t₀
  ┼╱────╲──→ ω           ───┼──────→ ω
                             │╲
                             │ ╲
```

### Frequency Shifting (Modulation)

$$x(t) \cdot e^{j\omega_0 t} \xleftrightarrow{\mathcal{F}} X(j(\omega - \omega_0))$$

> Multiplying by a complex exponential shifts the spectrum by $\omega_0$.

**Modulation by cosine** (AM modulation):

$$x(t)\cos(\omega_c t) \leftrightarrow \frac{1}{2}[X(j(\omega-\omega_c)) + X(j(\omega+\omega_c))]$$

```
MODULATION:

X(jω) baseband:            x(t)·cos(ωct):
    ╱╲                         ╱╲     ╱╲
   ╱  ╲                       ╱  ╲   ╱  ╲
  ╱    ╲                     ╱    ╲ ╱    ╲
 ╱──────╲──→ ω            ──╱──────╲╱──────╲──→ ω
  -W  0  W               -ωc    0    ωc
  
  Baseband                Shifted to ±ωc (carrier)
```

### Time Scaling

$$x(at) \xleftrightarrow{\mathcal{F}} \frac{1}{|a|} X\left(\frac{j\omega}{a}\right)$$

- $|a| > 1$: compression in time → expansion in frequency
- $|a| < 1$: expansion in time → compression in frequency

> **Time-bandwidth product** is conserved.

---

## 6.2.3 The Convolution Theorem

$$y(t) = x(t) * h(t) \xleftrightarrow{\mathcal{F}} Y(j\omega) = X(j\omega) \cdot H(j\omega)$$

> **Convolution in time = multiplication in frequency** — the single most important property!

```
TIME DOMAIN:                  FREQUENCY DOMAIN:

x(t) ──→ [h(t)] ──→ y(t)    X(jω) × H(jω) = Y(jω)

Convolution (hard)            Multiplication (easy!)

  ┌────────────────────────────────────────────┐
  │ To find output: Y = X · H                  │
  │ Much easier than computing convolution!     │
  └────────────────────────────────────────────┘
```

### Dual: Multiplication in Time

$$x(t) \cdot y(t) \xleftrightarrow{\mathcal{F}} \frac{1}{2\pi} X(j\omega) * Y(j\omega)$$

> Multiplication in time = convolution in frequency (with $1/2\pi$ factor).

---

## 6.2.4 Parseval's Theorem (Energy)

$$E = \int_{-\infty}^{\infty} |x(t)|^2 \, dt = \frac{1}{2\pi} \int_{-\infty}^{\infty} |X(j\omega)|^2 \, d\omega$$

- $|X(j\omega)|^2$ is the **energy spectral density** (ESD)
- Energy in frequency band $[\omega_1, \omega_2]$: $E_{[\omega_1,\omega_2]} = \frac{1}{2\pi}\int_{\omega_1}^{\omega_2}|X|^2 d\omega$

---

## 6.2.5 Differentiation Properties

### Time Differentiation

$$\frac{d^n x}{dt^n} \xleftrightarrow{\mathcal{F}} (j\omega)^n X(j\omega)$$

> Differentiation boosts high frequencies by factor $\omega$.

**Application**: Solving differential equations algebraically:

$$\frac{d^2y}{dt^2} + 3\frac{dy}{dt} + 2y = x(t) \implies [(j\omega)^2 + 3j\omega + 2]Y = X$$

$$H(j\omega) = \frac{Y}{X} = \frac{1}{(j\omega)^2 + 3j\omega + 2}$$

### Frequency Differentiation

$$(-jt)^n x(t) \xleftrightarrow{\mathcal{F}} \frac{d^n X}{d\omega^n}$$

**Application**: Finding the FT of $te^{-at}u(t)$:

$$e^{-at}u(t) \leftrightarrow \frac{1}{a+j\omega}$$

$$te^{-at}u(t) = (-j)^{-1}\frac{d}{d\omega}\frac{1}{a+j\omega} \cdot (-j)$$

$$te^{-at}u(t) \leftrightarrow \frac{1}{(a+j\omega)^2}$$

---

## 6.2.6 Symmetry Properties for Real Signals

If $x(t)$ is real:

$$X(-j\omega) = X^*(j\omega)$$

This means:
- $|X(-j\omega)| = |X(j\omega)|$ — even magnitude
- $\angle X(-j\omega) = -\angle X(j\omega)$ — odd phase
- $\text{Re}\{X\}$ is even, $\text{Im}\{X\}$ is odd

Additionally:
- Real + even $x(t)$ → $X(j\omega)$ is real and even
- Real + odd $x(t)$ → $X(j\omega)$ is purely imaginary and odd

---

## 6.2.7 Worked Examples

### Example 1: FT of $x(t) = e^{-2|t|}$

$$X(j\omega) = \int_{-\infty}^{0} e^{2t}e^{-j\omega t}dt + \int_0^{\infty} e^{-2t}e^{-j\omega t}dt$$

$$= \frac{1}{2 - j\omega} + \frac{1}{2 + j\omega} = \frac{4}{4 + \omega^2}$$

### Example 2: Using Convolution Theorem

Find the output of $h(t) = e^{-3t}u(t)$ with input $x(t) = e^{-2t}u(t)$:

$$H(j\omega) = \frac{1}{3+j\omega}, \quad X(j\omega) = \frac{1}{2+j\omega}$$

$$Y(j\omega) = \frac{1}{(2+j\omega)(3+j\omega)} = \frac{1}{2+j\omega} - \frac{1}{3+j\omega}$$

$$y(t) = (e^{-2t} - e^{-3t})u(t)$$

### Example 3: Energy via Parseval's

For $x(t) = e^{-3t}u(t)$:

Time domain: $E = \int_0^{\infty} e^{-6t}dt = 1/6$

Frequency domain: $E = \frac{1}{2\pi}\int_{-\infty}^{\infty}\frac{1}{9+\omega^2}d\omega = \frac{1}{2\pi}\cdot\frac{\pi}{3} = \frac{1}{6}$ ✓

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Convolution theorem | $x*h \leftrightarrow X\cdot H$ (most important!) |
| Time shift | Phase change only; $\|X\|$ unchanged |
| Frequency shift | Spectrum translation (modulation) |
| Time scaling | Compress time → expand freq (and vice versa) |
| Differentiation | Multiply by $j\omega$ → boosts high freq |
| Integration | Divide by $j\omega$ → boosts low freq |
| Duality | rect ↔ sinc, $\delta$ ↔ 1 |
| Parseval's energy | $\int\|x\|^2 = \frac{1}{2\pi}\int\|X\|^2$ |
| Real signal symmetry | Even $\|X\|$, odd $\angle X$ |

---

## Quick Revision Questions

1. **State the convolution theorem.**
   - $x(t)*h(t) \leftrightarrow X(j\omega) \cdot H(j\omega)$.

2. **What does a time delay $t_0$ do to the spectrum?**
   - Multiplies by $e^{-j\omega t_0}$; magnitude unchanged, phase has linear term $-\omega t_0$.

3. **If $x(t) \leftrightarrow X(j\omega)$, find the FT of $x(3t)$.**
   - $\frac{1}{3}X(j\omega/3)$.

4. **Multiplication in time corresponds to what in frequency?**
   - Convolution (scaled by $1/2\pi$).

5. **Why is the convolution theorem so important for LTI systems?**
   - It converts convolution (difficult integration) to simple multiplication in the frequency domain.

---

[← Previous: CTFT Definition](01-ctft.md) | [Next: Frequency Response →](03-frequency-response.md)
