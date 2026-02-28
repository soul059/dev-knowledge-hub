# Unit 4 — Differentiability: Left and Right Derivatives

[← Previous: Differentiability vs Continuity](03-differentiability-vs-continuity.md) · [Next: Unit 5 — Differentiation Rules →](../05-Differentiation-Rules/01-power-sum-product-rules.md)

---

## Chapter Overview

Just as limits can be one-sided, so can derivatives. The **left derivative** and **right derivative** measure the rate of change as we approach a point from one side. A function is differentiable at a point if and only if both one-sided derivatives exist and are **equal**.

---

## 4.4.1 Definitions

### Left-Hand Derivative (LHD)

$$f'(a^-) = \lim_{h \to 0^-} \frac{f(a+h) - f(a)}{h} = \lim_{x \to a^-} \frac{f(x) - f(a)}{x - a}$$

### Right-Hand Derivative (RHD)

$$f'(a^+) = \lim_{h \to 0^+} \frac{f(a+h) - f(a)}{h} = \lim_{x \to a^+} \frac{f(x) - f(a)}{x - a}$$

### Differentiability Criterion

$$\boxed{f'(a) \text{ exists} \iff f'(a^-) = f'(a^+) \text{ (both finite and equal)}}$$

---

## 4.4.2 Graphical Interpretation

The left derivative is the slope of the tangent from the left; the right derivative is the slope from the right.

```
            LHD = slope from left     RHD = slope from right
                 ╲                         ╱
                  ╲                       ╱
                   ╲                     ╱
                    ● ← point a         ●
                   ╱                     ╲
                  ╱                       ╲

  If LHD ≠ RHD → corner → not differentiable
  If LHD = RHD → smooth → differentiable
```

```
  y                              y
  │╲        ╱                    │        ╱
  │  ╲    ╱                      │      ╱
  │    ╲╱  LHD = -1              │    ╱    LHD = RHD = 1
  │     ↑  RHD = +1              │  ╱
  │   corner                     │╱   smooth
  └────┼──── x                   └──────── x
       0                              a
  NOT differentiable             DIFFERENTIABLE
```

---

## 4.4.3 Worked Examples

### Example 1: $f(x) = |x|$ at $x = 0$

$$f'(0^-) = \lim_{h \to 0^-} \frac{|h| - 0}{h} = \lim_{h \to 0^-} \frac{-h}{h} = -1$$

$$f'(0^+) = \lim_{h \to 0^+} \frac{|h| - 0}{h} = \lim_{h \to 0^+} \frac{h}{h} = 1$$

$f'(0^-) = -1 \ne 1 = f'(0^+)$ ⟹ Not differentiable at $x = 0$.

### Example 2: Piecewise function

$$f(x) = \begin{cases} x^2 & x \le 1 \\ 2x - 1 & x > 1 \end{cases}$$

**Continuity check at $x = 1$:**
- $f(1) = 1$
- LHL: $\lim_{x \to 1^-} x^2 = 1$; RHL: $\lim_{x \to 1^+} (2x-1) = 1$
- Continuous ✓

**Derivative check at $x = 1$:**

$$f'(1^-) = \lim_{h \to 0^-} \frac{(1+h)^2 - 1}{h} = \lim_{h \to 0^-} \frac{2h + h^2}{h} = \lim_{h \to 0^-} (2 + h) = 2$$

$$f'(1^+) = \lim_{h \to 0^+} \frac{[2(1+h) - 1] - 1}{h} = \lim_{h \to 0^+} \frac{2h}{h} = 2$$

$f'(1^-) = f'(1^+) = 2$ ⟹ **Differentiable at $x = 1$**, $f'(1) = 2$.

### Example 3: Piecewise — NOT differentiable

$$g(x) = \begin{cases} x^2 & x \le 1 \\ 3x - 2 & x > 1 \end{cases}$$

**Continuity at $x = 1$:** $g(1) = 1$; LHL = 1; RHL = 1 ✓

**Derivatives:**

$$g'(1^-) = 2(1) = 2, \qquad g'(1^+) = 3$$

$2 \ne 3$ ⟹ **Not differentiable** at $x = 1$ (even though continuous).

### Example 4: Finding constants for differentiability

Find $a, b$ such that $f(x) = \begin{cases} ax^2 + b & x \le 2 \\ 3x + 1 & x > 2 \end{cases}$ is differentiable at $x = 2$.

**Continuity:** Require $f(2^-) = f(2^+)$:
$$4a + b = 7 \quad \cdots (1)$$

**Differentiability:** Require $f'(2^-) = f'(2^+)$:
- $f'(x) = 2ax$ for $x < 2$ → LHD $= 4a$
- $f'(x) = 3$ for $x > 2$ → RHD $= 3$
$$4a = 3 \Rightarrow a = \frac{3}{4} \quad \cdots (2)$$

From (1): $b = 7 - 4 \cdot \frac{3}{4} = 7 - 3 = 4$.

**Answer:** $a = \frac{3}{4}$, $b = 4$.

---

## 4.4.4 Classification of Non-Differentiable Points

| Type | LHD | RHD | Behavior |
|------|-----|-----|----------|
| **Corner** | Finite $L$ | Finite $R$ ($L \ne R$) | Sharp turn |
| **Cusp** | $+\infty$ | $-\infty$ (or vice versa) | Pointed peak |
| **Vertical tangent** | $+\infty$ | $+\infty$ (or both $-\infty$) | Graph goes vertical |
| **Discontinuity** | May not exist | May not exist | Function not continuous |

```
  CORNER            CUSP              VERTICAL TANGENT
  y                 y                 y
  │╲  ╱             │╲  ╱             │      │
  │  ╲╱              │ ╲╱              │    ─┤
  │  ╱╲              │  │              │   ╱  │
  │ ╱  ╲             │  │              │  ╱   │
  └──┼──── x         └──┼── x          └──┼──── x
     a                   a                 a
  LHD ≠ RHD         LHD = +∞          LHD = RHD = ∞
  (both finite)      RHD = -∞
```

---

## 4.4.5 Differentiability on an Interval

- $f$ is differentiable on **open interval** $(a, b)$ if $f'(x)$ exists at every $x \in (a, b)$.
- $f$ is differentiable on **closed interval** $[a, b]$ if:
  - $f'(x)$ exists for all $x \in (a, b)$
  - $f'(a^+)$ exists (right derivative at left endpoint)
  - $f'(b^-)$ exists (left derivative at right endpoint)

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Left derivative $f'(a^-)$ | Limit of diff. quotient as $h \to 0^-$ |
| Right derivative $f'(a^+)$ | Limit of diff. quotient as $h \to 0^+$ |
| Differentiable ⟺ | LHD = RHD (both finite) |
| Corner | LHD ≠ RHD (both finite) |
| Cusp | One-sided derivatives are $\pm\infty$ oppositely |
| Vertical tangent | Both one-sided derivatives are $+\infty$ or both $-\infty$ |
| Piecewise differentiability | Check continuity first, then match derivatives |

---

## Quick Revision Questions

**Q1.** Define the left-hand and right-hand derivatives at $x = a$.

**Q2.** Show that $f(x) = |x - 3|$ is not differentiable at $x = 3$ by computing LHD and RHD.

**Q3.** Find constants $p, q$ so that $f(x) = \begin{cases} px + q & x \le 1 \\ x^3 & x > 1 \end{cases}$ is differentiable at $x = 1$.

**Q4.** What type of non-differentiable point does $f(x) = x^{2/3}$ have at $x = 0$?

**Q5.** True or False: If $f'(a^-) = f'(a^+) = +\infty$, then $f$ is differentiable at $a$.

**Q6.** If $f$ is differentiable on $(0, 5)$ and right-differentiable at 0 and left-differentiable at 5, is it differentiable on $[0, 5]$?

---

[← Previous: Differentiability vs Continuity](03-differentiability-vs-continuity.md) · [Next: Unit 5 — Differentiation Rules →](../05-Differentiation-Rules/01-power-sum-product-rules.md)
