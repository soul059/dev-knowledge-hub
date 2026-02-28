# Unit 2 — Limits: Left and Right Hand Limits

[← Previous: Concept of Limit](01-concept-of-limit.md) · [Next: Laws of Limits →](03-laws-of-limits.md)

---

## Chapter Overview

When we approach a point $a$ on the number line, we can come from **two directions** — from the left (values less than $a$) or from the right (values greater than $a$). The **one-sided limits** capture what happens in each direction independently.

---

## 2.2.1 Definition

### Left-Hand Limit (LHL)

$$\lim_{x \to a^-} f(x) = L^- \quad \text{means: as } x \to a \text{ from below } (x < a), \ f(x) \to L^-$$

Formal: For every $\varepsilon > 0$, $\exists\, \delta > 0$ such that $a - \delta < x < a \Rightarrow |f(x) - L^-| < \varepsilon$.

### Right-Hand Limit (RHL)

$$\lim_{x \to a^+} f(x) = L^+ \quad \text{means: as } x \to a \text{ from above } (x > a), \ f(x) \to L^+$$

Formal: For every $\varepsilon > 0$, $\exists\, \delta > 0$ such that $a < x < a + \delta \Rightarrow |f(x) - L^+| < \varepsilon$.

### The Existence Theorem

$$\boxed{\lim_{x \to a} f(x) = L \quad \iff \quad \lim_{x \to a^-} f(x) = \lim_{x \to a^+} f(x) = L}$$

---

## 2.2.2 Graphical Interpretation

```
         LHL                  RHL
  ──────────▶ a ◀──────────
   x approaches     x approaches
   from left         from right

  y
  │
  │                    ╱── approaching from right → L⁺
  L⁺├─ ─ ─ ─ ─ ─ ─ ─●╱
  │                 ╱
  │                ╱
  │               ╱
  L⁻├─ ─ ─ ─ ─ ○╱── approaching from left → L⁻
  │            ╱
  │           ╱
  └──────────┼──────── x
             a

  Since L⁻ ≠ L⁺ → lim does NOT exist
```

---

## 2.2.3 Worked Examples

### Example 1: Step Function (Heaviside Function)

$$f(x) = \begin{cases} 0 & \text{if } x < 0 \\ 1 & \text{if } x \ge 0 \end{cases}$$

```
  y
  │
  1├          ●───────────
  │           │
  │           │
  0├──────────○
  │
  └───────────┼──────── x
              0
```

- $\lim_{x \to 0^-} f(x) = 0$
- $\lim_{x \to 0^+} f(x) = 1$
- Since $0 \ne 1$, $\lim_{x \to 0} f(x)$ **does not exist**.

### Example 2: Piecewise Linear

$$f(x) = \begin{cases} 2x + 1 & \text{if } x < 1 \\ x^2 + 2 & \text{if } x \ge 1 \end{cases}$$

- LHL: $\lim_{x \to 1^-} (2x + 1) = 2(1) + 1 = 3$
- RHL: $\lim_{x \to 1^+} (x^2 + 2) = 1 + 2 = 3$
- Since LHL = RHL = 3, $\lim_{x \to 1} f(x) = 3$ ✓

### Example 3: Absolute Value Ratio

$$f(x) = \frac{|x|}{x}$$

Rewrite:

$$f(x) = \begin{cases} \frac{-x}{x} = -1 & \text{if } x < 0 \\ \frac{x}{x} = 1 & \text{if } x > 0 \end{cases}$$

- $\lim_{x \to 0^-} f(x) = -1$
- $\lim_{x \to 0^+} f(x) = 1$
- $\lim_{x \to 0} f(x)$ **does not exist**.

```
  y
  │
  1├              ●────────────
  │               │
  0├───────────── ┼ ──────────── x
  │               │
 -1├──────────────●
  │
  └───────────────┼─── x
                  0
```

### Example 4: Greatest Integer Function $f(x) = \lfloor x \rfloor$

At $x = 2$:

- $\lim_{x \to 2^-} \lfloor x \rfloor = 1$ (just below 2, floor = 1)
- $\lim_{x \to 2^+} \lfloor x \rfloor = 2$ (just above 2, floor = 2)
- Limit does not exist at any integer.

```
  y
  │
  3├              ●─────────
  │        ○      │
  2├       ●──────○
  │  ○     │
  1├ ●─────○
  │  │
  0├─○
  └──┼──┼──┼──┼──── x
     0  1  2  3
  ● = included,  ○ = excluded
```

---

## 2.2.4 One-Sided Limits and Piecewise Functions — Strategy

For a piecewise function $f$ defined differently on the left and right of $a$:

```
Step 1: Compute LHL using the formula valid for x < a
Step 2: Compute RHL using the formula valid for x > a
Step 3: If LHL = RHL = L, then lim = L
         If LHL ≠ RHL, then lim DNE
```

### Decision Flowchart

```
           Is f piecewise at x = a?
                 │
         ┌──────┴──────┐
        YES             NO
         │               │
    Compute LHL      Use standard
    and RHL          evaluation
    separately       techniques
         │
    ┌────┴────┐
  LHL = RHL?  │
    │         NO → Limit DNE
   YES
    │
  Limit = LHL = RHL
```

---

## 2.2.5 One-Sided Infinite Limits

Sometimes one or both sided limits are infinite:

**Example:** $f(x) = \frac{1}{x}$

- $\lim_{x \to 0^+} \frac{1}{x} = +\infty$
- $\lim_{x \to 0^-} \frac{1}{x} = -\infty$

Since $+\infty \ne -\infty$, the two-sided limit does not exist (even as an infinite limit).

**Example:** $f(x) = \frac{1}{x^2}$

- $\lim_{x \to 0^+} \frac{1}{x^2} = +\infty$
- $\lim_{x \to 0^-} \frac{1}{x^2} = +\infty$

Both sides agree, so we write $\lim_{x \to 0} \frac{1}{x^2} = +\infty$.

---

## Summary Table

| Concept | Definition |
|---------|-----------|
| Left-hand limit $\lim_{x \to a^-}$ | Approach $a$ from values less than $a$ |
| Right-hand limit $\lim_{x \to a^+}$ | Approach $a$ from values greater than $a$ |
| Two-sided limit exists | LHL = RHL (and both exist) |
| Heaviside at 0 | LHL = 0, RHL = 1, limit DNE |
| $\|x\|/x$ at 0 | LHL = −1, RHL = +1, limit DNE |
| $\lfloor x \rfloor$ at integers | LHL = $n-1$, RHL = $n$, limit DNE |
| One-sided infinite limits | $1/x$ at 0: left → $-\infty$, right → $+\infty$ |

---

## Quick Revision Questions

**Q1.** Evaluate $\lim_{x \to 3^-} f(x)$ and $\lim_{x \to 3^+} f(x)$ where $f(x) = \begin{cases} x + 1 & x < 3 \\ 2x - 2 & x \ge 3 \end{cases}$. Does the limit exist?

**Q2.** For $f(x) = \frac{|x - 5|}{x - 5}$, find the one-sided limits at $x = 5$.

**Q3.** Why does $\lim_{x \to 0} \frac{1}{x}$ not exist, but $\lim_{x \to 0} \frac{1}{x^2}$ can be said to be $+\infty$?

**Q4.** At which points does $\lfloor x \rfloor$ have a two-sided limit? At which points does it not?

**Q5.** Give a function where $\lim_{x \to 2^-} f(x) = 5$ but $\lim_{x \to 2^+} f(x) = 7$.

**Q6.** True or False: If $f(a)$ exists, then $\lim_{x \to a} f(x)$ must exist. Justify.

---

[← Previous: Concept of Limit](01-concept-of-limit.md) · [Next: Laws of Limits →](03-laws-of-limits.md)
