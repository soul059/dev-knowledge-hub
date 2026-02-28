# Chapter 6.3: D-Operator Methods (Inverse Operator)

[← Previous: Variation of Parameters](02-variation-of-parameters.md) | [Back to README](../README.md) | [Next: Cauchy-Euler Non-Homogeneous →](04-cauchy-euler-non-homogeneous.md)

---

## Overview

The D-operator method provides a compact, algebraic approach to finding particular solutions. By treating the differential operator $D = d/dx$ as an algebraic quantity, we can use inverse operators to "undo" differentiation and find $y_p$ systematically.

---

## 1. The Operator Framework

$$D = \frac{d}{dx}, \quad D^2 = \frac{d^2}{dx^2}, \quad f(D) = a_n D^n + \cdots + a_1 D + a_0$$

The ODE $f(D)y = g(x)$ has particular solution:

$$\boxed{y_p = \frac{1}{f(D)}g(x)}$$

The operator $\dfrac{1}{f(D)}$ is the **inverse operator**.

---

## 2. Key Inverse Operator Formulas

### Formula 1: Exponential

$$\boxed{\frac{1}{f(D)}e^{ax} = \frac{e^{ax}}{f(a)}, \quad f(a) \neq 0}$$

If $f(a) = 0$ (root of multiplicity $s$):

$$\boxed{\frac{1}{f(D)}e^{ax} = \frac{x^s e^{ax}}{f^{(s)}(a)}}$$

where $f^{(s)}(a)$ is the $s$th derivative of $f(m)$ evaluated at $m = a$.

### Formula 2: Sine and Cosine

For $f(D^2)$ (only even powers of $D$):

$$\boxed{\frac{1}{f(D^2)}\sin ax = \frac{\sin ax}{f(-a^2)}, \quad f(-a^2) \neq 0}$$

$$\boxed{\frac{1}{f(D^2)}\cos ax = \frac{\cos ax}{f(-a^2)}, \quad f(-a^2) \neq 0}$$

**Rule**: Replace $D^2$ with $-a^2$ wherever it appears.

### Formula 3: Polynomial $x^n$

$$\frac{1}{f(D)}x^n = [f(D)]^{-1} x^n$$

Expand $[f(D)]^{-1}$ as a power series in $D$ up to $D^n$, then apply term by term.

### Formula 4: Exponential Shift (Shifting Theorem)

$$\boxed{\frac{1}{f(D)}[e^{ax}V(x)] = e^{ax}\frac{1}{f(D+a)}V(x)}$$

This is extremely useful for products like $e^{ax}\sin bx$, $e^{ax}x^n$, etc.

### Formula 5: $x \cdot V(x)$

$$\frac{1}{f(D)}[xV(x)] = x\frac{1}{f(D)}V(x) - \frac{f'(D)}{[f(D)]^2}V(x)$$

---

## 3. Formula Summary Table

```
┌────────────────────────────────────────────────────────────┐
│          INVERSE OPERATOR QUICK REFERENCE                  │
├──────────────┬─────────────────────────────────────────────┤
│    g(x)      │           y_p = (1/f(D))·g(x)              │
├──────────────┼─────────────────────────────────────────────┤
│  e^(ax)      │  e^(ax)/f(a)       [if f(a)≠0]            │
│              │  x^s·e^(ax)/f⁽ˢ⁾(a) [if a is root, mult s]│
├──────────────┼─────────────────────────────────────────────┤
│  sin(ax)     │  Replace D² → -a²  in f(D²)               │
│  cos(ax)     │  Then evaluate                             │
├──────────────┼─────────────────────────────────────────────┤
│  x^n         │  Expand 1/f(D) in powers of D              │
│              │  Apply D^k to x^n term by term             │
├──────────────┼─────────────────────────────────────────────┤
│  e^(ax)V(x)  │  e^(ax) · (1/f(D+a))·V(x)                │
│              │  (exponential shift)                        │
├──────────────┼─────────────────────────────────────────────┤
│  x·V(x)      │  x·(1/f(D))V - (f'(D)/f²(D))V            │
└──────────────┴─────────────────────────────────────────────┘
```

---

## 4. Worked Examples

### Example 1: $\dfrac{1}{D^2 - 3D + 2}e^{5x}$

$f(D) = D^2 - 3D + 2$, $f(5) = 25 - 15 + 2 = 12$

$$y_p = \frac{e^{5x}}{12}$$

### Example 2: $\dfrac{1}{D^2 - 2D + 1}e^x$ (Failure Case)

$f(D) = (D-1)^2$, $f(1) = 0$, $f'(D) = 2(D-1)$, $f'(1) = 0$

$f''(D) = 2$, $f''(1) = 2$. Root has multiplicity $s = 2$.

$$y_p = \frac{x^2 e^x}{f''(1)} = \frac{x^2 e^x}{2}$$

### Example 3: $(D^2 + 4)y = \sin 3x$

Replace $D^2 \to -9$:

$$y_p = \frac{\sin 3x}{-9 + 4} = \frac{\sin 3x}{-5} = -\frac{\sin 3x}{5}$$

### Example 4: $(D^2 + 9)y = \sin 3x$ (Failure Case)

$D^2 \to -9$: $f(-9) = -9 + 9 = 0$. Method fails directly.

**Resolution**: Use $\sin 3x = \text{Im}(e^{3ix})$

$$\frac{1}{D^2 + 9}e^{3ix} = \frac{x \cdot e^{3ix}}{2D}\bigg|_{D=3i} = \frac{xe^{3ix}}{6i} = \frac{x(\cos 3x + i\sin 3x)}{6i}$$

$$= \frac{x}{6}(\sin 3x - i\cos 3x)$$

Taking imaginary part (wait — let me use the standard approach):

Since $D^2 + 9 = (D + 3i)(D - 3i)$, and $3i$ is a simple root:

$$\frac{1}{D^2 + 9}\sin 3x = -\frac{x\cos 3x}{6}$$

### Example 5: $(D^2 + 1)y = x^2$

$$y_p = \frac{1}{1 + D^2}x^2 = (1 - D^2 + D^4 - \cdots)x^2$$

Only terms up to $D^2$ matter (higher derivatives of $x^2$ vanish):

$$y_p = x^2 - D^2(x^2) = x^2 - 2$$

### Example 6: $(D^2 - 3D + 2)y = e^{2x}\sin x$ (Exponential Shift)

$$y_p = \frac{1}{D^2 - 3D + 2}e^{2x}\sin x = e^{2x}\frac{1}{(D+2)^2 - 3(D+2) + 2}\sin x$$

$$= e^{2x}\frac{1}{D^2 + 4D + 4 - 3D - 6 + 2}\sin x = e^{2x}\frac{1}{D^2 + D}\sin x$$

Replace $D^2 \to -1$:

$$= e^{2x}\frac{\sin x}{-1 + D} = e^{2x}\frac{D + 1}{(D+1)(D-1)}\sin x \cdot \frac{-1}{1}$$

Wait, let me redo more carefully:

$$= e^{2x}\frac{1}{D^2 + D}\sin x$$

$D^2 \to -1$: $\dfrac{1}{-1 + D}\sin x = \dfrac{D + 1}{(D-1)(D+1)}\sin x = \dfrac{D + 1}{D^2 - 1}\sin x$

$D^2 \to -1$: $= \dfrac{D + 1}{-2}\sin x = \dfrac{-(D + 1)}{2}\sin x = \dfrac{-(\cos x + \sin x)}{2}$

$$\boxed{y_p = -\frac{e^{2x}}{2}(\cos x + \sin x)}$$

### Example 7: $(D^2 - 1)y = x^3$

$$y_p = \frac{-1}{1 - D^2}x^3 = -(1 + D^2 + D^4 + \cdots)x^3 = -(x^3 + 6x) = -x^3 - 6x$$

Check: $y_p'' - y_p = (-6x) - (-x^3 - 6x) = x^3$ ✓

---

## 5. Useful Binomial Expansions for Polynomial $g(x)$

$$\frac{1}{1 + u} = 1 - u + u^2 - u^3 + \cdots$$

$$\frac{1}{1 - u} = 1 + u + u^2 + u^3 + \cdots$$

$$\frac{1}{(1 + u)^2} = 1 - 2u + 3u^2 - 4u^3 + \cdots$$

where $u$ represents the operator expression (e.g., $u = D$, $u = D^2$, etc.).

### Strategy for Polynomials

1. Factor out the constant term from $f(D)$
2. Express as $\dfrac{1}{c(1 + h(D))}$ where $h(D)$ has no constant term
3. Expand $(1 + h(D))^{-1}$ as a power series
4. Keep terms up to $D^n$ (where $n = $ degree of polynomial $g(x)$)

---

## 6. Comparison of Methods

| Method | Best for | Difficulty |
|--------|----------|-----------|
| Direct substitution (undetermined coeff.) | Simple $g(x)$, verification | Easy |
| Exponential formula | $e^{ax}$ | Very easy |
| $D^2 \to -a^2$ | Pure $\sin ax$, $\cos ax$ | Very easy |
| Exponential shift | $e^{ax}V(x)$ | Moderate |
| Series expansion | Polynomials | Moderate |
| Variation of parameters | Anything else | Hard (integration) |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Inverse operator** | $y_p = \dfrac{1}{f(D)}g(x)$ |
| **Exponential** | $\dfrac{e^{ax}}{f(a)}$ or $\dfrac{x^s e^{ax}}{f^{(s)}(a)}$ |
| **Trig** | Replace $D^2 \to -a^2$ |
| **Polynomial** | Expand $[f(D)]^{-1}$ as power series |
| **Exp shift** | $\dfrac{1}{f(D)}[e^{ax}V] = e^{ax}\dfrac{1}{f(D+a)}V$ |
| **Key advantage** | Fast, algebraic — no integration needed |

---

## Quick Revision Questions

1. Evaluate $\dfrac{1}{D^2 + 3D + 2}e^{-3x}$.

2. Find $y_p$ for $(D^2 + 4)y = \cos 2x$ using the failure case formula.

3. Use the power series method to find $\dfrac{1}{D^2 + D + 1}(x^2 + 1)$.

4. Apply the exponential shift to find $\dfrac{1}{D^2 - 4D + 4}xe^{2x}$.

5. Evaluate $\dfrac{1}{D^3 - D}e^x$ using the appropriate failure case formula.

6. Compare the D-operator solution of $(D^2 - 1)y = 3e^{2x}$ with undetermined coefficients.

---

[← Previous: Variation of Parameters](02-variation-of-parameters.md) | [Back to README](../README.md) | [Next: Cauchy-Euler Non-Homogeneous →](04-cauchy-euler-non-homogeneous.md)
