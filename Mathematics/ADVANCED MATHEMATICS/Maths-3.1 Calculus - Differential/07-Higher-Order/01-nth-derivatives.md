# Unit 7 — Higher-Order Derivatives: $n^{th}$ Derivatives

[← Previous: Hyperbolic Derivatives](../06-Derivatives-Standard/05-hyperbolic-derivatives.md) · [Next: Leibniz Theorem →](02-leibniz-theorem.md)

---

## Chapter Overview

Differentiating a function repeatedly gives **higher-order derivatives**. This chapter develops techniques to find $n^{th}$ derivatives of standard functions and recognize patterns that allow us to write closed-form expressions.

---

## 7.1.1 Notation

The $n^{th}$ derivative of $y = f(x)$ is written as:

| Notation | Style |
|----------|-------|
| $f^{(n)}(x)$ | Lagrange |
| $\dfrac{d^n y}{dx^n}$ | Leibniz |
| $D^n y$ | Operator |
| $y_n$ | Subscript |

For small $n$: $f'$, $f''$, $f'''$, $f^{(4)}$, $f^{(5)}$, ...

---

## 7.1.2 $n^{th}$ Derivatives of Standard Functions

### Power Functions

$$\frac{d^n}{dx^n}(x^m) = \frac{m!}{(m-n)!}\,x^{m-n} \quad \text{for } n \leq m$$

$$\frac{d^n}{dx^n}(x^m) = 0 \quad \text{for } n > m$$

**Special case:** $\frac{d^n}{dx^n}(x^n) = n!$

### Exponential Functions

$$\frac{d^n}{dx^n}(e^{ax}) = a^n e^{ax}$$

**Special case:** $\frac{d^n}{dx^n}(e^x) = e^x$

### Trigonometric Functions

$$\frac{d^n}{dx^n}(\sin ax) = a^n \sin\left(ax + \frac{n\pi}{2}\right)$$

$$\frac{d^n}{dx^n}(\cos ax) = a^n \cos\left(ax + \frac{n\pi}{2}\right)$$

**Derivation for $\sin x$:**

```
  n = 0:  sin x         = sin(x + 0·π/2)
  n = 1:  cos x         = sin(x + 1·π/2)
  n = 2:  -sin x        = sin(x + 2·π/2)
  n = 3:  -cos x        = sin(x + 3·π/2)
  n = 4:  sin x         = sin(x + 4·π/2)   ← cycle repeats
```

### Logarithmic Function

$$\frac{d^n}{dx^n}(\ln x) = \frac{(-1)^{n-1}(n-1)!}{x^n} \quad \text{for } n \geq 1$$

**Derivation:**

```
  n = 1:  1/x        =  (-1)^0 · 0!/x^1  =  1/x        ✓
  n = 2:  -1/x²      =  (-1)^1 · 1!/x^2  = -1/x²       ✓
  n = 3:  2/x³       =  (-1)^2 · 2!/x^3  =  2/x³       ✓
  n = 4:  -6/x⁴      =  (-1)^3 · 3!/x^4  = -6/x⁴       ✓
```

### General Exponential

$$\frac{d^n}{dx^n}(a^x) = a^x (\ln a)^n$$

---

## 7.1.3 Complete Table of $n^{th}$ Derivatives

| $f(x)$ | $f^{(n)}(x)$ |
|---------|---------------|
| $x^m$ ($n \leq m$) | $\dfrac{m!}{(m-n)!}\,x^{m-n}$ |
| $e^{ax}$ | $a^n e^{ax}$ |
| $a^x$ | $a^x (\ln a)^n$ |
| $\sin(ax+b)$ | $a^n\sin(ax+b+n\pi/2)$ |
| $\cos(ax+b)$ | $a^n\cos(ax+b+n\pi/2)$ |
| $\ln(ax+b)$ | $\dfrac{(-1)^{n-1}(n-1)!\,a^n}{(ax+b)^n}$ |
| $(ax+b)^m$ | $\dfrac{m!\,a^n}{(m-n)!}(ax+b)^{m-n}$ |
| $\frac{1}{ax+b}$ | $\dfrac{(-1)^n \cdot n! \cdot a^n}{(ax+b)^{n+1}}$ |

---

## 7.1.4 $n^{th}$ Derivative via Partial Fractions

For rational functions, decompose into partial fractions first.

### Example: $y = \frac{1}{(x-1)(x-2)}$

Partial fractions: $y = \frac{1}{x-2} - \frac{1}{x-1}$

Using $\frac{d^n}{dx^n}\left(\frac{1}{x-a}\right) = \frac{(-1)^n \cdot n!}{(x-a)^{n+1}}$:

$$y_n = \frac{(-1)^n \cdot n!}{(x-2)^{n+1}} - \frac{(-1)^n \cdot n!}{(x-1)^{n+1}}$$

$$= (-1)^n \cdot n!\left[\frac{1}{(x-2)^{n+1}} - \frac{1}{(x-1)^{n+1}}\right]$$

### Example: $y = \frac{x}{(x+1)(x+2)}$

Partial fractions: $y = \frac{-1}{x+1} + \frac{2}{x+2}$

$$y_n = \frac{-(-1)^n \cdot n!}{(x+1)^{n+1}} + \frac{2(-1)^n \cdot n!}{(x+2)^{n+1}}$$

$$= (-1)^n \cdot n!\left[\frac{2}{(x+2)^{n+1}} - \frac{1}{(x+1)^{n+1}}\right]$$

---

## 7.1.5 $n^{th}$ Derivative of $e^{ax}\sin(bx + c)$ and $e^{ax}\cos(bx + c)$

These are found using the result:

$$\frac{d^n}{dx^n}[e^{ax}\sin(bx+c)] = r^n e^{ax}\sin(bx + c + n\phi)$$

where $r = \sqrt{a^2 + b^2}$ and $\phi = \tan^{-1}\frac{b}{a}$.

Similarly:

$$\frac{d^n}{dx^n}[e^{ax}\cos(bx+c)] = r^n e^{ax}\cos(bx + c + n\phi)$$

**Derivation sketch:** Write $e^{ax}\sin(bx) = \text{Im}(e^{(a+ib)x})$. Then $D^n e^{(a+ib)x} = (a+ib)^n e^{(a+ib)x}$. Express $a + ib = re^{i\phi}$ in polar form.

### Example: $y = e^{2x}\sin(3x)$

Here $a = 2$, $b = 3$: $r = \sqrt{4+9} = \sqrt{13}$, $\phi = \tan^{-1}\frac{3}{2}$.

$$y_n = (\sqrt{13})^n \, e^{2x}\sin\left(3x + n\tan^{-1}\frac{3}{2}\right)$$

---

## 7.1.6 Recognizing Patterns

**Strategy for finding $f^{(n)}(x)$:**

```
  ┌─────────────────────────────────────────────┐
  │  1. Compute f'(x), f''(x), f'''(x), f⁴(x) │
  │  2. Look for a pattern                      │
  │  3. Write general formula for f⁽ⁿ⁾(x)      │
  │  4. Verify: does it match for n = 1,2,3,4?  │
  └─────────────────────────────────────────────┘
```

### Example: $y = \frac{1}{1+x}$

$$y' = -(1+x)^{-2}, \quad y'' = 2(1+x)^{-3}, \quad y''' = -6(1+x)^{-4}$$

Pattern: $y_n = \frac{(-1)^n \cdot n!}{(1+x)^{n+1}}$

Check $n=1$: $\frac{(-1)^1 \cdot 1!}{(1+x)^2} = \frac{-1}{(1+x)^2}$ ✓

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| $n^{th}$ derivative of $x^m$ | $\frac{m!}{(m-n)!}x^{m-n}$ |
| $n^{th}$ derivative of $e^{ax}$ | $a^n e^{ax}$ |
| $n^{th}$ derivative of $\sin(ax+b)$ | $a^n \sin(ax+b+n\pi/2)$ |
| $n^{th}$ derivative of $\ln(ax+b)$ | $\frac{(-1)^{n-1}(n-1)!a^n}{(ax+b)^n}$ |
| $n^{th}$ derivative of $1/(ax+b)$ | $\frac{(-1)^n n! a^n}{(ax+b)^{n+1}}$ |
| $e^{ax}\sin(bx+c)$ | $r^n e^{ax}\sin(bx+c+n\phi)$ |
| Strategy | Compute first few → spot pattern → verify |

---

## Quick Revision Questions

**Q1.** Find the $n^{th}$ derivative of $y = e^{-3x}$.

**Q2.** Find the $n^{th}$ derivative of $y = \ln(2x + 5)$.

**Q3.** Find the 10th derivative of $y = x^{12}$.

**Q4.** Find $y_n$ for $y = \frac{1}{x^2 - 1}$ using partial fractions.

**Q5.** Find the $n^{th}$ derivative of $y = e^x \cos x$.

**Q6.** What is $f^{(100)}(x)$ if $f(x) = \sin(2x)$?

---

[← Previous: Hyperbolic Derivatives](../06-Derivatives-Standard/05-hyperbolic-derivatives.md) · [Next: Leibniz Theorem →](02-leibniz-theorem.md)
