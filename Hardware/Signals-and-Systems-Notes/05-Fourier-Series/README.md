# Unit 5: Fourier Series

## Unit Overview

The Fourier Series decomposes **periodic signals** into sums of harmonically related sinusoids or complex exponentials. This is the gateway to frequency-domain analysis — representing signals by their frequency content rather than their time-domain waveform.

---

## Chapters

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Trigonometric Fourier Series](01-trigonometric-fourier-series.md) | Sine-cosine representation, computing $a_0$, $a_n$, $b_n$ |
| 02 | [Exponential Fourier Series](02-exponential-fourier-series.md) | Complex exponential form, $c_n$ coefficients, relation to trigonometric form |
| 03 | [Fourier Coefficients](03-fourier-coefficients.md) | Computation techniques, symmetry shortcuts, common waveforms |
| 04 | [Properties of Fourier Series](04-properties-of-fourier-series.md) | Linearity, time shift, differentiation, Parseval's relation |
| 05 | [Parseval's Theorem and Power Spectrum](05-parsevals-theorem.md) | Power in frequency domain, power spectral density |

---

## Key Formulas

| Formula | Equation |
|---------|----------|
| Trig FS | $x(t) = a_0 + \sum_{n=1}^{\infty}(a_n\cos n\omega_0 t + b_n\sin n\omega_0 t)$ |
| Exponential FS | $x(t) = \sum_{n=-\infty}^{\infty} c_n e^{jn\omega_0 t}$ |
| DC coefficient | $a_0 = c_0 = \frac{1}{T}\int_T x(t)dt$ |
| Parseval's | $\frac{1}{T}\int_T \|x(t)\|^2 dt = \sum \|c_n\|^2$ |

---

## Prerequisites

- Unit 1-2: Signal basics, periodicity, standard signals
- Trigonometry and complex exponentials ($e^{j\theta} = \cos\theta + j\sin\theta$)

---

[← Previous Unit: LTI Systems](../04-LTI-Systems/README.md) | [Next Unit: Fourier Transform →](../06-Fourier-Transform/README.md)
