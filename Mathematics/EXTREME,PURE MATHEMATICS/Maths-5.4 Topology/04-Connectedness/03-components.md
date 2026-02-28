# Chapter 4.3 — Components

[← Previous: Path-Connected Spaces](02-path-connected-spaces.md) | [Back to README](../README.md) | [Next: Local Connectedness →](04-local-connectedness.md)

---

## 1. Overview

Even when a space is not connected, it can be decomposed into maximal connected pieces called **connected components**. Similarly, **path components** are the maximal path-connected pieces. This chapter studies these decompositions.

---

## 2. Connected Components

> **Def 3.1 (Connected Component).** The **connected component** of $x \in X$ is:
>
> $$C(x) = \bigcup \{ A \subseteq X : A \text{ is connected and } x \in A \}$$
>
> It is the **largest connected subset** containing $x$.

> **Prop 3.2.** Connected components have the following properties:
> 1. $C(x)$ is connected (union of connected sets sharing a point).
> 2. $C(x)$ is **closed** (closure of connected = connected, so $\overline{C(x)}$ is connected; maximality forces $C(x) = \overline{C(x)}$).
> 3. Connected components **partition** $X$: either $C(x) = C(y)$ or $C(x) \cap C(y) = \emptyset$.
> 4. $X$ is connected iff it has exactly **one** component.

> **⚠️ Warning:** Components need not be **open**!

```
Components partition X:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ┌───────┐  ┌──────────┐  ┌────┐
  │  C₁   │  │    C₂    │  │ C₃ │
  │       │  │          │  │    │
  │       │  │          │  │    │
  └───────┘  └──────────┘  └────┘

  X = C₁ ⊔ C₂ ⊔ C₃   (disjoint union)
  Each Cᵢ connected, closed, maximal
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Path Components

> **Def 3.3 (Path Component).** The **path component** of $x$ is:
>
> $$P(x) = \{ y \in X : \text{there is a path from } x \text{ to } y \}$$

> **Prop 3.4.**
> 1. "Reachable by a path" is an equivalence relation (reflexive by constant path, symmetric by reverse, transitive by concatenation).
> 2. Path components partition $X$.
> 3. Each path component is path-connected (maximal).
> 4. Each path component is contained in a connected component: $P(x) \subseteq C(x)$.
> 5. Path components need not be open or closed.

---

## 4. Components vs Path Components

| Property | Connected Components | Path Components |
|----------|:---:|:---:|
| Always closed | ✓ | ✗ |
| Always open | ✗ (open iff locally connected) | ✗ (open iff locally path-connected) |
| Partition $X$ | ✓ | ✓ |
| Contained in the other | Path ⊆ Connected | Connected ⊇ Path |
| Equal when... | $X$ locally path-connected | $X$ locally path-connected |

---

## 5. Examples

### 5.1 $\mathbb{Q}$ (Standard Topology)

Connected components = **singletons** $\{q\}$ for each $q \in \mathbb{Q}$.

Path components = singletons.

$\mathbb{Q}$ is **totally disconnected**: every component is a single point.

### 5.2 $\mathbb{R} \setminus \{0\}$

Two connected (and path) components: $(-\infty, 0)$ and $(0, \infty)$.

### 5.3 The Topologist's Sine Curve

- **One connected component** (the whole space — it is connected).
- **Two path components:** the oscillating curve $\{(x, \sin(1/x)) : x > 0\}$ and the vertical segment $\{0\} \times [-1,1]$.

This shows that a connected space can have **multiple path components**.

### 5.4 The Cantor Set

Totally disconnected: every connected component is a singleton. Yet it is uncountable and compact.

---

## 6. Totally Disconnected Spaces

> **Def 3.5 (Totally Disconnected).** $X$ is **totally disconnected** if every connected component is a single point.

**Examples:** $\mathbb{Q}$, $\mathbb{Z}$, the Cantor set, any discrete space, $\mathbb{R} \setminus \mathbb{Q}$.

> **Prop 3.6.** Every discrete space is totally disconnected, but not every totally disconnected space is discrete ($\mathbb{Q}$ is not discrete).

---

## 7. Quasi-components

> **Def 3.7 (Quasi-component).** The **quasi-component** of $x$ is:
>
> $$Q(x) = \bigcap \{ A : A \text{ is clopen and } x \in A \}$$

> **Prop 3.8.**
> 1. $C(x) \subseteq Q(x)$ always.
> 2. In **locally connected** spaces: $C(x) = Q(x)$.
> 3. In general: $C(x) \subsetneq Q(x)$ is possible.

Quasi-components are relevant in spaces that are not locally connected.

---

## 8. Number of Components as an Invariant

> **Prop 3.9.** If $f: X \to Y$ is a homeomorphism, then $X$ and $Y$ have the same number of connected components (and path components).

This is why "$\mathbb{R} \setminus \{0\}$ has 2 components but $\mathbb{R}^2 \setminus \{(0,0)\}$ has 1" proves $\mathbb{R} \not\cong \mathbb{R}^2$.

---

## 9. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Connected component $C(x)$ | Maximal connected set containing $x$ |
| Path component $P(x)$ | Maximal path-connected set containing $x$ |
| Both partition $X$ | Into disjoint pieces |
| $P(x) \subseteq C(x)$ | Always |
| $P(x) = C(x)$ | When $X$ is locally path-connected |
| Components are closed | Always; but not always open |
| Totally disconnected | All components are singletons |
| Quasi-component | Intersection of all clopen sets containing $x$ |
| Number of components | Topological invariant |

---

## 10. Quick Revision Questions

1. Define connected component and prove it is closed.

2. Give an example where the connected component of a point is not open.

3. Show that the topologist's sine curve has one connected component but two path components.

4. Define "totally disconnected" and give three examples.

5. What is a quasi-component? When does it equal the connected component?

6. Prove that the number of connected components is a topological invariant.

---

[← Previous: Path-Connected Spaces](02-path-connected-spaces.md) | [Back to README](../README.md) | [Next: Local Connectedness →](04-local-connectedness.md)
