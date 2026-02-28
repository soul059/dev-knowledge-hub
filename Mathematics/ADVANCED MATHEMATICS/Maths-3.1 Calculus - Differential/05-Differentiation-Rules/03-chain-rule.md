# Unit 5 — Differentiation Rules: Chain Rule

[← Previous: Quotient Rule](02-quotient-rule.md) · [Next: Implicit Differentiation →](04-implicit-differentiation.md)

---

## Chapter Overview

The **Chain Rule** is the most important and widely used differentiation rule. It tells us how to differentiate **composite functions** — functions of functions. Once you master the chain rule, you can differentiate almost anything.

---

## 5.3.1 Statement

If $y = f(g(x))$, where $f$ and $g$ are both differentiable, then:

$$\boxed{\frac{dy}{dx} = f'(g(x)) \cdot g'(x)}$$

**Leibniz notation (chain):** If $y = f(u)$ and $u = g(x)$:

$$\frac{dy}{dx} = \frac{dy}{du} \cdot \frac{du}{dx}$$

**The chain of functions:**

```
  x ──▶ g(x) = u ──▶ f(u) = y

  dx ──▶ du/dx ──▶ dy/du

  dy/dx = (dy/du) × (du/dx)
```

---

## 5.3.2 Intuitive Understanding

If $u$ changes 3 times as fast as $x$, and $y$ changes 5 times as fast as $u$, then $y$ changes $5 \times 3 = 15$ times as fast as $x$.

**Rate of change multiplies along the chain.**

---

## 5.3.3 Proof Sketch

$$\frac{dy}{dx} = \lim_{\Delta x \to 0} \frac{f(g(x + \Delta x)) - f(g(x))}{\Delta x}$$

Let $\Delta u = g(x + \Delta x) - g(x)$. If $\Delta u \ne 0$:

$$= \lim_{\Delta x \to 0} \frac{f(g(x) + \Delta u) - f(g(x))}{\Delta u} \cdot \frac{\Delta u}{\Delta x} = f'(g(x)) \cdot g'(x) \quad \blacksquare$$

(A rigorous proof handles the case $\Delta u = 0$ carefully.)

---

## 5.3.4 Worked Examples

### Example 1: $(2x + 3)^5$

Let $u = 2x + 3$ (inner), $y = u^5$ (outer).

$$\frac{dy}{dx} = 5u^4 \cdot 2 = 10(2x+3)^4$$

### Example 2: $\sin(x^2)$

$$\frac{d}{dx}[\sin(x^2)] = \cos(x^2) \cdot 2x = 2x\cos(x^2)$$

### Example 3: $e^{3x^2 + 1}$

$$\frac{d}{dx}[e^{3x^2+1}] = e^{3x^2+1} \cdot 6x = 6xe^{3x^2+1}$$

### Example 4: $\ln(\cos x)$

$$\frac{d}{dx}[\ln(\cos x)] = \frac{1}{\cos x} \cdot (-\sin x) = -\tan x$$

### Example 5: $\sqrt{1 + x^2}$

$$\frac{d}{dx}[(1+x^2)^{1/2}] = \frac{1}{2}(1+x^2)^{-1/2} \cdot 2x = \frac{x}{\sqrt{1+x^2}}$$

### Example 6: Double chain — $\sin^2(\ln x) = [\sin(\ln x)]^2$

Outer: $u^2$; Middle: $\sin v$; Inner: $\ln x$.

$$\frac{d}{dx} = 2\sin(\ln x) \cdot \cos(\ln x) \cdot \frac{1}{x} = \frac{2\sin(\ln x)\cos(\ln x)}{x} = \frac{\sin(2\ln x)}{x}$$

---

## 5.3.5 General Patterns

| Function | Derivative (via chain rule) |
|----------|---------------------------|
| $[g(x)]^n$ | $n[g(x)]^{n-1} \cdot g'(x)$ |
| $\sin(g(x))$ | $\cos(g(x)) \cdot g'(x)$ |
| $\cos(g(x))$ | $-\sin(g(x)) \cdot g'(x)$ |
| $\tan(g(x))$ | $\sec^2(g(x)) \cdot g'(x)$ |
| $e^{g(x)}$ | $e^{g(x)} \cdot g'(x)$ |
| $\ln(g(x))$ | $\frac{g'(x)}{g(x)}$ |
| $a^{g(x)}$ | $a^{g(x)} \ln a \cdot g'(x)$ |

---

## 5.3.6 Chain Rule with Multiple Links

For triple composition $y = f(g(h(x)))$:

$$\frac{dy}{dx} = f'(g(h(x))) \cdot g'(h(x)) \cdot h'(x)$$

**Pipeline:**

```
  x ──▶ h(x) ──▶ g(·) ──▶ f(·) = y

  dy/dx = f' · g' · h'
  (each evaluated at the correct input)
```

### Example: $e^{\sin(x^3)}$

$h(x) = x^3$, $g(t) = \sin t$, $f(u) = e^u$

$$\frac{dy}{dx} = e^{\sin(x^3)} \cdot \cos(x^3) \cdot 3x^2 = 3x^2 \cos(x^3)\, e^{\sin(x^3)}$$

---

## 5.3.7 Common Mistakes

| Mistake | Correction |
|---------|-----------|
| $\frac{d}{dx}\sin(3x) = \cos(3x)$ | Missing inner derivative: $= 3\cos(3x)$ |
| $\frac{d}{dx}(2x+1)^4 = 4(2x+1)^3$ | Missing chain: $= 8(2x+1)^3$ |
| $\frac{d}{dx}e^{x^2} = e^{x^2}$ | Missing chain: $= 2xe^{x^2}$ |
| Applying chain rule when not needed | $\frac{d}{dx}(x^3) = 3x^2$ — no inner function, no chain |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Chain Rule | $\frac{d}{dx}[f(g(x))] = f'(g(x)) \cdot g'(x)$ |
| Leibniz form | $\frac{dy}{dx} = \frac{dy}{du} \cdot \frac{du}{dx}$ |
| Triple chain | $\frac{dy}{dx} = f' \cdot g' \cdot h'$ |
| Key principle | Differentiate outside, multiply by derivative of inside |

---

## Quick Revision Questions

**Q1.** Differentiate $(5x^2 - 3)^7$.

**Q2.** Find $\frac{d}{dx}[\cos(e^x)]$.

**Q3.** Differentiate $\ln(\sin x)$ and simplify.

**Q4.** If $y = [1 + \tan(x^2)]^3$, find $\frac{dy}{dx}$.

**Q5.** Show that $\frac{d}{dx}(e^{kx}) = ke^{kx}$ using the chain rule.

**Q6.** Find $\frac{d}{dx}\left[\sqrt{\ln(x^2+1)}\right]$.

---

[← Previous: Quotient Rule](02-quotient-rule.md) · [Next: Implicit Differentiation →](04-implicit-differentiation.md)
