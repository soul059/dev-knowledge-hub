# Chapter 5: Connected Sets

[← Previous: Heine–Borel Theorem](04-heine-borel-theorem.md) | [Back to README](../README.md) | [Next: Perfect Sets →](06-perfect-sets.md)

---

## Overview

A **connected set** cannot be split into two non-empty "separated" pieces. In $\mathbb{R}$, the connected sets are precisely the **intervals**. Connectedness is the topological property behind the **Intermediate Value Theorem**.

---

## 5.1 Separated and Disconnected Sets

**Definition 5.1.** Two sets $A, B \subseteq \mathbb{R}$ are **separated** if:

$$\overline{A} \cap B = \emptyset \quad \text{and} \quad A \cap \overline{B} = \emptyset$$

Neither set contains a limit point of the other.

**Definition 5.2.** A set $E \subseteq \mathbb{R}$ is **disconnected** if there exist non-empty separated sets $A, B$ with $E = A \cup B$.

A set is **connected** if it is not disconnected.

```
  Disconnected:    E = A ∪ B  (separated)
  
  ═══[A]═══          ═══[B]═══
          ↑          ↑
       No limit points of B     No limit points of A
       are in A                 are in B

  Connected:    Cannot be split this way
  
  ════════════[E]════════════
  Any partition into A ∪ B fails the separation requirement.
```

---

## 5.2 Equivalent Formulations

**Theorem 5.3.** The following are equivalent for $E \subseteq \mathbb{R}$:

1. $E$ is connected
2. There do not exist open sets $U, V$ with $E \subseteq U \cup V$, $E \cap U \neq \emptyset$, $E \cap V \neq \emptyset$, $E \cap U \cap V = \emptyset$
3. The only subsets of $E$ that are both open and closed relative to $E$ are $\emptyset$ and $E$ itself
4. Every continuous function $f: E \to \{0, 1\}$ is constant

---

## 5.3 Connected Sets in ℝ are Intervals

**Theorem 5.4.** $E \subseteq \mathbb{R}$ is connected $\iff$ $E$ is an interval.

(An **interval** means: if $a, b \in E$ and $a < c < b$, then $c \in E$.)

**Proof ($\Leftarrow$: intervals are connected).** Suppose $E$ is an interval and $E = A \cup B$ with $A, B$ non-empty and separated. Pick $a \in A$, $b \in B$, WLOG $a < b$.

Let $s = \sup(A \cap [a, b])$. Since $A$ is separated from $B$: $s \notin B$ (since $s \in \overline{A}$ and $\overline{A} \cap B = \emptyset$). And $s \in \overline{A}$, but if $s \in A$, then $s < b$ (since $b \in B$), and by the interval property, $(s, s + \varepsilon) \cap E \neq \emptyset$, and these points must be in $B$, contradicting $A \cap \overline{B} = \emptyset$. Contradiction. $\blacksquare$

**Proof ($\Rightarrow$: connected sets are intervals).** Suppose $E$ is not an interval. Then $\exists\, a < c < b$ with $a, b \in E$ but $c \notin E$. Set $A = E \cap (-\infty, c)$ and $B = E \cap (c, \infty)$. These are separated and $E = A \cup B$, so $E$ is disconnected. $\blacksquare$

```
  Connected subsets of ℝ:

  Intervals:
  (a, b), [a, b], [a, b), (a, b],
  (-∞, b), (-∞, b], (a, ∞), [a, ∞),
  (-∞, ∞), {a} (single point)

  NOT connected:
  {0, 1}     →  A = {0}, B = {1}
  [0,1]∪[2,3] → A = [0,1], B = [2,3]
  ℚ          →  A = ℚ ∩ (-∞,√2), B = ℚ ∩ (√2, ∞)
```

---

## 5.4 Path Connectedness

**Definition 5.5.** $E$ is **path connected** if $\forall\, a, b \in E$, $\exists$ a continuous function $\gamma: [0,1] \to E$ with $\gamma(0) = a$ and $\gamma(1) = b$.

In $\mathbb{R}$: path connected $\iff$ connected $\iff$ interval.

(In higher dimensions, connected does not always imply path connected, but in $\mathbb{R}$ they coincide.)

---

## 5.5 Connected Components

**Definition 5.6.** The **connected component** of $x \in E$ is the largest connected subset of $E$ containing $x$.

Every set decomposes uniquely into connected components.

**Example:** $E = [0,1] \cup [2,3] \cup \{5\}$ has three connected components: $[0,1]$, $[2,3]$, $\{5\}$.

---

## 5.6 Connectedness and the Intermediate Value Theorem

**Theorem 5.7 (IVT — topological version).** Continuous images of connected sets are connected.

**Proof.** Let $f: E \to \mathbb{R}$ be continuous and $E$ connected. Suppose $f(E) = A \cup B$ with $A, B$ separated. Then $E = f^{-1}(A) \cup f^{-1}(B)$. By continuity, these preimages are separated (limit points map to limit points). Since $E$ is connected, one must be empty, so one of $A, B$ is empty. $\blacksquare$

**Corollary (Classical IVT).** If $f: [a,b] \to \mathbb{R}$ is continuous and $f(a) < v < f(b)$, then $\exists\, c \in (a,b)$ with $f(c) = v$.

*Proof:* $[a,b]$ connected $\Rightarrow$ $f([a,b])$ connected $\Rightarrow$ $f([a,b])$ is an interval $\Rightarrow$ contains $v$. $\blacksquare$

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| Separated sets | $\overline{A} \cap B = \emptyset$ and $A \cap \overline{B} = \emptyset$ |
| Connected | Cannot be written as union of separated non-empty sets |
| In $\mathbb{R}$ | Connected $\iff$ interval |
| Path connected | In $\mathbb{R}$, same as connected |
| Connected components | Maximal connected subsets; partition $E$ |
| Continuous image | Connected $\to$ connected |
| IVT | Follows directly from connectedness |

---

## Quick Revision Questions

1. **Define** separated sets and connected sets.

2. **Prove** that the connected subsets of $\mathbb{R}$ are exactly the intervals.

3. **Is** $\mathbb{Q}$ connected? Prove your answer.

4. **Show** that continuous images of connected sets are connected.

5. **Derive** the Intermediate Value Theorem from the connectedness of $[a,b]$.

6. **Find** the connected components of $(-1, 0) \cup \{1\} \cup [2, 4]$.

---

[← Previous: Heine–Borel Theorem](04-heine-borel-theorem.md) | [Back to README](../README.md) | [Next: Perfect Sets →](06-perfect-sets.md)
