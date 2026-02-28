# Unit 4 — Differentiability: Definition (First Principles)

[← Previous: Intermediate Value Theorem](../03-Continuity/04-intermediate-value-theorem.md) · [Next: Geometric Interpretation →](02-geometric-interpretation.md)

---

## Chapter Overview

The **derivative** captures the idea of instantaneous rate of change. It is defined as a **limit** — the limit of the difference quotient as the interval shrinks to zero. This chapter introduces the derivative from **first principles** (the limit definition), and shows how to compute derivatives directly from the definition.

---

## 4.1.1 Motivation: From Average to Instantaneous

### Average Rate of Change

For a function $f$ over $[x, x+h]$:

$$\text{Average rate of change} = \frac{f(x+h) - f(x)}{h}$$

This is the **slope of the secant line** through $(x, f(x))$ and $(x+h, f(x+h))$.

```
  y
  │                  ● (x+h, f(x+h))
  │                ╱│
  │              ╱  │ ← rise = f(x+h) - f(x)
  │            ╱    │
  │     ●────╱      │
  │    (x, f(x))    │
  │          |──────|
  │          run = h
  └─────────────────── x
```

### Instantaneous Rate of Change

Let $h \to 0$: the secant line approaches the **tangent line**, and the average rate approaches the **instantaneous rate**.

```
  y                  secant lines → tangent
  │          ╱╱╱╱▶
  │        ╱╱╱╱╱
  │      ╱╱╱╱╱╱
  │     ●╱╱╱╱╱   ← as h → 0, secant → tangent
  │    ╱╱╱╱╱
  │   ╱╱╱╱
  └──────────── x
```

---

## 4.1.2 The Derivative — Definition

The **derivative** of $f$ at $x = a$ is:

$$\boxed{f'(a) = \lim_{h \to 0} \frac{f(a+h) - f(a)}{h}}$$

**Equivalent form (using $x \to a$):**

$$f'(a) = \lim_{x \to a} \frac{f(x) - f(a)}{x - a}$$

If this limit exists, we say $f$ is **differentiable** at $x = a$.

### The Derivative as a Function

If the limit exists for all $x$ in some domain, the derivative **function** is:

$$f'(x) = \lim_{h \to 0} \frac{f(x+h) - f(x)}{h}$$

### Notation

| Notation | Read as |
|----------|---------|
| $f'(x)$ | "f prime of x" |
| $\frac{dy}{dx}$ | "dy by dx" (Leibniz notation) |
| $\frac{df}{dx}$ | "df by dx" |
| $\dot{y}$ | "y dot" (Newton, physics) |
| $Df(x)$ | "D of f" (operator notation) |

---

## 4.1.3 Worked Examples from First Principles

### Example 1: $f(x) = x^2$

$$f'(x) = \lim_{h \to 0} \frac{(x+h)^2 - x^2}{h} = \lim_{h \to 0} \frac{x^2 + 2xh + h^2 - x^2}{h}$$

$$= \lim_{h \to 0} \frac{2xh + h^2}{h} = \lim_{h \to 0} (2x + h) = 2x$$

**Result:** $\frac{d}{dx}(x^2) = 2x$

### Example 2: $f(x) = x^3$

$$f'(x) = \lim_{h \to 0} \frac{(x+h)^3 - x^3}{h}$$

Expand $(x+h)^3 = x^3 + 3x^2h + 3xh^2 + h^3$:

$$= \lim_{h \to 0} \frac{3x^2h + 3xh^2 + h^3}{h} = \lim_{h \to 0} (3x^2 + 3xh + h^2) = 3x^2$$

**Result:** $\frac{d}{dx}(x^3) = 3x^2$

### Example 3: $f(x) = \frac{1}{x}$

$$f'(x) = \lim_{h \to 0} \frac{\frac{1}{x+h} - \frac{1}{x}}{h} = \lim_{h \to 0} \frac{x - (x+h)}{h \cdot x(x+h)} = \lim_{h \to 0} \frac{-h}{h \cdot x(x+h)}$$

$$= \lim_{h \to 0} \frac{-1}{x(x+h)} = \frac{-1}{x^2}$$

**Result:** $\frac{d}{dx}\left(\frac{1}{x}\right) = -\frac{1}{x^2}$

### Example 4: $f(x) = \sqrt{x}$

$$f'(x) = \lim_{h \to 0} \frac{\sqrt{x+h} - \sqrt{x}}{h}$$

Rationalize:

$$= \lim_{h \to 0} \frac{(\sqrt{x+h} - \sqrt{x})(\sqrt{x+h} + \sqrt{x})}{h(\sqrt{x+h} + \sqrt{x})} = \lim_{h \to 0} \frac{(x+h) - x}{h(\sqrt{x+h} + \sqrt{x})}$$

$$= \lim_{h \to 0} \frac{1}{\sqrt{x+h} + \sqrt{x}} = \frac{1}{2\sqrt{x}}$$

**Result:** $\frac{d}{dx}(\sqrt{x}) = \frac{1}{2\sqrt{x}}$

### Example 5: $f(x) = \sin x$

$$f'(x) = \lim_{h \to 0} \frac{\sin(x+h) - \sin x}{h}$$

Using $\sin(A+B) = \sin A \cos B + \cos A \sin B$:

$$= \lim_{h \to 0} \frac{\sin x \cos h + \cos x \sin h - \sin x}{h}$$

$$= \lim_{h \to 0} \left[\sin x \cdot \frac{\cos h - 1}{h} + \cos x \cdot \frac{\sin h}{h}\right]$$

$$= \sin x \cdot 0 + \cos x \cdot 1 = \cos x$$

**Result:** $\frac{d}{dx}(\sin x) = \cos x$

### Example 6: $f(x) = e^x$

$$f'(x) = \lim_{h \to 0} \frac{e^{x+h} - e^x}{h} = \lim_{h \to 0} \frac{e^x(e^h - 1)}{h} = e^x \cdot \lim_{h \to 0} \frac{e^h - 1}{h} = e^x \cdot 1 = e^x$$

**Result:** $\frac{d}{dx}(e^x) = e^x$ — the exponential function is its own derivative!

---

## 4.1.4 Summary of Basic Derivatives (from First Principles)

| $f(x)$ | $f'(x)$ |
|---------|---------|
| $c$ (constant) | $0$ |
| $x$ | $1$ |
| $x^2$ | $2x$ |
| $x^3$ | $3x^2$ |
| $x^n$ | $nx^{n-1}$ (general power rule) |
| $\frac{1}{x}$ | $-\frac{1}{x^2}$ |
| $\sqrt{x}$ | $\frac{1}{2\sqrt{x}}$ |
| $\sin x$ | $\cos x$ |
| $\cos x$ | $-\sin x$ |
| $e^x$ | $e^x$ |
| $\ln x$ | $\frac{1}{x}$ |

---

## 4.1.5 When the Derivative Does NOT Exist

The limit $\lim_{h \to 0} \frac{f(a+h) - f(a)}{h}$ may fail to exist if:

1. **Corner/Kink:** Left and right derivatives differ. Ex: $|x|$ at $x = 0$.
2. **Cusp:** Slopes go to $\pm\infty$. Ex: $x^{2/3}$ at $x = 0$.
3. **Vertical tangent:** Slope is infinite. Ex: $\sqrt[3]{x}$ at $x = 0$.
4. **Discontinuity:** Function is not continuous.

```
  Corner (|x| at 0)       Cusp (x^(2/3) at 0)     Vertical tangent
                                                     (x^(1/3) at 0)
  y                        y                         y
  │╲      ╱                │╲    ╱                   │       ╱
  │  ╲  ╱                  │  ╲╱                     │     ╱
  │    V                   │   │ ← cusp              │   ╱  ← tangent
  └────┼─── x              └───┼─── x               │──│────── x
       0                       0                     │  ╲
                                                     │    ╲
```

---

## 4.1.6 Real-World Application

| Context | $f(t)$ | $f'(t)$ |
|---------|--------|---------|
| Position | $s(t)$ = distance | $v(t)$ = velocity |
| Velocity | $v(t)$ = speed | $a(t)$ = acceleration |
| Population | $P(t)$ = population | Growth rate |
| Cost | $C(q)$ = total cost | Marginal cost |
| Revenue | $R(q)$ = total revenue | Marginal revenue |

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Derivative definition | $f'(a) = \lim_{h \to 0} \frac{f(a+h) - f(a)}{h}$ |
| Difference quotient | $\frac{f(a+h) - f(a)}{h}$ = slope of secant |
| As $h \to 0$ | Secant → Tangent, avg rate → instantaneous rate |
| Notation | $f'$, $\frac{dy}{dx}$, $\dot{y}$, $Df$ |
| DNE cases | Corners, cusps, vertical tangents, discontinuities |

---

## Quick Revision Questions

**Q1.** Write the definition of $f'(a)$ using the limit definition.

**Q2.** Find $f'(x)$ from first principles for $f(x) = 3x^2 + 2x$.

**Q3.** Derive $\frac{d}{dx}(\cos x) = -\sin x$ from first principles.

**Q4.** At what point is $f(x) = |x|$ not differentiable? Show this using the limit definition.

**Q5.** Find $f'(4)$ from first principles for $f(x) = \sqrt{x}$.

**Q6.** Why is $e^x$ considered a "special" function in calculus?

---

[← Previous: Intermediate Value Theorem](../03-Continuity/04-intermediate-value-theorem.md) · [Next: Geometric Interpretation →](02-geometric-interpretation.md)
