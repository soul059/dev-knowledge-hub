# Unit 7: Laplace Transform

## Unit Overview

The Laplace Transform generalizes the Fourier Transform by introducing a complex frequency variable $s = \sigma + j\omega$. It handles a broader class of signals (including growing exponentials and unstable systems) and is the primary tool for continuous-time system analysis, control theory, and circuit analysis.

---

## Chapters

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Definition and ROC](01-definition-and-roc.md) | Bilateral/unilateral LT, region of convergence, properties of ROC |
| 02 | [Laplace Transform Properties](02-properties.md) | Linearity, shifting, differentiation, initial/final value theorems |
| 03 | [Inverse Laplace Transform](03-inverse-laplace.md) | Partial fraction expansion, residue method, table lookup |
| 04 | [Transfer Function](04-transfer-function.md) | System function $H(s)$, poles, zeros, pole-zero plots |
| 05 | [System Analysis Using Laplace](05-system-analysis.md) | Solving ODEs, stability, transient/steady-state response |

---

## Key Formulas

| Formula | Equation |
|---------|----------|
| Forward | $X(s) = \int_{-\infty}^{\infty} x(t)e^{-st}dt$ |
| Inverse | $x(t) = \frac{1}{2\pi j}\oint X(s)e^{st}ds$ |
| Unilateral | $X(s) = \int_{0^-}^{\infty}x(t)e^{-st}dt$ |
| Transfer function | $H(s) = Y(s)/X(s)$ |
| Relation to FT | $X(j\omega) = X(s)\big|_{s=j\omega}$ (if $j\omega$ axis in ROC) |

---

[← Previous Unit: Fourier Transform](../06-Fourier-Transform/README.md) | [Next Unit: Z-Transform →](../08-Z-Transform/README.md)
