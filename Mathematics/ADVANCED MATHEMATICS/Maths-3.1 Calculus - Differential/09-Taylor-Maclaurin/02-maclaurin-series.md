# Unit 9 — Taylor & Maclaurin Series: Maclaurin Series

[← Previous: Taylor Series](01-taylor-series.md) · [Next: Standard Expansions →](03-standard-expansions.md)

---

## Chapter Overview

A **Maclaurin series** is a Taylor series centered at $a = 0$. It is the most commonly used form in practice. This chapter derives the Maclaurin series for all major functions from first principles.

---

## 9.2.1 Definition

$$\boxed{f(x) = \sum_{n=0}^{\infty}\frac{f^{(n)}(0)}{n!}x^n = f(0) + f'(0)x + \frac{f''(0)}{2!}x^2 + \frac{f'''(0)}{3!}x^3 + \cdots}$$

This is simply the Taylor series with $a = 0$.

---

## 9.2.2 Deriving Key Maclaurin Series

### $e^x$

| $n$ | $f^{(n)}(x)$ | $f^{(n)}(0)$ |
|-----|---------------|---------------|
| 0 | $e^x$ | 1 |
| 1 | $e^x$ | 1 |
| 2 | $e^x$ | 1 |
| $n$ | $e^x$ | 1 |

$$\boxed{e^x = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots = \sum_{n=0}^{\infty}\frac{x^n}{n!}}$$

Valid for all $x$.

---

### $\sin x$

| $n$ | $f^{(n)}(x)$ | $f^{(n)}(0)$ |
|-----|---------------|---------------|
| 0 | $\sin x$ | 0 |
| 1 | $\cos x$ | 1 |
| 2 | $-\sin x$ | 0 |
| 3 | $-\cos x$ | $-1$ |
| 4 | $\sin x$ | 0 |

Pattern: $0, 1, 0, -1, 0, 1, 0, -1, \ldots$ → only odd powers survive.

$$\boxed{\sin x = x - \frac{x^3}{3!} + \frac{x^5}{5!} - \frac{x^7}{7!} + \cdots = \sum_{n=0}^{\infty}\frac{(-1)^n x^{2n+1}}{(2n+1)!}}$$

Valid for all $x$.

---

### $\cos x$

| $n$ | $f^{(n)}(0)$ |
|-----|---------------|
| 0 | 1 |
| 1 | 0 |
| 2 | $-1$ |
| 3 | 0 |
| 4 | 1 |

Only even powers survive.

$$\boxed{\cos x = 1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \frac{x^6}{6!} + \cdots = \sum_{n=0}^{\infty}\frac{(-1)^n x^{2n}}{(2n)!}}$$

Valid for all $x$.

---

### $\ln(1+x)$

$f(0) = 0$, $f'(x) = (1+x)^{-1}$, $f''(x) = -(1+x)^{-2}$, ...

$f^{(n)}(0) = (-1)^{n-1}(n-1)!$

$$\boxed{\ln(1+x) = x - \frac{x^2}{2} + \frac{x^3}{3} - \frac{x^4}{4} + \cdots = \sum_{n=1}^{\infty}\frac{(-1)^{n-1}x^n}{n}}$$

Valid for $-1 < x \leq 1$.

---

### $(1+x)^\alpha$ — The Binomial Series

$$\boxed{(1+x)^\alpha = 1 + \alpha x + \frac{\alpha(\alpha-1)}{2!}x^2 + \frac{\alpha(\alpha-1)(\alpha-2)}{3!}x^3 + \cdots}$$

$$= \sum_{n=0}^{\infty}\binom{\alpha}{n}x^n$$

where $\binom{\alpha}{n} = \frac{\alpha(\alpha-1)\cdots(\alpha-n+1)}{n!}$.

Valid for $|x| < 1$.

**Special cases:**

| $\alpha$ | Series |
|----------|--------|
| $-1$ | $\frac{1}{1+x} = 1 - x + x^2 - x^3 + \cdots$ |
| $1/2$ | $\sqrt{1+x} = 1 + \frac{x}{2} - \frac{x^2}{8} + \frac{x^3}{16} - \cdots$ |
| $-1/2$ | $\frac{1}{\sqrt{1+x}} = 1 - \frac{x}{2} + \frac{3x^2}{8} - \frac{5x^3}{16} + \cdots$ |

---

### $\tan^{-1} x$

Since $\frac{d}{dx}\tan^{-1}x = \frac{1}{1+x^2} = 1 - x^2 + x^4 - x^6 + \cdots$

Integrating term by term:

$$\boxed{\tan^{-1}x = x - \frac{x^3}{3} + \frac{x^5}{5} - \frac{x^7}{7} + \cdots = \sum_{n=0}^{\infty}\frac{(-1)^n x^{2n+1}}{2n+1}}$$

Valid for $|x| \leq 1$.

Setting $x = 1$: $\frac{\pi}{4} = 1 - \frac{1}{3} + \frac{1}{5} - \frac{1}{7} + \cdots$ (Leibniz formula for $\pi$).

---

## 9.2.3 Relationship Between Series

```
  eˣ  ─── set x = ix ───→  cos x + i·sin x  (Euler)

  1/(1-x) ─── integrate ──→  -ln(1-x)

  1/(1+x²) ── integrate ──→  tan⁻¹(x)

  eˣ  ─── set x = -x ───→  e⁻ˣ
       ─── subtract ──────→  sinh x = (eˣ - e⁻ˣ)/2
       ─── add ───────────→  cosh x = (eˣ + e⁻ˣ)/2
```

---

## 9.2.4 Euler's Formula (Bonus)

Substituting $ix$ into the series for $e^x$:

$$e^{ix} = 1 + ix + \frac{(ix)^2}{2!} + \frac{(ix)^3}{3!} + \cdots$$

$$= \left(1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \cdots\right) + i\left(x - \frac{x^3}{3!} + \frac{x^5}{5!} - \cdots\right)$$

$$\boxed{e^{ix} = \cos x + i\sin x}$$

Setting $x = \pi$: $e^{i\pi} + 1 = 0$ — **Euler's Identity**, the most beautiful equation in mathematics.

---

## Summary Table

| Function | Maclaurin Series | Convergence |
|----------|-----------------|-------------|
| $e^x$ | $\sum \frac{x^n}{n!}$ | All $x$ |
| $\sin x$ | $\sum \frac{(-1)^n x^{2n+1}}{(2n+1)!}$ | All $x$ |
| $\cos x$ | $\sum \frac{(-1)^n x^{2n}}{(2n)!}$ | All $x$ |
| $\ln(1+x)$ | $\sum \frac{(-1)^{n-1}x^n}{n}$ | $(-1, 1]$ |
| $(1+x)^\alpha$ | $\sum \binom{\alpha}{n}x^n$ | $(-1, 1)$ |
| $\tan^{-1}x$ | $\sum \frac{(-1)^n x^{2n+1}}{2n+1}$ | $[-1, 1]$ |
| $\frac{1}{1-x}$ | $\sum x^n$ | $(-1, 1)$ |

---

## Quick Revision Questions

**Q1.** Derive the Maclaurin series for $e^{-x}$ and hence for $\cosh x = \frac{e^x + e^{-x}}{2}$.

**Q2.** Using the Maclaurin series, verify that $\sin^2 x + \cos^2 x = 1$ up to the $x^4$ term.

**Q3.** Find the Maclaurin series for $\frac{x}{1+x^2}$ up to $x^7$.

**Q4.** Use the series for $\ln(1+x)$ with $x = 1$ to show $\ln 2 = 1 - \frac{1}{2} + \frac{1}{3} - \frac{1}{4} + \cdots$.

**Q5.** Derive the Maclaurin series for $\sin^{-1}x$ using the binomial series for $(1-x^2)^{-1/2}$.

**Q6.** Show that $e^{i\pi} + 1 = 0$ using Euler's formula.

---

[← Previous: Taylor Series](01-taylor-series.md) · [Next: Standard Expansions →](03-standard-expansions.md)
