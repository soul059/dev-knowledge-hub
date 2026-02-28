# Unit 5 — Differentiation Rules: Quotient Rule

[← Previous: Power, Sum, and Product Rules](01-power-sum-product-rules.md) · [Next: Chain Rule →](03-chain-rule.md)

---

## Chapter Overview

The **Quotient Rule** handles derivatives of fractions — ratios of two functions. While it can always be derived from the product and chain rules, having it as a standalone formula is convenient for direct application.

---

## 5.2.1 Statement

If $f$ and $g$ are differentiable and $g(x) \ne 0$:

$$\boxed{\frac{d}{dx}\left[\frac{f(x)}{g(x)}\right] = \frac{f'(x)\,g(x) - f(x)\,g'(x)}{[g(x)]^2}}$$

**Mnemonic:** "Low d-High minus High d-Low, over Low squared"

$$\left(\frac{f}{g}\right)' = \frac{f'g - fg'}{g^2}$$

---

## 5.2.2 Proof

Using the product rule on $f(x) = \frac{f}{g} \cdot g$:

Let $q(x) = \frac{f(x)}{g(x)}$, so $f(x) = q(x) \cdot g(x)$.

By the product rule:

$$f'(x) = q'(x) \cdot g(x) + q(x) \cdot g'(x)$$

$$q'(x) = \frac{f'(x) - q(x)\,g'(x)}{g(x)} = \frac{f'(x) - \frac{f(x)}{g(x)}\,g'(x)}{g(x)} = \frac{f'(x)\,g(x) - f(x)\,g'(x)}{[g(x)]^2} \quad \blacksquare$$

---

## 5.2.3 Worked Examples

### Example 1: Basic rational function

$$\frac{d}{dx}\left(\frac{x+1}{x-1}\right)$$

$f = x+1$, $f' = 1$; $g = x-1$, $g' = 1$.

$$= \frac{1 \cdot (x-1) - (x+1) \cdot 1}{(x-1)^2} = \frac{x - 1 - x - 1}{(x-1)^2} = \frac{-2}{(x-1)^2}$$

### Example 2

$$\frac{d}{dx}\left(\frac{x^2}{x+3}\right)$$

$f = x^2$, $f' = 2x$; $g = x+3$, $g' = 1$.

$$= \frac{2x(x+3) - x^2 \cdot 1}{(x+3)^2} = \frac{2x^2 + 6x - x^2}{(x+3)^2} = \frac{x^2 + 6x}{(x+3)^2} = \frac{x(x+6)}{(x+3)^2}$$

### Example 3: Deriving $\frac{d}{dx}(\tan x)$

$$\frac{d}{dx}(\tan x) = \frac{d}{dx}\left(\frac{\sin x}{\cos x}\right)$$

$$= \frac{\cos x \cdot \cos x - \sin x \cdot (-\sin x)}{\cos^2 x} = \frac{\cos^2 x + \sin^2 x}{\cos^2 x} = \frac{1}{\cos^2 x} = \sec^2 x$$

### Example 4

$$\frac{d}{dx}\left(\frac{e^x}{x^2 + 1}\right) = \frac{e^x(x^2+1) - e^x \cdot 2x}{(x^2+1)^2} = \frac{e^x(x^2 - 2x + 1)}{(x^2+1)^2} = \frac{e^x(x-1)^2}{(x^2+1)^2}$$

---

## 5.2.4 When NOT to Use the Quotient Rule

Sometimes it's simpler to rewrite the expression first:

| Expression | Rewrite | Rule to use |
|-----------|---------|-------------|
| $\frac{5}{x^3}$ | $5x^{-3}$ | Power rule: $-15x^{-4}$ |
| $\frac{x^4 + x}{x}$ | $x^3 + 1$ | Direct: $3x^2$ |
| $\frac{3}{\sqrt{x}}$ | $3x^{-1/2}$ | Power rule: $-\frac{3}{2}x^{-3/2}$ |
| $\frac{\sin x}{3}$ | $\frac{1}{3}\sin x$ | Constant multiple: $\frac{1}{3}\cos x$ |

> **Tip:** If the denominator is just a constant or a simple power of $x$, avoid the quotient rule — simplify first.

---

## 5.2.5 Comparison: Product vs Quotient Rule

| | Product Rule | Quotient Rule |
|--|-------------|---------------|
| **For** | $f \cdot g$ | $f / g$ |
| **Formula** | $f'g + fg'$ | $\frac{f'g - fg'}{g^2}$ |
| **Sign** | Plus (+) | Minus (−) in numerator |
| **Denominator** | None | $g^2$ |
| **Alternative** | — | Can use product rule on $f \cdot g^{-1}$ |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Quotient Rule | $\left(\frac{f}{g}\right)' = \frac{f'g - fg'}{g^2}$ |
| $\tan x$ derivative | $\sec^2 x$ |
| $\cot x$ derivative | $-\csc^2 x$ |
| $\sec x$ derivative | $\sec x \tan x$ |
| $\csc x$ derivative | $-\csc x \cot x$ |

---

## Quick Revision Questions

**Q1.** Differentiate $\frac{3x + 2}{x^2 - 1}$ using the quotient rule.

**Q2.** Derive $\frac{d}{dx}(\cot x) = -\csc^2 x$ using the quotient rule.

**Q3.** Differentiate $\frac{x^2 + 1}{e^x}$ and simplify.

**Q4.** Why is it simpler to differentiate $\frac{8}{x^4}$ using the power rule instead of the quotient rule?

**Q5.** Find $\frac{d}{dx}\left(\frac{\ln x}{x}\right)$.

**Q6.** Derive $\frac{d}{dx}(\sec x)$ from $\frac{d}{dx}\left(\frac{1}{\cos x}\right)$.

---

[← Previous: Power, Sum, and Product Rules](01-power-sum-product-rules.md) · [Next: Chain Rule →](03-chain-rule.md)
