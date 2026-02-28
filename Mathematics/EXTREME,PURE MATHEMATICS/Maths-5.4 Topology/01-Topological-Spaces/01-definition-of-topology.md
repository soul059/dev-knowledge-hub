# Chapter 1.1 — Definition of Topology

[← Back to README](../README.md) | [Next: Open and Closed Sets →](02-open-and-closed-sets.md)

---

## 1. Overview

A **topology** on a set is the minimal structure needed to talk about continuity, convergence, and connectedness — without any notion of distance. This chapter introduces the axioms that define a topology and builds intuition for what "open set" means in this abstract setting.

---

## 2. Motivation: From Metric Spaces to Topologies

In analysis on $\mathbb{R}$, the key notion is the **open set**: a set $U \subseteq \mathbb{R}$ is open if every point in $U$ has an open interval around it still contained in $U$.

We observe three fundamental properties of open sets in $\mathbb{R}$:

1. $\emptyset$ and $\mathbb{R}$ are open.
2. Any **union** of open sets is open.
3. Any **finite intersection** of open sets is open.

**✦ Key Insight:** Topology abstracts exactly these three properties, discarding everything else (distance, angles, coordinates).

```
Conceptual Diagram: Abstraction Ladder
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Euclidean Geometry   (angles, distances, rigid motions)
        │
        ▼
  Metric Spaces        (distances only)
        │
        ▼
  Topological Spaces   (open sets only)   ◄── WE ARE HERE
        │
        ▼
  Set Theory           (just sets)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Definition of a Topology

> **Def 1.1 (Topology).** Let $X$ be a set. A **topology** on $X$ is a collection $\mathcal{T} \subseteq \mathcal{P}(X)$ (where $\mathcal{P}(X)$ is the power set of $X$) satisfying:
>
> **(T1)** $\emptyset \in \mathcal{T}$ and $X \in \mathcal{T}$.
>
> **(T2)** If $\{U_\alpha\}_{\alpha \in A}$ is any (possibly infinite) family of sets in $\mathcal{T}$, then $\bigcup_{\alpha \in A} U_\alpha \in \mathcal{T}$.
>
> **(T3)** If $U_1, U_2, \ldots, U_n \in \mathcal{T}$ (finitely many), then $\bigcap_{i=1}^{n} U_i \in \mathcal{T}$.

The pair $(X, \mathcal{T})$ is called a **topological space**. Members of $\mathcal{T}$ are called **open sets**.

---

### 3.1 Why Only Finite Intersections?

Consider $\mathbb{R}$ with the standard topology:

$$\bigcap_{n=1}^{\infty} \left(-\frac{1}{n},\, \frac{1}{n}\right) = \{0\}$$

The singleton $\{0\}$ is **not** open in $\mathbb{R}$. So allowing arbitrary intersections would collapse the topology.

> **⚠️ Warning:** Axiom (T3) requires only **finite** intersections. Infinite intersections of open sets need not be open.

---

## 4. The Axioms in Set-Operation Form

```
┌──────────────────────────────────────────────────────┐
│               TOPOLOGY AXIOMS ON SET X               │
│                                                      │
│   𝒯 ⊆ 𝒫(X)  satisfying:                            │
│                                                      │
│   (T1)  ∅ ∈ 𝒯,   X ∈ 𝒯                             │
│                                                      │
│   (T2)  { Uα } ⊆ 𝒯  ⟹  ⋃ Uα ∈ 𝒯                  │
│           any family        α                        │
│                                                      │
│   (T3)  U₁,…,Uₙ ∈ 𝒯  ⟹  U₁ ∩ ··· ∩ Uₙ ∈ 𝒯       │
│           finite                                     │
└──────────────────────────────────────────────────────┘
```

---

## 5. First Examples

### Example 1: Indiscrete (Trivial) Topology

$$\mathcal{T}_{\text{indiscrete}} = \{\emptyset, X\}$$

This is the **smallest** possible topology on $X$. Only two sets are open.

### Example 2: Discrete Topology

$$\mathcal{T}_{\text{discrete}} = \mathcal{P}(X)$$

This is the **largest** possible topology on $X$. Every subset is open.

### Example 3: Concrete Small Set

Let $X = \{a, b, c\}$. Consider $\mathcal{T} = \{\emptyset, \{a\}, \{a, b\}, X\}$.

**Verification:**
| Axiom | Check | Result |
|-------|-------|--------|
| (T1) | $\emptyset \in \mathcal{T}$? $X \in \mathcal{T}$? | ✓ |
| (T2) | All unions of members in $\mathcal{T}$? $\{a\} \cup \{a,b\} = \{a,b\} \in \mathcal{T}$, etc. | ✓ |
| (T3) | All finite intersections? $\{a\} \cap \{a,b\} = \{a\} \in \mathcal{T}$, etc. | ✓ |

So $\mathcal{T}$ is indeed a topology on $X$.

### Example 4: Non-Topology

Let $X = \{a,b,c\}$, and consider $\mathcal{S} = \{\emptyset, \{a\}, \{b\}, X\}$.

Check: $\{a\} \cup \{b\} = \{a,b\} \notin \mathcal{S}$. **Fails (T2).** Not a topology.

---

## 6. Counting Topologies on Finite Sets

The number of distinct topologies on a set of $n$ elements grows extremely rapidly:

| $n$ | \# of topologies |
|-----|-----------------|
| 0 | 1 |
| 1 | 1 |
| 2 | 4 |
| 3 | 29 |
| 4 | 355 |
| 5 | 6,942 |

```
All topologies on X = {a, b}:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 𝒯₁ = {∅, {a,b}}                (indiscrete)
 𝒯₂ = {∅, {a}, {a,b}}          (Sierpiński-type)
 𝒯₃ = {∅, {b}, {a,b}}          (Sierpiński-type)
 𝒯₄ = {∅, {a}, {b}, {a,b}}     (discrete)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 7. Neighborhoods

> **Def 1.2 (Neighborhood).** Let $(X, \mathcal{T})$ be a topological space and $x \in X$. A set $N \subseteq X$ is a **neighborhood** of $x$ if there exists an open set $U \in \mathcal{T}$ with $x \in U \subseteq N$.

Note: A neighborhood need **not** be open itself — it just needs to contain an open set around the point.

```
ASCII Diagram: Neighborhood of x
━━━━━━━━━━━━━━━━━━━━━━━━━━━
      ┌───────────── N ──────────────┐
      │                              │
      │     ╭─────── U ───────╮      │
      │     │                 │      │
      │     │       • x      │      │
      │     │                 │      │
      │     ╰─────────────────╯      │
      │        (open set U)          │
      └──────────────────────────────┘
         (neighborhood N of x)
━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 8. Equivalent Formulation via Neighborhoods

One can alternatively define a topology by specifying a **neighborhood system** $\mathcal{N}(x)$ for each $x \in X$ satisfying:

1. $x \in N$ for every $N \in \mathcal{N}(x)$.
2. If $N \in \mathcal{N}(x)$ and $N \subseteq M$, then $M \in \mathcal{N}(x)$.
3. If $N_1, N_2 \in \mathcal{N}(x)$, then $N_1 \cap N_2 \in \mathcal{N}(x)$.
4. For each $N \in \mathcal{N}(x)$, there exists $M \in \mathcal{N}(x)$ with $M \subseteq N$ and $M \in \mathcal{N}(y)$ for all $y \in M$.

Then $U$ is declared open $\iff$ $U \in \mathcal{N}(x)$ for every $x \in U$.

---

## 9. Topology as a Lattice

The collection of all topologies on $X$ forms a **complete lattice** under inclusion $\subseteq$:

- **Meet (infimum):** $\mathcal{T}_1 \wedge \mathcal{T}_2 = \mathcal{T}_1 \cap \mathcal{T}_2$ (always a topology).
- **Join (supremum):** $\mathcal{T}_1 \vee \mathcal{T}_2$ = smallest topology containing $\mathcal{T}_1 \cup \mathcal{T}_2$ (need to close under unions and finite intersections).
- **Bottom:** Indiscrete topology.
- **Top:** Discrete topology.

```
Lattice of topologies on {a, b}:

           {∅,{a},{b},{a,b}}      ← Discrete (top)
              /          \
   {∅,{a},{a,b}}    {∅,{b},{a,b}}
              \          /
           {∅, {a,b}}            ← Indiscrete (bottom)
```

---

## 10. Real-World Applications

| Application | How Topology Appears |
|-------------|---------------------|
| **Data Science** | Topological Data Analysis (TDA) uses persistent homology to extract shape features from point clouds |
| **Robotics** | Configuration spaces of robots are topological spaces; connectivity determines reachability |
| **Physics** | Phase spaces and spacetime are modeled as topological manifolds |
| **Computer Science** | Scott topology on domains in denotational semantics; digital topology in image processing |
| **Network Theory** | Network topology describes the arrangement of nodes and links abstractly |

---

## 11. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Topology $\mathcal{T}$ | Collection of subsets of $X$ closed under ∅, $X$, arbitrary unions, finite intersections |
| Topological space | Pair $(X, \mathcal{T})$ |
| Open set | Any member of $\mathcal{T}$ |
| Discrete topology | $\mathcal{T} = \mathcal{P}(X)$; every set is open |
| Indiscrete topology | $\mathcal{T} = \{\emptyset, X\}$; only trivial sets open |
| Neighborhood | Set containing an open set around the point |
| Finite intersection only | Infinite intersections of opens can fail to be open |
| Lattice of topologies | All topologies on $X$ form a complete lattice under $\subseteq$ |

---

## 12. Quick Revision Questions

1. **State the three axioms** that a collection $\mathcal{T}$ must satisfy to be a topology on $X$.

2. Let $X = \{1,2,3\}$ and $\mathcal{T} = \{\emptyset, \{1\}, \{2,3\}, X\}$. Is $\mathcal{T}$ a topology? Verify each axiom.

3. Why does the definition of a topology require only **finite** intersections of open sets? Give a counterexample on $\mathbb{R}$ showing that infinite intersections can fail.

4. How many topologies exist on a $2$-element set? List them all.

5. Give an example of a set that is a neighborhood of a point $x$ but is **not** itself an open set.

6. In the lattice of all topologies on $X$, what is the meet (infimum) of two topologies $\mathcal{T}_1$ and $\mathcal{T}_2$? Is $\mathcal{T}_1 \cup \mathcal{T}_2$ always a topology?

---

[← Back to README](../README.md) | [Next: Open and Closed Sets →](02-open-and-closed-sets.md)
