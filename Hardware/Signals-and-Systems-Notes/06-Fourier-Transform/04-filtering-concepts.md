# Chapter 6.4: Filtering Concepts

## Overview

Filters are LTI systems designed to selectively pass or reject certain frequency components. Understanding ideal and practical filters is essential for signal processing, communication, and control systems.

---

## 6.4.1 Ideal Filters

### Ideal Low-Pass Filter (LPF)

$$H(j\omega) = \begin{cases} 1, & |\omega| < \omega_c \\ 0, & |\omega| > \omega_c \end{cases}$$

```
|H(jω)|:                    h(t) = (ωc/π)sinc(ωct/π):
  1 ┤  ┌──────┐                  │  ╱╲
    │  │      │                  │ ╱  ╲
  0 ┤──┘      └──→ ω            │╱    ╲╲
   -ωc  0   ωc                 ─┼───────╲╲─→ t
                                 │
  Passes |ω| < ωc               Non-causal! (h(t) ≠ 0 for t < 0)
  Blocks |ω| > ωc               Not physically realizable
```

**Impulse response**: $h(t) = \frac{\omega_c}{\pi}\text{sinc}\left(\frac{\omega_c t}{\pi}\right)$

> The ideal LPF is **non-causal** ($h(t) \neq 0$ for $t < 0$) and therefore physically unrealizable.

### Ideal High-Pass Filter (HPF)

$$H_{HP}(j\omega) = 1 - H_{LP}(j\omega) = \begin{cases} 0, & |\omega| < \omega_c \\ 1, & |\omega| > \omega_c \end{cases}$$

### Ideal Band-Pass Filter (BPF)

$$H(j\omega) = \begin{cases} 1, & \omega_1 < |\omega| < \omega_2 \\ 0, & \text{otherwise} \end{cases}$$

### Ideal Band-Stop (Notch) Filter (BSF)

$$H(j\omega) = 1 - H_{BP}(j\omega)$$

```
IDEAL FILTER GALLERY:

LPF:          HPF:          BPF:          BSF:
  ┌──┐        ┌┐  ┌┐        ┌┐  ┌┐      ┌┐  ┌┐┌┐  ┌┐
  │  │        ││  ││        ││  ││      ││  ││││  ││
──┘  └──    ──┘└──┘└──    ──┘└──┘└──    ──┘└──┘└──┘└──
-ωc  ωc     -ωc  ωc     -ω₂-ω₁ω₁ω₂   -ω₂ω₁  ω₁ ω₂
```

---

## 6.4.2 Filter Specifications

```
PRACTICAL LPF SPECIFICATION:

|H(jω)| (dB)
  0 ┤─────╲
    │      ╲  Transition
 -Rp┤·······╲·····  ← Passband ripple (Rp dB)
    │        ╲
    │         ╲
 -As┤··········╲···  ← Stopband attenuation (As dB)
    │           ╲───
    ┤────────────────→ ω
    0  ωp  ωs
       │   │
    Passband  Stopband
     edge     edge
       
    ωp to ωs = Transition band
```

| Parameter | Symbol | Typical Values |
|-----------|--------|----------------|
| Passband edge | $\omega_p$ | Design spec |
| Stopband edge | $\omega_s$ | Design spec |
| Passband ripple | $R_p$ | 0.5-3 dB |
| Stopband attenuation | $A_s$ | 40-80 dB |
| Transition width | $\omega_s - \omega_p$ | Narrower = higher order |

---

## 6.4.3 Practical Filter Types

### Butterworth Filter (Maximally Flat)

$$|H(j\omega)|^2 = \frac{1}{1 + (\omega/\omega_c)^{2N}}$$

- **No ripple** in passband or stopband
- $-20N$ dB/decade roll-off
- Monotonic magnitude response

### Chebyshev Type I

- Equiripple in **passband**, monotonic in stopband
- Sharper transition than Butterworth for same order

### Chebyshev Type II

- Monotonic in passband, equiripple in **stopband**

### Elliptic (Cauer)

- Equiripple in **both** passband and stopband
- **Sharpest** transition for given order

```
COMPARISON (same order N):

|H| dB
  0 ┤══════╲
    │       ╲ 
    │  B     ╲╲
    │  u      ╲╲  C     E
    │  t       ╲╲ h     l
    │  t        ╲╲e    l
    │  e         ╲╲b   i
    │  r          ╲╲   p
    │  w           ╲╲  t
    │  o            ╲╲ i
    │  r             ╲╲c
    │  t              ╲╲
    ┤───────────────────╲──→ ω
    
  Elliptic has sharpest cutoff
  Butterworth has flattest passband
```

---

## 6.4.4 First-Order Filters

### First-Order LPF: $H(j\omega) = \frac{1}{1 + j\omega/\omega_c}$

| Parameter | Value |
|-----------|-------|
| DC gain | $\|H(0)\| = 1$ (0 dB) |
| Cutoff | $\|H(j\omega_c)\| = 1/\sqrt{2}$ (-3 dB) |
| High-freq slope | -20 dB/decade |
| Phase at DC | $0°$ |
| Phase at $\omega_c$ | $-45°$ |
| Phase at $\omega \to \infty$ | $-90°$ |

### First-Order HPF: $H(j\omega) = \frac{j\omega/\omega_c}{1 + j\omega/\omega_c}$

| Parameter | Value |
|-----------|-------|
| DC gain | 0 (-∞ dB) |
| High-freq gain | 1 (0 dB) |
| Low-freq slope | +20 dB/decade |

---

## 6.4.5 Second-Order Resonant Systems

$$H(j\omega) = \frac{\omega_n^2}{(j\omega)^2 + 2\zeta\omega_n(j\omega) + \omega_n^2}$$

Where:
- $\omega_n$ = natural frequency
- $\zeta$ = damping ratio

```
|H(jω)| for different ζ:

  │    ╱╲  ζ = 0.1 (underdamped, sharp peak)
  │   ╱  ╲
  │  ╱    ╲  ── ζ = 0.5
  ├──╱──────╲──── ζ = 0.707 (Butterworth)
  │ ╱        ╲╲
  │╱          ╲╲── ζ = 1.0 (critically damped)
  ┼────────────╲╲──→ ω
  0     ωn
  
  Quality factor: Q = 1/(2ζ)
  Higher Q → sharper resonance
```

---

## 6.4.6 All-Pass Filters

$$|H(j\omega)| = 1 \quad \forall \omega$$

Only the **phase** changes — used for phase equalization.

**First-order all-pass**:

$$H(j\omega) = \frac{a - j\omega}{a + j\omega}$$

```
All-pass filter:

|H(jω)|:                    ∠H(jω):
  1 ┤──────────────           0° ┤──────╲
    │                            │       ╲
    │                        -90°┤        ╲
    │                            │         ╲
  0 ┤──────────────→ ω      -180°┤──────────╲─→ ω
    No magnitude change          Phase shifts only
```

---

## 6.4.7 Filter Applications

| Application | Filter Type | Purpose |
|-------------|------------|---------|
| Audio treble/bass | LPF/HPF | Tone control |
| Anti-aliasing | LPF | Remove freq > Nyquist before sampling |
| Noise removal | LPF or BPF | Remove high-freq noise |
| DC blocking | HPF | Remove DC offset |
| 50/60 Hz removal | BSF (notch) | Remove power line interference |
| Channel selection | BPF | Select one radio station |
| Smoothing | LPF | Moving average, integration |
| Edge detection | HPF | Image processing |

---

## 6.4.8 Energy and Power Through Filters

If input $x(t)$ passes through $H(j\omega)$:

### Energy at output

$$E_y = \frac{1}{2\pi}\int_{-\infty}^{\infty}|H(j\omega)|^2 |X(j\omega)|^2 d\omega$$

### For ideal LPF (bandwidth $\omega_c$)

$$E_y = \frac{1}{2\pi}\int_{-\omega_c}^{\omega_c}|X(j\omega)|^2 d\omega$$

> Only energy within the passband is retained.

---

## 6.4.9 Paley-Wiener Criterion

A necessary condition for $|H(j\omega)|$ to be the magnitude of a causal system:

$$\int_{-\infty}^{\infty} \frac{|\ln|H(j\omega)||}{1 + \omega^2} d\omega < \infty$$

> $|H(j\omega)|$ can be zero at discrete frequencies but **cannot be zero over any continuous band** of frequencies.

This is why **ideal filters are unrealizable** — they have $|H| = 0$ over infinite bands.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Ideal LPF | Rect in $\omega$, sinc in $t$, non-causal |
| Ideal filters | Unrealizable (violate causality) |
| Butterworth | Maximally flat passband, -20N dB/dec |
| Chebyshev | Equiripple, sharper transition |
| Elliptic | Sharpest cutoff, ripple in both bands |
| Distortionless | Flat $\|H\|$, linear phase |
| All-pass | $\|H\| = 1$, phase equalization only |
| Paley-Wiener | Can't be zero over a continuous band |
| Anti-aliasing | LPF before sampling to prevent aliasing |

---

## Quick Revision Questions

1. **Why is an ideal LPF unrealizable?**
   - Its impulse response $h(t) = \omega_c/\pi \cdot \text{sinc}(\omega_c t/\pi)$ is non-causal (extends to $-\infty$).

2. **Which filter type gives the sharpest transition band?**
   - Elliptic (Cauer) filter.

3. **What is the roll-off rate of a 3rd-order Butterworth LPF?**
   - $-60$ dB/decade ($-20N$ dB/dec with $N=3$).

4. **What does an all-pass filter do?**
   - Passes all frequencies with unity gain; modifies only the phase.

5. **State the Paley-Wiener criterion informally.**
   - The magnitude of a causal filter cannot be exactly zero over a continuous frequency band.

6. **What filter is used before sampling a signal?**
   - Anti-aliasing low-pass filter with cutoff at the Nyquist frequency.

---

[← Previous: Frequency Response](03-frequency-response.md) | [Next: Unit 7 — Laplace Transform →](../07-Laplace-Transform/README.md)
