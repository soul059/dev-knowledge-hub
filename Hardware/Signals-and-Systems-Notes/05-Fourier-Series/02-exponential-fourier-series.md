# Chapter 5.2: Exponential Fourier Series

## Overview

The **exponential (complex) Fourier Series** uses complex exponentials $e^{jn\omega_0 t}$ as basis functions. This form is mathematically more compact and elegant than the trigonometric form, and it connects directly to the Fourier Transform and frequency-domain analysis.

---

## 5.2.1 The Representation

$$x(t) = \sum_{n=-\infty}^{\infty} c_n \, e^{jn\omega_0 t}$$

Where:

$$c_n = \frac{1}{T} \int_{T} x(t) \, e^{-jn\omega_0 t} \, dt$$

- $n = 0$: DC component ($c_0 = a_0$)
- $n > 0$: positive frequency harmonics
- $n < 0$: negative frequency harmonics (mathematical construct)
- $c_n$ are **complex** in general

```
ANALYSIS:                          SYNTHESIS:
                 ┌─────┐
x(t) ──→ Compute │ c_n │          c_n ──→ Sum eʲⁿω₀ᵗ ──→ x(t)
              cₙ │     │
                 └─────┘
                 
Time domain → Frequency domain → Time domain
```

---

## 5.2.2 Relationship to Trigonometric Form

Using Euler's formula: $\cos\theta = \frac{e^{j\theta} + e^{-j\theta}}{2}$, $\sin\theta = \frac{e^{j\theta} - e^{-j\theta}}{2j}$

$$c_0 = a_0$$

$$c_n = \frac{1}{2}(a_n - jb_n) \quad \text{for } n > 0$$

$$c_{-n} = \frac{1}{2}(a_n + jb_n) = c_n^* \quad \text{for } n > 0$$

### Conversion Table

| From | To | Formula |
|------|----|---------|
| $a_n, b_n$ → $c_n$ | $c_n = \frac{a_n - jb_n}{2}$, $c_{-n} = \frac{a_n + jb_n}{2}$ | $n > 0$ |
| $c_n$ → $a_n, b_n$ | $a_n = 2\text{Re}(c_n)$, $b_n = -2\text{Im}(c_n)$ | $n > 0$ |
| $c_n$ → $A_n, \phi_n$ | $A_n = 2|c_n|$, $\phi_n = \angle c_n$ | $n > 0$ |

---

## 5.2.3 Conjugate Symmetry

For **real-valued** signals $x(t)$:

$$c_{-n} = c_n^*$$

This means:
- $|c_{-n}| = |c_n|$ — magnitude spectrum is **even**
- $\angle c_{-n} = -\angle c_n$ — phase spectrum is **odd**

```
Magnitude spectrum (even):     Phase spectrum (odd):
  |cₙ|                          ∠cₙ
    ↑                              ↑
    │  ↑     ↑     ↑               │       ↑
    │  │  ↑  │  ↑  │               │    ↑  │
    │  │  │  │  │  │            ───┼──────────→ n
  ──┼──┴──┴──┴──┴──┴──→ n         │  ↓  │
   -3 -2 -1  0  1  2  3           │     ↓
    Even: |c₋ₙ| = |cₙ|            Odd: ∠c₋ₙ = -∠cₙ
```

---

## 5.2.4 Worked Example 1: Periodic Pulse Train

$$x(t) = \begin{cases} A, & |t| < \tau/2 \\ 0, & \tau/2 < |t| < T/2 \end{cases}, \quad \text{period } T$$

```
x(t):
  A ┤  ┌─┐       ┌─┐       ┌─┐
    │  │ │       │ │       │ │
  0 ┤──┘ └───────┘ └───────┘ └──→ t
   -T  -τ/2  0  τ/2   T
          ←τ→
        ←────T────→
```

**Compute $c_n$**:

$$c_n = \frac{1}{T} \int_{-\tau/2}^{\tau/2} A \, e^{-jn\omega_0 t} dt = \frac{A}{T} \cdot \frac{e^{-jn\omega_0 t}}{-jn\omega_0}\Bigg|_{-\tau/2}^{\tau/2}$$

$$= \frac{A}{T} \cdot \frac{2\sin(n\omega_0\tau/2)}{n\omega_0} = \frac{A\tau}{T} \cdot \frac{\sin(n\pi\tau/T)}{n\pi\tau/T}$$

$$\boxed{c_n = \frac{A\tau}{T} \, \text{sinc}\left(\frac{n\tau}{T}\right)}$$

Where $\text{sinc}(x) = \frac{\sin(\pi x)}{\pi x}$.

```
Magnitude spectrum |cₙ| (sinc envelope):

|cₙ|
  ↑
  │    ╱╲
  │   ╱  ╲
  │  ╱    ╲        ╱╲        ╱╲
  │ ╱      ╲  ↑↑  ╱  ╲  ↑↑ ╱  ╲
  ┼╱────↑↑──╲╱──╲╱────╲╱──╲╱────╲→ n
 ... -T/τ    0      T/τ  2T/τ ...
  
  ↑ = samples at integer n
  Envelope = Aτ/T × |sinc(nτ/T)|
  First null at n = T/τ
```

### Special Case: $\tau = T/4$ (duty cycle = 25%)

$$c_n = \frac{A}{4}\text{sinc}\left(\frac{n}{4}\right)$$

$c_0 = A/4$, first null at $n = 4$.

---

## 5.2.5 Worked Example 2: Square Wave (Exponential Form)

From the trigonometric result: $b_n = 4/(n\pi)$ for odd $n$, $a_n = 0$:

$$c_n = \frac{-jb_n}{2} = \frac{-2j}{n\pi} \quad \text{(odd } n\text{)}$$

$$c_n = \begin{cases} \frac{-2j}{n\pi} & n \text{ odd} \\ 0 & n \text{ even, } n \neq 0 \end{cases}$$

$$|c_n| = \frac{2}{|n|\pi}, \quad \angle c_n = \begin{cases} -\pi/2 & n > 0, \text{ odd} \\ +\pi/2 & n < 0, \text{ odd} \end{cases}$$

---

## 5.2.6 Worked Example 3: Complex Exponential Input

What if $x(t) = e^{j\omega_0 t}$ (already a single exponential)?

$$c_1 = 1, \quad c_n = 0 \text{ for } n \neq 1$$

Only one nonzero coefficient — a single spectral line at $\omega = \omega_0$.

---

## 5.2.7 Line Spectra

The Fourier coefficients $c_n$ exist only at discrete frequencies $n\omega_0$. When plotted:

- **Magnitude spectrum**: $|c_n|$ vs $n$ (or $n\omega_0$)
- **Phase spectrum**: $\angle c_n$ vs $n$

These are called **line spectra** — the signal's frequency content at specific harmonics.

```
Example: Full-wave rectified sine

Magnitude |cₙ|:               Phase ∠cₙ:
    ↑                            ↑
    │  ↑                      π ┤
    │  │                        │
    │  │  ↑                   0 ┤  ↑     ↑  ↑
    │  │  │  ↑                  │  │  ↑  │  │
  ──┼──┴──┴──┴──→ n           ──┼──┴──┴──┴──┴──→ n
    0  2  4  6                   0  2  4  6
    (only even harmonics due to half-period symmetry)
```

---

## 5.2.8 DT Fourier Series (DTFS)

For a periodic DT signal $x[n]$ with period $N$:

$$x[n] = \sum_{k=0}^{N-1} c_k \, e^{j(2\pi k/N)n}$$

$$c_k = \frac{1}{N} \sum_{n=0}^{N-1} x[n] \, e^{-j(2\pi k/N)n}$$

> Only $N$ coefficients ($k = 0, 1, \ldots, N-1$) — finite and exact (no convergence issues).

**Example**: $x[n] = \{1, 0, 1, 0\}$ with $N = 4$:

$$c_0 = \frac{1}{4}(1+0+1+0) = \frac{1}{2}$$

$$c_1 = \frac{1}{4}(1+0\cdot e^{-j\pi/2}+1\cdot e^{-j\pi}+0) = \frac{1}{4}(1+(-1)) = 0$$

$$c_2 = \frac{1}{4}(1+0+1\cdot e^{-j2\pi}+0) = \frac{1}{4}(1+1) = \frac{1}{2}$$

$$c_3 = 0$$

$$x[n] = \frac{1}{2} + \frac{1}{2}e^{j\pi n} = \frac{1}{2}(1 + (-1)^n)$$ ✓

---

## 5.2.9 Comparison: Trigonometric vs Exponential

| Feature | Trigonometric | Exponential |
|---------|--------------|-------------|
| Basis | $\cos(n\omega_0 t), \sin(n\omega_0 t)$ | $e^{jn\omega_0 t}$ |
| Index range | $n = 0, 1, 2, \ldots$ | $n = -\infty, \ldots, \infty$ |
| Coefficients | Real: $a_0, a_n, b_n$ | Complex: $c_n$ |
| Negative freq | Not explicit | Explicit ($n < 0$) |
| Compactness | Less compact | More compact |
| Physical intuition | Direct (real sinusoids) | Abstract (complex) |
| Transform connection | Indirect | Direct → Fourier Transform |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Exponential FS | $x(t) = \sum c_n e^{jn\omega_0 t}$ |
| Coefficient | $c_n = \frac{1}{T}\int_T x(t)e^{-jn\omega_0 t}dt$ |
| Conjugate symmetry | $c_{-n} = c_n^*$ for real signals |
| Pulse train | $c_n = \frac{A\tau}{T}\text{sinc}(n\tau/T)$ |
| Line spectrum | Discrete frequencies at $n\omega_0$ |
| DTFS | $N$ coefficients, finite sum, exact |
| Relation to trig | $c_n = (a_n - jb_n)/2$ for $n > 0$ |

---

## Quick Revision Questions

1. **Express $\cos(\omega_0 t)$ using complex exponentials and identify $c_n$.**
   - $\cos\omega_0 t = \frac{1}{2}e^{j\omega_0 t} + \frac{1}{2}e^{-j\omega_0 t}$. So $c_1 = c_{-1} = 1/2$, all others zero.

2. **What is the conjugate symmetry property?**
   - For real $x(t)$: $c_{-n} = c_n^*$, meaning $|c_{-n}| = |c_n|$ and $\angle c_{-n} = -\angle c_n$.

3. **How many DTFS coefficients exist for a signal with period $N = 8$?**
   - Exactly 8 coefficients: $c_0, c_1, \ldots, c_7$.

4. **What is $c_0$ for a periodic signal?**
   - The DC (average) value: $c_0 = a_0 = \frac{1}{T}\int_T x(t)dt$.

5. **A pulse train has duty cycle $\tau/T = 1/5$. At which harmonic does the first null occur?**
   - At $n = T/\tau = 5$.

---

[← Previous: Trigonometric Fourier Series](01-trigonometric-fourier-series.md) | [Next: Fourier Coefficients →](03-fourier-coefficients.md)
