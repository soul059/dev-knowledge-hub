# Chapter 6.3: Frequency Response of LTI Systems

## Overview

The **frequency response** $H(j\omega)$ describes how an LTI system modifies each frequency component of the input signal. It is the Fourier Transform of the impulse response $h(t)$ and provides a complete characterization of the system in the frequency domain.

---

## 6.3.1 Definition

$$H(j\omega) = \mathcal{F}\{h(t)\} = \int_{-\infty}^{\infty} h(t)e^{-j\omega t}dt$$

### Decomposition

$$H(j\omega) = |H(j\omega)| \, e^{j\angle H(j\omega)}$$

- $|H(j\omega)|$ = **magnitude response** (gain at each frequency)
- $\angle H(j\omega)$ = **phase response** (phase shift at each frequency)

---

## 6.3.2 Eigenfunction Interpretation

When input is $x(t) = e^{j\omega_0 t}$:

$$y(t) = H(j\omega_0) \cdot e^{j\omega_0 t}$$

For real sinusoidal input $x(t) = A\cos(\omega_0 t + \phi)$:

$$y(t) = A|H(j\omega_0)| \cos\left(\omega_0 t + \phi + \angle H(j\omega_0)\right)$$

> The system **scales** the amplitude by $|H|$ and **shifts** the phase by $\angle H$ at each frequency independently.

```
INPUT:                          OUTPUT:
A·cos(ω₀t + φ)                 A|H(jω₀)|·cos(ω₀t + φ + ∠H)

  ╱╲   ╱╲   ╱╲                    ╱╲     ╱╲     ╱╲
 ╱  ╲ ╱  ╲ ╱  ╲   → [H(jω)] →  ╱    ╲ ╱    ╲ ╱    ╲
╱────╳────╳────╲               ╱──────╳──────╳──────╲
                               
Same frequency, different       Scaled by |H|, shifted by ∠H
amplitude and phase
```

---

## 6.3.3 Input-Output in Frequency Domain

$$Y(j\omega) = H(j\omega) \cdot X(j\omega)$$

$$|Y(j\omega)| = |H(j\omega)| \cdot |X(j\omega)|$$

$$\angle Y(j\omega) = \angle H(j\omega) + \angle X(j\omega)$$

```
FREQUENCY DOMAIN I/O:

|X(jω)|               |H(jω)|              |Y(jω)|
   ╱╲                  ┌───┐                  ╱╲
  ╱  ╲        ×       │   │       =         ╱  ╲
 ╱    ╲               │   └──┐             ╱    └──
╱──────╲──→ ω         └──────┘→ ω         ╱────────→ ω

Input spectrum    System gain          Output spectrum
```

---

## 6.3.4 Examples

### Example 1: RC Low-Pass Filter

$$h(t) = \frac{1}{RC}e^{-t/RC}u(t) \implies H(j\omega) = \frac{1}{1 + j\omega RC}$$

Let $\omega_c = 1/RC$ (cutoff frequency):

$$|H(j\omega)| = \frac{1}{\sqrt{1 + (\omega/\omega_c)^2}}, \quad \angle H(j\omega) = -\arctan(\omega/\omega_c)$$

```
RC Low-Pass Filter:

|H(jω)| (Magnitude):          ∠H(jω) (Phase):
  1.0 ┤──────╲                   0° ┤──────╲
      │       ╲                     │       ╲
 0.707┤········╲·····(−3dB)   −45°  ┤········╲
      │         ╲                    │         ╲
      │          ╲╲             −90° ┤          ╲──
  0.0 ┤───────────╲╲──→ ω           ┤──────────────→ ω
      0    ωc    2ωc                 0    ωc    2ωc

  Passes low freq, attenuates high freq
  −3dB point at ω = ωc = 1/RC
```

### Example 2: Ideal Differentiator

$$y(t) = \frac{dx}{dt} \implies H(j\omega) = j\omega$$

$$|H| = |\omega|, \quad \angle H = +90° \text{ for } \omega > 0$$

```
|H(jω)|:                    ∠H(jω):
  │         ╱               +90° ┤ ───────────
  │       ╱                      │
  │     ╱                     0° ┤
  │   ╱                          │
  │ ╱                        −90°┤ ───────────
  ┼╱──────→ ω                    ┼──────────→ ω
  0                              0
  
  Amplifies high frequencies linearly
```

### Example 3: First-Order System

$$H(j\omega) = \frac{j\omega}{j\omega + a}$$

This is a **high-pass filter**: $|H(0)| = 0$, $|H(\infty)| = 1$.

---

## 6.3.5 Bode Plots

Bode plots display $|H|$ in decibels and $\angle H$ in degrees vs log frequency.

### Magnitude in Decibels

$$|H|_{dB} = 20\log_{10}|H(j\omega)|$$

### First-Order Low-Pass: $H(j\omega) = \frac{K}{1 + j\omega/\omega_c}$

```
Bode Magnitude Plot (dB vs log ω):

  20log₁₀K ┤───────────────╲
            │                ╲
            │                 ╲  slope: −20 dB/decade
            │                  ╲
            │                   ╲
            ┤────────────────────╲──────→ log ω
                    ωc (corner freq)

Bode Phase Plot (degrees vs log ω):

     0° ┤──────────╲
        │           ╲
   −45° ┤            ╲  (at ω = ωc)
        │             ╲
   −90° ┤──────────────╲────→ log ω
        0.1ωc   ωc   10ωc
```

### Bode Plot Rules

| Factor | Magnitude slope | Phase change |
|--------|----------------|--------------|
| $K$ (constant) | $20\log K$ (flat) | $0°$ |
| $j\omega$ (zero at origin) | $+20$ dB/dec | $+90°$ |
| $1/(j\omega)$ (pole at origin) | $-20$ dB/dec | $-90°$ |
| $(1+j\omega/\omega_c)$ (zero) | $+20$ dB/dec above $\omega_c$ | $0° \to +90°$ |
| $1/(1+j\omega/\omega_c)$ (pole) | $-20$ dB/dec above $\omega_c$ | $0° \to -90°$ |

---

## 6.3.6 Distortionless Transmission

A system transmits without distortion if:

$$y(t) = Kx(t - t_d)$$

This requires:

$$H(j\omega) = Ke^{-j\omega t_d}$$

$$|H(j\omega)| = K \quad \text{(constant gain)}$$

$$\angle H(j\omega) = -\omega t_d \quad \text{(linear phase)}$$

```
DISTORTIONLESS SYSTEM:

|H(jω)|:                  ∠H(jω):
  K ┤──────────────          0° ┤──────────╲
    │                           │           ╲
    │                           │            ╲ slope = −t_d
  0 ┤──────────────→ ω          │             ╲
                                ┤──────────────╲→ ω
  Flat magnitude                Linear phase
```

> Any deviation from flat magnitude or linear phase introduces **distortion**.

### Types of Distortion

| Type | Cause | Effect |
|------|-------|--------|
| **Amplitude distortion** | Non-flat $\|H\|$ | Some frequencies amplified/attenuated differently |
| **Phase distortion** | Non-linear $\angle H$ | Different frequencies delayed differently |
| **Group delay** | $\tau_g = -d(\angle H)/d\omega$ | Non-constant → dispersion |

---

## 6.3.7 Group Delay

$$\tau_g(\omega) = -\frac{d\angle H(j\omega)}{d\omega}$$

For distortionless transmission: $\tau_g = t_d$ (constant).

**Example**: RC filter with $H = 1/(1+j\omega RC)$:

$$\angle H = -\arctan(\omega RC) \implies \tau_g = \frac{RC}{1+(\omega RC)^2}$$

The group delay varies with frequency — phase distortion occurs.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Frequency response | $H(j\omega) = \mathcal{F}\{h(t)\}$ |
| I/O relation | $Y = H \cdot X$ (multiplication) |
| Sinusoidal response | Scale by $\|H\|$, shift phase by $\angle H$ |
| Bode plot | Log-frequency plot of $\|H\|_{dB}$ and $\angle H$ |
| -3 dB point | Half-power frequency ($\|H\| = 1/\sqrt{2}$) |
| Distortionless | Flat $\|H\|$ + linear $\angle H$ |
| Group delay | $\tau_g = -d\angle H/d\omega$; constant = no phase distortion |

---

## Quick Revision Questions

1. **What is the frequency response of the system $y' + 2y = x$?**
   - $H(j\omega) = 1/(j\omega + 2)$ — a low-pass filter with $\omega_c = 2$.

2. **What magnitude and phase conditions give distortionless transmission?**
   - Flat magnitude ($|H|$ = constant) and linear phase ($\angle H = -\omega t_d$).

3. **What is the Bode magnitude slope for a first-order pole?**
   - $-20$ dB/decade beyond the corner frequency.

4. **If $|H(j\omega_c)| = 1/\sqrt{2}$, what is $|H|_{dB}$?**
   - $20\log(1/\sqrt{2}) = -3$ dB (half-power point).

5. **What does a non-constant group delay indicate?**
   - Phase distortion — different frequency components experience different delays.

---

[← Previous: CTFT Properties](02-ctft-properties.md) | [Next: Filtering Concepts →](04-filtering-concepts.md)
