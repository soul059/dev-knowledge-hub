# Chapter 3: Periodic Function Integrals

[← Previous: King Property](02-king-property.md) | [Back to README](../README.md) | [Next: Leibniz Rule →](04-leibniz-rule.md)

---

## 3.1 Overview

When a function repeats its values at regular intervals, its integral over multiple periods has elegant properties. These results dramatically simplify calculations involving trigonometric and other periodic functions.

---

## 3.2 Periodicity — Definition Review

A function f is **periodic with period T** if:

$$f(x + T) = f(x) \quad \text{for all } x$$

```
  Common Periodic Functions:
  ┌─────────────────────────┬────────────┐
  │  Function               │  Period T  │
  ├─────────────────────────┼────────────┤
  │  sin x, cos x           │    2π      │
  │  tan x, cot x           │    π       │
  │  |sin x|, |cos x|       │    π       │
  │  sin²x, cos²x           │    π       │
  │  sin(nx), cos(nx)       │   2π/n     │
  └─────────────────────────┴────────────┘
```

---

## 3.3 Fundamental Periodicity Property

> **Theorem 1:** If f is periodic with period T, then:
>
> $$\int_0^{nT} f(x)\, dx = n\int_0^T f(x)\, dx$$
>
> for any positive integer n.

### Visual Interpretation

```
  f(x) with period T:
  
  │   ╱╲      ╱╲      ╱╲      ╱╲
  │  ╱  ╲    ╱  ╲    ╱  ╲    ╱  ╲
  ├─╱────╲──╱────╲──╱────╲──╱────╲── x
  │        ╲╱      ╲╱      ╲╱
  0    T       2T      3T      4T
  
  Each period contributes the SAME area.
  ∫₀⁴ᵀ f dx = 4 · ∫₀ᵀ f dx
```

### Proof

Split the interval [0, nT] into n sub-intervals:

$$\int_0^{nT} f(x)\, dx = \sum_{k=0}^{n-1} \int_{kT}^{(k+1)T} f(x)\, dx$$

In each sub-interval, substitute x = t + kT:

$$\int_{kT}^{(k+1)T} f(x)\, dx = \int_0^T f(t + kT)\, dt = \int_0^T f(t)\, dt$$

since f(t + kT) = f(t) by periodicity. Summing n identical terms:

$$\int_0^{nT} f(x)\, dx = n\int_0^T f(x)\, dx \quad \blacksquare$$

---

## 3.4 Shift Invariance Property

> **Theorem 2:** If f is periodic with period T, then:
>
> $$\int_a^{a+T} f(x)\, dx = \int_0^T f(x)\, dx$$
>
> The integral over **any interval of length T** is the same.

### Proof

$$\int_a^{a+T} f(x)\, dx = \int_a^T f(x)\, dx + \int_T^{a+T} f(x)\, dx$$

In the second integral, let x = t + T:

$$\int_T^{a+T} f(x)\, dx = \int_0^a f(t+T)\, dt = \int_0^a f(t)\, dt$$

So: $\int_a^{a+T} f\, dx = \int_a^T f\, dx + \int_0^a f\, dx = \int_0^T f\, dx \quad \blacksquare$

### Visual

```
  │   ╱╲       ╱╲       ╱╲ 
  │  ╱  ╲     ╱  ╲     ╱  ╲
  ├─╱────╲───╱────╲───╱────╲── x
  │        ╲╱       ╲╱
  0    a   T   a+T  2T
  
  ├───────────┤         ← [0, T]:  Area = A
       ├───────────┤    ← [a, a+T]: Also Area = A!
  
  The "lost" area on the left reappears on the right.
```

---

## 3.5 General Periodicity Property

> **Theorem 3 (Combining Theorems 1 and 2):**
>
> $$\int_a^{a+nT} f(x)\, dx = n\int_0^T f(x)\, dx$$

**Proof:** Apply shift invariance first (start from 0), then the n-period property:

$$\int_a^{a+nT} f(x)\, dx = \int_0^{nT} f(x)\, dx = n\int_0^T f(x)\, dx \quad \blacksquare$$

---

## 3.6 Worked Examples

### Example 1: Basic Application

$$\int_0^{4\pi} \sin^2 x\, dx$$

sin²x has period π. So 4π = 4 · π:

$$= 4\int_0^{\pi} \sin^2 x\, dx = 4 \cdot \frac{\pi}{2} = 2\pi$$

---

### Example 2: Absolute Value Periodic Function

$$\int_0^{5\pi} |\cos x|\, dx$$

|cos x| has period π. So 5π = 5 · π:

$$= 5\int_0^{\pi} |\cos x|\, dx = 5 \cdot 2 = 10$$

(Since ∫₀^π |cos x| dx = ∫₀^{π/2} cos x dx + ∫_{π/2}^π (−cos x) dx = 1 + 1 = 2.)

---

### Example 3: Shifted Starting Point

$$\int_5^{5+2\pi} \sin x\, dx$$

sin x has period 2π. The interval has length 2π = 1 · T:

$$= \int_0^{2\pi} \sin x\, dx = [-\cos x]_0^{2\pi} = -1 + 1 = 0$$

---

### Example 4: Combination

$$\int_0^{8\pi} |\sin x|\, dx$$

|sin x| has period π. So 8π = 8 · π:

$$= 8\int_0^{\pi} |\sin x|\, dx = 8\int_0^{\pi} \sin x\, dx = 8 \cdot 2 = 16$$

---

### Example 5: Non-Standard Multiple

$$\int_{-\pi/3}^{11\pi/3} \cos^2 x\, dx$$

Length of interval = 11π/3 − (−π/3) = 12π/3 = 4π. Period of cos²x is π. So 4π = 4T:

$$= 4\int_0^{\pi} \cos^2 x\, dx = 4 \cdot \frac{\pi}{2} = 2\pi$$

---

## 3.7 Real-World Application

**Signal Processing:** Periodic integrals appear when computing the **average power** of an AC signal:

$$P_{avg} = \frac{1}{T}\int_0^T [v(t)]^2\, dt$$

Since the signal is periodic, we can compute over any convenient period.

**Fourier Coefficients:**

$$a_n = \frac{2}{T}\int_0^T f(t)\cos(n\omega_0 t)\, dt$$

The shift invariance property means we can choose any convenient starting point.

---

## Summary Table

| Property | Formula | Condition |
|---|---|---|
| n periods from 0 | ∫₀^{nT} f dx = n ∫₀ᵀ f dx | f periodic, period T |
| Shift invariance | ∫ₐ^{a+T} f dx = ∫₀ᵀ f dx | f periodic, period T |
| General (combined) | ∫ₐ^{a+nT} f dx = n ∫₀ᵀ f dx | f periodic, period T |
| ∫₀^π sin²x dx | π/2 | Standard result |
| ∫₀^π \|sin x\| dx | 2 | Standard result |
| ∫₀^π \|cos x\| dx | 2 | Standard result |

---

## Quick Revision Questions

1. If f has period 2π, what is ∫₅^{5+6π} f(x) dx in terms of ∫₀^{2π} f(x) dx?

2. Evaluate ∫₀^{3π} |cos x| dx using the periodicity of |cos x|.

3. Prove that ∫ₐ^{a+T} f(x) dx is independent of a when f has period T.

4. Evaluate ∫₀^{10π} sin²x dx.

5. What is the period of |sin 3x|, and evaluate ∫₀^{2π} |sin 3x| dx.

6. An AC voltage v(t) = 220 sin(100πt) has period T = 0.02 s. Express ∫₀^{0.1} v²(t) dt using the periodicity property.

---

[← Previous: King Property](02-king-property.md) | [Back to README](../README.md) | [Next: Leibniz Rule →](04-leibniz-rule.md)
