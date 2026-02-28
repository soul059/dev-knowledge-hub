# Unit 9 — Taylor & Maclaurin Series: Applications

[← Previous: Standard Expansions](03-standard-expansions.md) · [Back to README →](../README.md)

---

## Chapter Overview

Taylor and Maclaurin series are not merely theoretical — they are powerful **computational tools**. This chapter demonstrates how to use series expansions to evaluate limits, approximate functions, estimate integrals, solve equations, and prove inequalities.

---

## 9.4.1 Evaluating Limits Using Series

Many limits that appear as $\frac{0}{0}$ or other indeterminate forms simplify dramatically when we replace functions by their series expansions.

### Strategy

1. Expand numerator and denominator in powers of $x$ (as $x \to 0$).
2. Cancel common factors.
3. Read off the limit.

### Worked Example 1

$$\lim_{x\to 0}\frac{\sin x - x}{x^3}$$

$$= \lim_{x\to 0}\frac{\left(x - \frac{x^3}{6}+\frac{x^5}{120}-\cdots\right)-x}{x^3}$$

$$= \lim_{x\to 0}\frac{-\frac{x^3}{6}+\frac{x^5}{120}-\cdots}{x^3}$$

$$= \lim_{x\to 0}\left(-\frac{1}{6}+\frac{x^2}{120}-\cdots\right) = \boxed{-\frac{1}{6}}$$

### Worked Example 2

$$\lim_{x\to 0}\frac{e^x - 1 - x - \frac{x^2}{2}}{x^3}$$

$$= \lim_{x\to 0}\frac{\left(1+x+\frac{x^2}{2}+\frac{x^3}{6}+\cdots\right)-1-x-\frac{x^2}{2}}{x^3}$$

$$= \lim_{x\to 0}\frac{\frac{x^3}{6}+\frac{x^4}{24}+\cdots}{x^3} = \frac{1}{6}$$

### Worked Example 3

$$\lim_{x\to 0}\frac{1-\cos x}{x^2}$$

$$= \lim_{x\to 0}\frac{1-\left(1-\frac{x^2}{2}+\frac{x^4}{24}-\cdots\right)}{x^2} = \lim_{x\to 0}\frac{\frac{x^2}{2}-\frac{x^4}{24}+\cdots}{x^2} = \frac{1}{2}$$

---

## 9.4.2 Approximating Function Values

When $x$ is small, higher-order terms contribute negligibly. We truncate the series to get a polynomial approximation.

### Linear Approximation (Tangent Line)

$$f(a+h) \approx f(a) + hf'(a)$$

### Quadratic Approximation

$$f(a+h) \approx f(a) + hf'(a) + \frac{h^2}{2}f''(a)$$

### Worked Example: Approximate $\sqrt{4.1}$

Write $\sqrt{4.1} = \sqrt{4+0.1} = 2\sqrt{1+0.025}$.

Using $\sqrt{1+u} \approx 1 + \frac{u}{2} - \frac{u^2}{8}$, with $u = 0.025$:

$$\sqrt{1+0.025} \approx 1 + 0.0125 - 0.0000390625 = 1.0124609375$$

$$\sqrt{4.1} \approx 2 \times 1.0124609375 = 2.024922$$

Exact value: $\sqrt{4.1} = 2.024846\ldots$ Error $\approx 7.6 \times 10^{-5}$.

### Worked Example: Approximate $e^{0.1}$

$$e^{0.1} \approx 1 + 0.1 + \frac{0.01}{2} + \frac{0.001}{6} + \frac{0.0001}{24}$$

$$= 1 + 0.1 + 0.005 + 0.000\overline{166} + 0.0000041\overline{6} = 1.10517\ldots$$

Exact: $e^{0.1} = 1.10517\ldots$ — remarkably accurate with just 5 terms!

---

## 9.4.3 Approximating Definite Integrals

Some integrals have **no closed-form antiderivative**. Series provide a systematic way to compute approximate values.

### Method

1. Expand the integrand as a series.
2. Integrate term by term.
3. Evaluate at the limits.

### Worked Example 1: $\int_0^1 e^{-x^2}\,dx$

The function $e^{-x^2}$ has no elementary antiderivative.

$$e^{-x^2} = 1 - x^2 + \frac{x^4}{2!} - \frac{x^6}{3!} + \frac{x^8}{4!} - \cdots$$

$$\int_0^1 e^{-x^2}\,dx = \left[x - \frac{x^3}{3} + \frac{x^5}{5\cdot 2!} - \frac{x^7}{7\cdot 3!} + \frac{x^9}{9\cdot 4!} - \cdots\right]_0^1$$

$$= 1 - \frac{1}{3} + \frac{1}{10} - \frac{1}{42} + \frac{1}{216} - \cdots$$

$$\approx 1 - 0.3333 + 0.1 - 0.0238 + 0.00463 - \cdots \approx 0.7468$$

(Exact value $= \frac{\sqrt{\pi}}{2}\text{erf}(1) = 0.74682\ldots$)

### Worked Example 2: $\int_0^{0.5} \frac{\sin x}{x}\,dx$

$$\frac{\sin x}{x} = 1 - \frac{x^2}{3!} + \frac{x^4}{5!} - \frac{x^6}{7!} + \cdots$$

$$\int_0^{0.5} \frac{\sin x}{x}\,dx = \left[x - \frac{x^3}{3\cdot 3!} + \frac{x^5}{5\cdot 5!} - \cdots\right]_0^{0.5}$$

$$= 0.5 - \frac{0.125}{18} + \frac{0.03125}{600} - \cdots$$

$$= 0.5 - 0.00694 + 0.0000521 - \cdots \approx 0.4931$$

---

## 9.4.4 Error Estimation

### Alternating Series Estimation Theorem

For an alternating series with terms decreasing in absolute value to 0, the error from truncating after $n$ terms is bounded by the absolute value of the first omitted term:

$$|R_n| \leq |a_{n+1}|$$

```
  Partial sum              Actual sum
  ─────────────────────────────────────────────
  
  S₁  ●─────────────────────────────── too big
  S₂  ────●──────────────────────────  too small
  S₃  ──────●────────────────────────  too big
  S₄  ───────●───────────────────────  too small
  ...        ↓
  S    ────────● ← converges (actual value)
```

### Lagrange Remainder Bound

For non-alternating series, use the Lagrange remainder:

$$|R_n(x)| \leq \frac{M}{(n+1)!}|x-a|^{n+1}$$

where $M = \max|f^{(n+1)}(c)|$ on the interval between $a$ and $x$.

### Worked Example

How many terms of $e^x = \sum \frac{x^n}{n!}$ are needed to compute $e^1$ with error $< 10^{-6}$?

We need $\frac{1}{(n+1)!} < 10^{-6}$, so $(n+1)! > 10^6$.

$9! = 362880$, $10! = 3628800 > 10^6$. So $n+1 = 10$, i.e., $n = 9$ (use terms up to $x^9 / 9!$).

---

## 9.4.5 Proving Inequalities

Series can prove analytical inequalities rigorously.

### Example 1: $\sin x < x$ for $x > 0$

$$\sin x = x - \frac{x^3}{3!} + \frac{x^5}{5!} - \cdots$$

Group terms in pairs starting from the subtracted term:

$$x - \sin x = \frac{x^3}{3!} - \frac{x^5}{5!} + \cdots = \frac{x^3}{6}\left(1 - \frac{x^2}{20}+\cdots\right) > 0 \text{ for small } x$$

More rigorously, each bracketed pair $\frac{x^{2k-1}}{(2k-1)!} - \frac{x^{2k+1}}{(2k+1)!} = \frac{x^{2k-1}}{(2k-1)!}\left(1-\frac{x^2}{2k(2k+1)}\right) > 0$ for $0 < x < \sqrt{2k(2k+1)}$.

### Example 2: $e^x \geq 1 + x$ for all $x$

$$e^x = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots$$

$$e^x - (1+x) = \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots$$

For $x \geq 0$: every term is non-negative, so $e^x \geq 1+x$.

For $x < 0$: use the Lagrange remainder — $e^x = 1+x+\frac{e^c}{2}x^2 \geq 1+x$ since $e^c > 0$.

---

## 9.4.6 Finding Series Solutions to Equations

### Worked Example: Find the first 3 nonzero terms of the solution to $y = \sin(y) + x$

Let $y = a_1 x + a_2 x^2 + a_3 x^3 + \cdots$

From $y - \sin y = x$:

$$y - \sin y = \frac{y^3}{6} - \frac{y^5}{120}+\cdots = x$$

Substitute $y \approx a_1 x$:

$$\frac{a_1^3 x^3}{6} = x \implies a_1^3 = 6 \implies a_1 = 6^{1/3}$$

This technique is called **series reversion** and is used extensively in applied mathematics.

---

## 9.4.7 Summing Series Using Known Expansions

### Worked Example 1

Find $\sum_{n=0}^{\infty}\frac{1}{n!}$.

This is $e^x$ at $x = 1$:

$$\sum_{n=0}^{\infty}\frac{1}{n!} = e$$

### Worked Example 2

Find $\sum_{n=0}^{\infty}\frac{(-1)^n}{2n+1} = 1 - \frac{1}{3}+\frac{1}{5}-\frac{1}{7}+\cdots$

This is $\tan^{-1}(x)$ at $x = 1$:

$$\sum_{n=0}^{\infty}\frac{(-1)^n}{2n+1} = \tan^{-1}(1) = \frac{\pi}{4}$$

This is the famous **Leibniz formula for $\pi$**.

### Worked Example 3

Find $\sum_{n=1}^{\infty}\frac{(-1)^{n+1}}{n} = 1 - \frac{1}{2}+\frac{1}{3}-\frac{1}{4}+\cdots$

This is $\ln(1+x)$ at $x = 1$:

$$\sum_{n=1}^{\infty}\frac{(-1)^{n+1}}{n} = \ln 2$$

---

## 9.4.8 Application Decision Flowchart

```
  ┌────────────────────┐
  │ What do you need?  │
  └────────┬───────────┘
           │
     ┌─────┼──────┬──────────┬──────────────┐
     ▼     ▼      ▼          ▼              ▼
  Limit  Approx  Integral  Inequality    Sum of
         value   estimate               series
     │     │      │          │              │
     ▼     ▼      ▼          ▼              ▼
  Expand  Taylor  Expand &  Group terms   Identify
  num &   about   integrate to show       as known
  denom   nearby  term by   non-         f(x) at
  → 0/0   point   term      negative     x = c
  cancel
```

---

## Summary Table

| Application | Method | Key Idea |
|------------|--------|----------|
| Evaluating limits | Expand in series, cancel common powers | Replaces L'Hôpital for complex indeterminate forms |
| Approximation | Truncate Taylor polynomial | Linear, quadratic, or higher-order approximation |
| Definite integrals | Integrate series term by term | Works even when no closed-form antiderivative exists |
| Error estimation | Alternating series theorem or Lagrange remainder | Bound the first omitted term |
| Proving inequalities | Show remainder has definite sign | Group terms in pairs or use remainder theorem |
| Summing series | Recognize as known expansion at specific $x$ | Connects numerical series to known functions |

---

## Quick Revision Questions

**Q1.** Evaluate $\displaystyle\lim_{x\to 0}\frac{e^x - 1 - x}{x^2}$ using series expansion.

**Q2.** Approximate $\sin(0.1)$ using the first three nonzero terms of the Maclaurin series. Estimate the error.

**Q3.** Estimate $\displaystyle\int_0^{0.5}\frac{\ln(1+x)}{x}\,dx$ using the first four terms of the series.

**Q4.** How many terms of $\cos x = \sum \frac{(-1)^n x^{2n}}{(2n)!}$ are needed to compute $\cos(0.5)$ with error $< 10^{-8}$?

**Q5.** Use series to prove: $\cos x \geq 1 - \frac{x^2}{2}$ for all $x$.

**Q6.** Find $\displaystyle\sum_{n=0}^{\infty}\frac{(-1)^n}{n!\cdot 2^n}$ by identifying the appropriate function and substitution.

---

[← Previous: Standard Expansions](03-standard-expansions.md) · [Back to README →](../README.md)
