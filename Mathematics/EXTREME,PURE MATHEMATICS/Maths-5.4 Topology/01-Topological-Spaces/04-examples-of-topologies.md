# Chapter 1.4 — Examples of Topologies

[← Previous: Basis and Subbasis](03-basis-and-subbasis.md) | [Back to README](../README.md) | [Next: Comparing Topologies →](05-comparing-topologies.md)

---

## 1. Overview

This chapter is a **catalogue** of important topologies that serve as examples and counterexamples throughout the subject. Many results in general topology are tested against these; knowing them well builds the intuition needed to navigate the theory.

---

## 2. The Standard (Euclidean) Topology on $\mathbb{R}^n$

**Basis:** Open balls $B_r(x) = \{y \in \mathbb{R}^n : \|x - y\| < r\}$ for $r > 0$.

In $\mathbb{R}^1$: open intervals $(a,b)$.

**Properties:**
- Hausdorff (T₂), second countable, metrizable
- Connected, locally path-connected
- $\sigma$-compact, locally compact

```
ℝ¹:   ←──────●════════════●──────→
              a    (a,b)    b

ℝ²:   Open disk B_r(x):
       ┌─────────────────┐
       │    ╭─────╮      │
       │   ╱  • x  ╲     │
       │  │    r     │    │
       │   ╲        ╱     │
       │    ╰─────╯      │
       └─────────────────┘
```

---

## 3. The Discrete Topology

**Definition:** $\mathcal{T} = \mathcal{P}(X)$ (every subset is open).

**Basis:** Singletons $\{\{x\} : x \in X\}$.

**Properties:**
| Property | Value |
|----------|-------|
| Every set open | ✓ |
| Every set closed | ✓ |
| Every function from $X$ to any space is continuous | ✓ |
| Hausdorff | ✓ |
| Compact | ✓ iff $X$ is finite |
| Connected | ✓ iff $|X| \leq 1$ |
| Metrizable | ✓ (use discrete metric) |

**Discrete metric:** $d(x,y) = \begin{cases} 0 & x = y \\ 1 & x \neq y \end{cases}$

---

## 4. The Indiscrete (Trivial) Topology

**Definition:** $\mathcal{T} = \{\emptyset, X\}$.

**Properties:**
| Property | Value |
|----------|-------|
| Open sets | Only $\emptyset$ and $X$ |
| Hausdorff | ✗ (if $|X| > 1$) |
| Compact | ✓ (trivially — any cover includes $X$) |
| Connected | ✓ (no nontrivial clopen sets) |
| Every function to $X$ is continuous | ✓ |
| Metrizable | ✗ (if $|X| > 1$) |

---

## 5. The Cofinite Topology

> **Def 4.1.** The **cofinite topology** on $X$ is:
>
> $$\mathcal{T}_{\text{cof}} = \{U \subseteq X : X \setminus U \text{ is finite}\} \cup \{\emptyset\}$$

Open sets are those whose complements are finite (plus $\emptyset$).

**Properties:**
| Property | Value |
|----------|-------|
| T₁ | ✓ (singletons are closed) |
| Hausdorff | ✗ (if $X$ is infinite — open sets must overlap) |
| Compact | ✓ (any open cover has finite subcover) |
| Connected | ✓ (if $X$ is infinite) |
| Metrizable | ✗ (on infinite sets) |

**Proof of compactness:** Let $\{U_\alpha\}$ cover $X$. Pick any $U_{\alpha_0}$. Then $X \setminus U_{\alpha_0}$ is finite, say $\{x_1, \ldots, x_n\}$. For each $x_i$, choose $U_{\alpha_i}$ containing $x_i$. Then $\{U_{\alpha_0}, U_{\alpha_1}, \ldots, U_{\alpha_n}\}$ is a finite subcover. ∎

---

## 6. The Cocountable Topology

> **Def 4.2.** The **cocountable topology** on $X$ is:
>
> $$\mathcal{T}_{\text{cc}} = \{U \subseteq X : X \setminus U \text{ is countable}\} \cup \{\emptyset\}$$

**Properties:**
- T₁ but not Hausdorff (on uncountable $X$)
- If $X$ is uncountable: not first countable, not metrizable
- Finer than cofinite: $\mathcal{T}_{\text{cof}} \subseteq \mathcal{T}_{\text{cc}}$

---

## 7. The Lower Limit (Sorgenfrey) Topology on $\mathbb{R}$

> **Def 4.3.** The **lower limit topology** (or **Sorgenfrey line** $\mathbb{R}_\ell$) has basis:
>
> $$\mathcal{B} = \{[a, b) : a < b,\; a, b \in \mathbb{R}\}$$

```
Basis element [a, b):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    ←────●════════════○────→
         a   [a,b)    b
         │              │
      included       excluded
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Properties:**
| Property | Standard $\mathbb{R}$ | Sorgenfrey $\mathbb{R}_\ell$ |
|----------|----------------------|------------------------------|
| Hausdorff | ✓ | ✓ |
| Second countable | ✓ | ✗ |
| First countable | ✓ | ✓ |
| Separable | ✓ | ✓ ($\mathbb{Q}$ is dense) |
| Lindelöf | ✓ | ✓ |
| Metrizable | ✓ | ✗ |
| Connected | ✓ | ✗ |
| Paracompact | ✓ | ✓ |

**✦ Key Insight:** $\mathbb{R}_\ell$ is strictly finer than standard $\mathbb{R}$. Every standard open set is Sorgenfrey-open, but $[0,1)$ is Sorgenfrey-open and not standard-open.

---

## 8. The Sorgenfrey Plane $\mathbb{R}_\ell \times \mathbb{R}_\ell$

The product of $\mathbb{R}_\ell$ with itself. Basis elements: $[a,b) \times [c,d)$.

**Key pathology:** Separable but **not** Lindelöf, **not** second countable. The anti-diagonal $\{(x, -x) : x \in \mathbb{R}\}$ is closed and discrete.

---

## 9. The Particular Point Topology

> **Def 4.4.** Fix $p \in X$. The **particular point topology** is:
>
> $$\mathcal{T}_p = \{U \subseteq X : p \in U\} \cup \{\emptyset\}$$

A set is open iff it contains $p$ (or is empty).

**Properties:** T₀ but not T₁, connected, not Hausdorff.

---

## 10. The Sierpiński Space

The simplest non-trivial topology:

$$X = \{0, 1\}, \quad \mathcal{T} = \{\emptyset, \{1\}, \{0,1\}\}$$

```
Sierpiński Space:
━━━━━━━━━━━━━━━━━
  Points: 0, 1
  Open sets: ∅, {1}, {0,1}

  0 ——→ 1
  (0 is in the closure of {1})
  ({0} is closed but not open)
━━━━━━━━━━━━━━━━━
```

This space is universal in the sense that continuous functions $X \to \{0,1\}$ (Sierpiński) correspond to open sets of $X$.

---

## 11. The Order Topology

> **Def 4.5.** Let $(X, \leq)$ be a totally ordered set with $|X| \geq 2$. The **order topology** has basis consisting of:
>
> - Open intervals $(a, b) = \{x : a < x < b\}$
> - Rays $[a_0, b) = \{x : a_0 \leq x < b\}$ if $a_0 = \min X$
> - Rays $(a, b_0] = \{x : a < x \leq b_0\}$ if $b_0 = \max X$

**Examples:**
- $\mathbb{R}$ with the usual order → standard topology
- $\mathbb{Z}$ with the usual order → discrete topology (each $\{n\} = (n-1, n+1)$)
- $[0,1]$ with the usual order → subspace topology from $\mathbb{R}$

---

## 12. The K-Topology on $\mathbb{R}$

> **Def 4.6.** Let $K = \{1/n : n \in \mathbb{N}\}$. The **K-topology** $\mathbb{R}_K$ has basis:
>
> $$\mathcal{B}_K = \{(a,b) : a < b\} \cup \{(a,b) \setminus K : a < b\}$$

**Properties:**
- Strictly finer than standard $\mathbb{R}$
- Hausdorff but **not** regular (key counterexample for separation axioms!)
- $K$ is closed in $\mathbb{R}_K$, and $\{0\}$ is closed, but they cannot be separated by disjoint open sets

---

## 13. Subspace Topology

> **Def 4.7 (Subspace Topology).** Let $(X, \mathcal{T})$ be a topological space and $Y \subseteq X$. The **subspace topology** on $Y$ is:
>
> $$\mathcal{T}_Y = \{U \cap Y : U \in \mathcal{T}\}$$

```
ASCII Diagram: Subspace topology
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ┌────────── X ──────────────┐
  │                           │
  │   ╭──── U (open in X)    │
  │   │  ┌─── Y ───┐         │
  │   │  │▓▓▓▓▓▓▓▓▓│         │
  │   │  │ U ∩ Y   │         │
  │   │  │(open in Y)        │
  │   ╰──│─────────│         │
  │      └─────────┘         │
  └───────────────────────────┘

  Open in Y = (Open in X) ∩ Y
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> **⚠️ Warning:** A set can be open in $Y$ without being open in $X$. Example: $(0,1]$ is open in $Y = [0,1]$? No — $(0,1] = (0,2) \cap [0,1]$, and $(0,2)$ is open in $\mathbb{R}$, so $(0,1]$ is open in $[0,1]$. But $[0,1]$ itself is open in $Y$ (it equals $\mathbb{R} \cap [0,1]$).

Actually more carefully: $\{1\}$ is not open in $[0,1]$ since no open interval around 1 lies in $[0,1]$. But $(1/2, 1]$ **is** open in $[0,1]$: it equals $(1/2, 2) \cap [0,1]$.

---

## 14. Comparison Chart of Major Topologies on $\mathbb{R}$

| Property | Standard | Lower Limit $\mathbb{R}_\ell$ | K-topology $\mathbb{R}_K$ | Cofinite | Discrete |
|----------|----------|------|------|----------|----------|
| Hausdorff | ✓ | ✓ | ✓ | ✗ | ✓ |
| Regular | ✓ | ✓ | ✗ | ✗ | ✓ |
| Normal | ✓ | ✓ | ✗ | ✗ | ✓ |
| Second countable | ✓ | ✗ | ✓ | ✗ | ✗ |
| First countable | ✓ | ✓ | ✓ | ✗ | ✓ |
| Separable | ✓ | ✓ | ✓ | ✓* | ✗ |
| Connected | ✓ | ✗ | ✗ | ✓ | ✗ |
| Compact | ✗ | ✗ | ✗ | ✓ | ✗ |
| Metrizable | ✓ | ✗ | ✗ | ✗ | ✓ |

*On uncountable sets; on countable sets cofinite = discrete.

---

## 15. Summary Table

| Topology | On | Basis / Definition | Key Feature |
|----------|-----|-------------------|-------------|
| Standard $\mathbb{R}^n$ | $\mathbb{R}^n$ | Open balls $B_r(x)$ | The "default" topology |
| Discrete | Any $X$ | Singletons $\{x\}$ | Everything open |
| Indiscrete | Any $X$ | $\{\emptyset, X\}$ | Almost nothing open |
| Cofinite | Any $X$ | Complements of finite sets | T₁, compact, connected |
| Cocountable | Any $X$ | Complements of countable sets | T₁, not Hausdorff (uncountable) |
| Lower limit | $\mathbb{R}$ | Half-open intervals $[a,b)$ | Finer than standard; not metrizable |
| K-topology | $\mathbb{R}$ | $(a,b)$ and $(a,b) \setminus K$ | Hausdorff, not regular |
| Order topology | Totally ordered set | Open intervals + rays | Generalizes standard $\mathbb{R}$ |
| Particular point | Any $X$, fix $p$ | Sets containing $p$ | T₀, not T₁ |
| Sierpiński | $\{0,1\}$ | $\{\emptyset, \{1\}, X\}$ | Classifies open sets |
| Subspace | $Y \subseteq X$ | $\{U \cap Y\}$ | Inherited topology |

---

## 16. Quick Revision Questions

1. In the cofinite topology on $\mathbb{R}$, determine whether $(0,1)$ is open, closed, both, or neither.

2. Show that $\mathbb{R}_\ell$ is **not** connected by finding a separation (hint: use $(-\infty, 0)$ and $[0, \infty)$).

3. What is the closure of $\{0\}$ in the Sierpiński space $(\{0,1\}, \{\emptyset, \{1\}, \{0,1\}\})$?

4. Prove that the subspace topology on $Y \subseteq X$ is indeed a topology.

5. In the K-topology on $\mathbb{R}$, is $K = \{1/n : n \in \mathbb{N}\}$ open, closed, or neither?

6. Give an example of a topology on $\{a, b, c\}$ that is T₀ but **not** T₁.

---

[← Previous: Basis and Subbasis](03-basis-and-subbasis.md) | [Back to README](../README.md) | [Next: Comparing Topologies →](05-comparing-topologies.md)
