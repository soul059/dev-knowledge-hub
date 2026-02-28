# Chapter 1: Definition and Basic Properties of the Derivative

[← Previous: Unit 5 — Uniform Continuity](../05-Limits-and-Continuity/07-uniform-continuity.md) | [Back to README](../README.md) | [Next: Mean Value Theorems →](02-mean-value-theorems.md)

---

## Overview

The **derivative** measures the instantaneous rate of change of a function. In real analysis, the rigorous $\varepsilon$-$\delta$ definition reveals when differentiation is valid, what rules it obeys, and where it can fail. Differentiability implies continuity, but not conversely.

---

## 1.1 Definition

**Definition 1.1.** $f: (a,b) \to \mathbb{R}$ is **differentiable at $c \in (a,b)$** if the limit:

$$f'(c) = \lim_{x \to c} \frac{f(x) - f(c)}{x - c} = \lim_{h \to 0} \frac{f(c + h) - f(c)}{h}$$

exists (as a finite real number). The value $f'(c)$ is the **derivative** at $c$.

$f$ is **differentiable on $(a,b)$** if it is differentiable at every $c \in (a,b)$.

```
  The derivative as slope of tangent line:

  f(x) ▲
       │        ╱ ← tangent line, slope = f'(c)
       │      ╱● f(x)
       │    ╱╱
       │  ╱╱ ← secant: slope = [f(x)-f(c)]/(x-c)
       │╱╱
       ●╱ f(c)
       └──┼──┼──────► x
          c  x

  f'(c) = lim(secant slopes) as x → c
```

---

## 1.2 Differentiability Implies Continuity

**Theorem 1.2.** If $f$ is differentiable at $c$, then $f$ is continuous at $c$.

**Proof.**
$$f(x) - f(c) = \frac{f(x) - f(c)}{x - c} \cdot (x - c) \to f'(c) \cdot 0 = 0 \quad \text{as } x \to c$$

So $\lim_{x \to c} f(x) = f(c)$. $\blacksquare$

**The converse is FALSE:** $f(x) = |x|$ is continuous at $0$ but not differentiable.

```
  Hierarchy:

  Differentiable ⟹ Continuous ⟹ (has limit)
      ⇍               ⇍

  Weierstrass: ∃ continuous, NOWHERE differentiable functions!
```

---

## 1.3 Differentiation Rules

**Theorem 1.3.** If $f, g$ are differentiable at $c$:

| Rule | Formula |
|------|---------|
| **Sum** | $(f + g)'(c) = f'(c) + g'(c)$ |
| **Scalar** | $(kf)'(c) = k f'(c)$ |
| **Product** | $(fg)'(c) = f'(c)g(c) + f(c)g'(c)$ |
| **Quotient** | $(f/g)'(c) = \frac{f'(c)g(c) - f(c)g'(c)}{[g(c)]^2}$ if $g(c) \neq 0$ |
| **Chain Rule** | $(g \circ f)'(c) = g'(f(c)) \cdot f'(c)$ |

---

## 1.4 Chain Rule Proof

**Theorem 1.4 (Chain Rule).** If $f$ is differentiable at $c$ and $g$ is differentiable at $f(c)$, then $g \circ f$ is differentiable at $c$ and:

$$(g \circ f)'(c) = g'(f(c)) \cdot f'(c)$$

**Proof (Carathéodory's formulation).** Define:

$$\phi(y) = \begin{cases} \frac{g(y) - g(d)}{y - d} & y \neq d \\ g'(d) & y = d \end{cases}$$

where $d = f(c)$. Then $g(y) - g(d) = \phi(y)(y - d)$ for all $y$, and $\phi$ is continuous at $d$.

Similarly, $f(x) - f(c) = \psi(x)(x - c)$ where $\psi$ is continuous at $c$ with $\psi(c) = f'(c)$.

Then:
$$g(f(x)) - g(f(c)) = \phi(f(x)) \cdot (f(x) - f(c)) = \phi(f(x)) \cdot \psi(x) \cdot (x - c)$$

Dividing by $x - c$ and taking limits:
$$(g \circ f)'(c) = \phi(f(c)) \cdot \psi(c) = g'(f(c)) \cdot f'(c). \quad\blacksquare$$

---

## 1.5 Non-Differentiability

A function can fail to be differentiable due to:

| Type | Example | Left/Right derivatives |
|------|---------|----------------------|
| **Corner** | $\|x\|$ at $0$ | $f'_-(0) = -1 \neq 1 = f'_+(0)$ |
| **Cusp** | $x^{2/3}$ at $0$ | Derivatives $\to \pm \infty$ |
| **Vertical tangent** | $\sqrt[3]{x}$ at $0$ | $f'(0) = +\infty$ |
| **Oscillation** | $x \sin(1/x)$ at $0$ | Limit doesn't exist |

```
  Corner:           Cusp:            Vertical tangent:
  
  ▲   ╱             ▲  │             ▲        │
  │  ╱               │ │              │       │
  │ ╱                │╲│╱             │      ╱
  │╱                 │ ●              │    ╱
  ●╲                 │                │  ╱
  │  ╲               │                │╱
  │   ╲              │                ●
```

---

## 1.6 Weierstrass Function

**Theorem 1.5 (Weierstrass, 1872).** There exists a continuous function $f: \mathbb{R} \to \mathbb{R}$ that is **nowhere differentiable**.

$$f(x) = \sum_{n=0}^{\infty} a^n \cos(b^n \pi x), \quad 0 < a < 1, \; b \text{ odd}, \; ab > 1 + \frac{3\pi}{2}$$

This was a revolutionary counterexample showing that continuous functions can be far worse than expected.

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| Derivative | $f'(c) = \lim \frac{f(x)-f(c)}{x-c}$ |
| Differentiable $\Rightarrow$ continuous | $f(x) - f(c) = \frac{f(x)-f(c)}{x-c}(x-c) \to 0$ |
| Converse fails | $\|x\|$, Weierstrass function |
| Sum, product, quotient | Standard rules hold |
| Chain rule | $(g \circ f)' = (g' \circ f) \cdot f'$; Carathéodory proof |
| Non-differentiability | Corners, cusps, vertical tangents, oscillation |
| Weierstrass | Continuous, nowhere differentiable |

---

## Quick Revision Questions

1. **State** the definition of the derivative. Explain why the limit must be finite.

2. **Prove** that differentiability implies continuity.

3. **Prove** the chain rule using Carathéodory's formulation.

4. **Show** $f(x) = |x|$ is not differentiable at $0$.

5. **Prove** the product rule from the definition.

6. **Why** was the Weierstrass function so surprising historically?

---

[← Previous: Unit 5 — Uniform Continuity](../05-Limits-and-Continuity/07-uniform-continuity.md) | [Back to README](../README.md) | [Next: Mean Value Theorems →](02-mean-value-theorems.md)
