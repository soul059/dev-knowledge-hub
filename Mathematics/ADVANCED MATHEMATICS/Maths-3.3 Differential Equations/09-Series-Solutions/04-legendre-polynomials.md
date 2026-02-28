# Chapter 9.4: Legendre Polynomials

[← Previous: Bessel Functions](03-bessel-functions.md) | [Back to README](../README.md) | [Next: Laplace Transform — Definition & Properties →](../10-Laplace-Transform/01-definition-and-properties.md)

---

## Overview

**Legendre's equation** arises in problems with spherical symmetry — gravitational fields, electrostatics, and quantum mechanics. Its polynomial solutions, the **Legendre polynomials** $P_n(x)$, form a complete orthogonal set on $[-1, 1]$.

---

## 1. Legendre's Equation

$$\boxed{(1 - x^2)y'' - 2xy' + n(n+1)y = 0}$$

Equivalently: $\dfrac{d}{dx}\left[(1 - x^2)\dfrac{dy}{dx}\right] + n(n+1)y = 0$ (Sturm-Liouville form)

- Singular points: $x = \pm 1$
- $x = 0$ is an ordinary point → power series works

---

## 2. Series Solution

Let $y = \sum_{k=0}^{\infty} a_k x^k$ about $x_0 = 0$.

**Recurrence relation**:

$$a_{k+2} = \frac{(k-n)(k+n+1)}{(k+2)(k+1)} a_k$$

Two series:
- **Even series** (from $a_0$): $y_1 = a_0\left(1 + \sum \text{even powers}\right)$
- **Odd series** (from $a_1$): $y_2 = a_1\left(x + \sum \text{odd powers}\right)$

### Termination → Polynomials

When $k = n$: $a_{n+2} = 0$, and the series **terminates**.

- If $n$ is even: the even series terminates → polynomial
- If $n$ is odd: the odd series terminates → polynomial

These polynomial solutions, normalized so that $P_n(1) = 1$, are the **Legendre polynomials**.

---

## 3. Legendre Polynomials $P_n(x)$

| $n$ | $P_n(x)$ |
|-----|----------|
| 0 | $1$ |
| 1 | $x$ |
| 2 | $\frac{1}{2}(3x^2 - 1)$ |
| 3 | $\frac{1}{2}(5x^3 - 3x)$ |
| 4 | $\frac{1}{8}(35x^4 - 30x^2 + 3)$ |
| 5 | $\frac{1}{8}(63x^5 - 70x^3 + 15x)$ |

```
  LEGENDRE POLYNOMIALS P₀ ... P₃

   1 │─────P₀──────────────────── 
     │       ╱P₁               P₃╱
     │      ╱    P₂╲          ╱ 
     │     ╱        ╲       ╱   
  0  │────╱──────────╲────╱────
     │   ╱            ╲ ╱      
     │  ╱         P₂───╳       
     │ ╱P₁            ╱P₃     
  -1 │╱──────────────╱────────
    -1        0         1
```

---

## 4. Rodrigues' Formula

$$\boxed{P_n(x) = \frac{1}{2^n n!}\frac{d^n}{dx^n}(x^2 - 1)^n}$$

This provides a compact way to compute $P_n(x)$.

### Verification for $n = 2$

$(x^2 - 1)^2 = x^4 - 2x^2 + 1$

$\frac{d^2}{dx^2}(x^4 - 2x^2 + 1) = 12x^2 - 4$

$P_2 = \frac{1}{2^2 \cdot 2!}(12x^2 - 4) = \frac{12x^2 - 4}{8} = \frac{3x^2 - 1}{2}$ ✓

---

## 5. Generating Function

$$\frac{1}{\sqrt{1 - 2xt + t^2}} = \sum_{n=0}^{\infty} P_n(x)\,t^n, \quad |t| < 1$$

---

## 6. Recurrence Relations

$$(n+1)P_{n+1}(x) = (2n+1)x\,P_n(x) - n\,P_{n-1}(x)$$

$$P_n'(x) = \frac{n}{1-x^2}[P_{n-1}(x) - xP_n(x)]$$

---

## 7. Orthogonality

$$\boxed{\int_{-1}^{1} P_m(x)\,P_n(x)\,dx = \begin{cases} 0 & m \neq n \\ \dfrac{2}{2n+1} & m = n \end{cases}}$$

This is the key property for expanding functions in Legendre series:

$$f(x) = \sum_{n=0}^{\infty} c_n P_n(x), \quad c_n = \frac{2n+1}{2}\int_{-1}^{1} f(x)\,P_n(x)\,dx$$

---

## 8. Special Values

| Property | Value |
|----------|-------|
| $P_n(1)$ | $1$ |
| $P_n(-1)$ | $(-1)^n$ |
| $P_{2k+1}(0)$ | $0$ |
| $P_{2k}(0)$ | $\frac{(-1)^k (2k)!}{2^{2k}(k!)^2}$ |
| Parity | $P_n(-x) = (-1)^n P_n(x)$ |

---

## 9. Legendre Functions of the Second Kind $Q_n(x)$

The second linearly independent solution (non-polynomial, singular at $x = \pm 1$):

$$Q_0(x) = \frac{1}{2}\ln\frac{1+x}{1-x}$$

$$Q_1(x) = \frac{x}{2}\ln\frac{1+x}{1-x} - 1$$

General solution: $y = c_1 P_n(x) + c_2 Q_n(x)$

On $[-1,1]$: only $P_n$ is bounded → $c_2 = 0$ for physical problems.

---

## 10. Applications

```
  GRAVITATIONAL POTENTIAL EXPANSION

  For axially symmetric mass distribution:

  Φ(r,θ) = Σ (Aₙrⁿ + Bₙ/r^(n+1)) Pₙ(cos θ)
            n=0

  ┌──────────────────┐
  │     Earth         │
  │   ╭───────╮       │     r
  │   │       │──────────→ P(r,θ)
  │   │   ●   │  θ ↗  │
  │   │  axis │       │
  │   ╰───────╯       │
  └──────────────────┘

  P₀ = monopole (dominant)
  P₁ = dipole
  P₂ = quadrupole (Earth's oblateness)
```

| Application | Use of $P_n$ |
|------------|-------------|
| Gravitational multipole | $\Phi = \sum (A_n r^n + B_n r^{-(n+1)})P_n(\cos\theta)$ |
| Electrostatics | Potential in spherical coordinates |
| Quantum mechanics | Angular momentum (spherical harmonics $Y_l^m$) |
| Curve fitting | Legendre polynomial approximation |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Legendre eq.** | $(1-x^2)y'' - 2xy' + n(n+1)y = 0$ |
| **$P_n(x)$** | Polynomial solution, $P_n(1) = 1$ |
| **Rodrigues** | $P_n = \frac{1}{2^n n!}\frac{d^n}{dx^n}(x^2-1)^n$ |
| **Recurrence** | $(n+1)P_{n+1} = (2n+1)xP_n - nP_{n-1}$ |
| **Orthogonality** | $\int_{-1}^1 P_m P_n\,dx = \frac{2}{2n+1}\delta_{mn}$ |
| **$Q_n(x)$** | Second kind, singular at $x = \pm 1$ |
| **Generating fn.** | $(1-2xt+t^2)^{-1/2} = \sum P_n(x)t^n$ |

---

## Quick Revision Questions

1. Use the recurrence relation to compute $P_4(x)$ from $P_2$ and $P_3$.

2. Verify Rodrigues' formula for $P_3(x)$.

3. Compute $\int_{-1}^{1} x^2 P_2(x)\,dx$ using the Legendre expansion of $x^2$.

4. Why do physical problems on $[-1,1]$ exclude $Q_n(x)$?

5. Show that $P_n(x)$ has exactly $n$ real zeros in $(-1, 1)$.

6. Find the Legendre series coefficients $c_0, c_1, c_2$ for $f(x) = x^3$ on $[-1, 1]$.

---

[← Previous: Bessel Functions](03-bessel-functions.md) | [Back to README](../README.md) | [Next: Laplace Transform — Definition & Properties →](../10-Laplace-Transform/01-definition-and-properties.md)
