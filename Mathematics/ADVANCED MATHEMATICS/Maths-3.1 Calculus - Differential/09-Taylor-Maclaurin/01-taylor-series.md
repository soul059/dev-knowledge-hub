# Unit 9 — Taylor & Maclaurin Series: Taylor Series

[← Previous: Mean Value Theorems](../08-Applications/07-mean-value-theorems.md) · [Next: Maclaurin Series →](02-maclaurin-series.md)

---

## Chapter Overview

**Taylor series** express a function as an infinite polynomial centered at a point $a$. They are one of the most powerful tools in analysis, providing exact or approximate representations of functions as power series.

---

## 9.1.1 Taylor's Theorem (Finite Form)

> **Taylor's Theorem:** If $f$ has continuous derivatives up to order $n+1$ on an interval containing $a$, then for any $x$ in that interval:
>
> $$f(x) = f(a) + f'(a)(x-a) + \frac{f''(a)}{2!}(x-a)^2 + \cdots + \frac{f^{(n)}(a)}{n!}(x-a)^n + R_n(x)$$

where $R_n(x)$ is the **remainder** (error term).

---

## 9.1.2 The Taylor Series (Infinite Form)

If all derivatives exist and $R_n \to 0$ as $n \to \infty$:

$$\boxed{f(x) = \sum_{n=0}^{\infty} \frac{f^{(n)}(a)}{n!}(x-a)^n}$$

$$= f(a) + f'(a)(x-a) + \frac{f''(a)}{2!}(x-a)^2 + \frac{f'''(a)}{3!}(x-a)^3 + \cdots$$

```
  f(x) is approximated by polynomials of increasing degree:

  T₀(x) = f(a)                              (constant)
  T₁(x) = f(a) + f'(a)(x-a)                (linear)
  T₂(x) = T₁ + f''(a)(x-a)²/2!            (quadratic)
  T₃(x) = T₂ + f'''(a)(x-a)³/3!           (cubic)
  ⋮
  Each Tₙ matches f and its first n derivatives at x = a
```

---

## 9.1.3 Remainder Forms

### Lagrange Remainder (most common)

$$R_n(x) = \frac{f^{(n+1)}(c)}{(n+1)!}(x-a)^{n+1}$$

for some $c$ between $a$ and $x$.

### Cauchy Remainder

$$R_n(x) = \frac{f^{(n+1)}(c)}{n!}(x-c)^n(x-a)$$

### Key Uses of Remainder

- **Error estimation:** $|R_n(x)| \leq \frac{M}{(n+1)!}|x-a|^{n+1}$ where $M = \max|f^{(n+1)}|$
- **Convergence:** If $R_n(x) \to 0$ as $n \to \infty$, the Taylor series converges to $f(x)$

---

## 9.1.4 Constructing Taylor Series

### Example 1: $e^x$ about $a = 0$

$$f^{(n)}(x) = e^x \Rightarrow f^{(n)}(0) = 1 \text{ for all } n$$

$$e^x = \sum_{n=0}^{\infty}\frac{x^n}{n!} = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots$$

Converges for all $x \in \mathbb{R}$.

### Example 2: $\ln x$ about $a = 1$

$$f(x) = \ln x, \quad f(1) = 0$$
$$f'(x) = 1/x, \quad f'(1) = 1$$
$$f''(x) = -1/x^2, \quad f''(1) = -1$$
$$f'''(x) = 2/x^3, \quad f'''(1) = 2$$
$$f^{(n)}(x) = \frac{(-1)^{n-1}(n-1)!}{x^n}, \quad f^{(n)}(1) = (-1)^{n-1}(n-1)!$$

$$\ln x = \sum_{n=1}^{\infty}\frac{(-1)^{n-1}}{n}(x-1)^n = (x-1) - \frac{(x-1)^2}{2} + \frac{(x-1)^3}{3} - \cdots$$

Converges for $0 < x \leq 2$.

### Example 3: $\sin x$ about $a = \pi/6$

$$f(\pi/6) = 1/2, \quad f'(\pi/6) = \sqrt{3}/2, \quad f''(\pi/6) = -1/2, \quad f'''(\pi/6) = -\sqrt{3}/2$$

$$\sin x = \frac{1}{2} + \frac{\sqrt{3}}{2}\left(x-\frac{\pi}{6}\right) - \frac{1}{4}\left(x-\frac{\pi}{6}\right)^2 - \frac{\sqrt{3}}{12}\left(x-\frac{\pi}{6}\right)^3 + \cdots$$

---

## 9.1.5 Geometric Interpretation

```
  y
  |
  |   * * *  f(x) = eˣ (actual function)
  |  *    . .
  | *   .     T₃  (cubic approximation)
  |*  .    .
  *. T₁     .     T₂ (quadratic)
  .  (linear)
  +────────────────→ x
  0

  Higher-degree Taylor polynomials approximate
  the function more closely near x = a
```

The Taylor polynomial $T_n(x)$ is the **unique** polynomial of degree $\leq n$ that matches $f$ and its first $n$ derivatives at $x = a$.

---

## 9.1.6 Error Estimation Example

Estimate $e^{0.1}$ using the Taylor polynomial of degree 3, and bound the error.

$$e^{0.1} \approx 1 + 0.1 + \frac{(0.1)^2}{2} + \frac{(0.1)^3}{6} = 1 + 0.1 + 0.005 + 0.000167 = 1.105167$$

Error bound: $|R_3| \leq \frac{e^{0.1}}{4!}(0.1)^4 < \frac{3}{24}(0.0001) < 0.0000125$

Actual value: $e^{0.1} = 1.10517092...$, error $\approx 0.0000039$ ✓

---

## 9.1.7 When Does the Taylor Series Converge to $f$?

The Taylor series converges to $f(x)$ if and only if $R_n(x) \to 0$ as $n \to \infty$.

| Function | Convergence Set |
|----------|----------------|
| $e^x$ | All $x \in \mathbb{R}$ |
| $\sin x$, $\cos x$ | All $x \in \mathbb{R}$ |
| $\ln(1+x)$ | $-1 < x \leq 1$ |
| $\frac{1}{1-x}$ | $\|x\| < 1$ |
| $(1+x)^\alpha$ | $\|x\| < 1$ (in general) |

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| Taylor series at $a$ | $\sum \frac{f^{(n)}(a)}{n!}(x-a)^n$ |
| Taylor polynomial degree $n$ | $T_n(x) = \sum_{k=0}^n \frac{f^{(k)}(a)}{k!}(x-a)^k$ |
| Lagrange remainder | $R_n = \frac{f^{(n+1)}(c)}{(n+1)!}(x-a)^{n+1}$ |
| Error bound | $\|R_n\| \leq \frac{M}{(n+1)!}\|x-a\|^{n+1}$ |
| Convergence | $R_n \to 0$ as $n \to \infty$ |

---

## Quick Revision Questions

**Q1.** Write the Taylor series for $e^x$ about $a = 1$ up to the $(x-1)^3$ term.

**Q2.** Find the Taylor polynomial of degree 3 for $\cos x$ about $a = \pi/4$.

**Q3.** Estimate $\sqrt{1.1}$ using a degree-2 Taylor polynomial about $a = 1$ for $f(x) = \sqrt{x}$, and find the error bound.

**Q4.** Show that the Taylor series for $e^x$ converges for all $x$ by proving $R_n \to 0$.

**Q5.** Why do we need to verify $R_n \to 0$ separately? Give an example of a function whose Taylor series does NOT converge to the function.

**Q6.** State the relationship between Taylor's theorem and the Mean Value Theorem (hint: $n = 0$).

---

[← Previous: Mean Value Theorems](../08-Applications/07-mean-value-theorems.md) · [Next: Maclaurin Series →](02-maclaurin-series.md)
