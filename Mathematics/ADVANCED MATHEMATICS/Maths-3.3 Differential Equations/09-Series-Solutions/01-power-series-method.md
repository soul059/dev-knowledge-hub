# Chapter 9.1: Power Series Method

[← Previous: Phase Plane Analysis](../08-Systems-of-ODEs/03-phase-plane.md) | [Back to README](../README.md) | [Next: Frobenius Method →](02-frobenius-method.md)

---

## Overview

When an ODE has variable coefficients, the methods of Chapters 5–6 often fail. The **power series method** represents the solution as $y = \sum a_n x^n$ and determines the coefficients from a recurrence relation. This works at **ordinary points** of the ODE.

---

## 1. Ordinary and Singular Points

For $y'' + P(x)y' + Q(x)y = 0$:

- $x = x_0$ is an **ordinary point** if $P(x)$ and $Q(x)$ are both analytic at $x_0$
- $x = x_0$ is a **singular point** if either $P$ or $Q$ is not analytic at $x_0$

### Standard Form

From $a_2(x)y'' + a_1(x)y' + a_0(x)y = 0$:

$$P(x) = \frac{a_1(x)}{a_2(x)}, \quad Q(x) = \frac{a_0(x)}{a_2(x)}$$

Singular points occur where $a_2(x) = 0$.

```
  CLASSIFICATION OF POINTS

  Given: a₂(x)y'' + a₁(x)y' + a₀(x)y = 0

                 ┌───────────────┐
                 │ Is a₂(x₀) = 0?│
                 └───────┬───────┘
                    ╱         ╲
                 No             Yes
                 ╱               ╲
          ┌──────────┐    ┌──────────────┐
          │ ORDINARY  │    │   SINGULAR    │
          │  POINT    │    │    POINT      │
          └──────────┘    └──────┬───────┘
                              (Ch. 9.2)
```

### Example

$$x^2 y'' + xy' + (x^2 - n^2)y = 0 \quad \text{(Bessel's equation)}$$

$$P = \frac{1}{x}, \quad Q = \frac{x^2 - n^2}{x^2}$$

$x = 0$ is a singular point (both $P$ and $Q$ blow up).

---

## 2. The Power Series Method

### Procedure at an Ordinary Point

1. Assume $y = \displaystyle\sum_{n=0}^{\infty} a_n (x - x_0)^n$

2. Compute $y'$ and $y''$ term by term

3. Substitute into the ODE

4. Collect powers of $(x - x_0)$

5. Set coefficient of each power to zero → **recurrence relation**

6. Solve for $a_n$ in terms of $a_0$ and $a_1$ (the two arbitrary constants)

```
  POWER SERIES METHOD — STEP BY STEP

  ┌────────────────────────────────────┐
  │ 1. Assume y = Σ aₙxⁿ              │
  └──────────────┬─────────────────────┘
                 ▼
  ┌────────────────────────────────────┐
  │ 2. Differentiate:                  │
  │    y' = Σ naₙxⁿ⁻¹                 │
  │    y'' = Σ n(n-1)aₙxⁿ⁻²           │
  └──────────────┬─────────────────────┘
                 ▼
  ┌────────────────────────────────────┐
  │ 3. Substitute into ODE            │
  └──────────────┬─────────────────────┘
                 ▼
  ┌────────────────────────────────────┐
  │ 4. Shift indices so all sums      │
  │    have the same power of x       │
  └──────────────┬─────────────────────┘
                 ▼
  ┌────────────────────────────────────┐
  │ 5. Set each coefficient = 0       │
  │    → Recurrence relation          │
  └──────────────┬─────────────────────┘
                 ▼
  ┌────────────────────────────────────┐
  │ 6. Solve for aₙ; write y         │
  └────────────────────────────────────┘
```

---

## 3. Analytic Guarantee

**Theorem**: If $x_0$ is an ordinary point, then the power series solution converges at least for $|x - x_0| < R$, where $R$ is the distance from $x_0$ to the nearest singular point (in the complex plane).

---

## 4. Worked Examples

### Example 1: $y'' + y = 0$ (Verification)

Let $y = \sum_{n=0}^{\infty} a_n x^n$. Then $y'' = \sum_{n=2}^{\infty} n(n-1)a_n x^{n-2}$.

Substitute: $\sum_{n=2}^{\infty} n(n-1)a_n x^{n-2} + \sum_{n=0}^{\infty} a_n x^n = 0$

Shift first sum: let $m = n-2$, so $n = m+2$:

$$\sum_{m=0}^{\infty} (m+2)(m+1)a_{m+2} x^m + \sum_{m=0}^{\infty} a_m x^m = 0$$

**Recurrence**: $(m+2)(m+1)a_{m+2} + a_m = 0$

$$a_{m+2} = \frac{-a_m}{(m+2)(m+1)}$$

Even coefficients (from $a_0$):
$$a_2 = \frac{-a_0}{2!}, \quad a_4 = \frac{a_0}{4!}, \quad a_6 = \frac{-a_0}{6!}, \ldots$$

Odd coefficients (from $a_1$):
$$a_3 = \frac{-a_1}{3!}, \quad a_5 = \frac{a_1}{5!}, \ldots$$

$$y = a_0\underbrace{\left(1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \cdots\right)}_{\cos x} + a_1\underbrace{\left(x - \frac{x^3}{3!} + \frac{x^5}{5!} - \cdots\right)}_{\sin x}$$

### Example 2: $y'' - xy = 0$ (Airy's Equation)

Let $y = \sum a_n x^n$:

$$\sum_{n=2}^{\infty} n(n-1)a_n x^{n-2} - \sum_{n=0}^{\infty} a_n x^{n+1} = 0$$

Shift: $\sum_{m=0}^{\infty}(m+2)(m+1)a_{m+2}x^m - \sum_{m=1}^{\infty} a_{m-1}x^m = 0$

From $m = 0$: $2a_2 = 0 \implies a_2 = 0$

**Recurrence** (for $m \geq 1$):

$$a_{m+2} = \frac{a_{m-1}}{(m+2)(m+1)}$$

Starting from $a_0$: $a_3 = \frac{a_0}{6}$, $a_6 = \frac{a_0}{180}$, $a_9 = \frac{a_0}{12960}$, etc.

Starting from $a_1$: $a_4 = \frac{a_1}{12}$, $a_7 = \frac{a_1}{504}$, etc.

$$y = a_0\left(1 + \frac{x^3}{6} + \frac{x^6}{180} + \cdots\right) + a_1\left(x + \frac{x^4}{12} + \frac{x^7}{504} + \cdots\right)$$

These are the **Airy functions** $\text{Ai}(x)$ and $\text{Bi}(x)$.

### Example 3: $(1-x^2)y'' - 2xy' + n(n+1)y = 0$ (Legendre's Equation, about $x_0 = 0$)

Singular points at $x = \pm 1$, so $R = 1$.

**Recurrence**:

$$a_{k+2} = \frac{k(k+1) - n(n+1)}{(k+2)(k+1)} a_k = \frac{(k-n)(k+n+1)}{(k+2)(k+1)} a_k$$

When $k = n$: $a_{n+2} = 0$, and the series terminates → **polynomial solution** (Legendre polynomial $P_n(x)$).

### Example 4: $y'' + xy' + y = 0$

$x = 0$ is ordinary. Let $y = \sum a_n x^n$:

$$\sum_{n=2}^{\infty} n(n-1)a_n x^{n-2} + \sum_{n=1}^{\infty} n a_n x^n + \sum_{n=0}^{\infty} a_n x^n = 0$$

Shifting and combining:

$m = 0$: $2a_2 + a_0 = 0 \implies a_2 = -\frac{a_0}{2}$

$m \geq 1$: $(m+2)(m+1)a_{m+2} + (m+1)a_m = 0$

$$a_{m+2} = \frac{-a_m}{m+2}$$

$$y = a_0\left(1 - \frac{x^2}{2} + \frac{x^4}{8} - \frac{x^6}{48} + \cdots\right) + a_1\left(x - \frac{x^3}{3} + \frac{x^5}{15} - \cdots\right)$$

The first series is $a_0 e^{-x^2/2}$ (verify by expansion).

---

## 5. Index Shifting — Key Technique

To combine sums, make all powers of $x$ match:

| Original Sum | Substitution | New Sum |
|-------------|-------------|---------|
| $\sum_{n=2}^{\infty} n(n-1)a_n x^{n-2}$ | $m = n-2$ | $\sum_{m=0}^{\infty} (m+2)(m+1)a_{m+2} x^m$ |
| $\sum_{n=0}^{\infty} a_n x^{n+1}$ | $m = n+1$ | $\sum_{m=1}^{\infty} a_{m-1} x^m$ |
| $\sum_{n=1}^{\infty} n a_n x^n$ | $m = n$ | $\sum_{m=1}^{\infty} m\, a_m x^m$ |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Ordinary point** | $P(x)$, $Q(x)$ analytic at $x_0$ |
| **Singular point** | Where $a_2(x_0) = 0$ |
| **Trial solution** | $y = \sum a_n (x-x_0)^n$ |
| **Recurrence** | Relates $a_{n+2}$ (or $a_{n+k}$) to earlier $a_n$ |
| **Convergence radius** | $R = $ distance to nearest singular point |
| **Two series** | One from $a_0$, one from $a_1$ |

---

## Quick Revision Questions

1. Find the radius of convergence of the power series solution of $(1+x^2)y'' + 2xy' = 0$ about $x_0 = 0$.

2. Use the power series method to solve $y'' - 2xy' + 4y = 0$ about $x_0 = 0$. Find the first four nonzero terms.

3. Identify all singular points of $(x^2 - 4)y'' + 3xy' + y = 0$.

4. For Airy's equation $y'' = xy$, compute $a_3, a_4, a_5, a_6$ in terms of $a_0$ and $a_1$.

5. Why does the power series method always give exactly two linearly independent solutions at an ordinary point?

6. Solve $(1 - x)y' + y = 0$ by power series and verify against the exact solution.

---

[← Previous: Phase Plane Analysis](../08-Systems-of-ODEs/03-phase-plane.md) | [Back to README](../README.md) | [Next: Frobenius Method →](02-frobenius-method.md)
