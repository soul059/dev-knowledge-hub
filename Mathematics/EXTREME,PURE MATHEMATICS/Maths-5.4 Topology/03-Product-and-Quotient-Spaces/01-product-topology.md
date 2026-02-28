# Chapter 3.1 — Product Topology

[← Previous: Open and Closed Maps](../02-Continuity/04-open-and-closed-maps.md) | [Back to README](../README.md) | [Next: Box Topology →](02-box-topology.md)

---

## 1. Overview

The **product topology** is the "correct" way to topologize a Cartesian product of spaces. It is defined as the **coarsest** topology making all projection maps continuous. For finite products it coincides with the intuitive notion; for infinite products it is subtler than the naïve box topology.

---

## 2. Finite Products

### 2.1 Definition for Two Spaces

> **Def 1.1.** Let $(X, \mathcal{T}_X)$ and $(Y, \mathcal{T}_Y)$ be topological spaces. The **product topology** on $X \times Y$ is the topology with basis:
>
> $$\mathcal{B} = \{ U \times V : U \in \mathcal{T}_X,\; V \in \mathcal{T}_Y \}$$

Each $U \times V$ is called a **basic open rectangle**.

```
Product topology on X × Y:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Y
  ▲
  │  ┌─────────────┐
  │  │             │ ← V (open in Y)
  │  │  U × V     │
  │  │  (basic     │
  │  │   open set) │
  │  └─────────────┘
  │  ├─── U ───────┤
  └──────────────────────→ X
         (open in X)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 2.2 Why This Is a Basis

**(B1):** $X \times Y = X \times Y$ is itself a basic open set ($X \in \mathcal{T}_X$, $Y \in \mathcal{T}_Y$). ✓

**(B2):** $(U_1 \times V_1) \cap (U_2 \times V_2) = (U_1 \cap U_2) \times (V_1 \cap V_2)$, which is again a basic open set. ✓

---

## 3. Projection Maps

> **Def 1.2.** The **projection maps** are:
> $$\pi_X: X \times Y \to X, \quad \pi_X(x,y) = x$$
> $$\pi_Y: X \times Y \to Y, \quad \pi_Y(x,y) = y$$

> **Prop 1.3.** The projections $\pi_X, \pi_Y$ are:
> 1. **Continuous** (by design — the product topology makes them so).
> 2. **Open maps** (images of basic open sets are open).
> 3. **Surjective** (if both spaces are nonempty).

**Proof of continuity:** $\pi_X^{-1}(U) = U \times Y$, which is open in $X \times Y$. ✓

**Proof of openness:** $\pi_X(U \times V) = U$ (if $V \neq \emptyset$), which is open. ✓

---

## 4. Universal Property

> **Thm 1.4 (Universal Property of Product).** A function $f: Z \to X \times Y$ is continuous if and only if both coordinate functions $\pi_X \circ f: Z \to X$ and $\pi_Y \circ f: Z \to Y$ are continuous.

```
Universal Property Diagram:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
            Z
           ╱│╲
    π_X∘f ╱ │ ╲ π_Y∘f
        ╱   │f  ╲
       ╱    │    ╲
      ▼     ▼     ▼
      X ← X×Y →  Y
        π_X    π_Y

  f continuous ⟺ π_X∘f and π_Y∘f continuous
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

This property **characterizes** the product topology: it is the **unique** topology on $X \times Y$ satisfying this universal property (coarsest making projections continuous).

---

## 5. Subbasis Description

The product topology has as **subbasis**:

$$\mathcal{S} = \{\pi_X^{-1}(U) : U \in \mathcal{T}_X\} \cup \{\pi_Y^{-1}(V) : V \in \mathcal{T}_Y\}$$

Since $\pi_X^{-1}(U) = U \times Y$ and $\pi_Y^{-1}(V) = X \times V$, finite intersections give:

$$(U \times Y) \cap (X \times V) = U \times V$$

which are exactly the basis elements.

---

## 6. General (Arbitrary) Products

### 6.1 Cartesian Product

Given a family $\{X_\alpha\}_{\alpha \in A}$:

$$\prod_{\alpha \in A} X_\alpha = \left\{ f: A \to \bigcup_\alpha X_\alpha \;\Big|\; f(\alpha) \in X_\alpha \;\forall\, \alpha \right\}$$

A point in the product is a **choice function** selecting one element from each $X_\alpha$.

### 6.2 Product Topology

> **Def 1.5 (Product Topology).** The **product topology** on $\prod_\alpha X_\alpha$ has subbasis:
>
> $$\mathcal{S} = \left\{ \pi_\alpha^{-1}(U_\alpha) : \alpha \in A,\; U_\alpha \in \mathcal{T}_\alpha \right\}$$
>
> where $\pi_\alpha: \prod_\beta X_\beta \to X_\alpha$ is the $\alpha$-th projection.

**Basis elements** are **finite intersections** of subbasis elements:

$$\bigcap_{i=1}^{n} \pi_{\alpha_i}^{-1}(U_{\alpha_i}) = \prod_{\alpha \in A} V_\alpha$$

where $V_{\alpha_i} = U_{\alpha_i}$ for $i = 1,\ldots,n$ and $V_\alpha = X_\alpha$ for all other $\alpha$.

> **✦ Key Insight:** In a basic open set of the product topology, only **finitely many** coordinates are "constrained" — all others are the full space $X_\alpha$. This is the crucial difference from the box topology.

```
Product topology basis element (3 factors, constraint on α₁ only):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  π_{α₁}⁻¹(U_{α₁}) = U_{α₁} × X_{α₂} × X_{α₃} × ...
                       ╰─┬──╯   ╰──┬──╯   ╰──┬──╯
                     constrained   full      full
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 7. Properties of Product Spaces

| Property | Preserved by finite products? | Preserved by arbitrary products? |
|----------|:---:|:---:|
| Hausdorff | ✓ | ✓ |
| Compact | ✓ | ✓ (Tychonoff's theorem!) |
| Connected | ✓ | ✓ |
| Path-connected | ✓ | ✓ |
| First countable | ✓ | ✗ (uncountable product of non-trivial spaces) |
| Second countable | ✓ | ✗ |
| Metrizable | ✓ | ✗ (in general) |
| T₁ | ✓ | ✓ |

---

## 8. Convergence in Product Spaces

> **Thm 1.6.** A net (or sequence, if applicable) $(x_\lambda)$ in $\prod_\alpha X_\alpha$ converges to $x$ in the product topology if and only if $\pi_\alpha(x_\lambda) \to \pi_\alpha(x)$ for every $\alpha \in A$.

This is **coordinate-wise convergence**.

---

## 9. Examples

### 9.1 $\mathbb{R}^n$ with Standard Topology

$\mathbb{R}^n = \mathbb{R} \times \cdots \times \mathbb{R}$ ($n$ times). The product topology coincides with the topology from the Euclidean metric. Basis: open rectangles $(a_1,b_1) \times \cdots \times (a_n,b_n)$.

### 9.2 $\mathbb{R}^\omega$ (Countable Product)

$\mathbb{R}^\omega = \prod_{n=1}^{\infty} \mathbb{R}$. A basic open set constrains only finitely many coordinates. This is the space of real sequences with the topology of coordinate-wise convergence.

### 9.3 $\{0,1\}^A$ (Product of Discrete Two-Point Spaces)

For $A = \mathbb{N}$: this is homeomorphic to the **Cantor set**! The product topology has basis elements that fix finitely many coordinates.

---

## 10. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Product topology | Coarsest topology making all projections continuous |
| Basis (finite) | $\{U_1 \times \cdots \times U_n\}$ with each $U_i$ open |
| Subbasis (general) | $\{\pi_\alpha^{-1}(U_\alpha)\}$ |
| Basis (general) | Finite intersections of subbasis elements; constrain finitely many coords |
| Universal property | $f: Z \to \prod X_\alpha$ continuous $\iff$ each $\pi_\alpha \circ f$ continuous |
| Projections | Continuous, open, surjective |
| Convergence | Coordinate-wise |
| Key property | Tychonoff: arbitrary product of compacts is compact |

---

## 11. Quick Revision Questions

1. State the universal property of the product topology and explain why it characterizes the topology uniquely.

2. Describe a basis element for the product topology on $\mathbb{R}^\omega = \prod_{n=1}^\infty \mathbb{R}$. How many coordinates are constrained?

3. Prove that $\pi_X: X \times Y \to X$ is an open map.

4. Show that a function $f: Z \to X \times Y$ is continuous iff both $\pi_X \circ f$ and $\pi_Y \circ f$ are continuous.

5. Why is the product topology on $\mathbb{R}^n$ the same as the standard (Euclidean metric) topology?

6. Is the product of two metrizable spaces metrizable? What about an uncountable product?

---

[← Previous: Open and Closed Maps](../02-Continuity/04-open-and-closed-maps.md) | [Back to README](../README.md) | [Next: Box Topology →](02-box-topology.md)
