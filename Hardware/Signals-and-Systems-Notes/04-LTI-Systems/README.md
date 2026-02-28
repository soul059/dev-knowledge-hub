# Unit 4: LTI Systems

## Unit Overview

Linear Time-Invariant (LTI) systems are the cornerstone of signals and systems theory. Their special properties — superposition, time-invariance, and complete characterization by the impulse response — enable powerful analysis techniques including convolution, frequency response, and transfer functions.

---

## Chapters

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Impulse Response](01-impulse-response.md) | Definition, physical meaning, computing $h(t)$ and $h[n]$, examples |
| 02 | [Convolution Integral and Sum](02-convolution-integral-and-sum.md) | Convolution for CT/DT LTI analysis, graphical method, worked examples |
| 03 | [Properties of LTI Systems](03-properties-of-lti-systems.md) | Commutativity, associativity, distributivity, stability, causality via $h$ |
| 04 | [System Interconnection](04-system-interconnection.md) | Cascade, parallel, feedback connections, equivalent impulse responses |

---

## Key Formulas

| Formula | Equation |
|---------|----------|
| CT Convolution | $y(t) = x(t) * h(t) = \int_{-\infty}^{\infty} x(\tau)h(t-\tau)d\tau$ |
| DT Convolution | $y[n] = x[n] * h[n] = \sum_{k=-\infty}^{\infty} x[k]h[n-k]$ |
| Cascade | $h_{eq}(t) = h_1(t) * h_2(t)$ |
| Parallel | $h_{eq}(t) = h_1(t) + h_2(t)$ |
| BIBO Stability | $\int_{-\infty}^{\infty} |h(t)|dt < \infty$ |
| Causality | $h(t) = 0$ for $t < 0$ |

---

## Prerequisites

- Unit 1: Signal basics, standard signals ($\delta$, $u$)
- Unit 2: Signal operations (convolution basics)
- Unit 3: System properties (linearity, TI, stability, causality)

---

[← Previous Unit: Systems](../03-Systems/README.md) | [Next Unit: Fourier Series →](../05-Fourier-Series/README.md)
