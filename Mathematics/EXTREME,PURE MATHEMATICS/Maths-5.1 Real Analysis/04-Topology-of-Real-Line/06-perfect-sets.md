# Chapter 6: Perfect Sets and the Cantor Set

[← Previous: Connected Sets](05-connected-sets.md) | [Back to README](../README.md) | [Next: Unit 5 — Limit of a Function →](../05-Limits-and-Continuity/01-limit-of-a-function.md)

---

## Overview

A **perfect set** is closed with no isolated points — every point is a limit point. The **Cantor set** is the prototypical example: it is compact, perfect, uncountable, nowhere dense, and has measure zero. It demonstrates that "small" sets (in terms of length) can still be topologically "large" (uncountable).

---

## 6.1 Perfect Sets

**Definition 6.1.** A set $P \subseteq \mathbb{R}$ is **perfect** if:
1. $P$ is closed
2. $P$ has no isolated points (every point is a limit point)

Equivalently: $P$ is closed and $P = P'$ (equals its own set of limit points).

**Examples:**
| Set | Perfect? | Reason |
|-----|:--------:|--------|
| $[a, b]$ | ✅ | Closed; every point is a limit point |
| $\mathbb{R}$ | ✅ | Closed; every point is a limit point |
| $\{0\} \cup [1, 2]$ | ❌ | $0$ is isolated |
| $\mathbb{Z}$ | ❌ | All points are isolated |
| Cantor set $C$ | ✅ | (proved below) |
| $\{1/n\} \cup \{0\}$ | ❌ | Each $1/n$ is isolated |

---

## 6.2 Perfect Sets are Uncountable

**Theorem 6.2.** Every non-empty perfect set in $\mathbb{R}$ is uncountable.

**Proof.** Let $P$ be perfect and non-empty. $P$ is closed and every point is a limit point, so $P$ is infinite.

Suppose for contradiction that $P = \{x_1, x_2, x_3, \ldots\}$ is countable. We construct nested closed intervals $I_1 \supseteq I_2 \supseteq \cdots$ with $I_n \cap P \neq \emptyset$ and $x_n \notin I_n$.

- Since $x_1$ is a limit point of $P$, $\exists$ distinct $y_1, z_1 \in P$ near $x_1$. Choose a closed interval $I_1$ containing points of $P$ but not $x_1$.
- At step $n$: $I_{n-1} \cap P$ is infinite (since any point is a limit point). Choose $I_n \subseteq I_{n-1}$ with $I_n \cap P \neq \emptyset$ and $x_n \notin I_n$.
- $\{I_n\}$ are nested, closed, bounded, non-empty. By NIP, $\exists\, x \in \bigcap I_n$. Since each $I_n$ is closed and $I_n \cap P \neq \emptyset$ with $P$ closed, $x \in P$. But $x \neq x_n$ for all $n$, contradicting $P = \{x_1, x_2, \ldots\}$. $\blacksquare$

```
  Proof idea:
  
  P = {x₁, x₂, x₃, ...} (supposed countable listing)
  
  I₁: ═══════[══●══●══●══]══════  (avoids x₁)
  I₂:        [══●══●══]            (avoids x₂)
  I₃:           [══●══]            (avoids x₃)
  ...
  
  ∩Iₙ contains a point x ∈ P not in the list → contradiction!
```

---

## 6.3 The Cantor Set — Construction

The **Cantor (middle-thirds) set** is constructed by repeatedly removing middle thirds:

$$C_0 = [0, 1]$$
$$C_1 = [0, 1/3] \cup [2/3, 1]$$
$$C_2 = [0, 1/9] \cup [2/9, 1/3] \cup [2/3, 7/9] \cup [8/9, 1]$$
$$C = \bigcap_{n=0}^{\infty} C_n$$

```
  Cantor set construction:

  C₀: [═══════════════════════════════════]
       0                                   1

  C₁: [════════]          [════════════════]
      0       1/3        2/3              1

  C₂: [══]    [═══]      [═══]    [═══════]
      0  1/9 2/9 1/3    2/3 7/9  8/9     1

  C₃: [=] [=] [=] [=]    [=] [=]  [=] [=]

  C:  · · · · · · · ·    · · · ·  · · · ·
      (uncountably many points remain!)
```

---

## 6.4 Properties of the Cantor Set

**Theorem 6.3.** The Cantor set $C$ satisfies:

| Property | Status | Reason |
|----------|:------:|--------|
| Compact | ✅ | Intersection of closed bounded sets |
| Closed | ✅ | Intersection of closed sets |
| Bounded | ✅ | $C \subseteq [0, 1]$ |
| Non-empty | ✅ | Endpoints always remain |
| Perfect | ✅ | No isolated points (proved below) |
| Uncountable | ✅ | Follows from perfect + Theorem 6.2 |
| Nowhere dense | ✅ | Interior is empty |
| Measure zero | ✅ | Total length removed $= 1$ |
| Totally disconnected | ✅ | Contains no intervals |

---

## 6.5 Cantor Set and Ternary Expansions

**Theorem 6.4.** $x \in C$ if and only if $x$ has a base-3 (ternary) expansion using only digits $0$ and $2$:

$$x = \sum_{n=1}^{\infty} \frac{a_n}{3^n}, \quad a_n \in \{0, 2\}$$

```
  Ternary expansion correspondence:

  Step 1: Remove middle third → remove numbers with
          digit 1 in first ternary place
  
  [0, 1/3]  → first digit 0
  (1/3, 2/3) → first digit 1  ← REMOVED
  [2/3, 1]  → first digit 2
  
  Step 2: Remove middle third of each remaining
          → remove digit 1 in second place

  C = {0.a₁a₂a₃... (base 3) : each aₙ ∈ {0, 2}}
```

---

## 6.6 Cantor Set is Uncountable (Direct Proof)

Using ternary expansions, define $f: C \to [0, 1]$ by:

$$f\left(\sum \frac{a_n}{3^n}\right) = \sum \frac{a_n/2}{2^n}$$

This maps each ternary string of 0s and 2s to the binary string obtained by replacing 2 with 1. This function is **surjective** onto $[0, 1]$, so $|C| \geq |[0, 1]| = \mathfrak{c}$ (continuum).

Since $C \subseteq [0,1]$, we have $|C| = \mathfrak{c}$: **the Cantor set has the same cardinality as $\mathbb{R}$!**

---

## 6.7 Cantor Set Has Measure Zero

At stage $n$, we remove $2^{n-1}$ intervals each of length $1/3^n$:

$$\text{Total length removed} = \sum_{n=1}^{\infty} \frac{2^{n-1}}{3^n} = \frac{1}{3} \cdot \frac{1}{1 - 2/3} = 1$$

So the Cantor set has **Lebesgue measure zero** — despite being uncountable!

```
  The paradox:
  ┌─────────────────────────────────────────┐
  │  Cantor set C:                          │
  │  • Has measure (length) = 0             │
  │  • Has cardinality = |ℝ|               │
  │  • Contains no intervals               │
  │  • Yet is uncountable!                  │
  │                                         │
  │  "Almost all" of [0,1] is removed,      │
  │  yet uncountably many points remain.    │
  └─────────────────────────────────────────┘
```

---

## 6.8 C is Perfect (No Isolated Points)

**Proof.** Let $x \in C$ and $\varepsilon > 0$. We find another point of $C$ within $\varepsilon$ of $x$.

$x$ lies in an interval $I_n$ of $C_n$ with $|I_n| = 1/3^n$. Choose $n$ with $1/3^n < \varepsilon$. The interval $I_n$ has two sub-intervals in $C_{n+1}$: $x$ is in one, and the endpoint of the other is in $C$ and within $1/3^n < \varepsilon$ of $x$. $\blacksquare$

---

## 6.9 Generalizations: Fat Cantor Sets

By removing smaller middle portions, we can create **Cantor-like sets with positive measure**.

**Example:** At stage $n$, remove the middle $1/4^n$ instead of $1/3$:

$$\text{Total removed} = \sum_{n=1}^{\infty} \frac{2^{n-1}}{4^n} = \frac{1/4}{1 - 1/2} = \frac{1}{2}$$

This "fat Cantor set" has measure $1/2$, is still compact, perfect, nowhere dense, and uncountable — but has positive measure.

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| Perfect set | Closed + no isolated points ($P = P'$) |
| Perfect ⟹ uncountable | NIP argument avoids each listed point |
| Cantor set $C$ | $\bigcap C_n$; remove middle thirds |
| $C$ is compact, perfect | Intersection of closed bounded sets; no isolated points |
| $C$ is uncountable | $\|C\| = \mathfrak{c}$ via ternary expansion |
| $C$ has measure zero | Total removed $= 1$ |
| $C$ is nowhere dense | Contains no intervals |
| Ternary characterization | Digits $\in \{0, 2\}$ in base 3 |
| Fat Cantor sets | Positive measure, still perfect and nowhere dense |

---

## Quick Revision Questions

1. **Define** a perfect set. Give three examples.

2. **Prove** that every non-empty perfect set is uncountable.

3. **Describe** the construction of the Cantor set and show it has measure zero.

4. **Characterize** the Cantor set using ternary expansions. Use this to prove it is uncountable.

5. **Show** the Cantor set is perfect (has no isolated points).

6. **What is** a "fat Cantor set"? How does it differ from the standard Cantor set?

---

[← Previous: Connected Sets](05-connected-sets.md) | [Back to README](../README.md) | [Next: Unit 5 — Limit of a Function →](../05-Limits-and-Continuity/01-limit-of-a-function.md)
