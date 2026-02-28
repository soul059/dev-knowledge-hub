# Unit 2 — Limits: Indeterminate Forms

[← Previous: Evaluation Techniques](04-evaluation-techniques.md) · [Next: L'Hôpital's Rule →](06-lhopitals-rule.md)

---

## Chapter Overview

When direct substitution in a limit gives an expression like $\frac{0}{0}$, $\frac{\infty}{\infty}$, or $0 \cdot \infty$, the result is **not determined** by the form alone — it could be any value. These are called **indeterminate forms**. Recognizing them is the first step to knowing when special techniques (like L'Hôpital's Rule) are required.

---

## 2.5.1 The Seven Indeterminate Forms

| Form | Example | Why Indeterminate? |
|------|---------|-------------------|
| $\frac{0}{0}$ | $\lim_{x \to 0} \frac{\sin x}{x} = 1$ | Numerator and denominator both vanish; limit depends on *rates* |
| $\frac{\infty}{\infty}$ | $\lim_{x \to \infty} \frac{x^2}{e^x} = 0$ | Both grow without bound; limit depends on *relative growth* |
| $0 \cdot \infty$ | $\lim_{x \to 0^+} x \ln x = 0$ | One factor shrinks, the other explodes |
| $\infty - \infty$ | $\lim_{x \to \infty} (x - \sqrt{x^2 + x}) = -\frac{1}{2}$ | Two large quantities nearly cancel |
| $0^0$ | $\lim_{x \to 0^+} x^x = 1$ | Base → 0 (pulls toward 0), exponent → 0 (pulls toward 1) |
| $1^\infty$ | $\lim_{x \to \infty} (1 + \frac{1}{x})^x = e$ | Base → 1 (pulls toward 1), exponent → ∞ (pulls away) |
| $\infty^0$ | $\lim_{x \to \infty} x^{1/x} = 1$ | Base → ∞, exponent → 0 |

---

## 2.5.2 Forms That Are NOT Indeterminate

These **are** determined:

| Form | Result | Reason |
|------|--------|--------|
| $\frac{c}{0^+}$ ($c \ne 0$) | $+\infty$ or $-\infty$ | Nonzero divided by infinitesimal → blows up |
| $\frac{0}{c}$ ($c \ne 0$) | $0$ | Small quantity divided by finite → 0 |
| $0 \cdot c$ | $0$ | — |
| $\infty + \infty$ | $\infty$ | Both push to infinity |
| $0^\infty$ | $0$ | Small base raised to large power → 0 |
| $\infty^\infty$ | $\infty$ | — |

---

## 2.5.3 Identifying Indeterminate Forms — Worked Examples

### Example 1: $\frac{0}{0}$

$$\lim_{x \to 1} \frac{\ln x}{x - 1}$$

At $x = 1$: $\frac{\ln 1}{1 - 1} = \frac{0}{0}$ ← Indeterminate. Need L'Hôpital or substitution.

### Example 2: $\infty - \infty$

$$\lim_{x \to 0^+} \left(\frac{1}{x} - \frac{1}{\sin x}\right)$$

As $x \to 0^+$: $\frac{1}{x} \to \infty$ and $\frac{1}{\sin x} \to \infty$. Form: $\infty - \infty$ ← Indeterminate.

**Strategy:** Combine into a single fraction:

$$= \lim_{x \to 0^+} \frac{\sin x - x}{x \sin x} \quad \text{(now } \frac{0}{0}\text{)}$$

### Example 3: $1^\infty$

$$\lim_{x \to \infty} \left(1 + \frac{3}{x}\right)^x$$

Base $\to 1$, exponent $\to \infty$. Form: $1^\infty$ ← Indeterminate.

**Strategy:** Take logarithm.

Let $L = \lim_{x \to \infty} \left(1 + \frac{3}{x}\right)^x$

$$\ln L = \lim_{x \to \infty} x \ln\left(1 + \frac{3}{x}\right) \quad \text{(form: } \infty \cdot 0\text{)}$$

$$= \lim_{x \to \infty} \frac{\ln(1 + 3/x)}{1/x} \quad \text{(now } \frac{0}{0}\text{)} \to \text{use L'Hôpital}$$

Result: $\ln L = 3$, so $L = e^3$.

### Example 4: $0 \cdot \infty$

$$\lim_{x \to 0^+} x \ln x$$

$x \to 0^+$ and $\ln x \to -\infty$. Form: $0 \cdot (-\infty)$ ← Indeterminate.

**Strategy:** Rewrite as $\frac{0}{0}$ or $\frac{\infty}{\infty}$:

$$= \lim_{x \to 0^+} \frac{\ln x}{1/x} \quad \text{(form: } \frac{-\infty}{\infty}\text{)}$$

---

## 2.5.4 Conversion Strategies Between Forms

```
   0 · ∞  ──────▶  0/0  or  ∞/∞
                    (rewrite as fraction)

   ∞ - ∞  ──────▶  0/0  or  ∞/∞
                    (combine fractions or rationalize)

   0⁰, 1^∞, ∞⁰  ──▶  Take y = f(x)^g(x)
                       ln y = g(x)·ln f(x)  →  0·∞
                       Then convert to 0/0 or ∞/∞
```

### Conversion Table

| Original Form | Rewrite As |
|---------------|-----------|
| $f \cdot g$ where $f \to 0$, $g \to \infty$ | $\frac{f}{1/g}$ → $\frac{0}{0}$ **or** $\frac{g}{1/f}$ → $\frac{\infty}{\infty}$ |
| $f - g$ where both $\to \infty$ | $\frac{1/g - 1/f}{1/(fg)}$ or combine fractions |
| $f^g$ (exponential indeterminate) | $e^{g \ln f}$, then analyze $g \ln f$ |

---

## 2.5.5 The Exponential Limit Template for $1^\infty$ Forms

For limits of the form $\lim [1 + \phi(x)]^{\psi(x)}$ where $\phi(x) \to 0$ and $\psi(x) \to \infty$:

$$\lim [1 + \phi(x)]^{\psi(x)} = e^{\lim \, \phi(x) \cdot \psi(x)}$$

provided the limit of $\phi(x) \cdot \psi(x)$ exists.

**Example:**

$$\lim_{x \to \infty} \left(1 + \frac{5}{x}\right)^{2x} = e^{\lim_{x \to \infty} \frac{5}{x} \cdot 2x} = e^{10}$$

---

## Summary Table

| Form | Indeterminate? | Strategy |
|------|---------------|----------|
| $\frac{0}{0}$ | Yes | Factor, rationalize, trig, L'Hôpital |
| $\frac{\infty}{\infty}$ | Yes | Divide by dominant term, L'Hôpital |
| $0 \cdot \infty$ | Yes | Rewrite as $\frac{0}{0}$ or $\frac{\infty}{\infty}$ |
| $\infty - \infty$ | Yes | Combine fractions, rationalize |
| $0^0$ | Yes | Take logarithm, then L'Hôpital |
| $1^\infty$ | Yes | $e^{\lim g \ln f}$ or take logarithm |
| $\infty^0$ | Yes | Take logarithm |
| $\frac{c}{0}$ ($c \ne 0$) | **No** | Result is $\pm\infty$ |
| $0^\infty$ | **No** | Result is $0$ |

---

## Quick Revision Questions

**Q1.** List all seven indeterminate forms.

**Q2.** Identify the indeterminate form: $\lim_{x \to 0^+} (\sin x)^{\tan x}$.

**Q3.** Convert $\lim_{x \to 0^+} x^2 \cot x$ (a $0 \cdot \infty$ form) into $\frac{0}{0}$ form.

**Q4.** Why is $\frac{5}{0}$ not indeterminate, while $\frac{0}{0}$ is?

**Q5.** Show that $\lim_{x \to \infty} \left(1 + \frac{2}{x}\right)^{3x} = e^6$.

**Q6.** Identify the form and appropriate strategy for $\lim_{x \to \infty} (x - \ln x)$.

---

[← Previous: Evaluation Techniques](04-evaluation-techniques.md) · [Next: L'Hôpital's Rule →](06-lhopitals-rule.md)
