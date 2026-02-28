# Chapter 5.5: Error Estimation in Numerical Differentiation

[← Previous: Higher Order Derivatives](04-higher-order-derivatives.md) | [Next: Unit 6 — Trapezoidal Rule →](../06-Numerical-Integration/01-trapezoidal-rule.md)

---

## Overview

Numerical differentiation is **inherently ill-conditioned**: small data perturbations produce large derivative errors. Understanding how truncation error and roundoff error compete is essential for choosing the right formula and step size.

---

## 1. Two Sources of Error

### Truncation Error $E_T$

Comes from truncating the Taylor series:

$$f'(x_0) = \frac{f(x_0+h) - f(x_0)}{h} - \frac{h}{2}f''(\xi) = \frac{f_1 - f_0}{h} + O(h)$$

The dropped terms are $O(h)$ for forward, $O(h^2)$ for central.

### Roundoff Error $E_R$

Each $f(x_i)$ has a roundoff error $\epsilon$:

$$E_R \approx \frac{2\epsilon}{h} \quad \text{(for first derivative, 2-point formula)}$$

---

## 2. Total Error and Optimal Step Size

$$E_{\text{total}} = E_T + E_R$$

### Forward Difference $f'(x_0)$

$$E \approx \frac{|f''|}{2}h + \frac{2\epsilon}{h}$$

Minimising: $\frac{dE}{dh} = \frac{|f''|}{2} - \frac{2\epsilon}{h^2} = 0$

$$\boxed{h_{\text{opt}} = 2\sqrt{\frac{\epsilon}{|f''|}}} \quad \text{and} \quad E_{\min} = 2\sqrt{\epsilon \, |f''|}$$

### Central Difference $f'(x_0)$

$$E \approx \frac{|f'''|}{6}h^2 + \frac{\epsilon}{h}$$

$$\boxed{h_{\text{opt}} = \left(\frac{3\epsilon}{|f'''|}\right)^{1/3}} \quad \text{and} \quad E_{\min} = O(\epsilon^{2/3})$$

### Second Derivative (Central)

$$E \approx \frac{|f^{(4)}|}{12}h^2 + \frac{4\epsilon}{h^2}$$

$$\boxed{h_{\text{opt}} = \left(\frac{48\epsilon}{|f^{(4)}|}\right)^{1/4}}$$

---

## 3. Error vs Step Size — Visual

```
  log(Error)
      │
      │ ╲                              ╱
      │  ╲  Truncation               ╱  Roundoff
      │   ╲  (slope +1 or +2)      ╱   (slope −1 or −2)
      │    ╲                     ╱
      │     ╲                  ╱
      │      ╲               ╱
      │       ╲             ╱
      │        ╲           ╱
      │         ╲_________╱
      │              ●
      │          Minimum error at h_opt
      │
      └────────────────────────────── log(h)
            ← too small   too large →
```

---

## 4. Practical Guidelines

| Machine precision | $\epsilon \approx 10^{-16}$ (double precision) |
|---|---|
| Optimal $h$ for forward diff | $\approx 10^{-8}$ |
| Optimal $h$ for central diff | $\approx 10^{-5}$ |
| Optimal $h$ for 2nd deriv (central) | $\approx 10^{-4}$ |

---

## 5. Worked Example

Estimate $f'(1)$ for $f(x) = e^x$ using the central difference formula with $h = 0.1, 0.01, 0.001$.

Exact: $f'(1) = e = 2.718281828\ldots$

| $h$ | $\frac{f(1+h) - f(1-h)}{2h}$ | Absolute Error |
|-----|------------------------------|----------------|
| 0.1 | $\frac{e^{1.1} - e^{0.9}}{0.2} = 2.72281$ | $4.5 \times 10^{-3}$ |
| 0.01 | $\frac{e^{1.01} - e^{0.99}}{0.02} = 2.71833$ | $4.5 \times 10^{-5}$ |
| 0.001 | $\frac{e^{1.001} - e^{0.999}}{0.002} = 2.718282$ | $4.5 \times 10^{-7}$ |

Error decreases by factor 100 when $h$ decreases by factor 10 → confirms $O(h^2)$.

---

## 6. Error in the Difference Table Approach

If the difference table is used, the error in the $n$-th derivative is approximately:

$$|E_n| \leq \frac{|\delta^{n+2} f|_{\max}}{(n+2) \cdot h^n}$$

This uses the first neglected difference as an error estimate.

A practical rule: **stop including differences when they become erratic** (dominated by roundoff).

---

## 7. Comparison of Methods

| Method | $f'$ Error | $f''$ Error | Optimal $h$ (f') |
|--------|-----------|-------------|-------------------|
| Forward 2-pt | $O(h)$ | $O(h)$ | $O(\sqrt{\epsilon})$ |
| Forward 3-pt | $O(h^2)$ | — | $O(\epsilon^{1/3})$ |
| Central 2-pt | $O(h^2)$ | — | $O(\epsilon^{1/3})$ |
| Central 3-pt | — | $O(h^2)$ | $O(\epsilon^{1/4})$ |
| Central 5-pt | $O(h^4)$ | $O(h^4)$ | $O(\epsilon^{1/5})$ |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| **Truncation error** | From dropping Taylor series terms; decreases with $h$ |
| **Roundoff error** | From finite arithmetic; increases as $h \to 0$ |
| **Total error** | $E = E_T + E_R$; has a minimum at optimal $h$ |
| **$h_{\text{opt}}$ (forward)** | $2\sqrt{\epsilon/|f''|}$ |
| **$h_{\text{opt}}$ (central)** | $(3\epsilon/|f'''|)^{1/3}$ |
| **Key insight** | Numerical differentiation is ill-conditioned |
| **Richardson** | Combines two step sizes to cancel leading error |

---

## Quick Revision Questions

1. **What are the two sources of error in numerical differentiation?**

2. **Derive the optimal step size for the forward difference formula.**

3. **Why is numerical differentiation more sensitive to roundoff than numerical integration?**

4. **For double precision ($\epsilon \approx 10^{-16}$), what is the optimal $h$ for the central difference first derivative?**

5. **Show that the error in the central difference $f'$ formula is $O(h^2)$ by Taylor expansion.**

6. **What is Richardson extrapolation and how does it reduce truncation error?**

---

[← Previous: Higher Order Derivatives](04-higher-order-derivatives.md) | [Next: Unit 6 — Trapezoidal Rule →](../06-Numerical-Integration/01-trapezoidal-rule.md)
