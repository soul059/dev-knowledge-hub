# Unit 6 — Derivatives of Standard Functions: Trigonometric Derivatives

[← Previous: Polynomial Derivatives](01-polynomial-derivatives.md) · [Next: Inverse Trigonometric Derivatives →](03-inverse-trig-derivatives.md)

---

## Chapter Overview

This chapter presents the derivatives of all six trigonometric functions, their proofs, and worked examples involving combinations of trig functions with the differentiation rules.

---

## 6.2.1 Derivatives of the Six Trigonometric Functions

| $f(x)$ | $f'(x)$ |
|---------|----------|
| $\sin x$ | $\cos x$ |
| $\cos x$ | $-\sin x$ |
| $\tan x$ | $\sec^2 x$ |
| $\cot x$ | $-\csc^2 x$ |
| $\sec x$ | $\sec x \tan x$ |
| $\csc x$ | $-\csc x \cot x$ |

**Memory pattern — notice the negatives:**

```
    sin x  →  cos x          (positive)
    cos x  → -sin x          (negative)

    tan x  →  sec²x          (positive)
    cot x  → -csc²x          (negative)

    sec x  →  sec x tan x    (positive)
    csc x  → -csc x cot x    (negative)

    ┌─────────────────────────────────────┐
    │ "co" functions always get a MINUS   │
    └─────────────────────────────────────┘
```

---

## 6.2.2 Proofs from First Principles

### Proof: $\frac{d}{dx}(\sin x) = \cos x$

$$\frac{d}{dx}(\sin x) = \lim_{h \to 0}\frac{\sin(x+h) - \sin x}{h}$$

Using $\sin(x+h) = \sin x \cos h + \cos x \sin h$:

$$= \lim_{h \to 0}\frac{\sin x(\cos h - 1) + \cos x \sin h}{h}$$

$$= \sin x \cdot \lim_{h \to 0}\frac{\cos h - 1}{h} + \cos x \cdot \lim_{h \to 0}\frac{\sin h}{h}$$

$$= \sin x \cdot 0 + \cos x \cdot 1 = \cos x \quad \blacksquare$$

### Proof: $\frac{d}{dx}(\cos x) = -\sin x$

$$\frac{d}{dx}(\cos x) = \lim_{h \to 0}\frac{\cos(x+h) - \cos x}{h}$$

$$= \lim_{h \to 0}\frac{\cos x \cos h - \sin x \sin h - \cos x}{h}$$

$$= \cos x \cdot \underbrace{\lim_{h \to 0}\frac{\cos h - 1}{h}}_{0} - \sin x \cdot \underbrace{\lim_{h \to 0}\frac{\sin h}{h}}_{1} = -\sin x \quad \blacksquare$$

### Proof: $\frac{d}{dx}(\tan x) = \sec^2 x$

Using the quotient rule on $\tan x = \frac{\sin x}{\cos x}$:

$$\frac{d}{dx}\left(\frac{\sin x}{\cos x}\right) = \frac{\cos x \cdot \cos x - \sin x \cdot (-\sin x)}{\cos^2 x} = \frac{\cos^2 x + \sin^2 x}{\cos^2 x} = \frac{1}{\cos^2 x} = \sec^2 x \quad \blacksquare$$

---

## 6.2.3 Successive Derivatives of sin x and cos x

The derivatives of $\sin x$ cycle with period 4:

```
  sin x  ──→  cos x  ──→  -sin x  ──→  -cos x  ──→  sin x  ──→ ...
   f        f'          f''           f'''          f⁴
```

$$\frac{d^n}{dx^n}(\sin x) = \sin\left(x + \frac{n\pi}{2}\right)$$

Similarly:

$$\frac{d^n}{dx^n}(\cos x) = \cos\left(x + \frac{n\pi}{2}\right)$$

---

## 6.2.4 With Chain Rule: $\frac{d}{dx}[\text{trig}(u)]$

| $f(u)$ | $f'(u) \cdot u'$ |
|---------|-------------------|
| $\sin u$ | $\cos u \cdot u'$ |
| $\cos u$ | $-\sin u \cdot u'$ |
| $\tan u$ | $\sec^2 u \cdot u'$ |
| $\cot u$ | $-\csc^2 u \cdot u'$ |
| $\sec u$ | $\sec u \tan u \cdot u'$ |
| $\csc u$ | $-\csc u \cot u \cdot u'$ |

where $u = g(x)$.

---

## 6.2.5 Worked Examples

### Example 1: $y = \sin(3x)$

$$\frac{dy}{dx} = \cos(3x) \cdot 3 = 3\cos(3x)$$

### Example 2: $y = \cos^2 x = (\cos x)^2$

$$\frac{dy}{dx} = 2\cos x \cdot (-\sin x) = -2\sin x \cos x = -\sin 2x$$

### Example 3: $y = \tan(x^2 + 1)$

$$\frac{dy}{dx} = \sec^2(x^2 + 1) \cdot 2x$$

### Example 4: $y = x^2 \sin x$ (product rule)

$$\frac{dy}{dx} = 2x \sin x + x^2 \cos x$$

### Example 5: $y = \frac{\sin x}{1 + \cos x}$ (quotient rule)

$$\frac{dy}{dx} = \frac{\cos x(1 + \cos x) - \sin x(-\sin x)}{(1 + \cos x)^2}$$

$$= \frac{\cos x + \cos^2 x + \sin^2 x}{(1 + \cos x)^2} = \frac{1 + \cos x}{(1 + \cos x)^2} = \frac{1}{1 + \cos x}$$

### Example 6: $y = \sec^3(2x)$

$$\frac{dy}{dx} = 3\sec^2(2x) \cdot \sec(2x)\tan(2x) \cdot 2 = 6\sec^3(2x)\tan(2x)$$

---

## 6.2.6 Useful Identities for Simplification

| Identity | Use |
|----------|-----|
| $\sin^2 x + \cos^2 x = 1$ | Simplify sums of squares |
| $1 + \tan^2 x = \sec^2 x$ | Simplify $\tan$-$\sec$ expressions |
| $1 + \cot^2 x = \csc^2 x$ | Simplify $\cot$-$\csc$ expressions |
| $\sin 2x = 2\sin x \cos x$ | Simplify products |
| $\cos 2x = \cos^2 x - \sin^2 x$ | Simplify power expressions |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| $(\sin x)' = \cos x$ | Proven from first principles using $\lim \frac{\sin h}{h} = 1$ |
| "Co" functions | Co-function derivatives have a **minus sign** |
| Cycle of $\sin$ derivatives | Period 4: $\sin, \cos, -\sin, -\cos, \sin, \ldots$ |
| $n^{th}$ derivative of $\sin x$ | $\sin(x + n\pi/2)$ |
| Chain rule form | Multiply by inner derivative: $\cos(u) \cdot u'$ |

---

## Quick Revision Questions

**Q1.** Differentiate $y = 5\sin(2x) - 3\cos(4x)$.

**Q2.** Find $\frac{d}{dx}[\tan^2 x]$. Simplify your answer.

**Q3.** Prove that $\frac{d}{dx}(\sec x) = \sec x \tan x$ using the quotient rule.

**Q4.** Find the 50th derivative of $\cos x$.

**Q5.** Differentiate $y = \sin^3 x \cdot \cos x$.

**Q6.** Show that $\frac{d}{dx}\left[\frac{1 - \cos x}{\sin x}\right] = \frac{1}{1 + \cos x}$ (after simplifying to $\tan\frac{x}{2}$).

---

[← Previous: Polynomial Derivatives](01-polynomial-derivatives.md) · [Next: Inverse Trigonometric Derivatives →](03-inverse-trig-derivatives.md)
