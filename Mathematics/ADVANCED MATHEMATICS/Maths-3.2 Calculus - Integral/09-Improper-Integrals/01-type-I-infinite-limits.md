# Chapter 1: Type I — Improper Integrals with Infinite Limits

[← Previous: Work and Fluid Pressure](../08-Applications-Other/04-work-and-fluid-pressure.md) | [Back to README](../README.md) | [Next: Type II — Discontinuous Integrands →](02-type-II-discontinuous-integrand.md)

---

## 1.1 Overview

An **improper integral** extends integration to cases where either the interval is unbounded (Type I) or the integrand has a singularity (Type II). These integrals may converge to a finite value or diverge to infinity.

---

## 1.2 Definition — Type I (Infinite Limits)

> **Type I improper integrals** have one or both limits extending to infinity.

$$\int_a^{\infty} f(x)\, dx = \lim_{t \to \infty} \int_a^t f(x)\, dx$$

$$\int_{-\infty}^{b} f(x)\, dx = \lim_{t \to -\infty} \int_t^b f(x)\, dx$$

$$\int_{-\infty}^{\infty} f(x)\, dx = \int_{-\infty}^{c} f(x)\, dx + \int_c^{\infty} f(x)\, dx$$

(Both must converge independently for the total to converge.)

```
  Convergent:                    Divergent:
  
  │╲                             │╲
  │ ╲                            │ ╲         
  │  ╲___                       │  ╲
  │      ‾‾‾───────→ 0          │   ╲────────→ slow decay
  ├──────────────── x            ├──────────────── x
  a                              a
  
  ∫ₐ^∞ f dx = FINITE            ∫ₐ^∞ f dx = ∞
  (decays fast enough)           (doesn't decay fast enough)
```

---

## 1.3 The p-Integral (Most Important Example)

> $$\int_1^{\infty} \frac{1}{x^p}\, dx = \begin{cases} \frac{1}{p-1} & \text{if } p > 1 \quad \text{(CONVERGES)} \\[6pt] \infty & \text{if } p \leq 1 \quad \text{(DIVERGES)} \end{cases}$$

### Proof (p ≠ 1)

$$\int_1^t \frac{dx}{x^p} = \left[\frac{x^{1-p}}{1-p}\right]_1^t = \frac{t^{1-p} - 1}{1-p}$$

As t → ∞:
- If p > 1: exponent 1 − p < 0, so t^{1-p} → 0. Result: 1/(p−1). ✓
- If p < 1: exponent 1 − p > 0, so t^{1-p} → ∞. Result: ∞. ✗

### Proof (p = 1)

$$\int_1^t \frac{dx}{x} = \ln t \to \infty$$

### Visual Summary

```
  p-INTEGRAL CONVERGENCE
  ═══════════════════════
  
     p = 0.5:  1/√x     → DIVERGES (too slow)
     p = 1:    1/x       → DIVERGES (borderline)
     p = 1.5:  1/x^1.5  → CONVERGES = 2
     p = 2:    1/x²     → CONVERGES = 1
     p = 3:    1/x³     → CONVERGES = 1/2
  
  ┌──────────────────────────────────┐
  │  CRITICAL THRESHOLD: p = 1      │
  │  p > 1 → converges              │
  │  p ≤ 1 → diverges               │
  └──────────────────────────────────┘
```

---

## 1.4 Worked Examples

### Example 1: Basic Convergent Integral

$$\int_1^{\infty} \frac{1}{x^2}\, dx = \lim_{t\to\infty}\left[-\frac{1}{x}\right]_1^t = \lim_{t\to\infty}\left(-\frac{1}{t} + 1\right) = 1$$

---

### Example 2: Exponential Decay

$$\int_0^{\infty} e^{-x}\, dx = \lim_{t\to\infty}[-e^{-x}]_0^t = \lim_{t\to\infty}(-e^{-t} + 1) = 1$$

More generally: $\int_0^{\infty} e^{-ax}\, dx = \frac{1}{a}$ for a > 0.

---

### Example 3: Both Limits Infinite

$$\int_{-\infty}^{\infty} \frac{dx}{1+x^2} = \lim_{a\to -\infty}\lim_{b\to\infty} [\arctan x]_a^b = \frac{\pi}{2} - \left(-\frac{\pi}{2}\right) = \pi$$

---

### Example 4: Divergent Despite Oscillation

$$\int_1^{\infty} \frac{\sin x}{x}\, dx$$

This integral **converges conditionally** (by Dirichlet's test), even though it doesn't converge absolutely. Its value is related to the sine integral.

---

### Example 5: Gaussian Integral

$$\int_{-\infty}^{\infty} e^{-x^2}\, dx = \sqrt{\pi}$$

This famous result (proven by converting to polar coordinates) is fundamental to probability theory (the normal distribution).

---

## 1.5 Important Convergent Integrals

| Integral | Value | Condition |
|---|---|---|
| ∫₁^∞ 1/xᵖ dx | 1/(p−1) | p > 1 |
| ∫₀^∞ e⁻ᵃˣ dx | 1/a | a > 0 |
| ∫₀^∞ xⁿe⁻ˣ dx | n! | n = 0,1,2,... (Gamma function) |
| ∫₋∞^∞ 1/(1+x²) dx | π | — |
| ∫₋∞^∞ e⁻ˣ² dx | √π | Gaussian |
| ∫₀^∞ sin x/x dx | π/2 | Dirichlet integral |

---

## 1.6 Divergence

An improper integral **diverges** if the limit:
- equals ±∞
- does not exist (oscillation)

> **Warning:** For ∫₋∞^∞, you must evaluate both halves independently. The limit of ∫₋ₜᵗ (the "Cauchy principal value") may exist even when the integral diverges.

### Cautionary Example

$$\int_{-\infty}^{\infty} x\, dx$$

$$\lim_{t\to\infty}\int_{-t}^t x\, dx = \lim_{t\to\infty} 0 = 0 \quad \text{(Cauchy PV)}$$

But: $\int_{-\infty}^0 x\, dx = -\infty$ and $\int_0^{\infty} x\, dx = +\infty$

The integral **diverges** — the halves don't independently converge.

---

## Summary Table

| Concept | Key Point |
|---|---|
| Type I definition | One or both limits → ±∞ |
| Evaluation | Replace ∞ with limit variable t, take limit |
| p-test (∫₁^∞ 1/xᵖ) | Converges iff p > 1 |
| Exponential ∫₀^∞ e⁻ᵃˣ | Converges for a > 0 |
| Both limits infinite | Must split and evaluate independently |
| Cauchy principal value | Not sufficient for convergence |

---

## Quick Revision Questions

1. Evaluate ∫₁^∞ 1/x³ dx.

2. For what values of p does ∫₁^∞ 1/xᵖ dx converge? State the value.

3. Evaluate ∫₀^∞ xe⁻ˣ dx using integration by parts.

4. Show that ∫₋∞^∞ 1/(1+x²) dx = π.

5. Explain why ∫₋∞^∞ x dx diverges even though its Cauchy principal value is 0.

6. Does ∫₁^∞ 1/(x ln x) dx converge or diverge? (Hint: substitute u = ln x.)

---

[← Previous: Work and Fluid Pressure](../08-Applications-Other/04-work-and-fluid-pressure.md) | [Back to README](../README.md) | [Next: Type II — Discontinuous Integrands →](02-type-II-discontinuous-integrand.md)
