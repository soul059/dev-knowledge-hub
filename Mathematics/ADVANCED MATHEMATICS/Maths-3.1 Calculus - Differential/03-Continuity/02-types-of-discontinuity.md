# Unit 3 — Continuity: Types of Discontinuity

[← Previous: Definition of Continuity](01-definition-of-continuity.md) · [Next: Properties of Continuous Functions →](03-properties-of-continuous-functions.md)

---

## Chapter Overview

When a function fails to be continuous at a point, the nature of the failure falls into distinct categories. Classifying the type of discontinuity helps us understand the function's behavior and determine whether the discontinuity can be "fixed."

---

## 3.2.1 Classification of Discontinuities

```
                     Discontinuity at x = a
                            │
              ┌─────────────┴─────────────┐
         Limit exists                Limit does NOT exist
         (both sides equal)               │
              │                     ┌─────┴─────┐
         REMOVABLE             JUMP            ESSENTIAL
                           (finite jump)    (infinite or
                                            oscillatory)
```

---

## 3.2.2 Type 1: Removable Discontinuity

### Definition

$f$ has a **removable discontinuity** at $x = a$ if:

$$\lim_{x \to a} f(x) = L \quad \text{exists and is finite}$$

but either $f(a)$ is undefined or $f(a) \ne L$.

### Graphical Representation

```
  y                                 y
  │                                 │
  │      ╱                          │      ╱
  │     ╱                           │     ╱
  L├───○─── ← hole (f(a)           L├───○─── ← hole
  │  ╱       undefined)             │  ╱  ● ← f(a) ≠ L
  │ ╱                               │ ╱
  └──────┼── x                      └──────┼── x
         a                                 a

  Case 1: f(a) undefined            Case 2: f(a) ≠ limit
```

### Why "Removable"?

We can **fix** the function by (re)defining $f(a) = L$. The resulting function is continuous at $a$.

### Example

$$f(x) = \frac{x^2 - 1}{x - 1}, \quad x \ne 1$$

$\lim_{x \to 1} \frac{(x-1)(x+1)}{x-1} = 2$, but $f(1)$ is undefined.

**Fix:** Define $f(1) = 2$. Now $f$ is continuous at $x = 1$.

---

## 3.2.3 Type 2: Jump Discontinuity

### Definition

$f$ has a **jump discontinuity** at $x = a$ if:

$$\lim_{x \to a^-} f(x) \ne \lim_{x \to a^+} f(x)$$

Both one-sided limits exist and are finite, but they are **not equal**.

### Graphical Representation

```
  y
  │
  │         ●────────    ← right branch (RHL = L⁺)
  L⁺├─ ─ ─ │
  │         │
  │    jump │  = L⁺ - L⁻
  │         │
  L⁻├─ ─ ─ │
  │  ────────○           ← left branch (LHL = L⁻)
  │
  └─────────┼──────── x
            a
```

The **jump** = $|L^+ - L^-|$

### Example 1: Heaviside Function

$$H(x) = \begin{cases} 0 & x < 0 \\ 1 & x \ge 0 \end{cases}$$

- LHL at 0: $0$; RHL at 0: $1$
- Jump = 1
- Cannot be removed.

### Example 2: Signum Function

$$\text{sgn}(x) = \begin{cases} -1 & x < 0 \\ 0 & x = 0 \\ 1 & x > 0 \end{cases}$$

Jump discontinuity at $x = 0$ with jump = 2.

### Example 3: Greatest Integer Function

$f(x) = \lfloor x \rfloor$ has jump discontinuities at every integer $n$:

- LHL at $n$: $n - 1$
- RHL at $n$: $n$
- Jump = 1

```
  y
  │
  3├              ●─────────
  │        ○      │
  2├       ●──────○
  │  ○     │
  1├ ●─────○     Jump = 1
  │  │               at each integer
  0├─○
  └──┼──┼──┼──┼──── x
     0  1  2  3
```

---

## 3.2.4 Type 3: Essential (Infinite / Oscillatory) Discontinuity

### Definition

$f$ has an **essential discontinuity** at $x = a$ if the limit does not exist and is not a finite jump. This includes:

**(a) Infinite discontinuity:** One or both one-sided limits are $\pm\infty$.

**(b) Oscillatory discontinuity:** $f$ oscillates without settling on any value.

### Infinite Discontinuity

**Example:** $f(x) = \frac{1}{x}$ at $x = 0$.

- $\lim_{x \to 0^+} \frac{1}{x} = +\infty$
- $\lim_{x \to 0^-} \frac{1}{x} = -\infty$

```
  y
  │  │
  │  │  ╲
  │  │    ────
  │  │           → y = 0
  ───┼──────────── x
     │
  ───│
  ╱  │
  │  │
```

### Oscillatory Discontinuity

**Example:** $f(x) = \sin\left(\frac{1}{x}\right)$ at $x = 0$.

As $x \to 0$, the function oscillates between $-1$ and $+1$ infinitely often. No limit exists — not even a one-sided one.

```
  y
  1 ├  ╷╷╷╷┃┃┃┃║║║║ ────────
    │  ╵╵╵╵┃┃┃┃║║║║
  0 ├──────┃───║────────────── x
    │  ╷╷╷╷┃┃┃┃║║║║
 -1 ├  ╵╵╵╵┃┃┃┃║║║║ ────────
    │     ← increasingly rapid oscillation
    └──────┼──────── x
           0
```

---

## 3.2.5 Comparison Table

| Type | $\lim_{x \to a^-}$ | $\lim_{x \to a^+}$ | $\lim_{x \to a}$ | Fixable? |
|------|---------------------|---------------------|-------------------|----------|
| **Removable** | $= L$ | $= L$ | $= L$ (but ≠ $f(a)$ or $f(a)$ undef.) | **Yes** — set $f(a) = L$ |
| **Jump** | $= L^-$ | $= L^+$ | DNE ($L^- \ne L^+$) | **No** |
| **Infinite** | $\pm\infty$ | $\pm\infty$ | DNE | **No** |
| **Oscillatory** | DNE | DNE | DNE | **No** |

---

## 3.2.6 Worked Example: Full Classification

Classify the discontinuities of:

$$f(x) = \begin{cases} x + 1 & x < 0 \\ 2 & x = 0 \\ x^2 & 0 < x < 2 \\ 5 & x = 2 \\ 3x - 1 & x > 2 \end{cases}$$

**At $x = 0$:**
- LHL: $\lim_{x \to 0^-} (x + 1) = 1$
- RHL: $\lim_{x \to 0^+} x^2 = 0$
- $1 \ne 0$ → **Jump discontinuity**

**At $x = 2$:**
- LHL: $\lim_{x \to 2^-} x^2 = 4$
- RHL: $\lim_{x \to 2^+} (3x - 1) = 5$
- $4 \ne 5$ → **Jump discontinuity**

Alternatively, if we changed the problem so that at $x = 2$, LHL = RHL = 4 but $f(2) = 5$, we'd have a **removable discontinuity**.

---

## 3.2.7 Real-World Occurrences

| Discontinuity Type | Real-World Example |
|---------------------|-------------------|
| Removable | Pricing error filled in manually |
| Jump | Tax brackets (rate jumps at income thresholds) |
| Infinite | Electric field at a point charge |
| Oscillatory | High-frequency signal near a boundary |

---

## Summary Table

| Type | Key Feature | Can Be Fixed? |
|------|------------|---------------|
| Removable | Limit exists but ≠ f(a) or f(a) undefined | Yes |
| Jump | Left limit ≠ Right limit (both finite) | No |
| Infinite | At least one side → ±∞ | No |
| Oscillatory | No limit exists (function oscillates) | No |

---

## Quick Revision Questions

**Q1.** What are the three types of discontinuity? Give one example of each.

**Q2.** At $x = 2$, $f(x) = \frac{x^2 - 4}{x - 2}$ has what type of discontinuity? Can it be removed?

**Q3.** Why is a jump discontinuity called "non-removable"?

**Q4.** Classify the discontinuity of $f(x) = \frac{1}{(x - 1)^2}$ at $x = 1$.

**Q5.** For $f(x) = \lfloor x \rfloor$, classify the discontinuity at $x = 3$ and state the jump size.

**Q6.** Can a function have both removable and jump discontinuities? Give an example.

---

[← Previous: Definition of Continuity](01-definition-of-continuity.md) · [Next: Properties of Continuous Functions →](03-properties-of-continuous-functions.md)
