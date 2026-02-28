# Chapter 3.4 — Quotient Maps

[← Previous: Quotient Topology](03-quotient-topology.md) | [Back to README](../README.md) | [Next: Connected Spaces →](../04-Connectedness/01-connected-spaces.md)

---

## 1. Overview

A **quotient map** is the abstract version of the canonical projection $p: X \to X/\!\sim$. Many naturally arising maps (projections, identifications, group actions) are quotient maps. Understanding their properties is essential for working with quotient spaces.

---

## 2. Definition

> **Def 4.1 (Quotient Map).** A surjective function $p: X \to Y$ is a **quotient map** if:
>
> $$V \text{ is open in } Y \iff p^{-1}(V) \text{ is open in } X$$

Equivalently: $Y$ carries the **quotient topology** with respect to $p$ and the equivalence relation $x_1 \sim x_2 \iff p(x_1) = p(x_2)$.

> **Note:** Every quotient map is continuous (the "$\Leftarrow$" direction). The "$\Rightarrow$" direction (called the **saturation condition**) is what makes it more than just continuous.

---

## 3. Sufficient Conditions for Being a Quotient Map

> **Prop 4.2.** A continuous surjection $p: X \to Y$ is a quotient map if any of the following hold:
>
> 1. $p$ is an **open** map.
> 2. $p$ is a **closed** map.
> 3. $p$ is a continuous surjection from a **compact** space to a **Hausdorff** space.

```
Sufficient conditions diagram:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Continuous + surjective + open   ⟹ quotient map
  Continuous + surjective + closed ⟹ quotient map

  BUT:
  quotient map ⟹̸ open
  quotient map ⟹̸ closed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Proof of (3):** Since $X$ is compact and $Y$ is Hausdorff, every continuous map $X \to Y$ is closed (compact subsets of Hausdorff spaces are closed, and continuous images of compact sets are compact). So $p$ is a closed continuous surjection, hence a quotient map. ∎

---

## 4. Saturated Sets

> **Def 4.3 (Saturated).** A subset $A \subseteq X$ is **saturated** (with respect to $p: X \to Y$) if $A = p^{-1}(p(A))$, i.e., $A$ is a union of fibers of $p$.

> **Prop 4.4.** $p: X \to Y$ is a quotient map if and only if $p$ is a continuous surjection and: a saturated set $A$ is open in $X$ $\Rightarrow$ $p(A)$ is open in $Y$.

This reformulation shows we only need the "open" condition for saturated sets, not all sets.

```
Saturated vs non-saturated:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  X = {a, b, c, d, e}     p(a)=p(b)=1, p(c)=p(d)=2, p(e)=3

  Saturated: {a,b}      = p⁻¹({1})     ✓
             {a,b,c,d}  = p⁻¹({1,2})   ✓
             {e}         = p⁻¹({3})     ✓

  NOT saturated: {a,c}   ≠ p⁻¹(p({a,c})) = p⁻¹({1,2}) = {a,b,c,d}  ✗
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 5. Composing and Restricting Quotient Maps

> **Prop 4.5.** The composition of two quotient maps is a quotient map.

> **⚠️ Warning:** Restricting a quotient map to a subspace does **not** in general yield a quotient map.

> **Prop 4.6.** If $p: X \to Y$ is a quotient map and $A \subseteq Y$, then $p|_{p^{-1}(A)}: p^{-1}(A) \to A$ is a quotient map provided $A$ is **open** or **closed** in $Y$ (and both $p^{-1}(A)$, $A$ carry subspace topologies).

---

## 6. Quotient Maps vs Homeomorphisms

> **Prop 4.7.** A quotient map $p: X \to Y$ is a homeomorphism if and only if $p$ is **injective** (hence bijective).

So: quotient map = homeomorphism + possible identification of points.

---

## 7. Constructing Maps Out of Quotient Spaces

> **Thm 4.8.** Let $p: X \to X/\!\sim$ be a quotient map and $g: X \to Z$ be continuous with $g$ constant on each fiber ($x \sim y \Rightarrow g(x) = g(y)$). Then the induced map $\bar{g}: X/\!\sim \;\to Z$ is continuous. Moreover:
>
> - $\bar{g}$ is a quotient map if $g$ is a quotient map.
> - $\bar{g}$ is a homeomorphism if $g$ is a quotient map and $g$ separates fibers (injective on classes).

```
Induced map from quotient:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    X ──── g ────→ Z
    │              ↑
  p │             │ g̅ (induced)
    ▼             │
   X/~ ─ ─ ─ ─ ─╯

  g̅([x]) = g(x)  (well-defined since g
                   is constant on fibers)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 8. Examples of Quotient Maps

### 8.1 The Canonical Projection

$p: X \to X/\!\sim$ is always a quotient map (by definition of the quotient topology).

### 8.2 $\exp: \mathbb{R} \to S^1$

$\exp(t) = e^{2\pi i t}$. This is a continuous surjection, and it is an open map (images of open intervals are open arcs). Hence it is a quotient map. The fibers are $\{t + n : n \in \mathbb{Z}\}$, so $S^1 \cong \mathbb{R}/\mathbb{Z}$.

### 8.3 Matrix Groups

$\det: \text{GL}(n, \mathbb{R}) \to \mathbb{R}^*$ is a continuous surjection and an open map, hence a quotient map. The fiber over $1$ is $\text{SL}(n, \mathbb{R})$.

### 8.4 Collapsing a Subspace

$p: X \to X/A$ collapsing $A$ to a point. This is a quotient map. It is closed if $A$ is closed in $X$.

---

## 9. Quotient Maps and Group Actions

If a group $G$ acts on $X$ (continuously), the orbit space $X/G$ carries the quotient topology via $p: X \to X/G$, $p(x) = Gx$ (the orbit of $x$).

**Examples:**
- $S^1 = \mathbb{R}/\mathbb{Z}$
- $\mathbb{R}P^n = S^n / \{x \sim -x\} = S^n / \mathbb{Z}_2$
- Torus $T^2 = \mathbb{R}^2 / \mathbb{Z}^2$
- Klein bottle = $\mathbb{R}^2 / G$ for an appropriate group $G$

---

## 10. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Quotient map | Surjective; $V$ open $\iff$ $p^{-1}(V)$ open |
| Saturated set | $A = p^{-1}(p(A))$; union of fibers |
| Sufficient: open surjection | Continuous + surjective + open ⟹ quotient |
| Sufficient: closed surjection | Continuous + surjective + closed ⟹ quotient |
| Sufficient: compact→Hausdorff | Automatic by compactness arguments |
| Induced map | Well-defined and continuous if constant on fibers |
| Quotient + injective | = homeomorphism |
| Group action quotient | $X/G$ with quotient topology |

---

## 11. Quick Revision Questions

1. Define quotient map and explain how it differs from mere continuous surjection.

2. Give an example of a continuous surjection that is **not** a quotient map.

3. Define "saturated set" and explain its role in the characterization of quotient maps.

4. Prove that a continuous bijection from a compact space to a Hausdorff space is a homeomorphism (hence a quotient map).

5. Describe the quotient map $\mathbb{R} \to S^1$ via $t \mapsto e^{2\pi i t}$ and identify the fibers.

6. Under what conditions does a quotient map restrict to a quotient map on a subspace?

---

[← Previous: Quotient Topology](03-quotient-topology.md) | [Back to README](../README.md) | [Next: Connected Spaces →](../04-Connectedness/01-connected-spaces.md)
