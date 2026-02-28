# Unit 8: Z-Transform

## Overview

The Z-Transform is the discrete-time counterpart of the Laplace Transform. It converts discrete-time signals and difference equations into algebraic expressions in the complex $z$-plane, enabling analysis and design of digital filters, sampled-data systems, and discrete-time controllers.

---

## Chapters

1. [Definition and Region of Convergence](01-definition-and-roc.md)  
   Z-Transform definition, unilateral ZT, ROC properties, common transform pairs

2. [Properties of Z-Transform](02-properties.md)  
   Linearity, time shifting, convolution, z-domain differentiation, initial/final value theorems

3. [Inverse Z-Transform](03-inverse-z-transform.md)  
   Partial fraction expansion, power series, long division, contour integration

4. [Discrete-Time Transfer Function](04-transfer-function.md)  
   System function $H(z)$, pole-zero plots, FIR/IIR, difference equations

5. [Stability and Causality Analysis](05-stability-analysis.md)  
   Unit circle criterion, Jury stability test, pole placement, relationship to Laplace

---

## Key Formulas

| Formula | Expression |
|---------|-----------|
| Z-Transform | $X(z) = \sum_{n=-\infty}^{\infty} x[n]\,z^{-n}$ |
| Unilateral ZT | $X(z) = \sum_{n=0}^{\infty} x[n]\,z^{-n}$ |
| Inverse ZT | $x[n] = \frac{1}{2\pi j}\oint X(z)\,z^{n-1}\,dz$ |
| Transfer function | $H(z) = Y(z)/X(z)$ |
| Stability | All poles inside unit circle $|z| < 1$ |
| z ↔ s relation | $z = e^{sT}$ |

---

[← Previous: Unit 7 — Laplace Transform](../07-Laplace-Transform/README.md) | [Next: Unit 9 — Sampling →](../09-Sampling-and-Reconstruction/README.md)
