# Chapter 9.3: Reconstruction and Interpolation

## Overview

**Reconstruction** converts a discrete-time signal back to continuous-time. The ideal method uses sinc interpolation (equivalent to ideal low-pass filtering), but practical systems use zero-order hold (ZOH), first-order hold (FOH), or higher-order interpolation. This chapter covers the theory and practice of digital-to-analog conversion.

---

## 9.3.1 Ideal Reconstruction

### Theory

Pass the sampled signal through an ideal LPF with cutoff $\omega_c = \omega_s/2$ and gain $T_s$:

$$X(j\omega) = X_s(j\omega) \cdot H_{LPF}(j\omega)$$

$$H_{LPF}(j\omega) = \begin{cases} T_s & |\omega| \leq \omega_s/2 \\ 0 & |\omega| > \omega_s/2 \end{cases}$$

### Shannon (Whittaker-Shannon) Interpolation

$$\boxed{x(t) = \sum_{n=-\infty}^{\infty} x[n] \cdot \text{sinc}\left(\frac{t - nT_s}{T_s}\right)}$$

where $\text{sinc}(u) = \frac{\sin(\pi u)}{\pi u}$, and $\text{sinc}(0) = 1$.

```
SINC INTERPOLATION:

Samples x[n]:  ↑   ↑       ↑
               │   │  ↑    │   ↑
            ───┴───┴──┴────┴───┴──→ n
               0   1   2    3   4

Each sample × sinc:           Sum of all sincs:
    ╱╲                             ╱╲    ╱╲
   ╱  ╲  ╱╲╱╲                    ╱  ╲  ╱  ╲   ╱╲
──╱────╲╱────╲──→ t          ───╱────╲╱    ╲ ╱  ╲╲
  ↕ sinc centered at           ─────────────────────→ t
  each sample point            Reconstructed x(t)
```

### Why Ideal Reconstruction Is Impractical

1. $\text{sinc}(t)$ extends from $-\infty$ to $+\infty$ → **non-causal**
2. Requires infinite number of samples (past and future)
3. Slow decay of sinc tails → sensitive to truncation

---

## 9.3.2 Zero-Order Hold (ZOH)

The simplest practical reconstruction: hold each sample value constant until the next sample.

$$x_{ZOH}(t) = x[n] \quad \text{for } nT_s \leq t < (n+1)T_s$$

```
ZOH RECONSTRUCTION:

Samples:     ↑   ↑       ↑
             │   │  ↑    │   ↑
          ───┴───┴──┴────┴───┴──→ n

ZOH output:
          ┌───┐
          │   │   ┌─┐
      ┌───┘   │   │ │ ┌───┐
      │       └───┘ │ │   │ ┌─┐
  ────┘             └─┘   └─┘ └──→ t
  Staircase approximation
```

### ZOH Transfer Function

$$H_{ZOH}(j\omega) = T_s \cdot e^{-j\omega T_s/2} \cdot \text{sinc}\left(\frac{\omega T_s}{2\pi}\right)$$

$$|H_{ZOH}(j\omega)| = T_s \left|\text{sinc}\left(\frac{\omega T_s}{2\pi}\right)\right|$$

### ZOH Characteristics

| Property | Value |
|----------|-------|
| DC gain | $T_s$ |
| Magnitude droop at $\omega_s/2$ | $2/\pi \approx 0.637$ (−3.92 dB) |
| Phase | Linear: $-\omega T_s/2$ |
| Compensation needed | sinc compensation (inverse sinc) filter |

```
ZOH FREQUENCY RESPONSE:

|H(jω)|
  Ts ┤────╲
     │     ╲
     │      ╲          sinc envelope
     │       ╲
  0  ┤────────╲─────╱──────→ ω
     0      ωs/2    ωs     2ωs
     
  Droop of 3.92 dB at ω = ωs/2
  Nulls at multiples of ωs
```

---

## 9.3.3 First-Order Hold (FOH)

Linear interpolation between consecutive samples:

$$x_{FOH}(t) = x[n] + \frac{x[n+1] - x[n]}{T_s}(t - nT_s), \quad nT_s \leq t < (n+1)T_s$$

```
FOH (LINEAR INTERPOLATION):

Samples:     ↑   ↑       ↑
             │   │  ↑    │   ↑
          ───┴───┴──┴────┴───┴──→ n

FOH output:
             ╱╲
            ╱  ╲    ╱╲
        ╱──╱    ╲  ╱  ╲╲
       ╱         ╲╱    ╲╲
  ────╱                  ╲───→ t
  Piecewise linear (connect-the-dots)
```

### FOH Transfer Function

$$|H_{FOH}(j\omega)| = T_s \cdot \text{sinc}^2\left(\frac{\omega T_s}{2\pi}\right)$$

Better attenuation of spectral images than ZOH, but more droop in the passband.

---

## 9.3.4 Comparison of Reconstruction Methods

| Method | Interpolation | Continuity | Magnitude Droop at $f_s/2$ | Image Rejection |
|--------|--------------|------------|--------------------------|-----------------|
| Ideal (sinc) | Sinc function | Smooth ($C^\infty$) | None (perfect) | Perfect |
| ZOH | Piecewise constant | Discontinuous | 3.92 dB | Poor |
| FOH | Piecewise linear | Continuous ($C^0$) | 7.84 dB | Better |
| Cubic spline | Piecewise cubic | Smooth ($C^2$) | < 1 dB | Good |

```
RECONSTRUCTED SIGNALS COMPARISON:

Original x(t):     ╱╲    ╱╲
                  ╱  ╲  ╱  ╲
              ───╱────╲╱────╲──→ t

ZOH:            ┌─┐   ┌┐
              ┌─┘ └─┐ ││ ┌─┐
          ────┘     └─┘└─┘ └──→ t  (blocky)

FOH:            ╱╲   ╱╲
              ╱  ╲ ╱  ╲
          ───╱────×────╲──→ t  (angular)

Sinc:           ╱╲    ╱╲
              ╱  ╲  ╱  ╲
          ───╱────╲╱────╲──→ t  (smooth, exact)
```

---

## 9.3.5 Practical DAC Reconstruction

A real DAC system has multiple stages:

```
PRACTICAL RECONSTRUCTION CHAIN:

x[n] → [Digital  ] → [DAC    ] → [Analog    ] → x(t)
        [Interp.  ]   [ZOH    ]   [Smoothing ]
        [Filter   ]   [output ]   [Filter    ]
        
  1. Digital interpolation filter
     (oversampling + FIR filtering)
  2. DAC converts to analog (ZOH)
  3. Analog smoothing filter removes
     spectral images
```

### Step-by-Step Process

1. **Digital upsampling**: Insert zeros between samples (increase rate by $L$)
2. **Digital interpolation filter**: FIR LPF removes imaging artifacts
3. **DAC with ZOH**: Converts to staircase analog signal (at higher rate)
4. **Analog smoothing filter**: Gentle LPF removes remaining images

### Why Oversampling Helps Reconstruction

```
WITHOUT OVERSAMPLING (fs = 2fm):
  ╱╲     ╱╲      Need sharp analog filter
 ╱  ╲   ╱  ╲     to separate copies!
╱────╲─╱────╲──→ f
    fm  fs-fm

WITH 8× OVERSAMPLING (8fs):
  ╱╲                     ╱╲
 ╱  ╲                   ╱  ╲    Gentle analog
╱────╲─── ── ── ── ───╱────╲──→ f  filter works!
    fm              7fs     8fs
     ←── huge gap ──→
```

---

## 9.3.6 Sinc Compensation

To correct for ZOH magnitude droop, apply an **inverse sinc** (sinc compensation) digital filter before the DAC:

$$H_{comp}(e^{j\omega}) = \frac{1}{\text{sinc}(\omega/\omega_s)} \quad \text{for } |\omega| < \omega_s/2$$

This pre-emphasizes frequencies that ZOH attenuates, resulting in a flat overall response.

---

## 9.3.7 Reconstruction in the Z/Laplace Framework

### ZOH Transfer Function in $s$-Domain

$$G_{ZOH}(s) = \frac{1 - e^{-sT_s}}{s}$$

This is used in control systems for converting continuous-time controllers to discrete-time (impulse invariance, ZOH discretization).

### Connection to Z-Transform

When preceded by a sampler and followed by ZOH:

$$G(z) = (1 - z^{-1})\mathcal{Z}\left\{\frac{G_p(s)}{s}\right\}$$

where $G_p(s)$ is the plant transfer function.

---

## Summary Table

| Method | Quality | Complexity | Practical Use |
|--------|---------|------------|---------------|
| Ideal sinc | Perfect | Infinite (non-causal) | Theoretical benchmark |
| ZOH | Passable with filtering | Minimal | All DACs (inherent) |
| FOH | Better than ZOH | Moderate | Some DAC chips |
| Digital oversampling + ZOH | Excellent | Digital filter + simple analog | Modern audio DACs |
| Sinc compensation | Corrects ZOH droop | Digital filter | High-quality DACs |

---

## Quick Revision Questions

1. **What is the reconstruction formula for ideal interpolation?**
   - $x(t) = \sum_n x[n]\,\text{sinc}((t-nT_s)/T_s)$.

2. **Why is ideal sinc reconstruction impractical?**
   - The sinc function has infinite extent (non-causal), requiring knowledge of all future samples.

3. **What is the ZOH magnitude droop at $f_s/2$?**
   - $2/\pi \approx 0.637$, or about 3.92 dB of attenuation.

4. **How does oversampling help in DAC reconstruction?**
   - It pushes spectral images far from the signal band, allowing a gentle analog smoothing filter instead of a sharp one.

5. **What is sinc compensation?**
   - A digital pre-filter with inverse sinc response that compensates for the passband droop caused by the ZOH.

---

[← Previous: Aliasing](02-aliasing.md) | [Next: Practical Sampling Systems →](04-practical-systems.md)
