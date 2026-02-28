# Unit 3 — Continuity: Definition of Continuity

[← Previous: L'Hôpital's Rule](../02-Limits/06-lhopitals-rule.md) · [Next: Types of Discontinuity →](02-types-of-discontinuity.md)

---

## Chapter Overview

Intuitively, a **continuous function** is one whose graph can be drawn without lifting your pen. Formally, continuity is defined using **limits** — tying together the concepts from Unit 2. Continuity is essential because only continuous (and differentiable) functions behave "nicely" enough for calculus to work.

---

## 3.1.1 Definition of Continuity at a Point

A function $f(x)$ is **continuous at $x = a$** if all three of the following hold:

$$\boxed{
\begin{aligned}
&\text{(i)}\ \ f(a) \text{ is defined} \\
&\text{(ii)}\ \ \lim_{x \to a} f(x) \text{ exists} \\
&\text{(iii)} \ \lim_{x \to a} f(x) = f(a)
\end{aligned}
}$$

If any of these **fails**, $f$ is **discontinuous** at $a$.

### Three-Condition Check (Visual)

```
  Condition (i)          Condition (ii)         Condition (iii)
  f(a) is defined?      Limit exists?          Limit = f(a)?

  y                      y                      y
  │      ●               │    ╲╱ ← limit       │    ╲  ●  ← f(a)
  │      │               │     │    exists      │     ╲│
  │      │               │     ○ ← f(a) or     │      ○ ← limit
  └──────┼── x           │       undefined      └──────┼── x
         a               └──────┼── x                  a
                                a
  ✓ Yes                  ✓ Yes                  ✗ No → discontinuous
```

---

## 3.1.2 Equivalent Formulation (ε-δ)

$f$ is continuous at $a$ if: for every $\varepsilon > 0$, there exists $\delta > 0$ such that:

$$|x - a| < \delta \implies |f(x) - f(a)| < \varepsilon$$

> **Note:** Unlike the limit definition, here we include $x = a$ (no "excluding $a$" condition). This is because condition (iii) forces $f(a)$ to equal the limit.

---

## 3.1.3 Continuity on an Interval

| Type | Definition |
|------|-----------|
| **Continuous on open interval** $(a, b)$ | $f$ is continuous at every point in $(a, b)$ |
| **Continuous on closed interval** $[a, b]$ | Continuous on $(a, b)$, right-continuous at $a$, left-continuous at $b$ |
| **Right-continuous at $a$** | $\lim_{x \to a^+} f(x) = f(a)$ |
| **Left-continuous at $b$** | $\lim_{x \to b^-} f(x) = f(b)$ |

**Why one-sided at endpoints?** At $x = a$ (left endpoint of $[a, b]$), we can only approach from the right since $f$ may not be defined for $x < a$.

---

## 3.1.4 Continuity of Standard Functions

The following functions are continuous **on their entire domain**:

| Function Type | Examples | Domain |
|---------------|---------|--------|
| Polynomial | $x^n, ax^2 + bx + c$ | $\mathbb{R}$ |
| Rational | $\frac{p(x)}{q(x)}$ | $\{x : q(x) \ne 0\}$ |
| Root | $\sqrt[n]{x}$ | $[0, \infty)$ for even $n$; $\mathbb{R}$ for odd $n$ |
| Trigonometric | $\sin x, \cos x$ | $\mathbb{R}$ |
| | $\tan x$ | $\mathbb{R} \setminus \{\frac{\pi}{2} + n\pi\}$ |
| Exponential | $e^x, a^x$ | $\mathbb{R}$ |
| Logarithmic | $\ln x, \log_a x$ | $(0, \infty)$ |
| Absolute value | $\|x\|$ | $\mathbb{R}$ |
| Inverse trig | $\sin^{-1} x$ | $[-1, 1]$ |

---

## 3.1.5 Worked Examples

### Example 1: Verify continuity

Is $f(x) = x^2 - 3x + 2$ continuous at $x = 1$?

1. $f(1) = 1 - 3 + 2 = 0$ ✓ (defined)
2. $\lim_{x \to 1} (x^2 - 3x + 2) = 0$ ✓ (exists, by direct substitution)
3. $\lim_{x \to 1} f(x) = 0 = f(1)$ ✓

**Yes, continuous at $x = 1$.**

### Example 2: Piecewise function

$$f(x) = \begin{cases} x + 1 & x < 2 \\ 5 & x = 2 \\ 3x - 1 & x > 2 \end{cases}$$

Check at $x = 2$:

1. $f(2) = 5$ ✓
2. LHL $= \lim_{x \to 2^-} (x + 1) = 3$; RHL $= \lim_{x \to 2^+} (3x - 1) = 5$. Since $3 \ne 5$, limit DNE. ✗

**Discontinuous at $x = 2$.**

### Example 3: Making a function continuous

Find $k$ so that $f(x) = \begin{cases} \frac{x^2 - 9}{x - 3} & x \ne 3 \\ k & x = 3 \end{cases}$ is continuous at $x = 3$.

Need: $\lim_{x \to 3} f(x) = f(3) = k$.

$$\lim_{x \to 3} \frac{x^2 - 9}{x - 3} = \lim_{x \to 3} \frac{(x-3)(x+3)}{x-3} = \lim_{x \to 3} (x + 3) = 6$$

Therefore $k = 6$.

### Example 4: Continuity of a trig piecewise

$$f(x) = \begin{cases} \frac{\sin x}{x} & x \ne 0 \\ 1 & x = 0 \end{cases}$$

1. $f(0) = 1$ ✓
2. $\lim_{x \to 0} \frac{\sin x}{x} = 1$ ✓
3. $\lim_{x \to 0} f(x) = 1 = f(0)$ ✓

**Continuous at $x = 0$.** (This is the "corrected" version of $\frac{\sin x}{x}$.)

---

## 3.1.6 Algebra of Continuous Functions

If $f$ and $g$ are continuous at $x = a$, then:

| Operation | Result |
|-----------|--------|
| $f + g$ | Continuous at $a$ |
| $f - g$ | Continuous at $a$ |
| $c \cdot f$ | Continuous at $a$ |
| $f \cdot g$ | Continuous at $a$ |
| $f / g$ | Continuous at $a$, provided $g(a) \ne 0$ |
| $f \circ g$ | Continuous at $a$ (if $g$ continuous at $a$, $f$ continuous at $g(a)$) |

> **Consequence:** Any function built from polynomials, trig, exponentials, and logarithms using $+, -, \times, \div, \circ$ is continuous at every point of its domain.

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Continuity at a point | Three conditions: $f(a)$ defined, $\lim$ exists, $\lim = f(a)$ |
| ε-δ formulation | $\|x - a\| < \delta \Rightarrow \|f(x) - f(a)\| < \varepsilon$ |
| Right-continuous | $\lim_{x \to a^+} f(x) = f(a)$ |
| Left-continuous | $\lim_{x \to a^-} f(x) = f(a)$ |
| Continuous on $[a,b]$ | Continuous on $(a,b)$ + right-cont at $a$ + left-cont at $b$ |
| Standard functions | Continuous on their natural domains |
| Algebra of continuity | Sum, product, quotient (denom ≠ 0), composition preserve continuity |

---

## Quick Revision Questions

**Q1.** State the three conditions for continuity of $f$ at $x = a$.

**Q2.** Is $f(x) = \frac{1}{x}$ continuous at $x = 0$? Explain which condition fails.

**Q3.** Find $k$ so that $f(x) = \begin{cases} kx + 2 & x \le 1 \\ 3x & x > 1 \end{cases}$ is continuous at $x = 1$.

**Q4.** Is $|x|$ continuous at $x = 0$? Verify all three conditions.

**Q5.** Give an example of a function where $\lim_{x \to a} f(x)$ exists but $f$ is not continuous at $a$.

**Q6.** Explain why $f(x) = e^{\sin x}$ is continuous on all of $\mathbb{R}$.

---

[← Previous: L'Hôpital's Rule](../02-Limits/06-lhopitals-rule.md) · [Next: Types of Discontinuity →](02-types-of-discontinuity.md)
