# Chapter 2: Limit Points and Closure

[← Previous: Open and Closed Sets](01-open-and-closed-sets.md) | [Back to README](../README.md) | [Next: Compact Sets →](03-compact-sets.md)

---

## Overview

A **limit point** of a set $A$ is a point that can be "approached" by other points in $A$. The **closure** $\overline{A}$ is $A$ plus all its limit points. These concepts unify the ideas of open/closed sets and convergence, and are essential for understanding compactness and continuity.

---

## 2.1 Limit Points (Accumulation Points)

**Definition 2.1.** A point $x \in \mathbb{R}$ is a **limit point** (or **accumulation point**) of $A \subseteq \mathbb{R}$ if every $\varepsilon$-neighborhood of $x$ contains a point of $A$ different from $x$:

$$\forall\, \varepsilon > 0,\; (V_\varepsilon(x) \setminus \{x\}) \cap A \neq \emptyset$$

The set of all limit points of $A$ is denoted $A'$.

```
  x is a limit point of A:

  ──────●───●──●─●●●─[x]─●●●─●──●───●────────
        ▲   ▲  ▲ ▲▲▲      ▲▲▲ ▲  ▲   ▲
        Points of A cluster around x
        (x itself may or may not be in A)

  x is NOT a limit point (isolated):
  ──────────────────[x]────────────────────
                     ▲
                No other points of A nearby
```

**Equivalent formulation:** $x$ is a limit point of $A$ $\iff$ $\exists$ a sequence $(x_n) \subseteq A \setminus \{x\}$ with $x_n \to x$.

---

## 2.2 Isolated Points

**Definition 2.2.** A point $x \in A$ is an **isolated point** of $A$ if $x$ is NOT a limit point of $A$: $\exists\, \varepsilon > 0$ such that $V_\varepsilon(x) \cap A = \{x\}$.

**Partition:** Every point of $A$ is either a limit point of $A$ or an isolated point of $A$ (but not both).

| Set $A$ | Limit points $A'$ | Isolated points |
|---------|-------------------|-----------------|
| $(0, 1)$ | $[0, 1]$ | $\emptyset$ |
| $[0, 1]$ | $[0, 1]$ | $\emptyset$ |
| $\{1/n : n \in \mathbb{N}\}$ | $\{0\}$ | $\{1/n : n \in \mathbb{N}\}$ |
| $\mathbb{Z}$ | $\emptyset$ | $\mathbb{Z}$ |
| $\mathbb{Q}$ | $\mathbb{R}$ | $\emptyset$ |
| $\{1, 2, 3\}$ | $\emptyset$ | $\{1, 2, 3\}$ |
| $\{0\} \cup (1, 2)$ | $[1, 2]$ | $\{0\}$ |

---

## 2.3 Closure

**Definition 2.3.** The **closure** of $A$ is:

$$\overline{A} = A \cup A'$$

where $A'$ is the set of limit points of $A$.

**Equivalent characterizations:**
1. $\overline{A} = A \cup A'$
2. $\overline{A}$ = the smallest closed set containing $A$
3. $\overline{A} = \bigcap\{F : F \text{ closed}, F \supseteq A\}$
4. $x \in \overline{A} \iff \forall\, \varepsilon > 0$, $V_\varepsilon(x) \cap A \neq \emptyset$
5. $x \in \overline{A} \iff \exists\, (x_n) \subseteq A$ with $x_n \to x$

| Set $A$ | $\overline{A}$ |
|---------|----------------|
| $(0, 1)$ | $[0, 1]$ |
| $\mathbb{Q}$ | $\mathbb{R}$ |
| $\mathbb{Z}$ | $\mathbb{Z}$ |
| $\{1/n\}$ | $\{0\} \cup \{1/n\}$ |

---

## 2.4 Closed Sets via Limit Points

**Theorem 2.4.** $A$ is closed $\iff$ $A' \subseteq A$ $\iff$ $\overline{A} = A$.

**Proof.** 
- $A$ closed $\Rightarrow$ $A' \subseteq A$: If $x \in A'$, then every neighborhood of $x$ meets $A$, so any sequence in $A$ converges to $x$--hence $x \in A$ (closed sets contain limits).
- $A' \subseteq A$ $\Rightarrow$ $A$ closed: Show $A^c$ is open. If $x \in A^c$, then $x \notin A'$ (since $A' \subseteq A$), so $\exists\, \varepsilon > 0$ with $V_\varepsilon(x) \cap A = \emptyset$ (possibly $\{x\}$), but $x \notin A$, so $V_\varepsilon(x) \cap A = \emptyset$, meaning $V_\varepsilon(x) \subseteq A^c$. $\blacksquare$

---

## 2.5 Properties of Closure

**Theorem 2.5 (Kuratowski Closure Axioms).**

1. $\overline{\emptyset} = \emptyset$
2. $A \subseteq \overline{A}$
3. $\overline{\overline{A}} = \overline{A}$ (closure is idempotent)
4. $\overline{A \cup B} = \overline{A} \cup \overline{B}$

```
  Closure "fills in" the boundary:

  A = (0,1) ∪ (2,3):
  
  ──┤     ├───┤     ├──
    0     1   2     3

  Ā = [0,1] ∪ [2,3]:
  
  ──[─────]───[─────]──
    0     1   2     3

  Closure adds the endpoints / boundary points.
```

---

## 2.6 Dense Sets

**Definition 2.6.** $A$ is **dense** in $\mathbb{R}$ if $\overline{A} = \mathbb{R}$.

Equivalently: every non-empty open interval contains a point of $A$.

| Set | Dense in $\mathbb{R}$? |
|-----|:---------------------:|
| $\mathbb{Q}$ | ✅ |
| $\mathbb{R} \setminus \mathbb{Q}$ | ✅ |
| $\{n\sqrt{2} \bmod 1 : n \in \mathbb{N}\}$ | ✅ |
| $\mathbb{Z}$ | ❌ |
| Cantor set | ❌ (it's nowhere dense) |

**Definition 2.7.** $A$ is **nowhere dense** if $\overline{A}$ has empty interior: $(\overline{A})^\circ = \emptyset$.

---

## 2.7 Derived Set Properties

**Theorem 2.8.** 
- $A \subseteq B \Rightarrow A' \subseteq B'$
- $(A \cup B)' = A' \cup B'$
- $\overline{A}$ is always closed (i.e., $(\overline{A})' \subseteq \overline{A}$)
- Countable sets can have uncountable limit point sets (e.g., rationals in a Cantor-like construction)

---

## 2.8 Relationship Diagram

```
  For a set A ⊆ ℝ:

  ┌──────────────────────────────────┐
  │             Ā (closure)          │
  │  ┌────────────────────────────┐  │
  │  │          A                 │  │
  │  │  ┌──────────────────────┐  │  │
  │  │  │    A° (interior)     │  │  │
  │  │  └──────────────────────┘  │  │
  │  └────────────────────────────┘  │
  │  ← ∂A (boundary) = Ā \ A° →     │
  └──────────────────────────────────┘

  A° ⊆ A ⊆ Ā
  ∂A = Ā \ A°

  A is open   ⟺  A = A°
  A is closed ⟺  A = Ā
  A is clopen ⟺  A = A° = Ā  (so ∂A = ∅)
```

---

## Summary Table

| Concept | Definition / Key Fact |
|---------|----------------------|
| Limit point of $A$ | Every nbhd of $x$ meets $A \setminus \{x\}$ |
| Isolated point | $x \in A$ with a nbhd meeting $A$ only at $x$ |
| Closure $\overline{A}$ | $A \cup A'$ = smallest closed superset of $A$ |
| $A$ closed | $\iff A' \subseteq A \iff \overline{A} = A$ |
| Dense set | $\overline{A} = \mathbb{R}$; e.g., $\mathbb{Q}$ |
| Nowhere dense | $(\overline{A})^\circ = \emptyset$ |
| Interior $\subseteq$ Set $\subseteq$ Closure | $A^\circ \subseteq A \subseteq \overline{A}$ |
| Boundary | $\partial A = \overline{A} \setminus A^\circ$ |

---

## Quick Revision Questions

1. **Find** the limit points and isolated points of $A = \{1/n : n \in \mathbb{N}\} \cup [2, 3)$.

2. **Prove** that $x$ is a limit point of $A$ iff there exists a sequence in $A \setminus \{x\}$ converging to $x$.

3. **Show** that $\overline{A}$ is always closed.

4. **Prove** $\overline{A \cup B} = \overline{A} \cup \overline{B}$. Does $\overline{A \cap B} = \overline{A} \cap \overline{B}$?

5. **What is** the closure, interior, and boundary of $\mathbb{Q}$?

6. **True or False:** Every point of a closed set is a limit point. Give a counterexample if false.

---

[← Previous: Open and Closed Sets](01-open-and-closed-sets.md) | [Back to README](../README.md) | [Next: Compact Sets →](03-compact-sets.md)
