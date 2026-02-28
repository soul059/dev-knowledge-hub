# Unit 9 — Taylor & Maclaurin Series: Standard Expansions

[← Previous: Maclaurin Series](02-maclaurin-series.md) · [Next: Applications of Series →](04-applications.md)

---

## Chapter Overview

This chapter is a **comprehensive reference** of all standard Maclaurin/Taylor expansions needed in calculus. It includes expansions derived by substitution, multiplication, composition, and term-by-term operations on known series.

---

## 9.3.1 Master List of Standard Expansions

### Exponential Family

$$e^x = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots \qquad (\text{all } x)$$

$$e^{-x} = 1 - x + \frac{x^2}{2!} - \frac{x^3}{3!} + \cdots \qquad (\text{all } x)$$

$$a^x = e^{x\ln a} = 1 + x\ln a + \frac{(x\ln a)^2}{2!} + \cdots \qquad (\text{all } x)$$

### Trigonometric Family

$$\sin x = x - \frac{x^3}{3!} + \frac{x^5}{5!} - \cdots \qquad (\text{all } x)$$

$$\cos x = 1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \cdots \qquad (\text{all } x)$$

$$\tan x = x + \frac{x^3}{3} + \frac{2x^5}{15} + \frac{17x^7}{315} + \cdots \qquad (|x| < \pi/2)$$

### Inverse Trigonometric Family

$$\sin^{-1}x = x + \frac{x^3}{6} + \frac{3x^5}{40} + \frac{15x^7}{336} + \cdots \qquad (|x| \leq 1)$$

$$\tan^{-1}x = x - \frac{x^3}{3} + \frac{x^5}{5} - \frac{x^7}{7} + \cdots \qquad (|x| \leq 1)$$

### Hyperbolic Family

$$\sinh x = x + \frac{x^3}{3!} + \frac{x^5}{5!} + \cdots \qquad (\text{all } x)$$

$$\cosh x = 1 + \frac{x^2}{2!} + \frac{x^4}{4!} + \cdots \qquad (\text{all } x)$$

$$\tanh x = x - \frac{x^3}{3} + \frac{2x^5}{15} - \cdots \qquad (|x| < \pi/2)$$

### Logarithmic Family

$$\ln(1+x) = x - \frac{x^2}{2} + \frac{x^3}{3} - \frac{x^4}{4} + \cdots \qquad (-1 < x \leq 1)$$

$$\ln(1-x) = -x - \frac{x^2}{2} - \frac{x^3}{3} - \frac{x^4}{4} - \cdots \qquad (-1 \leq x < 1)$$

$$\ln\frac{1+x}{1-x} = 2\left(x + \frac{x^3}{3} + \frac{x^5}{5} + \cdots\right) \qquad (|x| < 1)$$

### Binomial / Algebraic

$$\frac{1}{1-x} = 1 + x + x^2 + x^3 + \cdots \qquad (|x| < 1)$$

$$\frac{1}{1+x} = 1 - x + x^2 - x^3 + \cdots \qquad (|x| < 1)$$

$$\frac{1}{(1-x)^2} = 1 + 2x + 3x^2 + 4x^3 + \cdots \qquad (|x| < 1)$$

$$(1+x)^\alpha = 1 + \alpha x + \frac{\alpha(\alpha-1)}{2!}x^2 + \cdots \qquad (|x| < 1)$$

$$\sqrt{1+x} = 1 + \frac{x}{2} - \frac{x^2}{8} + \frac{x^3}{16} - \cdots \qquad (|x| < 1)$$

$$\frac{1}{\sqrt{1+x}} = 1 - \frac{x}{2} + \frac{3x^2}{8} - \frac{5x^3}{16} + \cdots \qquad (|x| < 1)$$

---

## 9.3.2 Techniques for Deriving New Expansions

### Technique 1: Substitution

Replace $x$ with an expression in a known series.

**Example:** $e^{-x^2}$

Substitute $-x^2$ for $x$ in $e^x$:

$$e^{-x^2} = 1 - x^2 + \frac{x^4}{2!} - \frac{x^6}{3!} + \cdots$$

### Technique 2: Multiplication of Series

**Example:** $e^x \sin x$

$$e^x \sin x = \left(1 + x + \frac{x^2}{2} + \frac{x^3}{6} + \cdots\right)\left(x - \frac{x^3}{6} + \cdots\right)$$

Collecting terms:

$$= x + x^2 + \frac{x^3}{3} - \frac{x^5}{30} + \cdots$$

### Technique 3: Division of Series

**Example:** $\tan x = \frac{\sin x}{\cos x}$

Perform long division of the $\sin x$ series by the $\cos x$ series:

$$\frac{x - x^3/6 + x^5/120 - \cdots}{1 - x^2/2 + x^4/24 - \cdots} = x + \frac{x^3}{3} + \frac{2x^5}{15} + \cdots$$

### Technique 4: Term-by-Term Differentiation

If $f(x) = \sum a_n x^n$, then $f'(x) = \sum n a_n x^{n-1}$ (within radius of convergence).

**Example:** From $\frac{1}{1-x} = \sum x^n$:

$$\frac{1}{(1-x)^2} = \frac{d}{dx}\left(\frac{1}{1-x}\right) = \sum_{n=1}^{\infty}nx^{n-1} = 1 + 2x + 3x^2 + \cdots$$

### Technique 5: Term-by-Term Integration

$\int f(x)\,dx = C + \sum \frac{a_n}{n+1}x^{n+1}$

**Example:** From $\frac{1}{1+x^2} = 1 - x^2 + x^4 - \cdots$:

$$\tan^{-1}x = \int_0^x \frac{dt}{1+t^2} = x - \frac{x^3}{3} + \frac{x^5}{5} - \cdots$$

---

## 9.3.3 Quick Reference: Comparison of Similar Series

```
  sin x  = x - x³/3! + x⁵/5! - ...    (alternating, odd powers)
  sinh x = x + x³/3! + x⁵/5! + ...    (all positive, odd powers)

  cos x  = 1 - x²/2! + x⁴/4! - ...    (alternating, even powers)
  cosh x = 1 + x²/2! + x⁴/4! + ...    (all positive, even powers)

  ln(1+x)  = x - x²/2 + x³/3 - ...    (alternating)
  ln(1-x)  = -x - x²/2 - x³/3 - ...   (all negative)

  tan⁻¹x = x - x³/3 + x⁵/5 - ...     (alternating, odd powers)
  tanh⁻¹x = x + x³/3 + x⁵/5 + ...    (all positive, odd powers)
```

The pattern: **hyperbolic** versions have **no alternation** compared to trig versions.

---

## 9.3.4 Convergence Summary

| Series | Radius of Convergence |
|--------|-----------------------|
| $e^x$, $\sin x$, $\cos x$, $\sinh x$, $\cosh x$ | $\infty$ (all $x$) |
| $\ln(1+x)$ | $R = 1$ |
| $(1+x)^\alpha$ | $R = 1$ |
| $\frac{1}{1-x}$ | $R = 1$ |
| $\tan x$ | $R = \pi/2$ |
| $\tan^{-1}x$ | $R = 1$ |

---

## Summary Table

| Technique | When to Use |
|-----------|------------|
| Direct computation | Function's $n^{th}$ derivative at 0 has a pattern |
| Substitution | Replace $x$ with $-x$, $x^2$, $2x$, etc. in known series |
| Multiplication | Product of two functions with known series |
| Division | Quotient of two functions |
| Differentiation | Derive from antiderivative's known series |
| Integration | Derive from derivative's known series |

---

## Quick Revision Questions

**Q1.** Find the Maclaurin series for $e^{x^2}$ up to $x^8$.

**Q2.** Use the series for $\ln(1+x)$ and $\ln(1-x)$ to derive the series for $\ln\frac{1+x}{1-x}$.

**Q3.** Find the first four nonzero terms of the Maclaurin series for $\sec x = 1/\cos x$.

**Q4.** Derive the Maclaurin series for $\frac{x}{e^x - 1}$ up to $x^3$.

**Q5.** Using known series, find the expansion of $e^{\sin x}$ up to $x^4$.

**Q6.** Verify that $\sinh^{-1}x = x - \frac{x^3}{6} + \frac{3x^5}{40} - \cdots$ by using the binomial series for $(1+x^2)^{-1/2}$ and integrating.

---

[← Previous: Maclaurin Series](02-maclaurin-series.md) · [Next: Applications of Series →](04-applications.md)
