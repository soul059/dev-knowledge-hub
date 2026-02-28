# Chapter 5.3: Fourier Coefficients — Computation Techniques

## Overview

Computing Fourier coefficients by direct integration can be tedious. This chapter presents **symmetry shortcuts**, **coefficient tables** for common waveforms, and efficient techniques that dramatically reduce computation.

---

## 5.3.1 Symmetry-Based Shortcuts

### Even Symmetry: $x(t) = x(-t)$

$$b_n = 0 \quad \text{(all sine terms vanish)}$$

$$a_n = \frac{4}{T} \int_0^{T/2} x(t) \cos(n\omega_0 t) \, dt$$

> Only cosine terms survive. Integration range halved.

### Odd Symmetry: $x(t) = -x(-t)$

$$a_0 = 0, \quad a_n = 0 \quad \text{(all cosine terms vanish)}$$

$$b_n = \frac{4}{T} \int_0^{T/2} x(t) \sin(n\omega_0 t) \, dt$$

> Only sine terms survive. Integration range halved.

### Half-Wave Symmetry: $x(t) = -x(t \pm T/2)$

$$a_n = b_n = 0 \quad \text{for even } n$$

> Only **odd harmonics** exist. The signal looks the same inverted and shifted by half a period.

```
Even function:            Odd function:         Half-wave symmetric:
    │  ╱╲                     │╲                    │  ╱╲
  ╲╱│╱  ╲╱╲               ╱╲│  ╲╱            ╲╱  │╱  ╲╱
  ──┼──────→ t           ──┼──────→ t         ──┼──────→ t
    │                    ╱  │╲                    │
    │                       │                    │
  Mirror about t=0      Rotate 180°           Invert + shift T/2
  → only cosines        → only sines          → only odd harmonics
```

### Quarter-Wave Symmetry

If a signal has **both** half-wave AND even/odd symmetry:

| Symmetry | Surviving Terms |
|----------|----------------|
| HW + Even | $a_n$, odd $n$ only |
| HW + Odd | $b_n$, odd $n$ only |

Integration range reduces to $[0, T/4]$ — **one quarter period**!

$$a_n = \frac{8}{T}\int_0^{T/4} x(t)\cos(n\omega_0 t)dt \quad \text{(HW + Even, odd } n\text{)}$$

$$b_n = \frac{8}{T}\int_0^{T/4} x(t)\sin(n\omega_0 t)dt \quad \text{(HW + Odd, odd } n\text{)}$$

---

## 5.3.2 Symmetry Detection Flowchart

```
                   START
                     │
                     ▼
           ┌────────────────┐
           │ Is x(t) = x(-t)?│
           └───────┬────────┘
              YES  │  NO
              ▼    │  ▼
         bₙ = 0   │  ┌────────────┐
                   │  │x(t)=-x(-t)?│
                   │  └──────┬─────┘
                   │    YES  │  NO
                   │    ▼    │  ▼
                   │ aₙ=a₀=0│  General
                   │         │  (all terms)
                   ▼         ▼
           ┌────────────────────┐
           │ x(t) = -x(t±T/2)? │
           └───────┬────────────┘
              YES  │  NO
              ▼    │  ▼
        Even n     │  All harmonics
        terms = 0  │  present
```

---

## 5.3.3 Common Waveform Coefficients

### Square Wave (Odd HW-symmetric, amplitude $A$)

$$b_n = \frac{4A}{n\pi}, \quad n = 1, 3, 5, \ldots$$

### Triangular Wave (Even HW-symmetric, amplitude $A$)

$$a_n = \frac{8A}{n^2\pi^2}, \quad n = 1, 3, 5, \ldots$$

### Sawtooth Wave (Odd, amplitude $A$, period $T$)

$$b_n = \frac{-2A}{n\pi}(-1)^{n+1} = \frac{2A}{n\pi}(-1)^{n+1}, \quad n = 1, 2, 3, \ldots$$

### Rectified Sine (Even, period $T/2$)

$$a_0 = \frac{2A}{\pi}, \quad a_n = \frac{4A}{\pi}\frac{(-1)^{n+1}}{4n^2-1}, \quad n = 1,2,3,\ldots$$

### Pulse Train (duty cycle $d = \tau/T$)

$$c_n = Ad \cdot \text{sinc}(nd)$$

---

## 5.3.4 Coefficient Decay Rate

The smoothness of $x(t)$ determines how fast the coefficients decay:

| Signal Property | Coefficient Decay | Example |
|-----------------|-------------------|---------|
| Discontinuous | $c_n \sim 1/n$ | Square wave |
| Continuous, corners | $c_n \sim 1/n^2$ | Triangle wave |
| Smooth (differentiable) | $c_n \sim 1/n^3$ or faster | Smooth pulse |

> **Smoother signals need fewer harmonics** for accurate representation.

```
Coefficient magnitude decay:

|cₙ|
  │╲
  │ ╲ 1/n (square wave - slow decay)
  │  ╲
  │   ╲╲╲ 1/n² (triangle wave - faster)
  │      ╲╲╲╲╲ 1/n³ (smooth signal)
  ┼──────────────→ n
  0  5  10  15  20
```

---

## 5.3.5 Effect of DC Offset and Scaling

### DC Offset

If $y(t) = x(t) + D$:

$$c_0^{(y)} = c_0^{(x)} + D, \quad c_n^{(y)} = c_n^{(x)} \text{ for } n \neq 0$$

### Amplitude Scaling

If $y(t) = Ax(t)$:

$$c_n^{(y)} = A \cdot c_n^{(x)}$$

### Time Reversal

If $y(t) = x(-t)$:

$$c_n^{(y)} = c_{-n}^{(x)} = [c_n^{(x)}]^*$$

---

## 5.3.6 Effect of Waveform Modifications

### Shifting Baseline

Adding a constant shifts only $c_0$:

```
Original:                 Shifted up by D:
  A ┤  ┌───┐              A+D ┤  ┌───┐
    │  │   │                   │  │   │
  0 ┤──┘   └──              D ┤──┘   └──
 -A ┤       ← symmetric      0 ┤       ← asymmetric
    c₀ = 0                     c₀ = D
```

### Half-Range Expansion

If a signal is defined on $[0, L]$ only, you can expand it as:
- **Even extension** → cosine series only ($b_n = 0$)
- **Odd extension** → sine series only ($a_n = 0$)

---

## 5.3.7 Coefficient Computation Example

**Find FS coefficients for triangular wave**:

$$x(t) = \begin{cases} \frac{4A}{T}t, & 0 \leq t \leq T/4 \\ \frac{4A}{T}(T/2 - t), & T/4 \leq t \leq 3T/4 \\ \frac{4A}{T}(t - T), & 3T/4 \leq t \leq T \end{cases}$$

**Symmetry check**:
- Even? Yes (mirror symmetric about $t = 0$) → $b_n = 0$
- Half-wave? Yes ($x(t) = -x(t + T/2)$) → even harmonics = 0
- Quarter-wave symmetric → use $[0, T/4]$ only

$$a_n = \frac{8}{T}\int_0^{T/4} \frac{4A}{T}t \cdot \cos(n\omega_0 t)dt, \quad n \text{ odd}$$

Integration by parts:

$$a_n = \frac{8A}{n^2\pi^2}(-1)^{(n-1)/2}, \quad n = 1, 3, 5, \ldots$$

$$a_1 = \frac{8A}{\pi^2}, \quad a_3 = \frac{-8A}{9\pi^2}, \quad a_5 = \frac{8A}{25\pi^2}$$

```
Triangular wave coefficients:

  8A/π² ┤  ↑
        │  │
        │  │
        │  │        ↑
  0     ┤──┤────↓───┤───↓───→ n
        │  1    3   5   7
        
  Decays as 1/n² — MUCH faster than square wave's 1/n
  → Triangle wave is a "smoother" version of square wave
```

---

## 5.3.8 Differentiation/Integration of Series

### Differentiation

If $x(t) \leftrightarrow c_n$, then:

$$\frac{dx}{dt} \leftrightarrow jn\omega_0 c_n$$

> Differentiating a square wave gives impulse pairs (consistent with $c_n$ growing by factor $n$).

### Integration

$$\int x(t)dt \leftrightarrow \frac{c_n}{jn\omega_0} \quad (n \neq 0)$$

> Integrating a square wave gives a triangle wave (consistent with $c_n$ gaining factor $1/n$).

```
RELATIONSHIP CHAIN:

  Impulse pairs      Differentiate      Square wave
  cₙ ~ 1           ←──────────────      cₙ ~ 1/n
                    ──────────────→    
                      Integrate         
                                     ←──────────────
                                       Differentiate
                                     ──────────────→
                                       Integrate      Triangle wave
                                                       cₙ ~ 1/n²
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Even symmetry | $b_n = 0$, only cosines |
| Odd symmetry | $a_0 = a_n = 0$, only sines |
| Half-wave symmetry | Even harmonics = 0 |
| Quarter-wave | Both HW + even/odd, integrate over $T/4$ |
| Discontinuous signal | $c_n \sim 1/n$ (slow decay) |
| Continuous signal | $c_n \sim 1/n^2$ or faster |
| Differentiation | Multiplies $c_n$ by $jn\omega_0$ |
| Integration | Divides $c_n$ by $jn\omega_0$ |

---

## Quick Revision Questions

1. **A signal has only cosine terms. What symmetry does it have?**
   - Even symmetry: $x(t) = x(-t)$.

2. **A square wave has $1/n\ $ coefficient decay. A triangle wave?**
   - $1/n^2$ — triangle is smoother (continuous, only corners).

3. **What is the advantage of quarter-wave symmetry?**
   - Integration is over only $T/4$ (quarter period) and only odd harmonics need computing.

4. **How does adding a DC offset $D$ to a signal affect its FS?**
   - Only $c_0$ changes: $c_0 \to c_0 + D$. All other $c_n$ unchanged.

5. **If differentiating $x(t)$ gives the FS coefficient $jn\omega_0 c_n$, what signal has $c_n = 1/(jn\omega_0)$?**
   - The integral of a signal whose FS coefficients are all $c_n = 1$, i.e., the integral of $\delta_T(t)$ (periodic impulse train).

---

[← Previous: Exponential Fourier Series](02-exponential-fourier-series.md) | [Next: Properties of Fourier Series →](04-properties-of-fourier-series.md)
