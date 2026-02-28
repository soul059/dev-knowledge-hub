# Chapter 5: Intermediate Value Theorem

[← Previous: Properties of Continuous Functions](04-properties-of-continuous-functions.md) | [Back to README](../README.md) | [Next: Extreme Value Theorem →](06-extreme-value-theorem.md)

---

## Overview

The **Intermediate Value Theorem (IVT)** states that a continuous function on an interval takes every value between $f(a)$ and $f(b)$. It is one of the most important consequences of the completeness of $\mathbb{R}$ and the connectedness of intervals.

---

## 5.1 Statement

**Theorem 5.1 (Intermediate Value Theorem).** Let $f: [a,b] \to \mathbb{R}$ be continuous with $f(a) \neq f(b)$. For every value $v$ between $f(a)$ and $f(b)$, there exists $c \in (a, b)$ with $f(c) = v$.

```
  IVT:  f continuous on [a,b]

  f(b) ●─────────────────────────●
       │                       ╱
   v   │─ ─ ─ ─ ─ ─ ─ ─ ─ ─╱─ ─ ─ ← must cross this line!
       │                  ╱
       │               ╱
  f(a) ●─────────── ╱
       │          ╱
       └──────┼──┼──────────┼──────► x
              a  c          b

  Every horizontal line between f(a) and f(b) 
  must be crossed. No "jumping over" values!
```

---

## 5.2 Proof

**Proof.** Assume $f(a) < v < f(b)$ (the other case is symmetric).

Let $S = \{x \in [a, b] : f(x) < v\}$.

- $S \neq \emptyset$ since $a \in S$
- $S$ is bounded above by $b$
- Let $c = \sup S$

**Claim:** $f(c) = v$.

**$f(c) \geq v$:** If $f(c) < v$, then by continuity, $f(x) < v$ for $x$ in a neighborhood of $c$. So $c + \delta \in S$ for small $\delta > 0$, contradicting $c = \sup S$.

**$f(c) \leq v$:** If $f(c) > v$, then by continuity, $f(x) > v$ for $x$ in $(c - \delta, c + \delta)$. So no $x \in (c-\delta, c)$ satisfies $f(x) < v$, meaning $\sup S \leq c - \delta$, contradicting $c = \sup S$.

Therefore $f(c) = v$. $\blacksquare$

```
  Proof structure using LUB:

  a═══════════════════c═════════════b
  ◄── f < v on S ───►|← f ≥ v ──►
                      ▲
                    c = sup S
                   f(c) = v

  Key: LUB property of ℝ guarantees c exists.
  Continuity forces f(c) = v exactly.
```

---

## 5.3 Topological Proof

**Alternative proof.** $[a,b]$ is connected. $f$ continuous $\Rightarrow$ $f([a,b])$ is connected $\Rightarrow$ $f([a,b])$ is an interval. Since $f(a), f(b) \in f([a,b])$, every value between them is in $f([a,b])$. $\blacksquare$

---

## 5.4 Applications

**Application 1: Root Finding.** If $f$ is continuous and $f(a) < 0 < f(b)$, then $\exists\, c$ with $f(c) = 0$.

**Application 2: Fixed Point Theorem.** If $f: [0,1] \to [0,1]$ is continuous, then $f$ has a **fixed point**: $\exists\, c$ with $f(c) = c$.

*Proof.* Let $g(x) = f(x) - x$. Then $g(0) = f(0) \geq 0$ and $g(1) = f(1) - 1 \leq 0$. By IVT, $\exists\, c$ with $g(c) = 0$, i.e., $f(c) = c$. $\blacksquare$

```
  Fixed Point Theorem:

  y ▲
  1 │─────────────────● 
    │               ╱ │
    │  f(x)      ╱   │
    │         ╱╱      │  y = x line
    │      ╱╱ ●←fixed │
    │   ╱╱    point   │
    │╱╱               │
    ●─────────────────┼──► x
  0                   1

  f: [0,1] → [0,1] continuous
  ⟹ f(x) = x has a solution
```

**Application 3: $n$-th Roots.** $\forall\, a > 0$, $n \in \mathbb{N}$, $\exists\, x > 0$ with $x^n = a$. Apply IVT to $f(x) = x^n$ on $[0, \max(a, 1)]$.

**Application 4: Odd-degree polynomials have roots.** $p(x) \to -\infty$ as $x \to -\infty$ and $p(x) \to +\infty$ as $x \to +\infty$. By IVT, $p$ has a real root.

---

## 5.5 Bisection Method

The IVT proof gives an algorithm: **bisection**.

Given $f(a) < 0 < f(b)$, to find $c$ with $f(c) = 0$:
1. Let $m = (a+b)/2$
2. If $f(m) = 0$: done
3. If $f(m) < 0$: replace $a$ with $m$
4. If $f(m) > 0$: replace $b$ with $m$
5. Repeat. Interval halves each step: error $\leq (b-a)/2^n$ after $n$ steps.

```
  Bisection: finding √2 (root of x² - 2 = 0)

  n  [a, b]          m       f(m) = m²-2
  0  [1, 2]         1.5       0.25 > 0 → b = 1.5
  1  [1, 1.5]       1.25     -0.4375 < 0 → a = 1.25
  2  [1.25, 1.5]    1.375    -0.109 < 0 → a = 1.375
  3  [1.375, 1.5]   1.4375    0.066 > 0 → b = 1.4375
  4  [1.375, 1.4375] 1.40625 -0.022 < 0 → a = 1.40625
  ...
  → √2 ≈ 1.41421...
```

---

## 5.6 IVT Converse is False

**Warning:** The converse of IVT does not hold. A function can satisfy the intermediate value property without being continuous.

**Example:** $f(x) = \sin(1/x)$ for $x \neq 0$ and $f(0) = 0$ satisfies IVP but is discontinuous at $0$.

**Darboux's Theorem** (Unit 6): Derivatives always satisfy the IVP, even if discontinuous.

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| IVT | Continuous on $[a,b]$ → takes every value between $f(a)$ and $f(b)$ |
| Proof technique | $c = \sup \{x : f(x) < v\}$; continuity forces $f(c) = v$ |
| Topological proof | Connected → continuous image connected → interval |
| Fixed Point | $f: [0,1] \to [0,1]$ continuous has $f(c) = c$ |
| Root finding | $f(a) < 0 < f(b)$ → $\exists$ root in $(a,b)$ |
| Bisection method | Halve interval each step; error $\leq (b-a)/2^n$ |
| Converse fails | IVP $\not\Rightarrow$ continuity |

---

## Quick Revision Questions

1. **State and prove** the Intermediate Value Theorem.

2. **Prove** the Fixed Point Theorem for $f: [0,1] \to [0,1]$.

3. **Use IVT** to show every positive real number has an $n$-th root.

4. **Show** every odd-degree polynomial has at least one real root.

5. **Explain** the bisection method. How many iterations for 10 decimal places of accuracy starting from $[0, 2]$?

6. **Give** an example of a discontinuous function that satisfies the intermediate value property.

---

[← Previous: Properties of Continuous Functions](04-properties-of-continuous-functions.md) | [Back to README](../README.md) | [Next: Extreme Value Theorem →](06-extreme-value-theorem.md)
