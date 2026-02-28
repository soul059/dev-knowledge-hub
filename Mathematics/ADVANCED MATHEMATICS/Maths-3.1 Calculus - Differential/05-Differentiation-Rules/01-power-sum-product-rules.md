# Unit 5 — Differentiation Rules: Power, Sum, and Product Rules

[← Previous: Left and Right Derivatives](../04-Differentiability/04-left-right-derivatives.md) · [Next: Quotient Rule →](02-quotient-rule.md)

---

## Chapter Overview

Rather than computing every derivative from first principles, we develop **rules** that let us differentiate quickly and systematically. This chapter covers the three most fundamental rules: the **Power Rule**, the **Sum/Difference Rule**, and the **Product Rule**.

---

## 5.1.1 The Constant Rule

$$\boxed{\frac{d}{dx}(c) = 0}$$

The derivative of any constant is zero — a horizontal line has zero slope.

**Proof:** $\lim_{h \to 0} \frac{c - c}{h} = 0$ $\blacksquare$

---

## 5.1.2 The Power Rule

$$\boxed{\frac{d}{dx}(x^n) = nx^{n-1}}$$

Valid for **any real** $n$ (integer, rational, irrational), wherever $x^n$ is defined.

### Proof (for positive integer $n$)

Using the binomial theorem:

$$(x+h)^n = x^n + nx^{n-1}h + \binom{n}{2}x^{n-2}h^2 + \cdots + h^n$$

$$\frac{(x+h)^n - x^n}{h} = nx^{n-1} + \binom{n}{2}x^{n-2}h + \cdots + h^{n-1}$$

As $h \to 0$, all terms with $h$ vanish: result is $nx^{n-1}$. $\blacksquare$

### Examples

| $f(x)$ | Apply power rule | $f'(x)$ |
|---------|-----------------|---------|
| $x^5$ | $5x^{5-1}$ | $5x^4$ |
| $x^{-2} = \frac{1}{x^2}$ | $-2x^{-3}$ | $-\frac{2}{x^3}$ |
| $x^{1/2} = \sqrt{x}$ | $\frac{1}{2}x^{-1/2}$ | $\frac{1}{2\sqrt{x}}$ |
| $x^{-1/3}$ | $-\frac{1}{3}x^{-4/3}$ | $-\frac{1}{3x^{4/3}}$ |
| $x^{\pi}$ | $\pi x^{\pi - 1}$ | $\pi x^{\pi - 1}$ |

---

## 5.1.3 The Constant Multiple Rule

$$\boxed{\frac{d}{dx}[c \cdot f(x)] = c \cdot f'(x)}$$

Constants "pass through" the derivative.

**Example:** $\frac{d}{dx}(7x^3) = 7 \cdot 3x^2 = 21x^2$

---

## 5.1.4 The Sum and Difference Rules

$$\boxed{\frac{d}{dx}[f(x) \pm g(x)] = f'(x) \pm g'(x)}$$

The derivative of a sum (or difference) is the sum (or difference) of the derivatives.

**Proof (sum):**

$$\lim_{h \to 0} \frac{[f(x+h) + g(x+h)] - [f(x) + g(x)]}{h}$$

$$= \lim_{h \to 0} \frac{f(x+h) - f(x)}{h} + \lim_{h \to 0} \frac{g(x+h) - g(x)}{h} = f'(x) + g'(x) \quad \blacksquare$$

### Extended: Derivative of a polynomial

$$\frac{d}{dx}(a_n x^n + a_{n-1}x^{n-1} + \cdots + a_1 x + a_0) = na_n x^{n-1} + (n-1)a_{n-1}x^{n-2} + \cdots + a_1$$

**Example:**

$$\frac{d}{dx}(3x^4 - 5x^3 + 2x - 7) = 12x^3 - 15x^2 + 2$$

---

## 5.1.5 The Product Rule

$$\boxed{\frac{d}{dx}[f(x) \cdot g(x)] = f'(x) \cdot g(x) + f(x) \cdot g'(x)}$$

**Mnemonic:** "first-times-derivative-of-second plus second-times-derivative-of-first"

Or: $(fg)' = f'g + fg'$

### Proof

$$\frac{d}{dx}[f \cdot g] = \lim_{h \to 0} \frac{f(x+h)g(x+h) - f(x)g(x)}{h}$$

Add and subtract $f(x+h)g(x)$ in the numerator:

$$= \lim_{h \to 0} \frac{f(x+h)g(x+h) - f(x+h)g(x) + f(x+h)g(x) - f(x)g(x)}{h}$$

$$= \lim_{h \to 0} \left[f(x+h) \cdot \frac{g(x+h) - g(x)}{h} + g(x) \cdot \frac{f(x+h) - f(x)}{h}\right]$$

$$= f(x) \cdot g'(x) + g(x) \cdot f'(x) \quad \blacksquare$$

### Example 1

$$\frac{d}{dx}[(x^2)(x^3)] = 2x \cdot x^3 + x^2 \cdot 3x^2 = 2x^4 + 3x^4 = 5x^4$$

**Verification:** $x^2 \cdot x^3 = x^5$, and $\frac{d}{dx}(x^5) = 5x^4$ ✓

### Example 2

$$\frac{d}{dx}[x \sin x] = 1 \cdot \sin x + x \cdot \cos x = \sin x + x \cos x$$

### Example 3

$$\frac{d}{dx}[e^x \cos x] = e^x \cos x + e^x(-\sin x) = e^x(\cos x - \sin x)$$

### Extended Product Rule (Three Functions)

$$(fgh)' = f'gh + fg'h + fgh'$$

**Example:**

$$\frac{d}{dx}[x^2 \sin x \, e^x] = 2x \sin x \, e^x + x^2 \cos x \, e^x + x^2 \sin x \, e^x$$

$$= e^x(2x \sin x + x^2 \cos x + x^2 \sin x)$$

---

## 5.1.6 Process Flowchart

```
  Given: differentiate an expression
          │
  ┌───────┴───────┐
  │ Is it a sum   │
  │ of terms?     │
  └───────┬───────┘
     YES  │  NO
     │    │
     │    ├── Is it a product of two functions?
     │    │        │
     │    │   YES: Use Product Rule
     │    │   NO:  Is it a single power of x?
     │    │              │
     │    │         YES: Use Power Rule
     │    │         NO:  Other rules (quotient, chain...)
     │
  Differentiate term by term
  (Sum Rule + Power Rule + Constant Multiple)
```

---

## Summary Table

| Rule | Formula |
|------|---------|
| Constant | $\frac{d}{dx}(c) = 0$ |
| Power | $\frac{d}{dx}(x^n) = nx^{n-1}$ |
| Constant Multiple | $\frac{d}{dx}(cf) = cf'$ |
| Sum/Difference | $\frac{d}{dx}(f \pm g) = f' \pm g'$ |
| Product | $\frac{d}{dx}(fg) = f'g + fg'$ |
| Triple Product | $(fgh)' = f'gh + fg'h + fgh'$ |

---

## Quick Revision Questions

**Q1.** Differentiate $f(x) = 4x^7 - 3x^4 + 2x - 9$.

**Q2.** Find $\frac{d}{dx}(\sqrt{x} + \frac{1}{x^3})$.

**Q3.** Use the product rule to differentiate $x^2 \ln x$.

**Q4.** Prove the product rule from the definition of the derivative.

**Q5.** Differentiate $(x^2 + 1)(x^3 - x)$ using the product rule, then verify by expanding first.

**Q6.** Find $\frac{d}{dx}(x^2 e^x \sin x)$ using the triple product rule.

---

[← Previous: Left and Right Derivatives](../04-Differentiability/04-left-right-derivatives.md) · [Next: Quotient Rule →](02-quotient-rule.md)
