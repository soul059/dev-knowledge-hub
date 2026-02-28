# Unit 2 — Limits: Evaluation Techniques

[← Previous: Laws of Limits](03-laws-of-limits.md) · [Next: Indeterminate Forms →](05-indeterminate-forms.md)

---

## Chapter Overview

When direct substitution gives a valid result, that's the limit. But when substitution yields an **indeterminate form** like $\frac{0}{0}$, we need algebraic or trigonometric manipulation to rewrite the expression into a form where the limit can be evaluated. This chapter covers the main techniques.

---

## 2.4.1 Decision Flowchart

```
  ┌─────────────────────────────────┐
  │   Substitute x = a into f(x)   │
  └──────────────┬──────────────────┘
                 │
        ┌────────┴────────┐
    Defined?              ├── YES → That's the limit!
        │
       NO (0/0, ∞/∞, etc.)
        │
  ┌─────┴───────────────────────────────────┐
  │ Choose a technique:                     │
  │ 1. Factor and cancel                    │
  │ 2. Rationalize (conjugates)             │
  │ 3. Trig identities + standard limits    │
  │ 4. Substitution                         │
  │ 5. Multiply/divide by strategic form    │
  │ 6. L'Hôpital's Rule (see 2.6)          │
  └─────────────────────────────────────────┘
```

---

## 2.4.2 Technique 1: Factoring and Cancellation

**When to use:** $\frac{0}{0}$ form where numerator and denominator share a common factor.

### Example 1

$$\lim_{x \to 2} \frac{x^2 - 4}{x - 2} = \lim_{x \to 2} \frac{(x-2)(x+2)}{x-2} = \lim_{x \to 2} (x + 2) = 4$$

### Example 2

$$\lim_{x \to -3} \frac{x^2 + 5x + 6}{x^2 + 3x} = \lim_{x \to -3} \frac{(x+2)(x+3)}{x(x+3)} = \lim_{x \to -3} \frac{x+2}{x} = \frac{-1}{-3} = \frac{1}{3}$$

### Example 3

$$\lim_{x \to 1} \frac{x^3 - 1}{x - 1} = \lim_{x \to 1} \frac{(x-1)(x^2 + x + 1)}{x - 1} = \lim_{x \to 1} (x^2 + x + 1) = 3$$

**Useful factoring formulae:**

| Expression | Factored Form |
|-----------|--------------|
| $x^2 - a^2$ | $(x-a)(x+a)$ |
| $x^3 - a^3$ | $(x-a)(x^2 + ax + a^2)$ |
| $x^3 + a^3$ | $(x+a)(x^2 - ax + a^2)$ |
| $x^n - a^n$ | $(x-a)(x^{n-1} + x^{n-2}a + \cdots + a^{n-1})$ |

---

## 2.4.3 Technique 2: Rationalization (Conjugate Multiplication)

**When to use:** Expressions involving $\sqrt{\ }$ that give $\frac{0}{0}$.

### Example 1

$$\lim_{x \to 0} \frac{\sqrt{x+4} - 2}{x}$$

Multiply by conjugate:

$$= \lim_{x \to 0} \frac{(\sqrt{x+4} - 2)(\sqrt{x+4} + 2)}{x(\sqrt{x+4} + 2)} = \lim_{x \to 0} \frac{(x+4) - 4}{x(\sqrt{x+4} + 2)}$$

$$= \lim_{x \to 0} \frac{x}{x(\sqrt{x+4} + 2)} = \lim_{x \to 0} \frac{1}{\sqrt{x+4} + 2} = \frac{1}{2 + 2} = \frac{1}{4}$$

### Example 2

$$\lim_{x \to 3} \frac{x - 3}{\sqrt{x+1} - 2}$$

Multiply by conjugate of denominator:

$$= \lim_{x \to 3} \frac{(x-3)(\sqrt{x+1}+2)}{(\sqrt{x+1}-2)(\sqrt{x+1}+2)} = \lim_{x \to 3} \frac{(x-3)(\sqrt{x+1}+2)}{(x+1)-4}$$

$$= \lim_{x \to 3} \frac{(x-3)(\sqrt{x+1}+2)}{x-3} = \lim_{x \to 3} (\sqrt{x+1}+2) = \sqrt{4}+2 = 4$$

---

## 2.4.4 Technique 3: Trigonometric Manipulation

**Core standard limits to exploit:**

$$\lim_{x \to 0} \frac{\sin x}{x} = 1, \qquad \lim_{x \to 0} \frac{\tan x}{x} = 1, \qquad \lim_{x \to 0} \frac{1 - \cos x}{x^2} = \frac{1}{2}$$

### Example 1

$$\lim_{x \to 0} \frac{\sin 5x}{x} = \lim_{x \to 0} 5 \cdot \frac{\sin 5x}{5x} = 5 \cdot 1 = 5$$

### Example 2

$$\lim_{x \to 0} \frac{\sin 3x}{\sin 7x} = \lim_{x \to 0} \frac{\sin 3x}{3x} \cdot \frac{7x}{\sin 7x} \cdot \frac{3x}{7x} = 1 \cdot 1 \cdot \frac{3}{7} = \frac{3}{7}$$

### Example 3

$$\lim_{x \to 0} \frac{\tan x - \sin x}{x^3}$$

$$= \lim_{x \to 0} \frac{\sin x(\frac{1}{\cos x} - 1)}{x^3} = \lim_{x \to 0} \frac{\sin x}{x} \cdot \frac{1 - \cos x}{x^2} \cdot \frac{1}{\cos x} = 1 \cdot \frac{1}{2} \cdot 1 = \frac{1}{2}$$

---

## 2.4.5 Technique 4: Substitution

**When to use:** To transform the limit into a standard form.

### Example 1

$$\lim_{x \to 0} \frac{\sin(x^2)}{x^2}$$

Let $t = x^2$. As $x \to 0$, $t \to 0$.

$$= \lim_{t \to 0} \frac{\sin t}{t} = 1$$

### Example 2

$$\lim_{x \to 1} \frac{\sin(x - 1)}{x^2 - 1}$$

Let $t = x - 1$. As $x \to 1$, $t \to 0$.

$$= \lim_{t \to 0} \frac{\sin t}{(t+1)^2 - 1} = \lim_{t \to 0} \frac{\sin t}{t^2 + 2t} = \lim_{t \to 0} \frac{\sin t}{t} \cdot \frac{1}{t + 2} = 1 \cdot \frac{1}{2} = \frac{1}{2}$$

---

## 2.4.6 Technique 5: Exponential Limits

Use the standard results:

$$\lim_{x \to 0} \frac{e^x - 1}{x} = 1, \qquad \lim_{x \to 0} \frac{a^x - 1}{x} = \ln a, \qquad \lim_{x \to 0} \frac{\ln(1+x)}{x} = 1$$

### Example 1

$$\lim_{x \to 0} \frac{e^{3x} - 1}{x} = \lim_{x \to 0} 3 \cdot \frac{e^{3x} - 1}{3x} = 3 \cdot 1 = 3$$

### Example 2

$$\lim_{x \to 0} \frac{e^x - e^{-x}}{x}$$

$$= \lim_{x \to 0} \frac{e^x - 1}{x} + \lim_{x \to 0} \frac{1 - e^{-x}}{x} = 1 + \lim_{x \to 0} \frac{e^{-x}(e^x - 1)}{x}$$

Alternatively:

$$= \lim_{x \to 0} \frac{e^x - 1}{x} + \lim_{x \to 0} \frac{e^{-x} - 1}{-x} = 1 + 1 = 2$$

---

## 2.4.7 Technique 6: Limits at Infinity

**Strategy:** Divide by the highest power of $x$ in the denominator.

### Example

$$\lim_{x \to \infty} \frac{2x^2 + 3}{5x^2 - x + 1}$$

Divide by $x^2$:

$$= \lim_{x \to \infty} \frac{2 + \frac{3}{x^2}}{5 - \frac{1}{x} + \frac{1}{x^2}} = \frac{2}{5}$$

### Functions involving radicals at infinity

$$\lim_{x \to \infty} (\sqrt{x^2 + x} - x)$$

Multiply by conjugate:

$$= \lim_{x \to \infty} \frac{(\sqrt{x^2+x}-x)(\sqrt{x^2+x}+x)}{\sqrt{x^2+x}+x} = \lim_{x \to \infty} \frac{x^2+x-x^2}{\sqrt{x^2+x}+x} = \lim_{x \to \infty} \frac{x}{\sqrt{x^2+x}+x}$$

Divide by $x$ (note $\sqrt{x^2} = x$ for $x > 0$):

$$= \lim_{x \to \infty} \frac{1}{\sqrt{1+\frac{1}{x}}+1} = \frac{1}{1+1} = \frac{1}{2}$$

---

## Summary Table

| Technique | When to Use | Key Move |
|-----------|------------|----------|
| Factoring | Polynomial $\frac{0}{0}$ | Factor, cancel common term |
| Rationalization | Square roots in $\frac{0}{0}$ | Multiply by conjugate |
| Trig manipulation | Trig functions near 0 | Use $\frac{\sin x}{x} \to 1$ |
| Substitution | Transform to standard form | Let $t = g(x)$ |
| Exponential | $e^x$, $a^x$, $\ln$ near 0 | Use $\frac{e^x-1}{x} \to 1$ |
| Divide by $x^n$ | Limits at $\infty$ | Divide num & denom by highest power |

---

## Quick Revision Questions

**Q1.** Evaluate $\lim_{x \to 4} \frac{x^2 - 16}{x - 4}$ by factoring.

**Q2.** Find $\lim_{x \to 0} \frac{\sqrt{1+x} - 1}{x}$ using rationalization.

**Q3.** Compute $\lim_{x \to 0} \frac{\sin 4x}{\sin 6x}$.

**Q4.** Using substitution, evaluate $\lim_{x \to \pi} \frac{\sin(x - \pi)}{x - \pi}$.

**Q5.** Find $\lim_{x \to 0} \frac{e^{2x} - 1}{\sin 3x}$.

**Q6.** Evaluate $\lim_{x \to \infty} \frac{x + 1}{\sqrt{4x^2 + 3}}$.

---

[← Previous: Laws of Limits](03-laws-of-limits.md) · [Next: Indeterminate Forms →](05-indeterminate-forms.md)
