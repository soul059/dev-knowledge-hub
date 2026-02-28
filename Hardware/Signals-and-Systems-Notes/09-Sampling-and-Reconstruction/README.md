# Unit 9: Sampling and Reconstruction

## Overview

Sampling bridges the continuous-time and discrete-time worlds. This unit covers the **Nyquist-Shannon Sampling Theorem**, the phenomenon of aliasing, reconstruction using interpolation, and practical considerations in analog-to-digital and digital-to-analog conversion.

---

## Chapters

1. [Sampling Theorem](01-sampling-theorem.md)  
   Ideal sampling, Nyquist rate, Nyquist frequency, spectral analysis of sampled signals

2. [Aliasing](02-aliasing.md)  
   Frequency folding, anti-aliasing filters, spectral overlap, practical examples

3. [Reconstruction and Interpolation](03-reconstruction.md)  
   Ideal reconstruction (sinc interpolation), zero-order hold, first-order hold, practical DAC

4. [Practical Sampling Systems](04-practical-systems.md)  
   ADC/DAC architectures, quantization, sample-and-hold, oversampling, sigma-delta modulation

---

## Key Formulas

| Formula | Expression |
|---------|-----------|
| Nyquist rate | $f_s \geq 2f_{max}$ |
| Sampling period | $T_s = 1/f_s$ |
| Sampled signal spectrum | $X_s(j\omega) = \frac{1}{T_s}\sum_{k=-\infty}^{\infty}X(j(\omega - k\omega_s))$ |
| Ideal reconstruction | $x(t) = \sum_{n} x[n]\,\text{sinc}\left(\frac{t - nT_s}{T_s}\right)$ |
| Frequency resolution | $\Delta f = f_s / N$ |

---

[← Previous: Unit 8 — Z-Transform](../08-Z-Transform/README.md) | [Back to Main README →](../README.md)
