# Unit 5 — Differentiation Rules: Logarithmic Differentiation

[← Previous: Implicit Differentiation](04-implicit-differentiation.md) · [Next: Parametric Differentiation →](06-parametric-differentiation.md)

---

## Chapter Overview

**Logarithmic differentiation** is a technique where we take the natural logarithm of both sides of $y = f(x)$ before differentiating. It is especially useful for:
- Functions with **variables in both base and exponent** (e.g., $x^x$)
- **Products and quotients of many factors**
- Functions where direct rules would be cumbersome

---

## 5.5.1 The Method

**Step-by-step:**

1. Let $y = f(x)$
2. Take $\ln$ of both sides: $\ln y = \ln f(x)$
3. Simplify using log properties
4. Differentiate both sides w.r.t. $x$ (using $\frac{d}{dx}[\ln y] = \frac{1}{y}\frac{dy}{dx}$)
5. Solve for $\frac{dy}{dx}$
6. Substitute back $y = f(x)$

---

## 5.5.2 Why It Works

The key identity is:

$$\frac{d}{dx}[\ln y] = \frac{1}{y} \cdot \frac{dy}{dx}$$

so

$$\frac{dy}{dx} = y \cdot \frac{d}{dx}[\ln y]$$

Logarithms convert:
- **Products → Sums**: $\ln(ab) = \ln a + \ln b$
- **Quotients → Differences**: $\ln\frac{a}{b} = \ln a - \ln b$
- **Powers → Multiples**: $\ln(a^n) = n \ln a$

---

## 5.5.3 Worked Examples

### Example 1: $y = x^x$ (Variable Base AND Exponent)

Neither power rule ($\frac{d}{dx}x^n = nx^{n-1}$) nor exponential rule ($\frac{d}{dx}a^x = a^x \ln a$) applies, because $x$ is in **both** base and exponent.

$$\ln y = x \ln x$$

Differentiate:

$$\frac{1}{y}\frac{dy}{dx} = 1 \cdot \ln x + x \cdot \frac{1}{x} = \ln x + 1$$

$$\frac{dy}{dx} = y(\ln x + 1) = x^x(\ln x + 1)$$

### Example 2: $y = x^{\sin x}$

$$\ln y = \sin x \cdot \ln x$$

$$\frac{1}{y}\frac{dy}{dx} = \cos x \cdot \ln x + \sin x \cdot \frac{1}{x}$$

$$\frac{dy}{dx} = x^{\sin x}\left(\cos x \cdot \ln x + \frac{\sin x}{x}\right)$$

### Example 3: $y = (\sin x)^{\cos x}$

$$\ln y = \cos x \cdot \ln(\sin x)$$

$$\frac{1}{y}\frac{dy}{dx} = -\sin x \cdot \ln(\sin x) + \cos x \cdot \frac{\cos x}{\sin x}$$

$$\frac{dy}{dx} = (\sin x)^{\cos x}\left(-\sin x \cdot \ln(\sin x) + \frac{\cos^2 x}{\sin x}\right)$$

### Example 4: Complicated product

$$y = \frac{x^3(x+1)^4}{\sqrt{x^2+1}}$$

$$\ln y = 3\ln x + 4\ln(x+1) - \frac{1}{2}\ln(x^2+1)$$

Differentiate:

$$\frac{1}{y}\frac{dy}{dx} = \frac{3}{x} + \frac{4}{x+1} - \frac{x}{x^2+1}$$

$$\frac{dy}{dx} = \frac{x^3(x+1)^4}{\sqrt{x^2+1}} \left(\frac{3}{x} + \frac{4}{x+1} - \frac{x}{x^2+1}\right)$$

### Example 5: $y = x^{1/x}$

$$\ln y = \frac{\ln x}{x}$$

$$\frac{1}{y}\frac{dy}{dx} = \frac{\frac{1}{x} \cdot x - \ln x \cdot 1}{x^2} = \frac{1 - \ln x}{x^2}$$

$$\frac{dy}{dx} = x^{1/x} \cdot \frac{1 - \ln x}{x^2}$$

---

## 5.5.4 When to Use Logarithmic Differentiation

| Situation | Example | Why Log Diff? |
|-----------|---------|--------------|
| Variable in base AND exponent | $x^x$, $x^{\sin x}$ | No standard rule covers this |
| Product of many factors | $x^3(x+1)^4\sqrt{x-2}$ | Logs simplify to sums |
| Complex quotients | $\frac{(x+1)^5(x+2)^3}{(x+3)^7}$ | Logs simplify to differences |
| General $f(x)^{g(x)}$ | $(\cos x)^{x^2}$ | Always use log diff |

---

## 5.5.5 The General Formula for $f(x)^{g(x)}$

$$\frac{d}{dx}\left[f(x)^{g(x)}\right] = f(x)^{g(x)}\left[g'(x)\ln f(x) + g(x)\frac{f'(x)}{f(x)}\right]$$

**Derivation:** Let $y = f^g$, so $\ln y = g \ln f$.

$$\frac{1}{y}\frac{dy}{dx} = g'\ln f + g \cdot \frac{f'}{f}$$

This formula combines the "exponential part" ($g' \ln f$, treating $f$ as constant) and the "base part" ($g \cdot f'/f$, treating $g$ as constant).

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Logarithmic differentiation | Take $\ln$ of both sides, then differentiate |
| Key identity | $\frac{d}{dx}[\ln y] = \frac{1}{y}\frac{dy}{dx}$ |
| For $f^g$ form | $\frac{dy}{dx} = f^g[g'\ln f + gf'/f]$ |
| For products | Logs turn products into sums |
| For quotients | Logs turn quotients into differences |
| For powers | Logs bring exponents down as coefficients |

---

## Quick Revision Questions

**Q1.** Differentiate $y = x^{x^2}$ using logarithmic differentiation.

**Q2.** Find $\frac{dy}{dx}$ for $y = (\ln x)^x$.

**Q3.** Differentiate $y = \frac{x^2 \sqrt{x+1}}{(2x-1)^3}$ using log diff.

**Q4.** Why can't we use the power rule on $x^x$?

**Q5.** Find $\frac{dy}{dx}$ for $y = (\tan x)^{\cot x}$.

**Q6.** Show that $\frac{d}{dx}(a^x) = a^x \ln a$ using logarithmic differentiation.

---

[← Previous: Implicit Differentiation](04-implicit-differentiation.md) · [Next: Parametric Differentiation →](06-parametric-differentiation.md)
