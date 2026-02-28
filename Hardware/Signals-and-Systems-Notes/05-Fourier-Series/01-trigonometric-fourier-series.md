# Chapter 5.1: Trigonometric Fourier Series

## Overview

The Trigonometric Fourier Series represents any periodic signal as a **sum of sines and cosines** at harmonically related frequencies. This is the most intuitive form of Fourier analysis since sines and cosines directly correspond to real-world oscillations.

---

## 5.1.1 The Representation

For a periodic signal $x(t)$ with period $T$ and fundamental frequency $\omega_0 = 2\pi/T$:

$$x(t) = a_0 + \sum_{n=1}^{\infty} \left[ a_n \cos(n\omega_0 t) + b_n \sin(n\omega_0 t) \right]$$

Where:
- $a_0$ = DC component (average value)
- $a_n \cos(n\omega_0 t)$ = cosine harmonics
- $b_n \sin(n\omega_0 t)$ = sine harmonics
- $n = 1$: **fundamental** frequency $\omega_0$
- $n = 2, 3, \ldots$: **harmonics** ($2\omega_0, 3\omega_0, \ldots$)

```
Signal decomposition:

x(t) = a₀   +  a₁cos(ω₀t) + b₁sin(ω₀t)  +  a₂cos(2ω₀t) + ...
       ──      ─────────────────────────      ──────────────────
        DC           1st harmonic                2nd harmonic
        │                │                          │
        ▼                ▼                          ▼
    ─────────     ╱╲    ╱╲    ╱╲          ╱╲╱╲╱╲╱╲╱╲╱╲
    constant       ╲╱    ╲╱    ╲╱          one period
```

---

## 5.1.2 Computing the Coefficients

### DC Coefficient $a_0$

$$a_0 = \frac{1}{T} \int_{0}^{T} x(t) \, dt$$

> Average value of the signal over one period.

### Cosine Coefficients $a_n$

$$a_n = \frac{2}{T} \int_{0}^{T} x(t) \cos(n\omega_0 t) \, dt, \quad n = 1, 2, 3, \ldots$$

### Sine Coefficients $b_n$

$$b_n = \frac{2}{T} \int_{0}^{T} x(t) \sin(n\omega_0 t) \, dt, \quad n = 1, 2, 3, \ldots$$

> Integration can be over **any** complete period: $[0, T]$, $[-T/2, T/2]$, etc.

### Why These Formulas Work — Orthogonality

The key property is **orthogonality** of sinusoids:

$$\int_0^T \cos(m\omega_0 t)\cos(n\omega_0 t)dt = \begin{cases} T & m = n = 0 \\ T/2 & m = n \neq 0 \\ 0 & m \neq n \end{cases}$$

$$\int_0^T \sin(m\omega_0 t)\cos(n\omega_0 t)dt = 0 \quad \forall m, n$$

---

## 5.1.3 Amplitude-Phase Form

The trigonometric FS can be rewritten as:

$$x(t) = A_0 + \sum_{n=1}^{\infty} A_n \cos(n\omega_0 t + \phi_n)$$

Where:

$$A_0 = a_0, \quad A_n = \sqrt{a_n^2 + b_n^2}, \quad \phi_n = -\arctan\left(\frac{b_n}{a_n}\right)$$

```
Amplitude spectrum:          Phase spectrum:
  A                            φ
  │                            │
A₁┤  ↑                    φ₁ ┤  ↑
  │  │                        │  │
A₂┤  │  ↑                 φ₂ ┤  │  ↑
  │  │  │                     │  │  │
A₃┤  │  │  ↑              φ₃ ┤  │  │  ↑
  │  │  │  │                  │  │  │  │
  ┼──┴──┴──┴──→ nω₀          ┼──┴──┴──┴──→ nω₀
  0  ω₀ 2ω₀ 3ω₀             0  ω₀ 2ω₀ 3ω₀
  
  LINE SPECTRUM — exists only at harmonic frequencies
```

---

## 5.1.4 Worked Example 1: Square Wave

$$x(t) = \begin{cases} 1, & 0 < t < T/2 \\ -1, & T/2 < t < T \end{cases}$$

```
x(t):
  1 ┤  ┌────┐    ┌────┐    ┌────┐
    │  │    │    │    │    │    │
  0 ┤──┘    │    │    │    │    │──→ t
    │       │    │    │    │
 -1 ┤       └────┘    └────┘
    0  T/2   T  3T/2  2T
```

**DC**: $a_0 = \frac{1}{T}\int_0^T x(t)dt = \frac{1}{T}[T/2 - T/2] = 0$

**Cosine**: $a_n = \frac{2}{T}\int_0^{T/2}\cos(n\omega_0 t)dt + \frac{2}{T}\int_{T/2}^{T}(-1)\cos(n\omega_0 t)dt = 0$ (by symmetry: odd function)

**Sine**: 
$$b_n = \frac{2}{T}\int_0^{T/2}\sin(n\omega_0 t)dt - \frac{2}{T}\int_{T/2}^{T}\sin(n\omega_0 t)dt$$

$$= \frac{2}{n\pi}(1 - \cos n\pi) = \begin{cases} \frac{4}{n\pi} & n \text{ odd} \\ 0 & n \text{ even} \end{cases}$$

$$\boxed{x(t) = \frac{4}{\pi}\left[\sin(\omega_0 t) + \frac{1}{3}\sin(3\omega_0 t) + \frac{1}{5}\sin(5\omega_0 t) + \cdots\right]}$$

```
Fourier synthesis of square wave:

1st harmonic:     1st + 3rd:          1st + 3rd + 5th:
    ╱╲               ┌──╲               ┌────┐
   ╱  ╲             ╱│   ╲             ╱│    │╲
──╱────╲──      ───╱─┘    ╲──      ───╱─┘    └─╲──
  ╲    ╱           ╲    ┌─╱          ╲     ┌──╱
   ╲  ╱             ╲│  ╱            ╲│   │╱
    ╲╱               └──╱              └────┘

  Getting closer to the square wave with more terms!
```

---

## 5.1.5 Worked Example 2: Sawtooth Wave

$$x(t) = \frac{t}{T}, \quad 0 < t < T \quad \text{(period } T\text{)}$$

```
x(t):
  1 ┤        ╱│       ╱│       ╱│
    │       ╱ │      ╱ │      ╱ │
    │      ╱  │     ╱  │     ╱  │
    │     ╱   │    ╱   │    ╱   │
  0 ┤    ╱    │   ╱    │   ╱    │──→ t
    0    T    2T   3T
```

**DC**: $a_0 = \frac{1}{T}\int_0^T \frac{t}{T}dt = \frac{1}{2}$

**Cosine**: $a_n = \frac{2}{T}\int_0^T \frac{t}{T}\cos(n\omega_0 t)dt = 0$

**Sine**: $b_n = \frac{2}{T}\int_0^T \frac{t}{T}\sin(n\omega_0 t)dt = -\frac{1}{n\pi}$

$$\boxed{x(t) = \frac{1}{2} - \frac{1}{\pi}\sum_{n=1}^{\infty} \frac{1}{n}\sin(n\omega_0 t)}$$

---

## 5.1.6 Worked Example 3: Rectified Cosine

$$x(t) = |\cos(\omega_0 t)| \quad \text{(full-wave rectified, period } T/2\text{)}$$

The fundamental frequency doubles: $\omega_0' = 2\omega_0$, $T' = T/2$.

$$a_0 = \frac{2}{\pi}$$

$$a_n = \frac{4}{\pi} \cdot \frac{(-1)^{n+1}}{4n^2 - 1}$$

$$b_n = 0 \quad \text{(even function)}$$

$$x(t) = \frac{2}{\pi} + \frac{4}{\pi}\left[\frac{1}{3}\cos(2\omega_0 t) - \frac{1}{15}\cos(4\omega_0 t) + \cdots\right]$$

---

## 5.1.7 Dirichlet Conditions

A periodic signal $x(t)$ has a Fourier Series representation if:

1. $x(t)$ is absolutely integrable over one period: $\int_T |x(t)|dt < \infty$
2. $x(t)$ has a finite number of maxima and minima in one period
3. $x(t)$ has a finite number of discontinuities in one period

> At discontinuities, the FS converges to the **midpoint**: $\frac{x(t^+) + x(t^-)}{2}$

---

## 5.1.8 Gibbs Phenomenon

At points of discontinuity, the partial Fourier sum exhibits **overshoot** of approximately **9%** of the jump, regardless of the number of terms.

```
Gibbs phenomenon at a discontinuity:

  1.09 ┤     ╱╲  ← ~9% overshoot (never goes away!)
  1.00 ┤ ───╱──╲───────────  Ideal square wave
       │   ╱    ╲╱╲
       │  ╱        ╲╱─── oscillations diminish
  0.00 ┤─╱─────────────── 
       │╱╲╱╲
 -0.09 ┤    ╲╱  ← ~9% undershoot
 -1.00 ┤ ──────────────── 
       0
```

---

## 5.1.9 Convergence Types

| Type | Definition |
|------|------------|
| **Pointwise** | FS converges to $x(t)$ at each continuous point |
| **Uniform** | FS converges uniformly if $x(t)$ is continuous |
| **Mean-square** | $\lim_{N\to\infty} \int_T |x(t) - x_N(t)|^2 dt = 0$ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Trig FS | $x(t) = a_0 + \sum(a_n\cos + b_n\sin)$ |
| $a_0$ | DC = average = $\frac{1}{T}\int_T x(t)dt$ |
| $a_n$ | $\frac{2}{T}\int_T x(t)\cos(n\omega_0 t)dt$ |
| $b_n$ | $\frac{2}{T}\int_T x(t)\sin(n\omega_0 t)dt$ |
| Orthogonality | Basis functions are mutually orthogonal |
| Line spectrum | Exists only at discrete harmonic frequencies |
| Square wave FS | Only odd harmonics with $1/n$ decay |
| Dirichlet conditions | Sufficient conditions for FS existence |
| Gibbs phenomenon | ~9% overshoot at discontinuities |

---

## Quick Revision Questions

1. **What is the DC coefficient of $x(t) = 3 + 2\cos(5t)$?**
   - $a_0 = 3$ (the constant term).

2. **A signal has only sine terms in its FS. What symmetry does it have?**
   - Odd symmetry: $x(t) = -x(-t)$.

3. **How many harmonics does a pure cosine signal have?**
   - One. $x(t) = A\cos(\omega_0 t)$ already is its own Fourier Series.

4. **What happens to the FS at a discontinuity?**
   - It converges to the midpoint of the jump; Gibbs overshoot of ~9% persists.

5. **Write the first three nonzero terms of the square wave FS.**
   - $\frac{4}{\pi}[\sin\omega_0 t + \frac{1}{3}\sin 3\omega_0 t + \frac{1}{5}\sin 5\omega_0 t]$.

---

[← Previous: Unit 4 — System Interconnection](../04-LTI-Systems/04-system-interconnection.md) | [Next: Exponential Fourier Series →](02-exponential-fourier-series.md)
