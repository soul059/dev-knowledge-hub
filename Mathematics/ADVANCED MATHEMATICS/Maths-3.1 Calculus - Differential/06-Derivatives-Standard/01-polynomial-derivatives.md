# Unit 6 — Derivatives of Standard Functions: Polynomial Derivatives

[← Previous: Parametric Differentiation](../05-Differentiation-Rules/06-parametric-differentiation.md) · [Next: Trigonometric Derivatives →](02-trigonometric-derivatives.md)

---

## Chapter Overview

Polynomials are the simplest functions to differentiate. Every polynomial derivative follows directly from the **power rule** and **sum rule**. This chapter provides a systematic reference for differentiating all polynomials and rational functions.

---

## 6.1.1 Review of the Power Rule

$$\frac{d}{dx}(x^n) = nx^{n-1} \quad \text{for all } n \in \mathbb{R}$$

Combined with the constant multiple and sum rules:

$$\frac{d}{dx}\left[\sum_{k=0}^{n} a_k x^k\right] = \sum_{k=1}^{n} k \cdot a_k \cdot x^{k-1}$$

---

## 6.1.2 Standard Derivatives Table

| $f(x)$ | $f'(x)$ | Notes |
|---------|----------|-------|
| $c$ (constant) | $0$ | |
| $x$ | $1$ | |
| $x^2$ | $2x$ | |
| $x^3$ | $3x^2$ | |
| $x^n$ | $nx^{n-1}$ | $n \in \mathbb{R}$ |
| $x^{-1} = \frac{1}{x}$ | $-x^{-2} = -\frac{1}{x^2}$ | |
| $x^{-n} = \frac{1}{x^n}$ | $-nx^{-n-1}$ | |
| $x^{1/2} = \sqrt{x}$ | $\frac{1}{2}x^{-1/2} = \frac{1}{2\sqrt{x}}$ | |
| $x^{1/n} = \sqrt[n]{x}$ | $\frac{1}{n}x^{(1/n)-1}$ | |
| $x^{p/q}$ | $\frac{p}{q}x^{(p/q)-1}$ | Rational exponents |

---

## 6.1.3 Polynomial Examples

### Degree 1 (Linear)

$$f(x) = 3x + 5 \quad \Rightarrow \quad f'(x) = 3$$

### Degree 2 (Quadratic)

$$f(x) = 4x^2 - 7x + 2 \quad \Rightarrow \quad f'(x) = 8x - 7$$

### Degree 3 (Cubic)

$$f(x) = x^3 - 6x^2 + 11x - 6 \quad \Rightarrow \quad f'(x) = 3x^2 - 12x + 11$$

### Degree 4 (Quartic)

$$f(x) = 2x^4 - 3x^3 + x^2 - 5x + 1$$
$$f'(x) = 8x^3 - 9x^2 + 2x - 5$$

### General Degree $n$

$$f(x) = a_n x^n + a_{n-1}x^{n-1} + \cdots + a_1 x + a_0$$
$$f'(x) = na_n x^{n-1} + (n-1)a_{n-1}x^{n-2} + \cdots + a_1$$

**Key observation:** Differentiating a polynomial of degree $n$ gives a polynomial of degree $n-1$.

---

## 6.1.4 Negative and Fractional Powers

### Example 1: $y = \frac{3}{x^4}$

Rewrite: $y = 3x^{-4}$

$$\frac{dy}{dx} = 3(-4)x^{-5} = -\frac{12}{x^5}$$

### Example 2: $y = 5\sqrt[3]{x^2}$

Rewrite: $y = 5x^{2/3}$

$$\frac{dy}{dx} = 5 \cdot \frac{2}{3}x^{-1/3} = \frac{10}{3\sqrt[3]{x}}$$

### Example 3: $y = \frac{2}{\sqrt{x}} + 4x^{3/2}$

Rewrite: $y = 2x^{-1/2} + 4x^{3/2}$

$$\frac{dy}{dx} = -x^{-3/2} + 6x^{1/2} = -\frac{1}{x\sqrt{x}} + 6\sqrt{x}$$

---

## 6.1.5 Expanding Before Differentiating

Sometimes, expanding a product is easier than using the product rule.

### Example: $y = (x+2)(3x^2-1)$

Expand: $y = 3x^3 + 6x^2 - x - 2$

$$\frac{dy}{dx} = 9x^2 + 12x - 1$$

### Example: $y = (\sqrt{x} + 1)^2$

Expand: $y = x + 2\sqrt{x} + 1 = x + 2x^{1/2} + 1$

$$\frac{dy}{dx} = 1 + x^{-1/2} = 1 + \frac{1}{\sqrt{x}}$$

---

## 6.1.6 Rational Functions (Simplify First)

### Example: $y = \frac{x^3 + 2x^2 - x}{x}$

Simplify: $y = x^2 + 2x - 1$

$$\frac{dy}{dx} = 2x + 2$$

### Example: $y = \frac{x^2 + 3x + 2}{x + 1}$

Factor: $y = \frac{(x+1)(x+2)}{x+1} = x + 2$ (for $x \neq -1$)

$$\frac{dy}{dx} = 1$$

---

## 6.1.7 Successive Derivatives of Polynomials

```
  f(x) = x⁴ - 3x² + 5x - 2    (degree 4)
      ↓  differentiate
  f'(x) = 4x³ - 6x + 5         (degree 3)
      ↓  differentiate
  f''(x) = 12x² - 6             (degree 2)
      ↓  differentiate
  f'''(x) = 24x                 (degree 1)
      ↓  differentiate
  f⁴(x) = 24                   (degree 0)
      ↓  differentiate
  f⁵(x) = 0                    (all further derivatives = 0)
```

> **Fact:** The $n^{th}$ derivative of a polynomial of degree $n$ is $n! \cdot a_n$ (a constant), and all higher derivatives are zero.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Power rule | $\frac{d}{dx}(x^n) = nx^{n-1}$ for all real $n$ |
| Polynomial derivative | Reduces degree by 1 |
| $n^{th}$ derivative of degree $n$ | Equals $n! \cdot$(leading coefficient) |
| $(n+1)^{th}$ derivative | Equals $0$ |
| Negative powers | Rewrite $1/x^n = x^{-n}$ first |
| Fractional powers | Rewrite $\sqrt[n]{x^m} = x^{m/n}$ first |
| Tip | Expand or simplify before differentiating when possible |

---

## Quick Revision Questions

**Q1.** Differentiate $f(x) = 7x^5 - 3x^3 + 2x - 9$.

**Q2.** Find $\frac{dy}{dx}$ for $y = \frac{4}{x^3} - \frac{2}{\sqrt{x}}$.

**Q3.** Find the third derivative of $g(x) = x^4 - 2x^3 + x^2 - x + 1$.

**Q4.** Differentiate $y = (2x-1)^3$ by expanding first.

**Q5.** What is the 6th derivative of $f(x) = x^5 + x^4 + x^3$?

**Q6.** Simplify and differentiate: $y = \frac{x^4 - 16}{x^2 + 4}$.

---

[← Previous: Parametric Differentiation](../05-Differentiation-Rules/06-parametric-differentiation.md) · [Next: Trigonometric Derivatives →](02-trigonometric-derivatives.md)
