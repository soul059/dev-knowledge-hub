# Chapter 10.2: Inverse Laplace Transform

[← Previous: Definition & Properties](01-definition-and-properties.md) | [Back to README](../README.md) | [Next: Solving ODEs with Laplace Transform →](03-solving-odes-with-laplace.md)

---

## Overview

The **inverse Laplace transform** takes us back from $F(s)$ to $f(t)$. In practice, we use tables, partial fractions, and convolution — not the complex integral formula. Mastery of partial fraction decomposition is the essential skill here.

---

## 1. Definition

$$f(t) = \mathcal{L}^{-1}\{F(s)\} = \frac{1}{2\pi i}\int_{\gamma - i\infty}^{\gamma + i\infty} e^{st}F(s)\,ds$$

(Bromwich integral — rarely computed directly. Use the methods below instead.)

**Linearity**: $\mathcal{L}^{-1}\{aF + bG\} = af + bg$

---

## 2. Standard Inverse Transforms

| $F(s)$ | $f(t) = \mathcal{L}^{-1}\{F(s)\}$ |
|--------|-----------------------------------|
| $\dfrac{1}{s}$ | $1$ |
| $\dfrac{1}{s^2}$ | $t$ |
| $\dfrac{n!}{s^{n+1}}$ | $t^n$ |
| $\dfrac{1}{s-a}$ | $e^{at}$ |
| $\dfrac{\omega}{s^2+\omega^2}$ | $\sin\omega t$ |
| $\dfrac{s}{s^2+\omega^2}$ | $\cos\omega t$ |
| $\dfrac{1}{(s-a)^{n+1}}$ | $\dfrac{t^n e^{at}}{n!}$ |
| $\dfrac{\omega}{(s-a)^2+\omega^2}$ | $e^{at}\sin\omega t$ |
| $\dfrac{s-a}{(s-a)^2+\omega^2}$ | $e^{at}\cos\omega t$ |

---

## 3. Partial Fraction Decomposition

The central technique for computing inverse transforms.

```
  PARTIAL FRACTIONS — DECISION TREE

  ┌───────────────────┐
  │ Factor denominator │
  └────────┬──────────┘
           ▼
  ┌────────┴────────────────┐
  │ What kinds of factors?   │
  └──┬──────────┬───────────┘
     ▼          ▼
  Linear    Quadratic
  (s-a)     (s²+bs+c)
     │          │
     ▼          ▼
  ┌──┴──┐   ┌──┴──┐
  Simple Repeated Simple Repeated
  A/(s-a) A/(s-a) (As+B)/ ...
          +B/(s-a)² (s²+bs+c)
```

### Case 1: Distinct Linear Factors

$$\frac{P(s)}{(s-a)(s-b)} = \frac{A}{s-a} + \frac{B}{s-b}$$

### Case 2: Repeated Linear Factors

$$\frac{P(s)}{(s-a)^3} = \frac{A}{s-a} + \frac{B}{(s-a)^2} + \frac{C}{(s-a)^3}$$

### Case 3: Irreducible Quadratic Factors

$$\frac{P(s)}{(s^2 + \omega^2)} = \frac{As + B}{s^2 + \omega^2}$$

Then complete the square if needed.

---

## 4. Worked Examples

### Example 1: Distinct Linear

$$F(s) = \frac{5s + 3}{(s+1)(s-2)}$$

$$\frac{5s+3}{(s+1)(s-2)} = \frac{A}{s+1} + \frac{B}{s-2}$$

$s = -1$: $\frac{-2}{-3} = \frac{2}{3} = A$

$s = 2$: $\frac{13}{3} = B$

$$f(t) = \frac{2}{3}e^{-t} + \frac{13}{3}e^{2t}$$

### Example 2: Repeated Factors

$$F(s) = \frac{s+3}{(s+1)^2(s-1)}$$

$$= \frac{A}{s+1} + \frac{B}{(s+1)^2} + \frac{C}{s-1}$$

$s = -1$: $\frac{2}{-2} = -1 = B$

$s = 1$: $\frac{4}{4} = 1 = C$

Comparing $s^2$ coefficients: $A + C = 0 \implies A = -1$

$$f(t) = -e^{-t} - te^{-t} + e^{t}$$

### Example 3: Quadratic Factor

$$F(s) = \frac{2s + 5}{s^2 + 4s + 13}$$

Complete the square: $s^2 + 4s + 13 = (s+2)^2 + 9$

$$F(s) = \frac{2(s+2) + 1}{(s+2)^2 + 9} = 2\cdot\frac{s+2}{(s+2)^2 + 9} + \frac{1}{3}\cdot\frac{3}{(s+2)^2 + 9}$$

$$f(t) = 2e^{-2t}\cos 3t + \frac{1}{3}e^{-2t}\sin 3t$$

### Example 4: Mixed

$$F(s) = \frac{s^2 + 1}{s(s^2 + 4)}$$

$$= \frac{A}{s} + \frac{Bs + C}{s^2 + 4}$$

$s = 0$: $1/4 = A$

Comparing $s^2$: $A + B = 1 \implies B = 3/4$

Comparing constants: $4A + C = 1 \implies C = 0$

$$f(t) = \frac{1}{4} + \frac{3}{4}\cos 2t$$

### Example 5: Using the First Shift

$$F(s) = \frac{3}{(s-2)^4}$$

$$\mathcal{L}^{-1}\left\{\frac{3}{s^4}\right\} = \frac{3}{6}t^3 = \frac{t^3}{2}$$

Apply shift ($s \to s-2$ means multiply by $e^{2t}$):

$$f(t) = \frac{t^3}{2}e^{2t}$$

---

## 5. Completing the Square — Key Technique

For $\dfrac{f(s)}{as^2 + bs + c}$:

$$as^2 + bs + c = a\left[\left(s + \frac{b}{2a}\right)^2 + \frac{4ac - b^2}{4a^2}\right]$$

Then rewrite numerator in terms of $(s + b/2a)$.

---

## 6. Heaviside's Cover-Up Rule

For distinct linear factors, a fast shortcut:

$$\mathcal{L}^{-1}\left\{\frac{P(s)}{(s-a_1)(s-a_2)\cdots(s-a_n)}\right\} = \sum_{k=1}^n \frac{P(a_k)}{\prod_{j \neq k}(a_k - a_j)} e^{a_k t}$$

---

## Summary Table

| Technique | When to Use |
|-----------|------------|
| **Direct table lookup** | $F(s)$ matches a standard form |
| **Partial fractions** | Rational $F(s)$ with factorable denominator |
| **Completing the square** | Irreducible quadratic in denominator |
| **First shift** | $(s-a)$ appears in denominator |
| **Cover-up rule** | Distinct linear factors only |
| **Multiplication by $t$** | $\mathcal{L}^{-1}\{F'(s)\} = -tf(t)$ |

---

## Quick Revision Questions

1. Find $\mathcal{L}^{-1}\left\{\dfrac{3s - 5}{s^2 + 2s + 5}\right\}$.

2. Decompose $\dfrac{s^2}{(s+1)(s-1)(s+2)}$ into partial fractions and invert.

3. Find $\mathcal{L}^{-1}\left\{\dfrac{1}{s^2(s+3)}\right\}$.

4. Compute $\mathcal{L}^{-1}\left\{\dfrac{e^{-2s}}{s^2 + 1}\right\}$ (involves the second shifting theorem).

5. Use the cover-up rule to find $\mathcal{L}^{-1}\left\{\dfrac{2s+1}{(s-1)(s+2)(s-3)}\right\}$.

6. Find $\mathcal{L}^{-1}\left\{\ln\dfrac{s+1}{s-1}\right\}$ using the property $\mathcal{L}\{f/t\} = \int_s^\infty F(\sigma)\,d\sigma$.

---

[← Previous: Definition & Properties](01-definition-and-properties.md) | [Back to README](../README.md) | [Next: Solving ODEs with Laplace Transform →](03-solving-odes-with-laplace.md)
