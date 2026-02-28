# Unit 6: Fourier Transform

## Unit Overview

The Fourier Transform extends frequency analysis from **periodic signals** (Fourier Series) to **aperiodic signals**. It provides a continuous frequency spectrum and is the foundation for frequency-domain system analysis, filtering, and signal processing.

---

## Chapters

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Continuous-Time Fourier Transform](01-ctft.md) | Definition, derivation from FS, transform pairs, existence conditions |
| 02 | [CTFT Properties](02-ctft-properties.md) | Linearity, duality, time/frequency shifting, convolution theorem |
| 03 | [Frequency Response of LTI Systems](03-frequency-response.md) | $H(j\omega)$, magnitude/phase response, Bode plots |
| 04 | [Filtering Concepts](04-filtering-concepts.md) | Ideal filters, practical filters, bandwidth, distortionless transmission |

---

## Key Formulas

| Formula | Equation |
|---------|----------|
| Forward (Analysis) | $X(j\omega) = \int_{-\infty}^{\infty} x(t)e^{-j\omega t}dt$ |
| Inverse (Synthesis) | $x(t) = \frac{1}{2\pi}\int_{-\infty}^{\infty}X(j\omega)e^{j\omega t}d\omega$ |
| Convolution Theorem | $x*h \leftrightarrow X\cdot H$ |
| Parseval's | $\int\|x\|^2 dt = \frac{1}{2\pi}\int\|X\|^2 d\omega$ |

---

## Prerequisites

- Unit 5: Fourier Series (exponential form, $c_n$ coefficients)
- Complex exponentials and Euler's formula

---

[← Previous Unit: Fourier Series](../05-Fourier-Series/README.md) | [Next Unit: Laplace Transform →](../07-Laplace-Transform/README.md)
