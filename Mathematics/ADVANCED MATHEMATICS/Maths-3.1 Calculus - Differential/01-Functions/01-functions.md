# Unit 1 — Functions

[← Back to README](../README.md) · [Next: Unit 2 — Limits →](../02-Limits/01-concept-of-limit.md)

---

## Chapter Overview

A **function** is the most fundamental object in calculus. Before we can talk about limits, derivatives, or rates of change, we need a precise language for describing how one quantity depends on another. This unit builds that language from the ground up.

---

## 1.1 Definition and Types of Functions

### 1.1.1 Definition

A **function** $f$ from a set $A$ to a set $B$ is a rule that assigns to **each** element $x \in A$ **exactly one** element $y \in B$.

We write:

$$f : A \to B \quad \text{or} \quad y = f(x)$$

- $x$ is the **independent variable** (input / argument).
- $y$ is the **dependent variable** (output / value).
- $A$ is the **domain** of $f$.
- $B$ is the **codomain** of $f$.
- The set $\{f(x) : x \in A\} \subseteq B$ is the **range** (image) of $f$.

**Function as a "machine":**

```
         ┌──────────────┐
  x ───▶ │   f(x) = y   │ ───▶ y
 input   └──────────────┘    output
```

> **Key point:** Every input has *exactly one* output. A relation that gives two outputs for one input is **not** a function.

### 1.1.2 Vertical Line Test

A curve in the $xy$-plane is the graph of a function of $x$ if and only if **no vertical line** intersects the curve more than once.

```
  y                           y
  │    ╭─╮                    │     ╭╮
  │   ╱   ╲                   │    ╱  ╲
  │  ╱     ╲                  │   │    │
  │ ╱       ╲                 │   │    │   ← two y-values
  │╱         ╲                │    ╲  ╱      for same x
  └───────────── x            │     ╰╯
  PASSES the test             └──────────── x
  (function)                  FAILS the test
                              (not a function)
```

### 1.1.3 Types of Functions

| Type | Definition | Example |
|------|-----------|---------|
| **One-to-one (Injective)** | $f(a) = f(b) \Rightarrow a = b$ | $f(x) = 2x + 3$ |
| **Onto (Surjective)** | Range = Codomain | $f : \mathbb{R} \to \mathbb{R},\ f(x) = x^3$ |
| **Bijective** | Both injective and surjective | $f(x) = x^3$ on $\mathbb{R}$ |
| **Even** | $f(-x) = f(x)$ for all $x$ | $f(x) = x^2,\ \cos x$ |
| **Odd** | $f(-x) = -f(x)$ for all $x$ | $f(x) = x^3,\ \sin x$ |
| **Periodic** | $f(x + T) = f(x)$ for some $T > 0$ | $\sin x$ (period $2\pi$) |
| **Constant** | $f(x) = c$ for all $x$ | $f(x) = 5$ |
| **Identity** | $f(x) = x$ | — |
| **Polynomial** | $f(x) = a_nx^n + \cdots + a_0$ | $3x^2 - x + 7$ |
| **Rational** | $f(x) = \frac{p(x)}{q(x)}$, polynomials $p, q$ | $\frac{x+1}{x^2-4}$ |
| **Algebraic** | Involves only algebraic operations | $\sqrt{x^2 + 1}$ |
| **Transcendental** | Not algebraic | $e^x, \ln x, \sin x$ |
| **Piecewise** | Different rules for different intervals | $\|x\|$ |

### 1.1.4 Horizontal Line Test (for Injectivity)

A function is **one-to-one** if and only if **no horizontal line** intersects its graph more than once.

```
  y                           y
  │       ╱                   │   ╭──╮
  │      ╱                    │  ╱    ╲
  │─────╱───── y = c          │─╱──────╲── y = c  (hits twice!)
  │    ╱                      │╱        ╲
  │   ╱                       │          ╲
  └──────────── x             └──────────── x
  ONE-TO-ONE                  NOT ONE-TO-ONE
  (f(x) = 2x + 1)            (f(x) = x²)
```

---

## 1.2 Domain and Range

### 1.2.1 Finding the Domain

The **natural domain** is the largest set of real numbers for which $f(x)$ is defined.

**Rules to remember:**

| Restriction | Condition |
|-------------|-----------|
| Denominator ≠ 0 | If $f(x) = \frac{g(x)}{h(x)}$, exclude $x$ where $h(x) = 0$ |
| Even root ≥ 0 | If $f(x) = \sqrt[2n]{g(x)}$, require $g(x) \ge 0$ |
| Logarithm argument > 0 | If $f(x) = \ln(g(x))$, require $g(x) > 0$ |
| $\tan x$, $\sec x$ undefined | Exclude $x = \frac{\pi}{2} + n\pi$ |

**Example 1:** Find the domain of $f(x) = \frac{1}{\sqrt{x - 2}}$.

- Require $x - 2 > 0$ (not just $\ge 0$ because the square root is in the denominator).
- Domain: $x > 2$, i.e., $(2, \infty)$.

**Example 2:** Find the domain of $f(x) = \ln(4 - x^2)$.

- Require $4 - x^2 > 0 \Rightarrow x^2 < 4 \Rightarrow -2 < x < 2$.
- Domain: $(-2, 2)$.

### 1.2.2 Finding the Range

Strategy: Set $y = f(x)$ and solve for $x$ in terms of $y$. The values of $y$ for which a real $x$ exists form the range.

**Example:** Find the range of $f(x) = \frac{x}{x+1}$, $x \ne -1$.

$$y = \frac{x}{x+1} \Rightarrow y(x+1) = x \Rightarrow yx + y = x \Rightarrow x(y-1) = -y \Rightarrow x = \frac{-y}{y-1}$$

This gives a real $x$ for all $y \ne 1$. So Range $= \mathbb{R} \setminus \{1\}$.

### 1.2.3 Domain–Range Quick Reference

| Function | Domain | Range |
|----------|--------|-------|
| $x^n$ ($n$ even) | $\mathbb{R}$ | $[0, \infty)$ |
| $x^n$ ($n$ odd) | $\mathbb{R}$ | $\mathbb{R}$ |
| $\sqrt{x}$ | $[0, \infty)$ | $[0, \infty)$ |
| $\frac{1}{x}$ | $\mathbb{R} \setminus \{0\}$ | $\mathbb{R} \setminus \{0\}$ |
| $e^x$ | $\mathbb{R}$ | $(0, \infty)$ |
| $\ln x$ | $(0, \infty)$ | $\mathbb{R}$ |
| $\sin x$ | $\mathbb{R}$ | $[-1, 1]$ |
| $\cos x$ | $\mathbb{R}$ | $[-1, 1]$ |
| $\tan x$ | $\mathbb{R} \setminus \{\frac{\pi}{2}+n\pi\}$ | $\mathbb{R}$ |
| $|x|$ | $\mathbb{R}$ | $[0, \infty)$ |

---

## 1.3 Composition of Functions

### 1.3.1 Definition

Given two functions $f$ and $g$, the **composite function** $(f \circ g)$ is defined by:

$$(f \circ g)(x) = f\big(g(x)\big)$$

**Pipeline diagram:**

```
         ┌──────────┐      ┌──────────┐
  x ───▶ │  g(x)    │ ───▶ │  f(·)    │ ───▶ f(g(x))
         └──────────┘      └──────────┘
           inner              outer
```

### 1.3.2 Domain of a Composite Function

$$\text{Domain of } f \circ g = \{x \in \text{Dom}(g) : g(x) \in \text{Dom}(f)\}$$

### 1.3.3 Properties

| Property | Status |
|----------|--------|
| Commutativity: $f \circ g = g \circ f$ ? | **NO** (in general) |
| Associativity: $f \circ (g \circ h) = (f \circ g) \circ h$ ? | **YES** |
| Identity: $f \circ I = I \circ f = f$ ? | **YES** ($I(x) = x$) |

**Example:** Let $f(x) = x^2$ and $g(x) = x + 3$.

- $(f \circ g)(x) = f(g(x)) = f(x+3) = (x+3)^2$
- $(g \circ f)(x) = g(f(x)) = g(x^2) = x^2 + 3$

Clearly $f \circ g \ne g \circ f$.

### 1.3.4 Decomposition of Functions

Many complex functions can be broken into simpler compositions — a skill crucial for the **Chain Rule** (Unit 5).

| Composite Function | Outer $f$ | Inner $g$ |
|---------------------|-----------|-----------|
| $\sin(x^2)$ | $\sin u$ | $u = x^2$ |
| $e^{\cos x}$ | $e^u$ | $u = \cos x$ |
| $\sqrt{\ln x}$ | $\sqrt{u}$ | $u = \ln x$ |
| $(\tan x)^5$ | $u^5$ | $u = \tan x$ |

---

## 1.4 Inverse Functions

### 1.4.1 Definition

If $f : A \to B$ is **bijective**, its **inverse** $f^{-1} : B \to A$ is the function satisfying:

$$f^{-1}(f(x)) = x \quad \text{for all } x \in A$$
$$f(f^{-1}(y)) = y \quad \text{for all } y \in B$$

**Diagram:**

```
         f
  A ──────────▶ B
  │              │
  │  f⁻¹        │  f
  │◀─────────── │
  A ◀────────── B
         f⁻¹
```

### 1.4.2 Finding the Inverse

**Step-by-step method:**

1. Write $y = f(x)$.
2. Swap $x$ and $y$.
3. Solve for $y$.
4. The result is $y = f^{-1}(x)$.

**Example:** Find $f^{-1}$ for $f(x) = 3x - 7$.

1. $y = 3x - 7$
2. $x = 3y - 7$
3. $3y = x + 7 \Rightarrow y = \frac{x + 7}{3}$
4. $f^{-1}(x) = \frac{x + 7}{3}$

**Verification:** $f(f^{-1}(x)) = 3 \cdot \frac{x+7}{3} - 7 = x + 7 - 7 = x$ ✓

### 1.4.3 Graph of Inverse Functions

The graph of $f^{-1}$ is the **reflection** of the graph of $f$ across the line $y = x$.

```
  y
  │       ╱  y = x
  │      ╱ ╱ ← f(x) = 2x + 1
  │     ╱╱
  │    ╱╱
  │   ╱╱
  │  ╱╱ ← f⁻¹(x) = (x-1)/2
  │ ╱╱
  │╱╱
  └──────────────── x
```

### 1.4.4 Key Properties

| Property | Formula |
|----------|---------|
| Domain of $f^{-1}$ | = Range of $f$ |
| Range of $f^{-1}$ | = Domain of $f$ |
| $(f^{-1})^{-1}$ | $= f$ |
| Graph symmetry | Reflected about $y = x$ |

> **Warning:** $f^{-1}(x) \ne \frac{1}{f(x)}$. The notation $f^{-1}$ denotes the *inverse function*, not the reciprocal.

---

## 1.5 Graphs of Standard Functions

Understanding the shape, symmetry, and key features of standard graphs is essential for all later topics.

### 1.5.1 Constant Function $f(x) = c$

```
  y
  │
  c ├──────────────────  ← horizontal line
  │
  └──────────────────── x
```

### 1.5.2 Identity Function $f(x) = x$

```
  y
  │       ╱
  │      ╱
  │     ╱
  │    ╱
  │   ╱
  │  ╱
  │ ╱
  │╱
  └──────────────── x
       slope = 1
```

### 1.5.3 Quadratic Function $f(x) = x^2$

```
  y
  │
  │  ╲         ╱
  │   ╲       ╱
  │    ╲     ╱
  │     ╲   ╱
  │      ╲ ╱
  │       V  ← vertex at (0,0)
  └──────────────── x
  Even function, symmetric about y-axis
```

### 1.5.4 Cubic Function $f(x) = x^3$

```
  y
  │           ╱
  │          ╱
  │         ╱
  │───────·──────── x   ← passes through origin
  │      ╱
  │     ╱
  │    ╱
```

### 1.5.5 Absolute Value Function $f(x) = |x|$

```
  y
  │
  │  ╲       ╱
  │   ╲     ╱
  │    ╲   ╱
  │     ╲ ╱
  │      V  ← corner at origin (not differentiable here!)
  └──────────────── x
```

### 1.5.6 Square Root Function $f(x) = \sqrt{x}$

```
  y
  │         ────────
  │       ╱
  │     ╱
  │   ╱
  │  ╱
  │ ╱
  │╱
  └──────────────── x
  Domain: [0, ∞)
```

### 1.5.7 Reciprocal Function $f(x) = \frac{1}{x}$

```
  y
  │    │
  │    │  ╲
  │    │    ──────
  │    │           asymptote: y = 0
  ─────┼──────────── x
       │
  ─────│
  ╱    │
  │    │
  asymptote: x = 0
```

### 1.5.8 Exponential Function $f(x) = e^x$

```
  y
  │              ╱
  │            ╱
  │          ╱
  │        ╱
  1 ├─────·          ← passes through (0,1)
  │     ╱
  │───╱───────────   ← asymptote y = 0
  │  ╱
  └──────────────── x
  Always positive, always increasing
```

### 1.5.9 Natural Logarithm $f(x) = \ln x$

```
  y
  │           ────────
  │         ╱
  │       ╱
  │     ╱
  │   ╱
  0 ├─·──────────── y = 0  ← crosses at (1, 0)
  │ ╱
  │╱
  ││
  └──────────────── x
  Domain: (0, ∞)
  Asymptote: x = 0
```

### 1.5.10 Trigonometric Functions

**$\sin x$:**
```
  y
  1 ├──╮     ╭──╮     ╭──╮
    │   ╲   ╱    ╲   ╱    ╲
  0 ├────╲─╱──────╲─╱──────── x
    │     V        V
 -1 ├
    0    π       2π      3π
  Period = 2π, Odd function
```

**$\cos x$:**
```
  y
  1 ├╮       ╭──╮       ╭─
    │ ╲     ╱    ╲     ╱
  0 ├──╲───╱──────╲───╱──── x
    │   ╲ ╱        ╲ ╱
 -1 ├    V          V
    0   π/2   π   3π/2  2π
  Period = 2π, Even function
```

**$\tan x$:**
```
  y
  │  │      ╱│      ╱│
  │  │     ╱ │     ╱ │
  │  │    ╱  │    ╱  │
  0──┼───╱───┼───╱───┼── x
  │  │  ╱    │  ╱    │
  │  │ ╱     │ ╱     │
  │  │╱      │╱      │
    -π/2     π/2    3π/2
  Period = π, Odd function
  Vertical asymptotes at x = π/2 + nπ
```

### 1.5.11 Graph Transformations (Summary)

| Transformation | Effect on Graph |
|----------------|-----------------|
| $f(x) + c$ | Shift **up** by $c$ |
| $f(x) - c$ | Shift **down** by $c$ |
| $f(x - c)$ | Shift **right** by $c$ |
| $f(x + c)$ | Shift **left** by $c$ |
| $-f(x)$ | Reflect about **x-axis** |
| $f(-x)$ | Reflect about **y-axis** |
| $cf(x)$ ($c > 1$) | Vertical **stretch** by factor $c$ |
| $f(cx)$ ($c > 1$) | Horizontal **compression** by factor $c$ |
| $|f(x)|$ | Reflect negative portions above x-axis |

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Function | Rule assigning each input exactly one output |
| Domain | Set of all valid inputs |
| Range | Set of all actual outputs |
| Injective | Different inputs → different outputs |
| Surjective | Every element in codomain is an output |
| Bijective | Both injective and surjective |
| Composition $f \circ g$ | Apply $g$ first, then $f$ |
| Inverse $f^{-1}$ | Undoes $f$; exists only if $f$ is bijective |
| Even function | Symmetric about $y$-axis: $f(-x) = f(x)$ |
| Odd function | Symmetric about origin: $f(-x) = -f(x)$ |
| Vertical line test | Tells if a graph is a function |
| Horizontal line test | Tells if a function is one-to-one |

---

## Quick Revision Questions

**Q1.** Is $f(x) = x^2$ a one-to-one function on $\mathbb{R}$? Justify using the horizontal line test.

**Q2.** Find the domain of $f(x) = \sqrt{3 - x} + \ln(x + 1)$.

**Q3.** If $f(x) = 2x + 5$ and $g(x) = x^2$, find $(f \circ g)(3)$ and $(g \circ f)(3)$.

**Q4.** Find the inverse of $f(x) = \frac{2x + 1}{x - 3}$, $x \ne 3$.

**Q5.** Classify $f(x) = x^3 + \sin x$ as even, odd, or neither. Prove your answer.

**Q6.** Sketch the graph of $f(x) = |x - 2| + 1$ by applying transformations to $|x|$.

---

[← Back to README](../README.md) · [Next: Unit 2 — Limits →](../02-Limits/01-concept-of-limit.md)
