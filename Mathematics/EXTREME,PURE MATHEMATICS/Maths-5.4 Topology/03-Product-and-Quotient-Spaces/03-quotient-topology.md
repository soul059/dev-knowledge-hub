# Chapter 3.3 — Quotient Topology

[← Previous: Box Topology](02-box-topology.md) | [Back to README](../README.md) | [Next: Quotient Maps →](04-quotient-maps.md)

---

## 1. Overview

The **quotient topology** provides a way to construct new spaces by "gluing" or "identifying" parts of existing spaces. It is the topology of equivalence classes — the **finest** topology making the projection map continuous. This construction underlies surfaces, manifolds, CW-complexes, and many other objects in topology.

---

## 2. Equivalence Relations and Quotient Sets

Recall: An **equivalence relation** $\sim$ on $X$ partitions $X$ into equivalence classes $[x] = \{y \in X : y \sim x\}$. The **quotient set** is:

$$X / \!\sim\; = \{[x] : x \in X\}$$

The **canonical projection** (or quotient map) is:

$$p: X \to X / \!\sim, \quad p(x) = [x]$$

---

## 3. Definition of the Quotient Topology

> **Def 3.1 (Quotient Topology).** Let $(X, \mathcal{T})$ be a topological space and $\sim$ an equivalence relation on $X$. The **quotient topology** on $X / \!\sim$ is:
>
> $$\mathcal{T}_{X/\sim} = \{ V \subseteq X / \!\sim \;:\; p^{-1}(V) \in \mathcal{T} \}$$
>
> A set $V$ of equivalence classes is open iff the union of those classes is open in $X$.

```
Quotient topology construction:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    X                    X/~
  ┌──────────┐       ┌──────────┐
  │ • • •    │       │          │
  │ │ │ │    │  p    │  [•]     │
  │ ▼ ▼ ▼   │ ───→  │          │
  │ ●●●●●   │       │  [●]     │
  │ (class)  │       │          │
  └──────────┘       └──────────┘

  V open in X/~ ⟺ p⁻¹(V) open in X
  (= union of equiv classes is open)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> **Prop 3.2.** The quotient topology is the **finest** topology on $X / \!\sim$ making $p$ continuous.

---

## 4. Fundamental Examples

### 4.1 Circle $S^1$ from $[0,1]$

Identify $0 \sim 1$ in $[0,1]$. The quotient space $[0,1]/\{0 \sim 1\}$ is homeomorphic to $S^1$.

```
[0,1] with 0 ~ 1:

  0 ●════════════════════● 1
  │                       │
  └───────── glue ────────┘
             ↓
         ╭───────╮
        ╱    ●    ╲    ← gluing point
       │           │
       │     S¹    │
        ╲         ╱
         ╰───────╯
```

### 4.2 Torus from $[0,1]^2$

Identify $(x,0) \sim (x,1)$ and $(0,y) \sim (1,y)$:

```
Square with identifications:

    b ●━━━━━━━━━● b
      ┃  →  →  → ┃
   a  ┃          ┃ a     →  Torus T²
      ┃  →  →  → ┃
    b ●━━━━━━━━━● b

  Arrows show identification direction.
  Top ↔ Bottom (same direction) → cylinder → torus
  Left ↔ Right (same direction)
```

### 4.3 Möbius Band from $[0,1]^2$

Identify $(0,y) \sim (1, 1-y)$ (twist one side):

```
    ●━━━━━━━━━━━● 
    ┃  →  →  →  ┃
  ↑ ┃           ┃ ↓    →  Möbius band
    ┃  →  →  →  ┃
    ●━━━━━━━━━━━●

  Left and right sides identified with opposite orientation.
```

### 4.4 Real Projective Plane $\mathbb{R}P^2$

Identify antipodal points on $S^2$: $x \sim -x$.

Or from $[0,1]^2$: identify $(x,0) \sim (1-x,1)$ and $(0,y) \sim (1,1-y)$.

### 4.5 Collapsing a Subspace

Let $A \subseteq X$. Define $X/A$ by collapsing $A$ to a single point: $a_1 \sim a_2$ for all $a_1, a_2 \in A$, and each $x \notin A$ is only equivalent to itself.

**Example:** $D^2 / S^1 \cong S^2$ (collapsing the boundary of a disk gives a sphere).

```
D² / S¹ ≅ S²:

  ╭────────╮         
 ╱ ╭──────╮ ╲       ●  (collapsed boundary)
│  │  D²  │  │  →  ╱ ╲
│  │      │  │    │ S² │
 ╲ ╰──────╯ ╱      ╲ ╱
  ╰────S¹──╯        ●  
```

---

## 5. Universal Property

> **Thm 3.3 (Universal Property of Quotient).** Let $p: X \to X/\!\sim$ be the quotient map. A function $f: X/\!\sim \;\to Y$ is continuous if and only if $f \circ p: X \to Y$ is continuous.

```
Universal Property:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    X ──── f∘p ────→ Y
    │                ↑
  p │               │ f
    ▼               │
   X/~ ─ ─ ─ ─ ─ ─ ╯
        (continuous iff f∘p is)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

This says: to define a continuous function on a quotient, define it on the original space (consistently with the equivalence relation) and check continuity there.

---

## 6. When Quotients Preserve Properties

| Property of $X$ | Inherited by $X/\!\sim$? | Condition |
|-----------------|:---:|---|
| Compact | ✓ | Always (continuous image) |
| Connected | ✓ | Always (continuous image) |
| Path-connected | ✓ | Always (continuous image) |
| Hausdorff | ✗ | Only if quotient map is "nice" |
| Metrizable | ✗ | Rarely |
| Second countable | ✗ | Only under strong conditions |

> **⚠️ Warning:** The quotient of a Hausdorff space need not be Hausdorff! For example, the "line with two origins" is a quotient of two copies of $\mathbb{R}$ that is not Hausdorff.

---

## 7. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Quotient topology | $V$ open in $X/\!\sim$ iff $p^{-1}(V)$ open in $X$ |
| Finest topology | Making $p: X \to X/\!\sim$ continuous |
| Universal property | $f: X/\!\sim \to Y$ continuous $\iff$ $f \circ p$ continuous |
| $S^1 = [0,1]/\{0\sim 1\}$ | Circle from interval |
| Torus | Square with opposite edges identified (same direction) |
| Möbius band | Square with one pair of edges identified (opposite direction) |
| $X/A$ | Collapse subspace $A$ to a point |
| Hausdorff? | Not always preserved by quotients |

---

## 8. Quick Revision Questions

1. Define the quotient topology and state its universal property.

2. Describe the equivalence relation on $[0,1]^2$ that produces the torus. What if you reverse one identification?

3. Prove that $D^2 / S^1 \cong S^2$ (describe the homeomorphism).

4. Give an example showing that quotients of Hausdorff spaces need not be Hausdorff.

5. If $f: X \to Y$ is continuous and surjective, under what conditions is $Y$ homeomorphic to $X / \!\sim$ where $x_1 \sim x_2 \iff f(x_1) = f(x_2)$?

6. Explain the construction of $\mathbb{R}P^2$ as a quotient of $S^2$.

---

[← Previous: Box Topology](02-box-topology.md) | [Back to README](../README.md) | [Next: Quotient Maps →](04-quotient-maps.md)
