# Chapter 1.5 — Comparing Topologies

[← Previous: Examples of Topologies](04-examples-of-topologies.md) | [Back to README](../README.md) | [Next: Continuous Functions →](../02-Continuity/01-continuous-functions.md)

---

## 1. Overview

Given a set $X$, many different topologies can be placed on it. This chapter develops the language and tools for comparing them — **finer**, **coarser**, **comparable** — and shows how this relates to continuity of the identity map and the lattice of all topologies.

---

## 2. Finer and Coarser Topologies

> **Def 5.1.** Let $\mathcal{T}_1$ and $\mathcal{T}_2$ be topologies on $X$.
>
> - $\mathcal{T}_1$ is **coarser** (or **weaker** or **smaller**) than $\mathcal{T}_2$ if $\mathcal{T}_1 \subseteq \mathcal{T}_2$.
> - $\mathcal{T}_2$ is **finer** (or **stronger** or **larger**) than $\mathcal{T}_1$ if $\mathcal{T}_1 \subseteq \mathcal{T}_2$.
> - They are **comparable** if $\mathcal{T}_1 \subseteq \mathcal{T}_2$ or $\mathcal{T}_2 \subseteq \mathcal{T}_1$.

```
Intuition Diagram:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  COARSEST                       FINEST
  (fewest open sets)             (most open sets)

  Indiscrete ──→ ... ──→ Standard ──→ ... ──→ Discrete
  {∅, X}                                      𝒫(X)

  "Less open sets"  ◄──────────►  "More open sets"
  "Coarser grain"                 "Finer grain"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> **⚠️ Warning:** The terminology is not uniform across textbooks. Some authors swap "stronger" and "weaker." The terms **finer** (= more open sets) and **coarser** (= fewer open sets) are unambiguous and preferred.

---

## 3. Comparison via Bases

> **Lem 5.2.** Let $\mathcal{B}_1$ and $\mathcal{B}_2$ be bases generating $\mathcal{T}_1$ and $\mathcal{T}_2$ on $X$. Then $\mathcal{T}_1 \subseteq \mathcal{T}_2$ if and only if:
>
> For every $B_1 \in \mathcal{B}_1$ and $x \in B_1$, there exists $B_2 \in \mathcal{B}_2$ with $x \in B_2 \subseteq B_1$.

**Intuition:** $\mathcal{T}_2$ is finer means its basis elements can **fit inside** every $\mathcal{T}_1$-basis element.

```
Basis comparison (𝒯₂ finer than 𝒯₁):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ╭──────── B₁ (from 𝓑₁) ────────╮
  │                                │
  │       ╭──── B₂ ────╮          │
  │       │     • x     │          │
  │       ╰─────────────╯          │
  │      (from 𝓑₂, fits inside)   │
  ╰────────────────────────────────╯

  ∀ B₁ ∈ 𝓑₁, ∀ x ∈ B₁, ∃ B₂ ∈ 𝓑₂:
    x ∈ B₂ ⊆ B₁
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 4. Key Comparisons on $\mathbb{R}$

### 4.1 Standard vs. Lower Limit

$\mathcal{T}_{\text{std}} \subsetneq \mathcal{T}_\ell$ — the Sorgenfrey topology is **strictly finer**.

**Proof:** Every $(a,b)$ is a union of $[a', b)$ intervals (e.g., $\bigcup_{n} [a + 1/n, b)$), so $\mathcal{T}_{\text{std}} \subseteq \mathcal{T}_\ell$.

But $[0,1) \in \mathcal{T}_\ell$ and $[0,1) \notin \mathcal{T}_{\text{std}}$ (no open interval around $0$ is contained in $[0,1)$). So the inclusion is strict. ∎

### 4.2 Standard vs. K-Topology

$\mathcal{T}_{\text{std}} \subsetneq \mathcal{T}_K$ — the K-topology is **strictly finer**.

**Proof:** $\mathcal{T}_K$ contains all standard open sets plus sets of the form $(a,b) \setminus K$, which are not standard-open. ∎

### 4.3 Lower Limit vs. K-Topology

These are **not comparable!**

- $[0,1) \in \mathcal{T}_\ell$ but $[0,1) \notin \mathcal{T}_K$ (no K-basis element $(-\epsilon, 1)$ or $(-\epsilon, 1) \setminus K$ equals $[0,1)$).
- $(-1,1) \setminus K \in \mathcal{T}_K$ but $(-1,1) \setminus K \notin \mathcal{T}_\ell$ (point $0$ cannot be isolated from $1/n$'s using $[a,b)$).

```
Hasse Diagram: Topologies on ℝ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

            Discrete
           ╱       ╲
          ╱         ╲
    Lower Limit    K-topology    (incomparable!)
          ╲         ╱
           ╲       ╱
           Standard
              │
           Cofinite
              │
           Indiscrete

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 5. Comparison and the Identity Map

> **Prop 5.3.** $\mathcal{T}_1 \subseteq \mathcal{T}_2$ if and only if the identity map $\text{id}: (X, \mathcal{T}_2) \to (X, \mathcal{T}_1)$ is continuous.

**Proof:** $\text{id}$ is continuous $\iff$ preimage of every $\mathcal{T}_1$-open set is $\mathcal{T}_2$-open $\iff$ every $\mathcal{T}_1$-open set is $\mathcal{T}_2$-open $\iff$ $\mathcal{T}_1 \subseteq \mathcal{T}_2$. ∎

> **Cor 5.4.** $\mathcal{T}_1 = \mathcal{T}_2$ if and only if $\text{id}: (X, \mathcal{T}_1) \to (X, \mathcal{T}_2)$ is a homeomorphism.

**✦ Key Insight:** Making a topology finer is equivalent to making the identity map from the new space to the old one continuous. Moving to a finer topology **preserves** continuity of functions going **out** but may **break** continuity of functions coming **in**.

---

## 6. Effects of Refinement on Properties

| Property | Preserved when making finer? | Preserved when making coarser? |
|----------|------------------------------|-------------------------------|
| Hausdorff (T₂) | ✓ (more open sets → still can separate) | ✗ |
| Compact | ✗ (more covers to worry about) | ✓ (fewer open covers) |
| Connected | ✗ (more potential separations) | ✓ (fewer clopen sets) |
| Second countable | ✗ in general | ✗ in general |
| Metrizable | ✗ in general | ✗ in general |
| T₁ | ✓ | ✗ |

**Intuition:**
- **Separation properties improve** with refinement (more open sets to separate points).
- **Compactness and connectedness degrade** with refinement (more open sets create more covers and more clopen sets).

---

## 7. Infima and Suprema of Topologies

Given a family $\{\mathcal{T}_\alpha\}_{\alpha \in A}$ of topologies on $X$:

> **Infimum (meet):** $\bigwedge_\alpha \mathcal{T}_\alpha = \bigcap_\alpha \mathcal{T}_\alpha$
> This is always a topology (verify the axioms directly for intersection).

> **Supremum (join):** $\bigvee_\alpha \mathcal{T}_\alpha$ is the **smallest topology containing** $\bigcup_\alpha \mathcal{T}_\alpha$.
> It is **not** simply $\bigcup_\alpha \mathcal{T}_\alpha$ in general (that union may fail to be closed under intersections).
> Instead: use $\bigcup_\alpha \mathcal{T}_\alpha$ as a **subbasis**, then generate.

---

## 8. The Initial and Final Topologies (Preview)

These constructions are studied in detail in Unit 3, but the comparison framework motivates them:

### Initial Topology
Given functions $f_\alpha: X \to (Y_\alpha, \mathcal{T}_\alpha)$, the **initial topology** on $X$ is the coarsest topology making all $f_\alpha$ continuous. It equals the topology generated by the subbasis $\{f_\alpha^{-1}(U) : U \in \mathcal{T}_\alpha\}$.

### Final Topology
Given functions $g_\alpha: (Y_\alpha, \mathcal{T}_\alpha) \to X$, the **final topology** on $X$ is the finest topology making all $g_\alpha$ continuous. A set $U \subseteq X$ is open iff $g_\alpha^{-1}(U) \in \mathcal{T}_\alpha$ for all $\alpha$.

```
Initial vs Final:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  INITIAL: arrows come IN to X
    (Y₁,𝒯₁) ──f₁──→ X ←──f₂── (Y₂,𝒯₂)
    Coarsest making all fᵢ continuous

  FINAL: arrows go OUT from Yₐ to X
    (Y₁,𝒯₁) ──g₁──→ X ←──g₂── (Y₂,𝒯₂)
    Finest making all gᵢ continuous
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 9. Worked Example: Finding All Topologies on a 3-Element Set and Ordering Them

Let $X = \{a, b, c\}$. There are 29 topologies. Here we list a few and order them:

```
Partial order (selected topologies on {a,b,c}):

  𝒫(X) = Discrete (29th, finest)
    │
    ├── {∅,{a},{b},{a,b},{a,b,c}}
    │     │
    │     ├── {∅,{a},{a,b},{a,b,c}}
    │     │     │
    ├─────┤     │
    │           │
    ├── {∅,{a},{a,b,c}}
    │     │
    ├─────┤
    │
  {∅,{a,b,c}} = Indiscrete (1st, coarsest)
```

---

## 10. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| $\mathcal{T}_1 \subseteq \mathcal{T}_2$ | $\mathcal{T}_1$ coarser, $\mathcal{T}_2$ finer |
| Comparable | One contains the other |
| Basis comparison | Finer basis elements fit inside coarser ones |
| Identity map criterion | $\mathcal{T}_1 \subseteq \mathcal{T}_2 \iff \text{id}: (X,\mathcal{T}_2) \to (X,\mathcal{T}_1)$ continuous |
| Meet $\bigwedge$ | Intersection of topologies |
| Join $\bigvee$ | Generated from union as subbasis |
| Refinement helps | Separation axioms (T₁, T₂, …) |
| Refinement hurts | Compactness, connectedness |
| Initial topology | Coarsest making given maps continuous (into $X$) |
| Final topology | Finest making given maps continuous (out of $Y_\alpha$ into $X$) |

---

## 11. Quick Revision Questions

1. Prove that $\mathcal{T}_{\text{std}} \subsetneq \mathcal{T}_\ell$ on $\mathbb{R}$ using the basis comparison lemma.

2. Are the cofinite topology and the standard topology on $\mathbb{R}$ comparable? Justify.

3. If $\mathcal{T}_1 \subseteq \mathcal{T}_2$ and $(X, \mathcal{T}_1)$ is compact, is $(X, \mathcal{T}_2)$ necessarily compact? Give a proof or counterexample.

4. Prove that the intersection of any family of topologies on $X$ is a topology.

5. Explain in one sentence why making a topology finer can destroy connectedness.

6. Let $f: (X, \mathcal{T}_1) \to (Y, \mathcal{T})$ be continuous. If $\mathcal{T}_2$ is finer than $\mathcal{T}_1$, is $f: (X, \mathcal{T}_2) \to (Y, \mathcal{T})$ still continuous? Prove it.

---

[← Previous: Examples of Topologies](04-examples-of-topologies.md) | [Back to README](../README.md) | [Next: Continuous Functions →](../02-Continuity/01-continuous-functions.md)
