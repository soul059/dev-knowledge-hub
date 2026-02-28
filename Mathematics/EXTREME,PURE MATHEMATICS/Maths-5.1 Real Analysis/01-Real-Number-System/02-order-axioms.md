# Chapter 2: Order Axioms

[← Previous: Field Axioms](01-field-axioms.md) | [Back to README](../README.md) | [Next: Completeness Axiom →](03-completeness-axiom.md)

---

## Overview

The field axioms give us algebra, but to do analysis we need a notion of **order** — the ability to say one number is "less than" another. The order axioms define a **total ordering** on ℝ that is compatible with the field operations.

---

## 2.1 The Order Axioms

**Definition.** An **ordered field** is a field $F$ with a subset $P \subset F$ (called the **positive cone**) satisfying:

| Axiom | Name | Statement |
|-------|------|-----------|
| O1 | Trichotomy | For every $a \in F$, exactly one holds: $a \in P$, $a = 0$, or $-a \in P$ |
| O2 | Closure under addition | If $a, b \in P$, then $a + b \in P$ |
| O3 | Closure under multiplication | If $a, b \in P$, then $a \cdot b \in P$ |

**Definition of ordering.** We define:
- $a > 0$ means $a \in P$ (i.e., $a$ is positive)
- $a < b$ means $b - a \in P$
- $a \leq b$ means $a < b$ or $a = b$

---

## 2.2 Visualizing the Trichotomy

```
   The Real Line Under Trichotomy
   ═══════════════════════════════

   ◄──────────────────┼──────────────────►
        -a ∈ P        0        a ∈ P
        (negative)             (positive)
                    (zero)

   Every real number falls into EXACTLY ONE category:

   ┌──────────────┐  ┌──────┐  ┌──────────────┐
   │  NEGATIVE    │  │ ZERO │  │  POSITIVE    │
   │  -a ∈ P      │  │ a=0  │  │   a ∈ P      │
   │  (a < 0)     │  │      │  │  (a > 0)     │
   └──────────────┘  └──────┘  └──────────────┘

   No element can belong to two categories simultaneously.
```

---

## 2.3 Consequences of the Order Axioms

### Theorem 2.1: Basic Order Properties
Let $F$ be an ordered field. Then:

**(a)** $a > 0 \iff -a < 0$

**(b)** $a > 0$ and $b < c \implies a \cdot b < a \cdot c$

**(c)** $a < 0$ and $b < c \implies a \cdot b > a \cdot c$ (inequality reverses!)

**(d)** $a \neq 0 \implies a^2 > 0$

**(e)** $1 > 0$

**Proof of (d).** If $a > 0$, then $a \in P$, so $a \cdot a = a^2 \in P$ by O3. If $a < 0$, then $-a \in P$, so $(-a)(-a) = a^2 \in P$ by O3. $\blacksquare$

**Proof of (e).** We have $1 = 1^2$, and since $1 \neq 0$ (by axiom M4), part (d) gives $1 > 0$. $\blacksquare$

### Theorem 2.2: Order Preserving Operations

```
  Operation on inequality a < b:
  ┌─────────────────────────────────────────────────┐
  │ Add c to both sides:    a + c < b + c           │
  │ Multiply by c > 0:      ac < bc   (preserved)   │
  │ Multiply by c < 0:      ac > bc   (reversed!)   │
  │ Take reciprocal (a,b>0): 1/b < 1/a (reversed!)  │
  └─────────────────────────────────────────────────┘
```

### Theorem 2.3: $\mathbb{Q}$ is an ordered field, $\mathbb{C}$ is not.

**Proof that $\mathbb{C}$ is not an ordered field.**
Suppose $\mathbb{C}$ were an ordered field. By Theorem 2.1(d), $i^2 > 0$, i.e., $-1 > 0$. But also $1 > 0$ (Theorem 2.1(e)). Adding: $0 = 1 + (-1) > 0$, contradiction with trichotomy. $\blacksquare$

---

## 2.4 Absolute Value

**Definition.** For $a \in \mathbb{R}$:
$$|a| = \begin{cases} a & \text{if } a \geq 0 \\ -a & \text{if } a < 0 \end{cases}$$

Equivalently: $|a| = \max(a, -a)$.

### Key Properties of Absolute Value

| Property | Statement | Condition |
|----------|-----------|-----------|
| Non-negativity | $|a| \geq 0$ | Always |
| Definiteness | $|a| = 0 \iff a = 0$ | Always |
| Multiplicativity | $|ab| = |a||b|$ | Always |
| **Triangle inequality** | $|a + b| \leq |a| + |b|$ | Always |
| Reverse triangle | $||a| - |b|| \leq |a - b|$ | Always |

### The Triangle Inequality — Proof

**Theorem 2.4.** $|a + b| \leq |a| + |b|$ for all $a, b \in \mathbb{R}$.

**Proof.** Note that $-|a| \leq a \leq |a|$ and $-|b| \leq b \leq |b|$. Adding:
$$-(|a| + |b|) \leq a + b \leq |a| + |b|$$
This means $|a + b| \leq |a| + |b|$. $\blacksquare$

### Geometric Interpretation

```
  |a - b| represents the DISTANCE between a and b on the real line:

  ◄────────────────────────────────────────────────►
         a                    b
         ├────────────────────┤
              |a - b| = d(a,b)

  Triangle inequality says: going from a to c directly
  is never longer than going via b.

                    b
                   ╱ ╲
          |a-b|  ╱     ╲  |b-c|
                ╱         ╲
              a ─────────── c
                  |a - c|

         |a - c| ≤ |a - b| + |b - c|
```

---

## 2.5 The ε-Characterization of Order

The following lemma is used constantly throughout analysis:

**Lemma 2.5.** Let $a \in \mathbb{R}$. If $a \geq 0$ and $a < \varepsilon$ for every $\varepsilon > 0$, then $a = 0$.

**Proof.** By contradiction. Suppose $a > 0$. Take $\varepsilon = a/2 > 0$. Then $a < a/2$, which gives $a/2 < 0$, contradicting $a > 0$. $\blacksquare$

**Why this matters:** This lemma allows us to prove two quantities are equal by showing their difference is smaller than any positive number.

```
  Strategy: To prove x = y, show |x - y| < ε for ALL ε > 0

  ┌──────────────────────────────────────────────┐
  │     ε₁ = 1       →   |x - y| < 1            │
  │     ε₂ = 0.1     →   |x - y| < 0.1          │
  │     ε₃ = 0.01    →   |x - y| < 0.01         │
  │     ε₄ = 0.001   →   |x - y| < 0.001        │
  │     ...           →   ...                     │
  │     For ALL ε > 0 →   |x - y| < ε            │
  │                                               │
  │     Conclusion: |x - y| = 0, so x = y        │
  └──────────────────────────────────────────────┘
```

---

## 2.6 Intervals

The order on ℝ lets us define intervals:

| Notation | Definition | Type |
|----------|-----------|------|
| $(a, b)$ | $\{x \in \mathbb{R} : a < x < b\}$ | Open |
| $[a, b]$ | $\{x \in \mathbb{R} : a \leq x \leq b\}$ | Closed |
| $[a, b)$ | $\{x \in \mathbb{R} : a \leq x < b\}$ | Half-open |
| $(a, b]$ | $\{x \in \mathbb{R} : a < x \leq b\}$ | Half-open |
| $(a, \infty)$ | $\{x \in \mathbb{R} : x > a\}$ | Unbounded open |
| $(-\infty, b]$ | $\{x \in \mathbb{R} : x \leq b\}$ | Unbounded closed |

```
  Open interval (a, b):       ○─────────────○
                               a             b
  Closed interval [a, b]:     ●─────────────●
                               a             b
  Half-open [a, b):           ●─────────────○
                               a             b
```

---

## 2.7 The Maximum and Minimum

**Definition.** For $a, b \in \mathbb{R}$:
$$\max(a, b) = \begin{cases} a & \text{if } a \geq b \\ b & \text{if } b > a \end{cases}$$

**Useful identity:** $\max(a, b) = \frac{1}{2}(a + b + |a - b|)$ and $\min(a, b) = \frac{1}{2}(a + b - |a - b|)$.

---

## 2.8 Real-World Application: Numerical Comparison

Ordered fields are fundamental in:
- **Sorting algorithms** — comparing elements requires a total order
- **Optimization** — finding maxima/minima requires ordering
- **Floating-point arithmetic** — IEEE 754 is designed to approximate an ordered field
- **Decision theory** — ranking outcomes requires order axioms

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Ordered field | A field with a positive cone $P$ satisfying O1–O3 |
| Trichotomy | Every element is positive, zero, or negative — exactly one |
| $a^2 > 0$ for $a \neq 0$ | Squares are always positive |
| $1 > 0$ | The multiplicative identity is positive |
| $\mathbb{C}$ not orderable | No ordering of $\mathbb{C}$ is compatible with field operations |
| Absolute value | $\|a\| = \max(a, -a)$ |
| Triangle inequality | $\|a + b\| \leq \|a\| + \|b\|$ |
| ε-characterization | $a \geq 0$ and $a < \varepsilon$ for all $\varepsilon > 0$ implies $a = 0$ |
| Intervals | Subsets of ℝ defined by inequalities |

---

## Quick Revision Questions

1. **State the three order axioms** using the positive cone formulation. Why is trichotomy essential?

2. **Prove that $1 > 0$** in any ordered field. Which other result do you need?

3. **Why can't $\mathbb{C}$ be made into an ordered field?** Give the key contradiction.

4. **Prove the reverse triangle inequality:** $||a| - |b|| \leq |a - b|$.

5. **Let $x, y \in \mathbb{R}$ with $|x - y| < \varepsilon$ for every $\varepsilon > 0$.** Prove that $x = y$.

6. **If $a < b$ and $c < 0$, what is the relationship between $ac$ and $bc$?** Prove it from the axioms.

---

[← Previous: Field Axioms](01-field-axioms.md) | [Back to README](../README.md) | [Next: Completeness Axiom →](03-completeness-axiom.md)
