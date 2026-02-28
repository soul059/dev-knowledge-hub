# Unit 6 — Derivatives of Standard Functions: Inverse Trigonometric Derivatives

[← Previous: Trigonometric Derivatives](02-trigonometric-derivatives.md) · [Next: Exponential & Logarithmic Derivatives →](04-exponential-log-derivatives.md)

---

## Chapter Overview

The inverse trigonometric functions ($\sin^{-1} x$, $\cos^{-1} x$, $\tan^{-1} x$, etc.) have algebraic derivatives involving square roots. This chapter derives each formula, explains their domains, and shows how to handle composite forms using the chain rule.

---

## 6.3.1 Standard Inverse Trigonometric Derivatives

| $f(x)$ | $f'(x)$ | Domain |
|---------|----------|--------|
| $\sin^{-1} x$ | $\dfrac{1}{\sqrt{1 - x^2}}$ | $\|x\| < 1$ |
| $\cos^{-1} x$ | $-\dfrac{1}{\sqrt{1 - x^2}}$ | $\|x\| < 1$ |
| $\tan^{-1} x$ | $\dfrac{1}{1 + x^2}$ | $x \in \mathbb{R}$ |
| $\cot^{-1} x$ | $-\dfrac{1}{1 + x^2}$ | $x \in \mathbb{R}$ |
| $\sec^{-1} x$ | $\dfrac{1}{\|x\|\sqrt{x^2 - 1}}$ | $\|x\| > 1$ |
| $\csc^{-1} x$ | $-\dfrac{1}{\|x\|\sqrt{x^2 - 1}}$ | $\|x\| > 1$ |

**Key pattern:**

```
  ┌──────────────────────────────────────────────┐
  │  sin⁻¹ and cos⁻¹  →  involve √(1 - x²)     │
  │  tan⁻¹ and cot⁻¹  →  involve (1 + x²)      │
  │  sec⁻¹ and csc⁻¹  →  involve |x|√(x² - 1)  │
  │                                              │
  │  Co-inverse functions have MINUS signs       │
  │  (cos⁻¹, cot⁻¹, csc⁻¹)                     │
  └──────────────────────────────────────────────┘
```

---

## 6.3.2 Proof: $\frac{d}{dx}(\sin^{-1} x)$

Let $y = \sin^{-1} x$, so $\sin y = x$ where $y \in [-\pi/2, \pi/2]$.

Differentiate both sides implicitly:

$$\cos y \cdot \frac{dy}{dx} = 1$$

$$\frac{dy}{dx} = \frac{1}{\cos y}$$

Since $\cos y = \sqrt{1 - \sin^2 y} = \sqrt{1 - x^2}$ (positive because $y \in [-\pi/2, \pi/2]$):

$$\frac{d}{dx}(\sin^{-1} x) = \frac{1}{\sqrt{1 - x^2}} \quad \blacksquare$$

---

## 6.3.3 Proof: $\frac{d}{dx}(\tan^{-1} x)$

Let $y = \tan^{-1} x$, so $\tan y = x$ where $y \in (-\pi/2, \pi/2)$.

$$\sec^2 y \cdot \frac{dy}{dx} = 1$$

$$\frac{dy}{dx} = \frac{1}{\sec^2 y} = \frac{1}{1 + \tan^2 y} = \frac{1}{1 + x^2} \quad \blacksquare$$

---

## 6.3.4 Relationship: $\sin^{-1} x + \cos^{-1} x = \frac{\pi}{2}$

Since $\sin^{-1} x + \cos^{-1} x = \frac{\pi}{2}$:

$$\frac{d}{dx}(\cos^{-1} x) = \frac{d}{dx}\left(\frac{\pi}{2} - \sin^{-1} x\right) = -\frac{1}{\sqrt{1 - x^2}}$$

Similarly:
- $\tan^{-1} x + \cot^{-1} x = \frac{\pi}{2} \Rightarrow (\cot^{-1} x)' = -\frac{1}{1+x^2}$
- $\sec^{-1} x + \csc^{-1} x = \frac{\pi}{2} \Rightarrow (\csc^{-1} x)' = -\frac{1}{|x|\sqrt{x^2-1}}$

---

## 6.3.5 Chain Rule Forms

| $f(u)$ where $u = g(x)$ | $f'(u) \cdot u'$ |
|--------------------------|-------------------|
| $\sin^{-1}(u)$ | $\dfrac{u'}{\sqrt{1 - u^2}}$ |
| $\cos^{-1}(u)$ | $-\dfrac{u'}{\sqrt{1 - u^2}}$ |
| $\tan^{-1}(u)$ | $\dfrac{u'}{1 + u^2}$ |

---

## 6.3.6 Worked Examples

### Example 1: $y = \sin^{-1}(2x)$

$$\frac{dy}{dx} = \frac{2}{\sqrt{1 - 4x^2}}$$

### Example 2: $y = \tan^{-1}\left(\frac{x}{a}\right)$

$$\frac{dy}{dx} = \frac{1/a}{1 + x^2/a^2} = \frac{1/a}{(a^2 + x^2)/a^2} = \frac{a}{a^2 + x^2}$$

This is a **very important** standard result:

$$\frac{d}{dx}\left[\tan^{-1}\frac{x}{a}\right] = \frac{a}{a^2 + x^2}$$

### Example 3: $y = \sin^{-1}\left(\frac{2x}{1 + x^2}\right)$

Let $x = \tan\theta$, so $\frac{2\tan\theta}{1 + \tan^2\theta} = \sin 2\theta$.

$$y = \sin^{-1}(\sin 2\theta) = 2\theta = 2\tan^{-1} x$$

$$\frac{dy}{dx} = \frac{2}{1 + x^2}$$

### Example 4: $y = \cos^{-1}\left(\frac{1 - x^2}{1 + x^2}\right)$

Using $x = \tan\theta$: $\frac{1 - \tan^2\theta}{1 + \tan^2\theta} = \cos 2\theta$

$$y = \cos^{-1}(\cos 2\theta) = 2\theta = 2\tan^{-1} x$$

$$\frac{dy}{dx} = \frac{2}{1 + x^2}$$

### Example 5: $y = \tan^{-1}\left(\frac{3x - x^3}{1 - 3x^2}\right)$

Using $x = \tan\theta$: numerator $= 3\tan\theta - \tan^3\theta$, denominator $= 1 - 3\tan^2\theta$, which equals $\tan 3\theta$.

$$y = \tan^{-1}(\tan 3\theta) = 3\theta = 3\tan^{-1} x$$

$$\frac{dy}{dx} = \frac{3}{1 + x^2}$$

---

## 6.3.7 Important Substitution Techniques

| Expression inside inverse trig | Substitution | Simplification |
|-------------------------------|--------------|----------------|
| $\frac{2x}{1+x^2}$ | $x = \tan\theta$ | $\sin 2\theta$ |
| $\frac{1-x^2}{1+x^2}$ | $x = \tan\theta$ | $\cos 2\theta$ |
| $\frac{2x}{1-x^2}$ | $x = \tan\theta$ | $\tan 2\theta$ |
| $\frac{3x - x^3}{1 - 3x^2}$ | $x = \tan\theta$ | $\tan 3\theta$ |
| $2x^2 - 1$ | $x = \cos\theta$ | $\cos 2\theta$ |
| $3x - 4x^3$ | $x = \sin\theta$ | $\sin 3\theta$ |
| $4x^3 - 3x$ | $x = \cos\theta$ | $\cos 3\theta$ |

---

## 6.3.8 Graphs of Inverse Trig Derivatives

```
  d/dx[sin⁻¹x]          d/dx[tan⁻¹x]
  = 1/√(1-x²)           = 1/(1+x²)

  f'(x)                  f'(x)
   |     *               1 +-----*-----+
   |    * *              |    *       *
   |   *   *             |  *           *
   |  *     *            | *               *
  1+*       *      0.5  +*                   *
   |*       *+           |
   +--+---+--→ x         +---+---+---+---→ x
  -1  0   1             -3  -1   0   1   3

  (vertical asymptotes   (bell-shaped curve,
   at x = ±1)            max value 1 at x = 0)
```

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| $(\sin^{-1} x)'$ | $\frac{1}{\sqrt{1-x^2}}$, proven via implicit diff |
| $(\tan^{-1} x)'$ | $\frac{1}{1+x^2}$, proven via implicit diff |
| Co-inverse relation | $\sin^{-1} + \cos^{-1} = \pi/2$ gives sign flip |
| Substitution trick | Use $x = \tan\theta$ to simplify complex arguments |
| $\tan^{-1}(x/a)$ | Derivative is $\frac{a}{a^2+x^2}$ (standard result) |

---

## Quick Revision Questions

**Q1.** Prove that $\frac{d}{dx}(\sec^{-1} x) = \frac{1}{|x|\sqrt{x^2-1}}$ using implicit differentiation.

**Q2.** Differentiate $y = \tan^{-1}(\sqrt{x})$.

**Q3.** If $y = \sin^{-1}\left(\frac{2x}{1+x^2}\right)$, show that $\frac{dy}{dx} = \frac{2}{1+x^2}$.

**Q4.** Find $\frac{dy}{dx}$ for $y = x\sin^{-1}x + \sqrt{1 - x^2}$.

**Q5.** Differentiate $y = \tan^{-1}\left(\frac{a + x}{1 - ax}\right)$ and simplify.

**Q6.** Show that $\frac{d}{dx}\left[\cos^{-1}\left(\frac{1-x^2}{1+x^2}\right)\right] = \frac{2}{1+x^2}$ for $x > 0$.

---

[← Previous: Trigonometric Derivatives](02-trigonometric-derivatives.md) · [Next: Exponential & Logarithmic Derivatives →](04-exponential-log-derivatives.md)
