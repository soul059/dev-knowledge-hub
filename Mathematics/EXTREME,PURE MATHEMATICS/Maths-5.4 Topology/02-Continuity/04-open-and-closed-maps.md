# Chapter 2.4 — Open and Closed Maps

[← Previous: Topological Properties](03-topological-properties.md) | [Back to README](../README.md) | [Next: Product Topology →](../03-Product-and-Quotient-Spaces/01-product-topology.md)

---

## 1. Overview

While continuity is about preimages of open sets, **open** and **closed maps** are about **images** of open (resp. closed) sets. These notions are distinct from continuity and play crucial roles in product topology (projections are open maps) and quotient topology (quotient maps are often not open nor closed).

---

## 2. Definitions

> **Def 4.1 (Open Map).** $f: X \to Y$ is an **open map** if for every open $U \subseteq X$, the image $f(U)$ is open in $Y$.

> **Def 4.2 (Closed Map).** $f: X \to Y$ is a **closed map** if for every closed $C \subseteq X$, the image $f(C)$ is closed in $Y$.

```
Comparison:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  CONTINUOUS:     V open in Y ⟹ f⁻¹(V) open in X
                  (preimages preserve openness)

  OPEN MAP:       U open in X ⟹ f(U) open in Y
                  (images preserve openness)

  CLOSED MAP:     C closed in X ⟹ f(C) closed in Y
                  (images preserve closedness)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> **⚠️ Warning:** These three notions are **independent**:
> - A function can be continuous but not open/closed.
> - A function can be open but not continuous.
> - A function can be open but not closed (and vice versa).

---

## 3. Relationships

```
Independence Diagram:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         Continuous
           │
           │  (independent)
           │
    Open ──┼── Closed
           │
           │
      Homeomorphism
      = continuous +
        bijective +
        open (or closed)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> **Prop 4.3.** A continuous bijection $f: X \to Y$ is a homeomorphism if and only if $f$ is an **open map** (equivalently, a **closed map**).

**Proof:** $f$ homeo $\iff$ $f^{-1}$ continuous $\iff$ $(f^{-1})^{-1}(U) = f(U)$ open for all $U$ open $\iff$ $f$ is open. The closed case follows by taking complements. ∎

---

## 4. Examples

### 4.1 Projection Maps (Open, Not Closed)

$\pi_1: \mathbb{R}^2 \to \mathbb{R}$, $\pi_1(x,y) = x$.

- **Open:** If $U \subseteq \mathbb{R}^2$ is open and $(a,b) \in U$, there's a basic open set $(a-\epsilon, a+\epsilon) \times (b-\epsilon, b+\epsilon) \subseteq U$. Its projection $(a-\epsilon, a+\epsilon)$ is open. ✓
- **Not closed:** The hyperbola $C = \{(x, 1/x) : x > 0\}$ is closed in $\mathbb{R}^2$, but $\pi_1(C) = (0, \infty)$, which is not closed in $\mathbb{R}$. ✗

```
Projection π₁ not closed:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  y
  │   ╲  hyperbola xy = 1
  │    ╲
  │     ╲___________
  │                  ──────→
  └──────────────────────── x
  0         (0, ∞)  ← not closed!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 4.2 Inclusion of a Closed Subspace (Closed, Not Open)

$\iota: [0,1] \hookrightarrow \mathbb{R}$. Closed subsets of $[0,1]$ are closed in $\mathbb{R}$ (closed subset of closed set is closed). But $[0,1]$ itself is not open in $\mathbb{R}$, so $\iota$ is not an open map.

### 4.3 Continuous, Not Open, Not Closed

$f: \mathbb{R} \to \mathbb{R}$, $f(x) = x^2$.
- Continuous: Yes.
- Open: $f((-1, 1)) = [0, 1)$, not open. ✗
- Closed: $f(\mathbb{Z}) = \{0, 1, 4, 9, \ldots\}$ is closed. Not immediately a counterexample. But $f(\{x : x \leq -1\}) = [1, \infty)$ which is closed. Actually, one can show $f$ **is** closed on $\mathbb{R}$ (proper maps are closed). But $f$ is **not** open. ✗$_{\text{open}}$ ✓$_{\text{closed}}$

### 4.4 Open Map That Is Not Continuous

Define $f: \mathbb{R} \to \mathbb{R}$ where $\mathbb{R}$ has the standard topology in the domain and the indiscrete topology in the codomain. Then $f$ maps any open set to a subset of $\mathbb{R}$, which is open in the indiscrete sense only if it is $\emptyset$ or $\mathbb{R}$. So this is subtle — easier example: identity $\text{id}: (\mathbb{R}, \mathcal{T}_{\text{std}}) \to (\mathbb{R}, \mathcal{T}_\ell)$ is open (standard open ⊆ Sorgenfrey open since $\mathcal{T}_{\text{std}} \subseteq \mathcal{T}_\ell$) but not continuous ($[0,1)$ is Sorgenfrey-open, but its preimage under id is $[0,1)$, which is not standard-open).

---

## 5. Open Maps and Basis Elements

> **Prop 4.4.** $f: X \to Y$ is open if and only if $f(B)$ is open in $Y$ for every basis element $B$ of $X$.

**Proof:** Every open $U = \bigcup B_\alpha$, so $f(U) = \bigcup f(B_\alpha)$, a union of opens. ∎

---

## 6. Closed Maps and Proper Maps

> **Def 4.5 (Proper Map).** $f: X \to Y$ is **proper** if $f^{-1}(K)$ is compact for every compact $K \subseteq Y$.

> **Thm 4.6.** If $f: X \to Y$ is continuous and proper, and $Y$ is Hausdorff, then $f$ is a closed map.

This connects compactness with closedness of maps — a very useful result.

---

## 7. Characterizations of Quotient Maps

> **Def 4.7 (Quotient Map).** A surjection $p: X \to Y$ is a **quotient map** if $V$ is open in $Y$ $\iff$ $p^{-1}(V)$ is open in $X$.

A quotient map is:
- Always continuous (preimage of open is open).
- Always surjective.
- **Not** necessarily open or closed.

```
Quotient map relationships:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Continuous surjection + open ⟹ quotient map
  Continuous surjection + closed ⟹ quotient map
  Quotient map ⟹ continuous surjection
  Quotient map ⟹̸ open
  Quotient map ⟹̸ closed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 8. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Open map | Images of open sets are open |
| Closed map | Images of closed sets are closed |
| Continuous | Preimages of open sets are open |
| Independence | These three properties are independent |
| Homeomorphism | = continuous bijection + open (or closed) |
| Projection $\pi_i$ | Open but not closed |
| Proper map | $f^{-1}(\text{compact}) = \text{compact}$; yields closed maps |
| Quotient map | Surjective; $V$ open $\iff$ $p^{-1}(V)$ open |
| Basis check for open | Suffices to check images of basis elements |

---

## 9. Quick Revision Questions

1. Give an example of a function that is continuous but neither open nor closed.

2. Prove: if $f$ is a continuous bijection and an open map, then $f$ is a homeomorphism.

3. Show that the projection $\pi_1: \mathbb{R}^2 \to \mathbb{R}$ is open. Then find a closed set whose projection is not closed.

4. Is a quotient map always an open map? Justify.

5. State the relationship between proper maps and closed maps.

6. Let $f: (X, \mathcal{T}_{\text{discrete}}) \to (Y, \mathcal{T})$ be any function. Is $f$ necessarily an open map? What conditions on $\mathcal{T}$ would ensure it?

---

[← Previous: Topological Properties](03-topological-properties.md) | [Back to README](../README.md) | [Next: Product Topology →](../03-Product-and-Quotient-Spaces/01-product-topology.md)
