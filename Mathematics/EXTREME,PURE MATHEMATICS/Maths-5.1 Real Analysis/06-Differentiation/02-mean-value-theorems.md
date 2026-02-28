# Chapter 2: Mean Value Theorems

[← Previous: Definition and Basic Properties](01-definition-and-basic-properties.md) | [Back to README](../README.md) | [Next: L'Hôpital's Rule →](03-lhopitals-rule.md)

---

## Overview

The **Mean Value Theorem (MVT)** connects the derivative (local information) to the function's overall behavior (global information). Along with **Rolle's Theorem** and the **Generalized MVT (Cauchy)**, it is the backbone of differential calculus proofs.

---

## 2.1 Local Extrema

**Definition 2.1.** $f$ has a **local maximum** at $c$ if $\exists\, \delta > 0$ with $f(x) \leq f(c)$ for $|x - c| < \delta$.

**Theorem 2.2 (Interior Extremum Theorem / Fermat).** If $f$ has a local extremum at $c$ and $f'(c)$ exists, then $f'(c) = 0$.

**Proof.** If $f$ has a local max at $c$: for $h > 0$ small, $\frac{f(c+h) - f(c)}{h} \leq 0$, so $f'(c) \leq 0$. For $h < 0$ small, $\frac{f(c+h) - f(c)}{h} \geq 0$, so $f'(c) \geq 0$. Thus $f'(c) = 0$. $\blacksquare$

```
  Fermat's Theorem:

  f(x) ▲
       │    ● ← local max, f'(c) = 0
       │   ╱╲    tangent line is horizontal
       │  ╱  ╲
       │ ╱    ╲
       │╱      ╲
       └──┼─────────► x
          c

  f'(c) exists + local extremum ⟹ f'(c) = 0
  (Converse fails: f(x) = x³ has f'(0) = 0 but no extremum)
```

---

## 2.2 Rolle's Theorem

**Theorem 2.3 (Rolle, 1691).** If $f: [a,b] \to \mathbb{R}$ is continuous on $[a,b]$, differentiable on $(a,b)$, and $f(a) = f(b)$, then $\exists\, c \in (a,b)$ with $f'(c) = 0$.

**Proof.** By EVT, $f$ attains max $M$ and min $m$ on $[a,b]$.

- If $M = m$: $f$ is constant, so $f'(c) = 0$ for all $c \in (a,b)$.
- If $M > m$: at least one is different from $f(a) = f(b)$, so it occurs at an interior point $c$. By Fermat's Theorem, $f'(c) = 0$. $\blacksquare$

```
  Rolle's Theorem:

  f(x) ▲
       │      ●  ← f'(c) = 0
       │    ╱  ╲
       │  ╱     ╲
  f(a) ●╱        ╲● f(b) = f(a)
       │
       └──┼──┼────┼──► x
          a  c    b

  Equal endpoints ⟹ horizontal tangent somewhere inside
```

---

## 2.3 Mean Value Theorem

**Theorem 2.4 (Lagrange's MVT).** If $f: [a,b] \to \mathbb{R}$ is continuous on $[a,b]$ and differentiable on $(a,b)$, then $\exists\, c \in (a,b)$ with:

$$f'(c) = \frac{f(b) - f(a)}{b - a}$$

**Proof.** Apply Rolle's Theorem to:

$$g(x) = f(x) - \left[f(a) + \frac{f(b) - f(a)}{b - a}(x - a)\right]$$

$g$ is continuous on $[a,b]$, differentiable on $(a,b)$, and $g(a) = g(b) = 0$. By Rolle, $\exists\, c$ with $g'(c) = 0$, i.e., $f'(c) = \frac{f(b) - f(a)}{b-a}$. $\blacksquare$

```
  MVT: The secant line is parallel to some tangent

  f(x) ▲
       │           ● f(b)
       │         ╱╱
       │ tangent╱╱ secant (dashed)
       │      ●╱╱
       │     ╱╱  ← parallel!
       │   ╱╱
  f(a) ●─╱╱
       └──┼──┼──────┼──► x
          a  c      b

  "Instantaneous rate = average rate somewhere"
```

---

## 2.4 Consequences of MVT

**Corollary 2.5.** If $f'(x) = 0$ for all $x \in (a,b)$, then $f$ is constant on $(a,b)$.

*Proof:* For $x, y \in (a,b)$, MVT gives $f(x) - f(y) = f'(c)(x-y) = 0$. $\blacksquare$

**Corollary 2.6.** If $f'(x) > 0$ on $(a,b)$, then $f$ is strictly increasing.

**Corollary 2.7.** If $|f'(x)| \leq M$ on $(a,b)$, then $|f(x) - f(y)| \leq M|x-y|$ (Lipschitz).

| $f'$ on $(a,b)$ | Conclusion about $f$ |
|:---------------:|---------------------|
| $f' = 0$ | $f$ constant |
| $f' > 0$ | $f$ strictly increasing |
| $f' < 0$ | $f$ strictly decreasing |
| $f' \geq 0$ | $f$ non-decreasing |
| $\|f'\| \leq M$ | $f$ is $M$-Lipschitz |

---

## 2.5 Cauchy's (Generalized) Mean Value Theorem

**Theorem 2.8 (Cauchy MVT).** If $f, g$ are continuous on $[a,b]$, differentiable on $(a,b)$, and $g'(x) \neq 0$ on $(a,b)$, then $\exists\, c \in (a,b)$:

$$\frac{f'(c)}{g'(c)} = \frac{f(b) - f(a)}{g(b) - g(a)}$$

**Proof.** Apply Rolle to $h(x) = [f(b) - f(a)]g(x) - [g(b) - g(a)]f(x)$. $\blacksquare$

Note: Cauchy MVT is the key ingredient for L'Hôpital's Rule.

---

## 2.6 Darboux's Theorem

**Theorem 2.9 (Darboux, 1875).** If $f$ is differentiable on $[a,b]$, then $f'$ has the **intermediate value property**: if $f'(a) < v < f'(b)$, then $\exists\, c \in (a,b)$ with $f'(c) = v$.

**Key insight:** Even though $f'$ may be discontinuous, it still satisfies the IVP!

**Proof.** Let $g(x) = f(x) - vx$. Then $g'(a) = f'(a) - v < 0$ and $g'(b) = f'(b) - v > 0$. By EVT, $g$ has a minimum on $[a,b]$. Since $g'(a) < 0$, the min is not at $a$; since $g'(b) > 0$, it's not at $b$. So the min is at an interior $c$, and $g'(c) = 0$, i.e., $f'(c) = v$. $\blacksquare$

---

## Summary Table

| Theorem | Hypothesis | Conclusion |
|---------|-----------|------------|
| Fermat | $f'(c)$ exists, local extremum at $c$ | $f'(c) = 0$ |
| Rolle | Cont. $[a,b]$, diff. $(a,b)$, $f(a)=f(b)$ | $\exists c: f'(c) = 0$ |
| MVT | Cont. $[a,b]$, diff. $(a,b)$ | $\exists c: f'(c) = \frac{f(b)-f(a)}{b-a}$ |
| Cauchy MVT | Same + $g'(x) \neq 0$ | $\frac{f'(c)}{g'(c)} = \frac{f(b)-f(a)}{g(b)-g(a)}$ |
| Darboux | $f$ differentiable on $[a,b]$ | $f'$ has IVP |

---

## Quick Revision Questions

1. **State and prove** Rolle's Theorem.

2. **Prove** the Mean Value Theorem using Rolle's Theorem.

3. **Show** using MVT: if $f'(x) = 0$ for all $x$, then $f$ is constant.

4. **State** the Cauchy MVT and explain why it generalizes the ordinary MVT.

5. **Prove** Darboux's Theorem. Why is it surprising?

6. **True or False:** If $f'(c) = 0$, then $f$ has a local extremum at $c$. Justify.

---

[← Previous: Definition and Basic Properties](01-definition-and-basic-properties.md) | [Back to README](../README.md) | [Next: L'Hôpital's Rule →](03-lhopitals-rule.md)
