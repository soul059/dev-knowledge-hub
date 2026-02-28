# Chapter 1.2 — Open and Closed Sets

[← Previous: Definition of Topology](01-definition-of-topology.md) | [Back to README](../README.md) | [Next: Basis and Subbasis →](03-basis-and-subbasis.md)

---

## 1. Overview

Open sets are the primitive objects in a topology. **Closed sets** are defined as their complements. This chapter develops the theory of both, introducing the essential operations of **interior**, **closure**, and **boundary** that let us dissect any subset of a topological space.

---

## 2. Closed Sets

> **Def 2.1 (Closed Set).** A subset $C \subseteq X$ is **closed** in $(X, \mathcal{T})$ if its complement $X \setminus C$ is open (i.e., $X \setminus C \in \mathcal{T}$).

### Properties of Closed Sets (De Morgan Duality)

> **Prop 2.2.** Let $(X, \mathcal{T})$ be a topological space. Then:
>
> **(C1)** $X$ and $\emptyset$ are closed.
>
> **(C2)** Any **intersection** of closed sets is closed.
>
> **(C3)** Any **finite union** of closed sets is closed.

**Proof sketch:** Apply De Morgan's laws to the topology axioms. E.g., $X \setminus \bigcap C_\alpha = \bigcup (X \setminus C_\alpha)$, which is a union of opens, hence open. ∎

```
Duality Table: Open vs Closed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  OPEN SETS                CLOSED SETS
  ─────────                ───────────
  ∅, X are open            ∅, X are closed
  Arbitrary ⋃ → open       Arbitrary ⋂ → closed
  Finite ⋂ → open          Finite ⋃ → closed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> **⚠️ Warning:** A set can be **both open and closed** ("clopen"), **neither**, or one without the other, depending on the topology.

### Example: Clopen Sets

In **any** topological space $(X, \mathcal{T})$:
- $\emptyset$ and $X$ are always clopen.
- In the **discrete** topology, every set is clopen.
- In the **indiscrete** topology, only $\emptyset$ and $X$ are clopen.

---

## 3. Interior, Closure, Boundary

### 3.1 Interior

> **Def 2.3 (Interior).** The **interior** of $A \subseteq X$ is:
>
> $$\text{Int}(A) = A° = \bigcup \{ U \in \mathcal{T} : U \subseteq A \}$$
>
> It is the **largest open set** contained in $A$.

### 3.2 Closure

> **Def 2.4 (Closure).** The **closure** of $A \subseteq X$ is:
>
> $$\overline{A} = \text{Cl}(A) = \bigcap \{ C : C \text{ closed},\; A \subseteq C \}$$
>
> It is the **smallest closed set** containing $A$.

### 3.3 Boundary

> **Def 2.5 (Boundary).** The **boundary** of $A$ is:
>
> $$\partial A = \text{Bd}(A) = \overline{A} \setminus \text{Int}(A)$$

### 3.4 Exterior

> **Def 2.6 (Exterior).** The **exterior** of $A$ is:
>
> $$\text{Ext}(A) = \text{Int}(X \setminus A)$$

```
ASCII Diagram: Decomposition of X relative to A
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┌──────────────── X ─────────────────────────┐
│                                            │
│   Ext(A)     ┌──── ∂A ────┐    Int(A)     │
│              │             │               │
│  (open,      │  (closed,   │  (open,      │
│  disjoint    │  "skin"     │  largest     │
│  from A̅)    │  of A)      │  open ⊆ A)  │
│              │             │               │
│              └─────────────┘               │
│                                            │
│         ◄──── A̅ = Int(A) ∪ ∂A ────►       │
└────────────────────────────────────────────┘

  X = Int(A) ⊔ ∂A ⊔ Ext(A)    (disjoint union)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 4. Properties of Interior and Closure

> **Prop 2.7 (Interior Properties).**
> 1. $\text{Int}(A) \subseteq A$
> 2. $\text{Int}(A)$ is open
> 3. $A$ is open $\iff$ $A = \text{Int}(A)$
> 4. $\text{Int}(\text{Int}(A)) = \text{Int}(A)$ (idempotent)
> 5. $\text{Int}(A \cap B) = \text{Int}(A) \cap \text{Int}(B)$

> **Prop 2.8 (Closure Properties).**
> 1. $A \subseteq \overline{A}$
> 2. $\overline{A}$ is closed
> 3. $A$ is closed $\iff$ $A = \overline{A}$
> 4. $\overline{\overline{A}} = \overline{A}$ (idempotent)
> 5. $\overline{A \cup B} = \overline{A} \cup \overline{B}$

> **Prop 2.9 (Duality).** For any $A \subseteq X$:
>
> $$\text{Int}(A) = X \setminus \overline{X \setminus A}$$
> $$\overline{A} = X \setminus \text{Int}(X \setminus A)$$

---

## 5. Kuratowski Closure Axioms

The closure operator $A \mapsto \overline{A}$ can alternatively **define** a topology:

> **Thm 2.10 (Kuratowski).** A function $\text{cl}: \mathcal{P}(X) \to \mathcal{P}(X)$ is the closure operator of some topology on $X$ if and only if:
>
> **(K1)** $\text{cl}(\emptyset) = \emptyset$
>
> **(K2)** $A \subseteq \text{cl}(A)$
>
> **(K3)** $\text{cl}(A \cup B) = \text{cl}(A) \cup \text{cl}(B)$
>
> **(K4)** $\text{cl}(\text{cl}(A)) = \text{cl}(A)$

The topology is recovered by: $C$ is closed $\iff$ $\text{cl}(C) = C$.

---

## 6. Limit Points and Derived Sets

> **Def 2.11 (Limit Point).** A point $x \in X$ is a **limit point** (or **accumulation point**) of $A \subseteq X$ if every open set $U$ containing $x$ satisfies:
>
> $$(U \setminus \{x\}) \cap A \neq \emptyset$$

> **Def 2.12 (Derived Set).** The set of all limit points of $A$ is the **derived set** $A'$.

> **Thm 2.13.** $\overline{A} = A \cup A'$.

**Proof:** ($\supseteq$) $A \subseteq \overline{A}$, and if $x \in A'$, every open set containing $x$ meets $A$, so $x$ cannot be in the interior of $X \setminus A$, hence $x \in \overline{A}$. ($\subseteq$) If $x \in \overline{A}$ and $x \notin A$, then every open set containing $x$ must meet $A$ (otherwise $x$ lies in an open set missing $A$, contradicting $x \in \overline{A}$), so $x \in A'$. ∎

### Example on $\mathbb{R}$ (standard topology)

| Set $A$ | $A'$ (limit points) | $\overline{A}$ |
|---------|---------------------|-----------------|
| $(0,1)$ | $[0,1]$ | $[0,1]$ |
| $\{1/n : n \in \mathbb{N}\}$ | $\{0\}$ | $\{0\} \cup \{1/n : n \in \mathbb{N}\}$ |
| $\mathbb{Z}$ | $\emptyset$ | $\mathbb{Z}$ |
| $\mathbb{Q}$ | $\mathbb{R}$ | $\mathbb{R}$ |

---

## 7. Dense and Nowhere Dense Sets

> **Def 2.14.** A subset $A$ of $(X, \mathcal{T})$ is:
> - **Dense** if $\overline{A} = X$ (equivalently, $A$ meets every nonempty open set).
> - **Nowhere dense** if $\text{Int}(\overline{A}) = \emptyset$.

**Examples:**
- $\mathbb{Q}$ is dense in $\mathbb{R}$.
- $\mathbb{Z}$ is nowhere dense in $\mathbb{R}$ (its closure is itself, and $\text{Int}(\mathbb{Z}) = \emptyset$).
- The Cantor set is closed and nowhere dense in $\mathbb{R}$.

---

## 8. Worked Example: Interior, Closure, Boundary

**Problem:** Let $X = \mathbb{R}$ with the standard topology and $A = [0,1) \cup \{2\}$. Find $\text{Int}(A)$, $\overline{A}$, $\partial A$.

**Solution:**

**Step 1 — Interior:**
The largest open set inside $A$. The interval $[0,1)$ has interior $(0,1)$. The isolated point $\{2\}$ has no open interval around it inside $A$. So:
$$\text{Int}(A) = (0,1)$$

**Step 2 — Closure:**
The smallest closed set containing $A$. We need to add the limit points. $0$ and $1$ are limit points of $[0,1)$; $2$ is already in $A$ but is not a limit point (it's isolated). So:
$$\overline{A} = [0,1] \cup \{2\}$$

**Step 3 — Boundary:**
$$\partial A = \overline{A} \setminus \text{Int}(A) = [0,1] \cup \{2\} \setminus (0,1) = \{0, 1, 2\}$$

```
Number line visualization:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
     ←──────────────────────────────────→
    -1    0    0.5    1    1.5    2    3
          [==========)          •
          │  Int(A)   │         │
          ●──────────●          ●
          0          1          2
          ∂A = {0, 1, 2}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 9. The 14-Set Problem (Kuratowski's Closure-Complement Theorem)

> **Thm 2.15 (Kuratowski, 1922).** Starting from any set $A \subseteq X$ and alternately applying closure and complement, one obtains **at most 14 distinct sets**.

This bound is achieved, for example, in $\mathbb{R}$ with:

$$A = (0,1) \cup (1,2) \cup \{3\} \cup ([4,5] \cap \mathbb{Q})$$

The 14 operations (using $c$ for complement and $k$ for closure): $A, kA, cA, ckA, kcA, kckA, ckcA, ckckA, kckA, \ldots$ This cycle closes after at most 14.

---

## 10. Summary Table

| Concept | Definition / Key Formula |
|---------|-------------------------|
| Closed set | $C$ is closed $\iff$ $X \setminus C$ is open |
| Interior $\text{Int}(A)$ | Largest open set $\subseteq A$; $\bigcup\{U \text{ open} : U \subseteq A\}$ |
| Closure $\overline{A}$ | Smallest closed set $\supseteq A$; $\bigcap\{C \text{ closed} : A \subseteq C\}$ |
| Boundary $\partial A$ | $\overline{A} \setminus \text{Int}(A)$ |
| Limit point of $A$ | Every open nbhd of $x$ meets $A \setminus \{x\}$ |
| Derived set $A'$ | Set of all limit points of $A$ |
| $\overline{A} = A \cup A'$ | Closure via limit points |
| Dense | $\overline{A} = X$ |
| Nowhere dense | $\text{Int}(\overline{A}) = \emptyset$ |
| Clopen | Simultaneously open and closed |
| Kuratowski axioms | $\text{cl}$ satisfying (K1)–(K4) defines a unique topology |

---

## 11. Quick Revision Questions

1. Prove that a set $A$ is open if and only if $A = \text{Int}(A)$.

2. Show that $\overline{A \cup B} = \overline{A} \cup \overline{B}$ but $\overline{A \cap B} \subseteq \overline{A} \cap \overline{B}$ (give an example where strict inclusion holds).

3. Let $X = \{a,b,c,d\}$ with $\mathcal{T} = \{\emptyset, \{a\}, \{a,b\}, \{a,b,c\}, X\}$. Find all closed sets. Compute $\overline{\{b\}}$ and $\text{Int}(\{a,b,c\})$.

4. Is $\mathbb{Q}$ closed in $\mathbb{R}$ (standard topology)? Justify using limit points.

5. Prove: $A$ is closed $\iff$ $A' \subseteq A$ (i.e., $A$ contains all its limit points).

6. Explain why $\partial A$ is always a closed set.

---

[← Previous: Definition of Topology](01-definition-of-topology.md) | [Back to README](../README.md) | [Next: Basis and Subbasis →](03-basis-and-subbasis.md)
