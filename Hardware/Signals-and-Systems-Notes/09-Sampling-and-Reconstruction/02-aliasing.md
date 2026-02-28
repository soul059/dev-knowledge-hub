# Chapter 9.2: Aliasing

## Overview

**Aliasing** occurs when a signal is sampled below the Nyquist rate ($f_s < 2f_m$). Higher-frequency components "fold" into lower frequencies, creating distortion that **cannot be removed** after sampling. Understanding aliasing is critical for proper ADC design and signal acquisition.

---

## 9.2.1 What Is Aliasing?

When $f_s < 2f_m$, the periodic spectral copies overlap:

```
NO ALIASING (fs > 2fm):
   ╱╲       ╱╲       ╱╲
  ╱  ╲     ╱  ╲     ╱  ╲
 ╱    ╲   ╱    ╲   ╱    ╲
╱──────╲─╱──────╲─╱──────╲──→ ω
       -ωs      0      ωs
  Copies separated → no distortion

ALIASING (fs < 2fm):
   ╱╲     ╱╲     ╱╲
  ╱ ╱╲  ╱╲╱ ╲  ╱╲╱ ╲
 ╱ ╱  ╲╱  ╲  ╲╱  ╲  ╲
╱─╱────╲───╲──╲───╲──╲──→ ω
      -ωs   0    ωs
  Copies OVERLAP → irreversible distortion!
```

---

## 9.2.2 Frequency Folding

A continuous-time sinusoid at frequency $f_0$, when sampled at $f_s$, appears as:

$$f_{apparent} = |f_0 - k \cdot f_s| \quad \text{for integer } k \text{ that minimizes } f_{apparent}$$

Or equivalently, the aliased frequency maps to:

$$f_{alias} = \text{mod}(f_0 + f_s/2, \; f_s) - f_s/2$$

### The Aliasing Effect

| Actual Frequency $f_0$ | Sampled at $f_s$ | Appears as |
|------------------------|-------------------|------------|
| 100 Hz | 1000 Hz | 100 Hz ✓ |
| 400 Hz | 1000 Hz | 400 Hz ✓ |
| 600 Hz | 1000 Hz | **400 Hz** (aliased!) |
| 900 Hz | 1000 Hz | **100 Hz** (aliased!) |
| 1100 Hz | 1000 Hz | **100 Hz** (aliased!) |

### Time-Domain View

```
TWO SINUSOIDS THAT GIVE IDENTICAL SAMPLES:

x₁(t) = cos(2π·100t)  (100 Hz, below Nyquist)
x₂(t) = cos(2π·900t)  (900 Hz, above Nyquist)

Sampled at fs = 1000 Hz (Ts = 1ms):

  x₁: ─╱╲───╱╲───╱╲───→ t   (slow oscillation)
       • •  • •  • •         (samples marked •)
       
  x₂: ╱╲╱╲╱╲╱╲╱╲╱╲╱╲╱→ t   (fast oscillation)
       • •  • •  • •         (SAME samples!)
       
  After sampling, x₁ and x₂ are INDISTINGUISHABLE
```

---

## 9.2.3 Aliasing Formula

For a real signal with frequency $f_0$ sampled at $f_s$:

$$f_{alias} = \begin{cases} f_0 \mod f_s & \text{if result} \leq f_s/2 \\ f_s - (f_0 \mod f_s) & \text{if result} > f_s/2 \end{cases}$$

### Frequency Folding Diagram

```
FOLDING DIAGRAM (fs = 1000 Hz):

Apparent  
freq (Hz)
  500 ┤       ╱╲       ╱╲
      │      ╱  ╲     ╱  ╲
      │     ╱    ╲   ╱    ╲
      │    ╱      ╲ ╱      ╲
  250 ┤   ╱        ×        ╲
      │  ╱        ╱ ╲        ╲
      │ ╱        ╱   ╲        ╲
      │╱        ╱     ╲        ╲
    0 ┼───┬───┬───┬───┬───┬───┬───→ Actual freq (Hz)
      0  250  500 750 1000 1250 1500
          fs/2  fs      3fs/2  2fs
          
  Frequencies above fs/2 fold back into 0─fs/2
```

---

## 9.2.4 Anti-Aliasing Filter

To prevent aliasing, apply a **low-pass filter before sampling** to remove frequencies above $f_s/2$:

```
ANTI-ALIASING SYSTEM:

  x(t) ──→ ┌──────────┐ ──→ ┌──────┐ ──→ x[n]
            │ Anti-alias│     │Sampler│
            │   LPF     │     │ (ADC) │
            │ fc = fs/2 │     │       │
            └──────────┘     └──────┘
            
  Filter MUST be analog (before sampling)!
  Digital filtering CANNOT undo aliasing.
```

### Ideal Anti-Aliasing Filter

$$H_{aa}(j\omega) = \begin{cases} 1 & |\omega| \leq \omega_s/2 \\ 0 & |\omega| > \omega_s/2 \end{cases}$$

### Practical Considerations

| Factor | Ideal | Practical |
|--------|-------|-----------|
| Transition band | Zero width | Finite width |
| Filter type | Brick-wall LPF | Butterworth, Chebyshev, Elliptic |
| Passband | Flat | Slight ripple acceptable |
| Stopband attenuation | Infinite | 60-100+ dB typical |
| Oversampling helps | Not needed | Relaxes filter requirements |

---

## 9.2.5 Aliasing Examples

### Example 1: Audio Aliasing

Record a 15 kHz tone at $f_s = 22.05$ kHz (half the CD rate):

- Nyquist frequency: $f_s/2 = 11.025$ kHz
- 15 kHz > 11.025 kHz → **aliased**
- Apparent frequency: $22.05 - 15 = 7.05$ kHz (audible artifact!)

### Example 2: Wagon Wheel Effect

In film (24 fps), a wheel spinning at 25 rev/s:

- $f_s = 24$ Hz, $f_0 = 25$ Hz
- Aliased frequency: $25 - 24 = 1$ Hz **backward**
- The wheel appears to rotate slowly backward!

### Example 3: Stroboscopic Effect

A strobe light flashing at $f_s$ observing a rotating object at $f_0$:

- If $f_0 = f_s$: object appears **stationary**
- If $f_0 = f_s + \Delta f$: object appears to rotate at $\Delta f$

---

## 9.2.6 Quantifying Aliasing

### Signal-to-Aliasing Ratio

$$\text{SAR} = \frac{\text{Signal power in } [0, f_s/2]}{\text{Aliased power folded into } [0, f_s/2]}$$

### Oversampling Ratio (OSR)

$$\text{OSR} = \frac{f_s}{2f_m}$$

| OSR | Guard Band | Filter Complexity |
|-----|------------|-------------------|
| 1.0 (Nyquist) | None | Impossible (ideal LPF) |
| 1.25 | 25% | Very steep filter |
| 2.0 | 100% | Moderate filter |
| 4.0+ | 300%+ | Gentle filter |

---

## 9.2.7 Oversampling to Combat Aliasing

```
OVERSAMPLING ADVANTAGE:

At Nyquist rate (OSR = 1):
  ╱╲     ╱╲     
 ╱  ╲   ╱  ╲   Need brick-wall filter!
╱────╲─╱────╲──→ ω
    fs/2  fs

With 4× oversampling (OSR = 4):
  ╱╲                 ╱╲     
 ╱  ╲               ╱  ╲   Gentle filter suffices
╱────╲─ ─ ─ ─ ─ ──╱────╲──→ ω
    fm     fs/2     fs
     ←── wide gap ──→
```

Oversampling at $K$ times Nyquist:
- Spreads quantization noise over wider bandwidth
- Allows simpler analog anti-aliasing filter
- Digital decimation filter can be used after sampling

---

## 9.2.8 Aliasing in 2D (Images)

Aliasing also occurs in spatial sampling (images):

| Artifact | Cause |
|----------|-------|
| **Moiré patterns** | Regular patterns sampled below Nyquist |
| **Jagged edges** | Diagonal lines on pixel grids |
| **Color artifacts** | In Bayer-pattern sensors |

Solution: Optical low-pass filter (OLPF) in camera + anti-aliasing in rendering.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Aliasing | High frequencies fold to lower ones when $f_s < 2f_m$ |
| Irreversible | Cannot be removed after sampling |
| Folding frequency | $f_s/2$ — frequencies above this get aliased |
| Anti-aliasing filter | Analog LPF before sampler, $f_c \leq f_s/2$ |
| Oversampling | $f_s \gg 2f_m$ relaxes filter requirements |
| Wagon wheel | Visual aliasing example in film |
| Prevention | Filter first, then sample at adequate rate |

---

## Quick Revision Questions

1. **A 6 kHz signal is sampled at 8 kHz. What frequency appears?**
   - $f_{alias} = 8 - 6 = 2$ kHz. The 6 kHz tone appears as 2 kHz.

2. **Why must the anti-aliasing filter be analog?**
   - Aliasing occurs at the moment of sampling. A digital filter processes already-sampled data where aliased components are indistinguishable from real ones.

3. **What is the folding frequency?**
   - $f_s/2$ — frequencies above this value get "folded" back into the range $[0, f_s/2]$.

4. **How does oversampling help with aliasing?**
   - It creates a wider gap between the signal bandwidth and $f_s/2$, allowing a simpler (less steep) analog anti-aliasing filter.

5. **A signal has 200, 500, and 1200 Hz components. What is the minimum $f_s$ to avoid aliasing?**
   - $f_s > 2 \times 1200 = 2400$ Hz.

6. **In a movie at 24 fps, at what wheel speed does the wheel appear stationary?**
   - At 24 rev/s (or any multiple of 24 rev/s), the wheel appears stationary due to stroboscopic effect.

---

[← Previous: Sampling Theorem](01-sampling-theorem.md) | [Next: Reconstruction →](03-reconstruction.md)
