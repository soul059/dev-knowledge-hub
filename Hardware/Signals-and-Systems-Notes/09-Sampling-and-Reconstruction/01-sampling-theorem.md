# Chapter 9.1: Sampling Theorem

## Overview

The **Nyquist-Shannon Sampling Theorem** states that a band-limited continuous-time signal can be perfectly recovered from its samples if the sampling rate exceeds twice the highest frequency component. This theorem forms the theoretical foundation for all digital signal processing, digital communications, and digital audio/video.

---

## 9.1.1 Ideal Sampling

### Mathematical Model

Ideal sampling multiplies $x(t)$ by an impulse train:

$$x_s(t) = x(t) \cdot p(t) = x(t) \sum_{n=-\infty}^{\infty} \delta(t - nT_s) = \sum_{n=-\infty}^{\infty} x(nT_s)\,\delta(t - nT_s)$$

where $T_s$ = sampling period, $f_s = 1/T_s$ = sampling frequency.

```
IDEAL SAMPLING PROCESS:

x(t) (continuous):              p(t) (impulse train):
  ╱╲    ╱╲                      ↑ ↑ ↑ ↑ ↑ ↑ ↑
 ╱  ╲  ╱  ╲   ╱╲               │ │ │ │ │ │ │
╱    ╲╱    ╲ ╱  ╲╲            ──┴─┴─┴─┴─┴─┴─┴──→ t
              ╲╱  ╲               Tₛ
─────────────────────→ t

x_s(t) = x(t)·p(t):
  ↑    ↑
  │  ↑ │         ↑
  │  │ │    ↑    │
  │  │ │  ↑ │  ↑ │
──┴──┴─┴──┴─┴──┴─┴──→ t
  Sampled signal (weighted impulses)
```

---

## 9.1.2 Frequency Domain Analysis

The Fourier Transform of the impulse train is also an impulse train:

$$p(t) = \sum_n \delta(t - nT_s) \xleftrightarrow{\mathcal{F}} P(j\omega) = \omega_s \sum_k \delta(\omega - k\omega_s)$$

where $\omega_s = 2\pi f_s = 2\pi/T_s$.

By the **multiplication property** (convolution in frequency):

$$X_s(j\omega) = \frac{1}{2\pi} X(j\omega) * P(j\omega) = \frac{1}{T_s}\sum_{k=-\infty}^{\infty} X(j(\omega - k\omega_s))$$

> The spectrum of the sampled signal is **periodic repetitions** of the original spectrum, shifted by multiples of $\omega_s$ and scaled by $1/T_s$.

```
SPECTRUM OF SAMPLED SIGNAL:

Original X(jω):
         ╱╲
        ╱  ╲
       ╱    ╲
  ────╱──────╲────→ ω
    -ωₘ  0  ωₘ

Sampled X_s(jω) (fs > 2fm, no aliasing):
   ╱╲       ╱╲       ╱╲       ╱╲       ╱╲
  ╱  ╲     ╱  ╲     ╱  ╲     ╱  ╲     ╱  ╲
 ╱    ╲   ╱    ╲   ╱    ╲   ╱    ╲   ╱    ╲
╱──────╲─╱──────╲─╱──────╲─╱──────╲─╱──────╲→ ω
       -ωs     -ωm  0   ωm       ωs      2ωs

Copies separated — can recover original with LPF!
```

---

## 9.1.3 Nyquist-Shannon Sampling Theorem

### Statement

A band-limited signal $x(t)$ with $X(j\omega) = 0$ for $|\omega| > \omega_m$ can be **uniquely determined** from its samples $x(nT_s)$ if:

$$\boxed{f_s > 2f_m \quad \text{or equivalently} \quad \omega_s > 2\omega_m}$$

### Key Definitions

| Term | Definition |
|------|-----------|
| **Nyquist rate** | $f_{Nyquist} = 2f_m$ — minimum sampling rate |
| **Nyquist frequency** | $f_m$ — highest frequency in the signal |
| **Nyquist interval** | $T_{Nyquist} = 1/(2f_m)$ — maximum sampling period |
| **Oversampling** | $f_s > 2f_m$ (sampling faster than required) |
| **Undersampling** | $f_s < 2f_m$ (causes aliasing) |

### Sampling at Exactly Nyquist Rate

Sampling at $f_s = 2f_m$ is **theoretically sufficient** but leaves no guard band — the spectral copies just touch. In practice, slightly higher rates are used.

---

## 9.1.4 Reconstruction (Preview)

If the sampling theorem is satisfied, the original signal can be recovered by passing $x_s(t)$ through an **ideal low-pass filter**:

$$H(j\omega) = \begin{cases} T_s & |\omega| < \omega_s/2 \\ 0 & |\omega| > \omega_s/2 \end{cases}$$

```
RECONSTRUCTION:

X_s(jω):
   ╱╲       ╱╲       ╱╲
  ╱  ╲     ╱  ╲     ╱  ╲
 ╱    ╲   ╱    ╲   ╱    ╲
╱──────╲─╱──────╲─╱──────╲→ ω
        -ωs/2   0   ωs/2

         ┌─────────┐
H(jω) =  │  Ideal  │  (LPF with cutoff ωs/2)
         │   LPF   │
         └─────────┘

Output X(jω):
         ╱╲
        ╱  ╲
       ╱    ╲
  ────╱──────╲────→ ω         ← ORIGINAL RECOVERED!
    -ωm  0   ωm
```

---

## 9.1.5 Reconstruction Formula (Shannon Interpolation)

$$x(t) = \sum_{n=-\infty}^{\infty} x(nT_s) \cdot \text{sinc}\left(\frac{t - nT_s}{T_s}\right)$$

where $\text{sinc}(u) = \frac{\sin(\pi u)}{\pi u}$.

```
SINC INTERPOLATION:

Each sample generates a sinc function:
x(nTs)·sinc[(t-nTs)/Ts]

       x[0]·sinc        x[1]·sinc        x[2]·sinc
         │╲╱│╲╱│          │╲╱│╲╱│          │╲╱│╲╱│
  ───────┼──┼──┼──────────┼──┼──┼──────────┼──┼──┼───→ t
         0  Ts 2Ts       Ts    2Ts       2Ts
         
Sum of all shifted sincs = original x(t)
```

---

## 9.1.6 Practical Examples

### Example 1: Audio Signal

- Music bandwidth: $f_m = 20$ kHz
- Nyquist rate: $2 \times 20 = 40$ kHz
- CD sampling rate: $f_s = 44.1$ kHz (10.25% oversampling)

### Example 2: Telephone

- Voice bandwidth: $f_m = 3.4$ kHz (band-limited by filter)
- Nyquist rate: $6.8$ kHz
- Telephony standard: $f_s = 8$ kHz

### Example 3: Multi-Tone Signal

$$x(t) = 2\cos(2\pi \cdot 300t) + 5\sin(2\pi \cdot 700t) + \cos(2\pi \cdot 1500t)$$

Highest frequency: $f_m = 1500$ Hz

Nyquist rate: $f_s \geq 3000$ Hz

---

## 9.1.7 Bandpass Sampling

For **bandpass signals** with bandwidth $B$ centered at $f_c$:

$$f_s \geq 2B \quad \text{(not } 2f_c \text{!)}$$

This allows much lower sampling rates for narrowband signals at high carrier frequencies.

| Signal Type | Min. Sampling Rate |
|-------------|-------------------|
| Baseband (0 to $f_m$) | $2f_m$ |
| Bandpass ($f_L$ to $f_H$, $B = f_H - f_L$) | $2B$ (under conditions) |

```
BANDPASS SAMPLING:

Baseband: need fs ≥ 2fH     Bandpass: need fs ≥ 2B only
  │╱╲│                         │    ╱╲│
  ╱  ╲                         │   ╱  ╲
 ╱    ╲                        │  ╱    ╲
╱──────╲─→ f                   │ ╱──────╲─→ f
0    fH                      fL   fH
     fs ≥ 2fH                  B = fH-fL
                               fs ≥ 2B << 2fH
```

---

## 9.1.8 Multi-Rate Signal Processing Preview

| Concept | Operation | Effect |
|---------|-----------|--------|
| **Downsampling** by $M$ | Keep every $M$-th sample | Reduces rate by $M$ |
| **Upsampling** by $L$ | Insert $L-1$ zeros between samples | Increases rate by $L$ |
| **Decimation** | Anti-alias filter + downsample | Rate reduction |
| **Interpolation** | Upsample + smoothing filter | Rate increase |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Sampling theorem | $f_s > 2f_m$ for perfect reconstruction |
| Nyquist rate | $2f_m$ — minimum sampling rate |
| Sampled spectrum | Periodic copies spaced by $\omega_s$ |
| Reconstruction | Ideal LPF extracts base copy |
| Shannon interpolation | Sum of shifted sinc functions |
| Bandpass sampling | $f_s \geq 2B$, not $2f_{max}$ |
| Oversampling | $f_s \gg 2f_m$ — relaxes filter requirements |

---

## Quick Revision Questions

1. **A signal has components at 100, 300, and 500 Hz. What is the minimum sampling rate?**
   - $f_s > 2 \times 500 = 1000$ Hz.

2. **Why is the CD sampling rate 44.1 kHz and not exactly 40 kHz?**
   - To provide a guard band for practical (non-ideal) anti-aliasing filter rolloff.

3. **What happens in the frequency domain when you sample a signal?**
   - The spectrum is replicated at multiples of $f_s$. If $f_s > 2f_m$, copies don't overlap.

4. **What is the sinc interpolation formula?**
   - $x(t) = \sum_n x(nT_s)\,\text{sinc}((t-nT_s)/T_s)$, which perfectly reconstructs $x(t)$ from its samples.

5. **What is the advantage of bandpass sampling?**
   - Narrowband signals at high carrier frequencies can be sampled at $2B$ instead of $2f_H$, greatly reducing the required sampling rate.

---

[← Previous: Unit 8 — Stability Analysis](../08-Z-Transform/05-stability-analysis.md) | [Next: Aliasing →](02-aliasing.md)
