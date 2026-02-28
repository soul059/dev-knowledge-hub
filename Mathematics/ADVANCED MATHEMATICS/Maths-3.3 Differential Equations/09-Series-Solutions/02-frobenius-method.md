# Chapter 9.2: Frobenius Method

[← Previous: Power Series Method](01-power-series-method.md) | [Back to README](../README.md) | [Next: Bessel Functions →](03-bessel-functions.md)

---

## Overview

When $x_0$ is a **regular singular point**, the standard power series method fails. The **Frobenius method** (method of extended power series) seeks solutions of the form $y = x^r \sum a_n x^n$, where $r$ may be fractional or negative, determined by the **indicial equation**.

---

## 1. Regular vs Irregular Singular Points

If $x = x_0$ is a singular point of $y'' + P(x)y' + Q(x)y = 0$:

- **Regular singular point**: $(x - x_0)P(x)$ and $(x - x_0)^2 Q(x)$ are both analytic at $x_0$
- **Irregular singular point**: otherwise

```
  CLASSIFICATION AT SINGULAR POINT x₀

  ┌──────────────────────────────────────┐
  │ Check: (x-x₀)P(x) analytic at x₀?  │
  │ AND  (x-x₀)²Q(x) analytic at x₀?   │
  └────────────────┬─────────────────────┘
              ╱         ╲
           Both          At least
           Yes           one No
            ╱              ╲
   ┌───────────────┐  ┌───────────────┐
   │    REGULAR     │  │   IRREGULAR    │
   │ SINGULAR POINT │  │ SINGULAR POINT │
   │ (Frobenius ok) │  │ (no general    │
   │                │  │  method)       │
   └───────────────┘  └───────────────┘
```

### Examples

| Equation | Singular Point | $(x-x_0)P$ | $(x-x_0)^2Q$ | Type |
|----------|---------------|-----------|-------------|------|
| $x^2y'' + xy' + (x^2-n^2)y = 0$ | $x=0$ | $1$ (analytic) | $x^2-n^2$ (analytic) | Regular |
| $x^3y'' + xy' + y = 0$ | $x=0$ | $1/x$ (not analytic) | $1/x$ (not analytic) | Irregular |
| $x(x-1)y'' + 3y' + 2y = 0$ | $x=0$ | $3/(x-1)$ (analytic at 0) | $2x/(x-1)$ (analytic at 0) | Regular |

---

## 2. The Frobenius Series

Assume:

$$y = x^r \sum_{n=0}^{\infty} a_n x^n = \sum_{n=0}^{\infty} a_n x^{n+r}, \quad a_0 \neq 0$$

### The Indicial Equation

Substituting into the ODE and collecting the lowest power of $x$ gives:

$$r(r-1) + p_0 r + q_0 = 0$$

where $p_0 = \lim_{x \to 0} xP(x)$ and $q_0 = \lim_{x \to 0} x^2 Q(x)$.

This quadratic in $r$ gives two roots $r_1 \geq r_2$.

---

## 3. Three Cases for Frobenius Solutions

Let $r_1$ and $r_2$ be the indicial roots with $r_1 \geq r_2$.

### Case 1: $r_1 - r_2 \notin \mathbb{Z}$ (roots do not differ by an integer)

$$y_1 = x^{r_1}\sum_{n=0}^{\infty} a_n x^n, \quad y_2 = x^{r_2}\sum_{n=0}^{\infty} b_n x^n$$

Both Frobenius series work directly.

### Case 2: $r_1 = r_2$ (equal roots)

$$y_1 = x^{r_1}\sum_{n=0}^{\infty} a_n x^n$$
$$y_2 = y_1 \ln x + x^{r_1}\sum_{n=1}^{\infty} c_n x^n$$

The second solution always contains a $\ln x$ term.

### Case 3: $r_1 - r_2 \in \mathbb{Z}^+$ (roots differ by a positive integer)

$$y_1 = x^{r_1}\sum_{n=0}^{\infty} a_n x^n$$
$$y_2 = C\, y_1 \ln x + x^{r_2}\sum_{n=0}^{\infty} b_n x^n$$

$C$ may or may not be zero. If $C = 0$, no $\ln x$ term appears.

```
  FROBENIUS — THREE CASES

  ┌───────────────────────────┐
  │ Indicial roots r₁ ≥ r₂   │
  └─────────────┬─────────────┘
                ▼
    ┌───────────┼──────────┐
    ▼           ▼          ▼
  r₁-r₂       r₁=r₂    r₁-r₂ =
  ∉ ℤ                   pos int
    │           │          │
    ▼           ▼          ▼
  Two         y₁ as     y₁ from r₁
  Frobenius   Frobenius  y₂ MAY have
  series      y₂ has    ln x term
  directly    ln x
```

---

## 4. Worked Examples

### Example 1: $2x y'' + y' + y = 0$

Standard form: $y'' + \frac{1}{2x}y' + \frac{1}{2x}y = 0$

$x = 0$ is singular. Check: $xP = \frac{1}{2}$ ✓, $x^2Q = \frac{x}{2}$ ✓ → Regular singular point.

$p_0 = \frac{1}{2}$, $q_0 = 0$

**Indicial equation**: $r(r-1) + \frac{1}{2}r = 0 \implies r^2 - \frac{1}{2}r = 0 \implies r(r - \frac{1}{2}) = 0$

$r_1 = \frac{1}{2}$, $r_2 = 0$. Difference = $\frac{1}{2} \notin \mathbb{Z}$ → **Case 1**.

Let $y = \sum a_n x^{n+r}$. Substituting into $2xy'' + y' + y = 0$:

$$\sum_{n=0}^{\infty} [2(n+r)(n+r-1) + (n+r)] a_n x^{n+r-1} + \sum_{n=0}^{\infty} a_n x^{n+r} = 0$$

Shift second sum: replace $n$ with $n-1$:

$$(n+r)(2n+2r-1)a_n + a_{n-1} = 0, \quad n \geq 1$$

$$a_n = \frac{-a_{n-1}}{(n+r)(2n+2r-1)}$$

**For $r_1 = 1/2$**: $a_n = \frac{-a_{n-1}}{(n+\frac{1}{2})(2n)} = \frac{-a_{n-1}}{n(2n+1)}$

$a_1 = \frac{-a_0}{3}$, $a_2 = \frac{a_0}{30}$, $a_3 = \frac{-a_0}{630}$

$$y_1 = x^{1/2}\left(1 - \frac{x}{3} + \frac{x^2}{30} - \frac{x^3}{630} + \cdots\right)$$

**For $r_2 = 0$**: $a_n = \frac{-a_{n-1}}{n(2n-1)}$

$a_1 = \frac{-a_0}{1}$, $a_2 = \frac{a_0}{6}$, $a_3 = \frac{-a_0}{90}$

$$y_2 = 1 - x + \frac{x^2}{6} - \frac{x^3}{90} + \cdots$$

### Example 2: $xy'' + y = 0$ (Equal Roots)

$P = 0$, $Q = 1/x$. $xP = 0$, $x^2 Q = x$ → Regular singular point.

Indicial: $r(r-1) = 0 \implies r_1 = 1, r_2 = 0$. Difference = 1 → **Case 3**.

For $r_1 = 1$: Recurrence gives $a_n = \frac{-a_{n-1}}{n(n+1)}$

$$y_1 = x\left(1 - \frac{x}{2} + \frac{x^2}{12} - \frac{x^3}{144} + \cdots\right)$$

### Example 3: Bessel's Equation of Order 0

$$x^2y'' + xy' + x^2y = 0$$

$p_0 = 1$, $q_0 = 0$. Indicial: $r^2 = 0 \implies r_1 = r_2 = 0$ → **Case 2**.

Recurrence: $n^2 a_n + a_{n-2} = 0 \implies a_n = \frac{-a_{n-2}}{n^2}$

Odd terms vanish. Even terms:

$$a_2 = \frac{-1}{4}, \quad a_4 = \frac{1}{64}, \quad a_6 = \frac{-1}{2304}$$

$$J_0(x) = 1 - \frac{x^2}{4} + \frac{x^4}{64} - \frac{x^6}{2304} + \cdots = \sum_{m=0}^{\infty} \frac{(-1)^m}{(m!)^2}\left(\frac{x}{2}\right)^{2m}$$

The second solution $Y_0(x)$ involves $J_0(x)\ln x$ + another series (see Chapter 9.3).

---

## 5. Summary Table

| Concept | Details |
|---------|---------|
| **Regular singular point** | $(x-x_0)P$ and $(x-x_0)^2Q$ analytic |
| **Frobenius form** | $y = x^r \sum a_n x^n$, $a_0 \neq 0$ |
| **Indicial equation** | $r(r-1) + p_0 r + q_0 = 0$ |
| **Case 1** ($r_1 - r_2 \notin \mathbb{Z}$) | Two Frobenius series directly |
| **Case 2** ($r_1 = r_2$) | Second solution has $\ln x$ |
| **Case 3** ($r_1 - r_2 \in \mathbb{Z}^+$) | Second solution may have $\ln x$ |
| **The larger root** $r_1$ | Always gives a valid Frobenius solution |

---

## Quick Revision Questions

1. Classify $x = 0$ for $(x^2 - x)y'' + (3x - 1)y' + y = 0$ as ordinary, regular singular, or irregular singular.

2. Find the indicial equation and roots for $x^2 y'' + x(\frac{1}{2} + x)y' - \frac{1}{2}y = 0$.

3. For $2x^2 y'' + xy' - (x+1)y = 0$, determine which Frobenius case applies and find the first three terms of $y_1$.

4. Why does the larger root $r_1$ always give a valid Frobenius solution?

5. Find the Frobenius solution for $xy'' + y' - y = 0$ (modified Bessel equation of order 0).

6. Explain the role of $\ln x$ in Case 2 and its physical significance.

---

[← Previous: Power Series Method](01-power-series-method.md) | [Back to README](../README.md) | [Next: Bessel Functions →](03-bessel-functions.md)
