# Chapter 9.3: Bessel Functions

[← Previous: Frobenius Method](02-frobenius-method.md) | [Back to README](../README.md) | [Next: Legendre Polynomials →](04-legendre-polynomials.md)

---

## Overview

**Bessel's equation** arises in problems with cylindrical symmetry — heat conduction in cylinders, vibrating circular membranes, and electromagnetic waveguides. Its solutions, called Bessel functions, are among the most important special functions in applied mathematics.

---

## 1. Bessel's Equation of Order $\nu$

$$\boxed{x^2 y'' + x y' + (x^2 - \nu^2)y = 0}$$

- $x = 0$ is a regular singular point
- $\nu \geq 0$ is the **order** (parameter)
- Indicial equation: $r^2 - \nu^2 = 0 \implies r = \pm\nu$

---

## 2. Bessel Function of the First Kind $J_\nu(x)$

Using Frobenius with $r = \nu$:

$$\boxed{J_\nu(x) = \sum_{m=0}^{\infty} \frac{(-1)^m}{m!\,\Gamma(m + \nu + 1)} \left(\frac{x}{2}\right)^{2m+\nu}}$$

where $\Gamma$ is the gamma function ($\Gamma(n+1) = n!$ for integers).

### Special Cases

$$J_0(x) = 1 - \frac{x^2}{4} + \frac{x^4}{64} - \frac{x^6}{2304} + \cdots = \sum_{m=0}^{\infty} \frac{(-1)^m}{(m!)^2}\left(\frac{x}{2}\right)^{2m}$$

$$J_1(x) = \frac{x}{2} - \frac{x^3}{16} + \frac{x^5}{384} - \cdots = \sum_{m=0}^{\infty} \frac{(-1)^m}{m!(m+1)!}\left(\frac{x}{2}\right)^{2m+1}$$

```
  BESSEL FUNCTIONS J₀(x) AND J₁(x)

   1 │  J₀
     │╲
     │  ╲        ╱╲
     │   ╲      ╱  ╲         ╱╲
  0  │────╲───╱────╲──────╱───╲──→ x
     │     ╲╱       ╲    ╱     ╲
     │               ╲  ╱
  -1 │                ╲╱
     0    2    4    6    8   10  12

   1 │      J₁
     │    ╱╲
     │   ╱  ╲
     │  ╱    ╲       ╱╲
  0  │╱───────╲────╱───╲────→ x
     │         ╲  ╱     ╲
     │          ╲╱
  -1 │
     0    2    4    6    8   10  12
```

---

## 3. Key Properties of $J_\nu$

| Property | Formula |
|----------|---------|
| $J_\nu(0)$ | $= 0$ for $\nu > 0$; $J_0(0) = 1$ |
| Oscillation | Infinite zeros, approximately periodic for large $x$ |
| Decay | Amplitude $\sim \sqrt{2/(\pi x)}$ for large $x$ |
| Asymptotics | $J_\nu(x) \approx \sqrt{\frac{2}{\pi x}}\cos\left(x - \frac{\nu\pi}{2} - \frac{\pi}{4}\right)$ for $x \gg 1$ |

---

## 4. Bessel Function of the Second Kind $Y_\nu(x)$

$$Y_\nu(x) = \frac{J_\nu(x)\cos(\nu\pi) - J_{-\nu}(x)}{\sin(\nu\pi)}$$

For integer $\nu = n$, this is defined by a limit and involves $\ln x$:

$$Y_0(x) = \frac{2}{\pi}\left[J_0(x)\left(\ln\frac{x}{2} + \gamma\right) + \sum_{m=1}^{\infty}\frac{(-1)^{m+1}H_m}{(m!)^2}\left(\frac{x}{2}\right)^{2m}\right]$$

where $\gamma \approx 0.5772$ (Euler-Mascheroni constant) and $H_m = 1 + \frac{1}{2} + \cdots + \frac{1}{m}$.

**Key**: $Y_\nu(x) \to -\infty$ as $x \to 0^+$ (singular at origin).

```
  J₀ vs Y₀

   1 │  J₀
     │╲.....
     │  ╲     .╱╲
     │   ╲  .╱   ╲
  0  │────╲╱──────╲────→ x
     │   .╱╲       ╲
     │  ╱ Y₀╲      
  -1 │.╱     ╲
     │        ╲ (Y₀ diverges
     │         ↓  at x = 0)
     0    2    4    6    8
```

---

## 5. General Solution of Bessel's Equation

$$\boxed{y = c_1 J_\nu(x) + c_2 Y_\nu(x)}$$

If the domain includes $x = 0$, we need $c_2 = 0$ (since $Y_\nu$ diverges there).

---

## 6. Recurrence Relations

$$J_{\nu-1}(x) + J_{\nu+1}(x) = \frac{2\nu}{x}J_\nu(x)$$

$$J_{\nu-1}(x) - J_{\nu+1}(x) = 2J_\nu'(x)$$

Useful consequences:

$$J_0'(x) = -J_1(x)$$

$$\frac{d}{dx}[x^\nu J_\nu(x)] = x^\nu J_{\nu-1}(x)$$

$$\frac{d}{dx}[x^{-\nu} J_\nu(x)] = -x^{-\nu} J_{\nu+1}(x)$$

---

## 7. Zeros of Bessel Functions

The positive zeros of $J_\nu(x)$, denoted $j_{\nu,k}$ ($k = 1, 2, 3, \ldots$), are crucial in applications:

| $k$ | $j_{0,k}$ | $j_{1,k}$ |
|-----|-----------|-----------|
| 1 | 2.4048 | 3.8317 |
| 2 | 5.5201 | 7.0156 |
| 3 | 8.6537 | 10.1735 |
| 4 | 11.7915 | 13.3237 |

---

## 8. Modified Bessel Equation

$$x^2 y'' + xy' - (x^2 + \nu^2)y = 0$$

(Note the minus sign before $x^2$.) Solutions:

$$I_\nu(x) = i^{-\nu}J_\nu(ix) \quad \text{(exponentially growing)}$$
$$K_\nu(x) \quad \text{(exponentially decaying)}$$

These are non-oscillatory, unlike $J_\nu$ and $Y_\nu$.

---

## 9. Applications

| Application | Equation | Relevant $\nu$ |
|------------|----------|----------------|
| Vibrating drum | $\nabla^2 u = c^{-2}u_{tt}$ | Integer |
| Heat in cylinder | $\nabla^2 T = \alpha^{-1}T_t$ | Integer |
| Waveguides | $\nabla^2 E + k^2 E = 0$ | Integer |
| Buckling of columns | $EI\,y'' + Py = 0$ (variable) | $1/3$ |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Bessel eq.** | $x^2y'' + xy' + (x^2 - \nu^2)y = 0$ |
| **$J_\nu(x)$** | First kind, bounded at $x = 0$ |
| **$Y_\nu(x)$** | Second kind, singular at $x = 0$ |
| **General sol.** | $y = c_1 J_\nu + c_2 Y_\nu$ |
| **Recurrence** | $J_{\nu-1} + J_{\nu+1} = (2\nu/x)J_\nu$ |
| **$J_0'$** | $= -J_1$ |
| **Modified eq.** | Replace $+x^2$ with $-x^2$; solutions $I_\nu$, $K_\nu$ |
| **Asymptotics** | $J_\nu \sim \sqrt{2/(\pi x)}\cos(\cdots)$ for large $x$ |

---

## Quick Revision Questions

1. Write the first four nonzero terms of $J_0(x)$ and verify it satisfies Bessel's equation of order 0.

2. Using the recurrence relation, express $J_2(x)$ in terms of $J_0(x)$ and $J_1(x)$.

3. Why is $Y_\nu(x)$ excluded from solutions on domains containing $x = 0$?

4. Find the indicial roots for Bessel's equation of order $\nu = 1/2$. What Frobenius case applies?

5. Show that $J_{1/2}(x) = \sqrt{\frac{2}{\pi x}}\sin x$.

6. A vibrating circular membrane has modes determined by zeros of $J_0$. What are the first three eigenfrequencies proportional to?

---

[← Previous: Frobenius Method](02-frobenius-method.md) | [Back to README](../README.md) | [Next: Legendre Polynomials →](04-legendre-polynomials.md)
