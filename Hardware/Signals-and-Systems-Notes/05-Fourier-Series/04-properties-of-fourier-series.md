# Chapter 5.4: Properties of Fourier Series

## Overview

The Fourier Series has a rich set of properties that allow manipulation in the frequency domain without computing coefficients from scratch. These properties parallel (and foreshadow) those of the Fourier Transform.

---

## 5.4.1 Notation

$$x(t) \xleftrightarrow{\text{FS}} c_n \quad \text{(FS pair, period } T, \omega_0 = 2\pi/T\text{)}$$

---

## 5.4.2 Linearity

$$\alpha x(t) + \beta y(t) \xleftrightarrow{\text{FS}} \alpha c_n + \beta d_n$$

> FS of a linear combination = same linear combination of individual FS coefficients.

Both signals must have the **same period** $T$.

---

## 5.4.3 Time Shifting

$$x(t - t_0) \xleftrightarrow{\text{FS}} c_n \, e^{-jn\omega_0 t_0}$$

> A time shift multiplies coefficients by a phase factor. **Magnitude spectrum unchanged!**

$$|c_n e^{-jn\omega_0 t_0}| = |c_n|, \quad \angle(c_n e^{-jn\omega_0 t_0}) = \angle c_n - n\omega_0 t_0$$

```
TIME SHIFT EFFECT:

Magnitude: |cₙ|              Phase: ∠cₙ
  │  ↑  ↑  ↑  ↑               │  ↑  ↑  ↑  ↑    Original
  ┼──┴──┴──┴──┴──→ n         ┼──┴──┴──┴──┴──→ n

  │  ↑  ↑  ↑  ↑               │     ↑        After shift
  ┼──┴──┴──┴──┴──→ n         ┼──↑──┴──↑──↑──→ n
  
  SAME magnitude!             Phase rotated by -nω₀t₀
```

---

## 5.4.4 Frequency Shifting (Modulation)

$$x(t) \, e^{jm\omega_0 t} \xleftrightarrow{\text{FS}} c_{n-m}$$

> Multiplication by $e^{jm\omega_0 t}$ shifts the spectrum by $m$ positions.

**Application — Modulation by cosine**:

$$x(t)\cos(\omega_0 t) \xleftrightarrow{\text{FS}} \frac{1}{2}c_{n-1} + \frac{1}{2}c_{n+1}$$

---

## 5.4.5 Time Reversal

$$x(-t) \xleftrightarrow{\text{FS}} c_{-n}$$

For real signals: $c_{-n} = c_n^*$, so $x(-t) \leftrightarrow c_n^*$.

---

## 5.4.6 Time Scaling

$$x(\alpha t) \text{ has period } T/|\alpha|$$

The coefficients $c_n$ themselves don't change, but the fundamental frequency becomes $|\alpha|\omega_0$.

---

## 5.4.7 Differentiation

$$\frac{dx(t)}{dt} \xleftrightarrow{\text{FS}} jn\omega_0 \, c_n$$

Higher derivatives:

$$\frac{d^k x(t)}{dt^k} \xleftrightarrow{\text{FS}} (jn\omega_0)^k \, c_n$$

> Differentiation emphasizes high frequencies (large $|n|$) and suppresses DC.

---

## 5.4.8 Integration

$$\int_{-\infty}^{t} x(\tau)d\tau \xleftrightarrow{\text{FS}} \frac{c_n}{jn\omega_0} \quad (n \neq 0, \text{ and } c_0 = 0)$$

> Integration emphasizes low frequencies and suppresses high frequencies.

Note: Integration is defined only if $c_0 = 0$ (the average of the signal must be zero for the integral to be periodic).

---

## 5.4.9 Conjugation

$$x^*(t) \xleftrightarrow{\text{FS}} c_{-n}^*$$

For real $x(t)$: $x^*(t) = x(t)$, confirming $c_{-n}^* = c_n$ (conjugate symmetry).

---

## 5.4.10 Multiplication (Windowing)

$$x(t) \cdot y(t) \xleftrightarrow{\text{FS}} \sum_{k=-\infty}^{\infty} c_k \, d_{n-k}$$

> Multiplication in time → **convolution in frequency** (discrete convolution of coefficient sequences).

---

## 5.4.11 Convolution

$$\int_T x(\tau)y(t-\tau)d\tau \xleftrightarrow{\text{FS}} T \cdot c_n \cdot d_n$$

> Periodic convolution in time → **multiplication in frequency**.

This is the dual of the multiplication property.

---

## 5.4.12 Properties Summary Table

| Property | Time Domain | FS Coefficients |
|----------|-------------|-----------------|
| **Linearity** | $\alpha x + \beta y$ | $\alpha c_n + \beta d_n$ |
| **Time shift** | $x(t-t_0)$ | $c_n e^{-jn\omega_0 t_0}$ |
| **Freq shift** | $x(t)e^{jm\omega_0 t}$ | $c_{n-m}$ |
| **Time reversal** | $x(-t)$ | $c_{-n}$ |
| **Conjugate** | $x^*(t)$ | $c_{-n}^*$ |
| **Differentiation** | $dx/dt$ | $jn\omega_0 c_n$ |
| **Integration** | $\int x \, dt$ | $c_n/(jn\omega_0)$, $c_0=0$ |
| **Multiplication** | $x(t) \cdot y(t)$ | $\sum c_k d_{n-k}$ |
| **Periodic conv** | $\int_T x(\tau)y(t-\tau)d\tau$ | $T \cdot c_n \cdot d_n$ |
| **Real signal** | $x(t) \in \mathbb{R}$ | $c_{-n} = c_n^*$ |
| **Even signal** | $x(t) = x(-t)$ | $c_n$ real and even |
| **Odd signal** | $x(t) = -x(-t)$ | $c_n$ purely imaginary and odd |

---

## 5.4.13 Applying Properties — Examples

### Example 1: Time-Shifted Square Wave

Square wave: $x(t) \leftrightarrow c_n = \frac{2}{jn\pi}$ (odd $n$)

Delayed by $T/4$: $x(t - T/4) \leftrightarrow c_n e^{-jn\omega_0 T/4} = c_n e^{-jn\pi/2}$

For $n = 1$: $c_1 e^{-j\pi/2} = \frac{2}{j\pi} \cdot (-j) = \frac{2}{\pi}$ (becomes real → even function!)

> Shifting a square wave by $T/4$ converts it from odd (sines) to even (cosines).

### Example 2: Derivative of Sawtooth

Sawtooth: $c_n = \frac{j}{n\pi}(-1)^n$ for $n \neq 0$.

Derivative of sawtooth = constant + periodic impulses.

FS of derivative: $jn\omega_0 \cdot c_n = jn\omega_0 \cdot \frac{j}{n\pi}(-1)^n = \frac{-\omega_0}{\pi}(-1)^n = \frac{-2}{T}(-1)^n$

All coefficients equal magnitude → the derivative contains impulses (consistent with the sawtooth having jumps).

### Example 3: Using Linearity

Given $x(t)$ with coefficients $c_n$ and $y(t) = 3x(t) + 2$:

$$d_n = \begin{cases} 3c_0 + 2 & n = 0 \\ 3c_n & n \neq 0 \end{cases}$$

---

## 5.4.14 Duality Between Time and Frequency

```
TIME DOMAIN              FREQUENCY DOMAIN
───────────              ────────────────
Shifting      ←──→       Phase rotation
Differentiating ←──→     Multiply by jnω₀
Integrating    ←──→      Divide by jnω₀
Multiplication ←──→      Convolution
Convolution    ←──→      Multiplication
Smooth signal  ←──→      Rapid c_n decay
Sharp edges    ←──→      Slow c_n decay
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Time shift | Phase rotation only; magnitude unchanged |
| Differentiation | Multiply by $jn\omega_0$ → emphasizes high freq |
| Integration | Divide by $jn\omega_0$ → emphasizes low freq |
| Multiplication | Convolution of coefficient sequences |
| Conjugate symmetry | $c_{-n} = c_n^*$ for real signals |
| Even signal | All $c_n$ real |
| Odd signal | All $c_n$ purely imaginary |
| Modulation | Spectrum shift by $m$ |

---

## Quick Revision Questions

1. **If $x(t) \leftrightarrow c_n$, what are the FS coefficients of $x(t-2)$?**
   - $c_n \cdot e^{-j2n\omega_0}$.

2. **Does a time shift change the magnitude spectrum?**
   - No. Only the phase changes.

3. **What property relates differentiation in time to the FS?**
   - $dx/dt \leftrightarrow jn\omega_0 c_n$.

4. **If $c_n$ are all real, what symmetry does $x(t)$ have?**
   - Even symmetry: $x(t) = x(-t)$.

5. **Multiplication of two periodic signals in time corresponds to what in frequency?**
   - Discrete convolution of their FS coefficient sequences.

---

[← Previous: Fourier Coefficients](03-fourier-coefficients.md) | [Next: Parseval's Theorem →](05-parsevals-theorem.md)
