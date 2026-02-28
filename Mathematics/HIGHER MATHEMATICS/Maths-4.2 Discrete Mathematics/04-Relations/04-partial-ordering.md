# Chapter 4.4: Partial Ordering

[← Previous: Equivalence Relations and Partitions](03-equivalence-relations-and-partitions.md) | [Back to README](../README.md) | [Next: Hasse Diagrams →](05-hasse-diagrams.md)

---

## 📋 Chapter Overview

A **partial order** imposes a hierarchical structure on a set where some elements can be compared but not necessarily all. This generalizes the familiar $\leq$ on numbers to arbitrary sets.

---

## 1. Definition

A relation $\preceq$ on a set $A$ is a **partial order** if it is:

1. **Reflexive:** $\forall a \in A, \; a \preceq a$
2. **Antisymmetric:** $\forall a, b \in A, \; a \preceq b \land b \preceq a \Rightarrow a = b$
3. **Transitive:** $\forall a, b, c \in A, \; a \preceq b \land b \preceq c \Rightarrow a \preceq c$

The pair $(A, \preceq)$ is called a **partially ordered set (poset)**.

---

## 2. Important Examples

| Poset | Set | Order Relation |
|-------|-----|---------------|
| $(\mathbb{Z}, \leq)$ | Integers | Usual ≤ |
| $(\mathbb{N}, \mid)$ | Natural numbers | Divisibility |
| $(\mathcal{P}(S), \subseteq)$ | Power set | Subset inclusion |
| $(S^*, \text{prefix})$ | Strings | "is a prefix of" |

---

## 3. Comparability

In a poset $(A, \preceq)$, two elements $a, b$ are:

- **Comparable** if $a \preceq b$ or $b \preceq a$
- **Incomparable** if neither $a \preceq b$ nor $b \preceq a$

```
  Divisibility on {2, 3, 6, 12}:
  
  2 | 6 ✓  (comparable)
  2 | 12 ✓ (comparable)
  3 | 6 ✓  (comparable)
  2 ∤ 3    (incomparable!)
  
  This is why it's "partial" — not everything
  can be compared.
```

---

## 4. Total Orders (Linear Orders)

A partial order where **every** pair of elements is comparable is a **total order**:

$$\forall a, b \in A, \; a \preceq b \text{ or } b \preceq a$$

| Example | Total? |
|---------|:------:|
| $(\mathbb{Z}, \leq)$ | ✅ |
| $(\mathbb{R}, \leq)$ | ✅ |
| Dictionary order on words | ✅ |
| $(\mathcal{P}(\{1,2,3\}), \subseteq)$ | ❌ (e.g., $\{1\}$ and $\{2\}$ incomparable) |
| $(\mathbb{N}, \mid)$ | ❌ (e.g., 2 and 3 incomparable) |

A total order is also called a **chain**.

---

## 5. Special Elements in a Poset

Let $(A, \preceq)$ be a poset and $B \subseteq A$:

| Element | Definition |
|---------|-----------|
| **Minimum** of $B$ | $a \in B$ such that $a \preceq b$ for all $b \in B$ |
| **Maximum** of $B$ | $a \in B$ such that $b \preceq a$ for all $b \in B$ |
| **Minimal** element | $a \in B$ such that no $b \in B$ has $b \prec a$ |
| **Maximal** element | $a \in B$ such that no $b \in B$ has $a \prec b$ |
| **Lower bound** of $B$ | $a \in A$ (not necessarily in $B$) with $a \preceq b$ for all $b \in B$ |
| **Upper bound** of $B$ | $a \in A$ with $b \preceq a$ for all $b \in B$ |
| **GLB / Infimum** ($\inf B$) | Greatest lower bound |
| **LUB / Supremum** ($\sup B$) | Least upper bound |

```
  ┌──────────────────────────────────────────────────┐
  │   Minimum vs. Minimal (They're different!)       │
  │                                                   │
  │   Minimum: ≤ everything (unique if exists)       │
  │   Minimal: nothing is strictly less (may be multiple) │
  │                                                   │
  │   Example: ({2,3,6}, |)                           │
  │   2 and 3 are MINIMAL (nothing divides them)     │
  │   But there is NO MINIMUM (2 ∤ 3, 3 ∤ 2)        │
  └──────────────────────────────────────────────────┘
```

---

## 6. Worked Example: Divisibility Poset

$(S, \mid)$ where $S = \{1, 2, 3, 4, 6, 12\}$:

```
  Divisibility relationships:
  
  1 | 2, 1 | 3, 1 | 4, 1 | 6, 1 | 12
  2 | 4, 2 | 6, 2 | 12
  3 | 6, 3 | 12
  4 | 12
  6 | 12
```

| Element | Description |
|---------|-------------|
| Minimum | 1 (divides everything) |
| Maximum | 12 (everything divides it) |
| Minimal elements | {1} |
| Maximal elements | {12} |

For $B = \{4, 6\}$:
- Lower bounds: $\{1, 2\}$ (divide both 4 and 6)
- Upper bounds: $\{12\}$ (both 4 and 6 divide 12)
- GLB = $\gcd(4,6) = 2$
- LUB = $\text{lcm}(4,6) = 12$

---

## 7. Well-Ordered Sets

A poset $(A, \preceq)$ is **well-ordered** if every non-empty subset has a **minimum element**.

| Set | Well-ordered? |
|-----|:---:|
| $(\mathbb{N}, \leq)$ | ✅ |
| $(\mathbb{Z}, \leq)$ | ❌ (no minimum for $\mathbb{Z}$ itself) |
| $(\mathbb{Q}^+, \leq)$ | ❌ (no minimum for $(0, 1) \cap \mathbb{Q}$) |

The **Well-Ordering Principle** states that $\mathbb{N}$ is well-ordered under $\leq$.

---

## 8. Strict Partial Orders

The **strict version** $\prec$ is defined as:

$$a \prec b \iff a \preceq b \text{ and } a \neq b$$

Properties: **Irreflexive**, **Asymmetric**, **Transitive**

You can convert between them:

$$a \preceq b \iff a \prec b \text{ or } a = b$$

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Partial Orders in Practice                │
  │                                                   │
  │  1. Task Scheduling                              │
  │     Tasks with dependencies form a poset          │
  │     "Must finish A before starting B"             │
  │                                                   │
  │  2. Version Control                               │
  │     Commits form a poset (ancestor relation)      │
  │                                                   │
  │  3. Class Hierarchy (OOP)                         │
  │     "is subclass of" is a partial order           │
  │                                                   │
  │  4. Package Dependencies                          │
  │     "depends on" induces a partial order          │
  │                                                   │
  │  5. File System                                   │
  │     "is subdirectory of" is a partial order       │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Partial order | Reflexive + Antisymmetric + Transitive |
| Poset | Set with a partial order |
| Total / Linear order | All elements comparable |
| Chain | Totally ordered subset |
| Antichain | Set of mutually incomparable elements |
| Minimum vs. Minimal | Min ≤ all; Minimal = nothing strictly below |
| GLB (infimum) | Greatest lower bound |
| LUB (supremum) | Least upper bound |
| Well-ordered | Every non-empty subset has a minimum |

---

## ❓ Quick Revision Questions

1. **Verify that $(\mathcal{P}(\{1,2\}), \subseteq)$ is a poset. Is it a total order?**

2. **In the divisibility poset on $\{1,2,3,5,6,10,15,30\}$, find all maximal and minimal elements.**

3. **Explain the difference between "maximum" and "maximal element" with an example.**

4. **Find the GLB and LUB of $\{6, 10\}$ in $(\{1,2,3,5,6,10,15,30\}, \mid)$.**

5. **Give an example of a poset that is NOT a total order.**

6. **Is $(\mathbb{Z}, \leq)$ well-ordered? Justify your answer.**

---

[← Previous: Equivalence Relations and Partitions](03-equivalence-relations-and-partitions.md) | [Back to README](../README.md) | [Next: Hasse Diagrams →](05-hasse-diagrams.md)
