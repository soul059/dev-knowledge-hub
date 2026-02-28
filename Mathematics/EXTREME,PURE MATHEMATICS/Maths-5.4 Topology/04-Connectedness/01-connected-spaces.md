# Chapter 4.1 — Connected Spaces

[← Previous: Quotient Maps](../03-Product-and-Quotient-Spaces/04-quotient-maps.md) | [Back to README](../README.md) | [Next: Path-Connected Spaces →](02-path-connected-spaces.md)

---

## 1. Overview

Intuitively, a space is **connected** if it is "in one piece" — it cannot be split into two disjoint nonempty open sets. Connectedness is a fundamental topological invariant preserved by continuous maps, and it forms the basis for the Intermediate Value Theorem.

---

## 2. Definition

> **Def 1.1 (Connected Space).** A topological space $X$ is **connected** if there is no **separation** of $X$: that is, there do not exist nonempty open sets $U, V \subseteq X$ with $U \cap V = \emptyset$ and $U \cup V = X$.

Equivalently:

> **Prop 1.2.** The following are equivalent:
> 1. $X$ is connected.
> 2. $X$ has no nontrivial clopen (simultaneously open and closed) subsets — the only clopen sets are $\emptyset$ and $X$.
> 3. $X$ cannot be written as $X = A \cup B$ with $A, B$ nonempty, $A \cap \overline{B} = \emptyset$, $\overline{A} \cap B = \emptyset$ (separation by closed sets).
> 4. Every continuous function $f: X \to \{0, 1\}$ (discrete topology) is constant.

```
Connected vs Disconnected:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  CONNECTED:                DISCONNECTED:
  ┌────────────────┐        ┌───────┐ ┌───────┐
  │                │        │  U    │ │    V  │
  │    X (one      │        │       │ │       │
  │     piece)     │        │       │ │       │
  └────────────────┘        └───────┘ └───────┘

  No separation exists      X = U ⊔ V (separation)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Key Examples

| Space | Connected? | Why? |
|-------|:---:|------|
| $\mathbb{R}$ (standard) | ✓ | Dedekind completeness / below |
| $(0,1) \cup (2,3)$ | ✗ | $(0,1)$ and $(2,3)$ form a separation |
| $\mathbb{Q}$ | ✗ | $(-\infty, \sqrt{2}) \cap \mathbb{Q}$ and $(\sqrt{2}, \infty) \cap \mathbb{Q}$ separate |
| $\mathbb{R}^n$ | ✓ | Path-connected (next chapter) |
| Indiscrete space | ✓ | Only clopen sets: $\emptyset, X$ |
| Discrete space ($|X|>1$) | ✗ | Every singleton is clopen |
| Cofinite on infinite $X$ | ✓ | Two nonempty opens must overlap (complements are finite) |

---

## 4. The Connected Subsets of $\mathbb{R}$

> **Thm 1.3.** A subset $S \subseteq \mathbb{R}$ is connected (in the subspace topology) if and only if $S$ is an **interval** (i.e., if $a, b \in S$ and $a < c < b$, then $c \in S$).

**Proof sketch ($\Leftarrow$):** Suppose $S$ is an interval and $S = U \cup V$ is a separation. Pick $a \in U, b \in V$ with $a < b$. Let $c = \sup(U \cap [a,b])$. Then $c \in \overline{U} \cap \overline{V}$, contradicting the separation property. ∎

**Proof sketch ($\Rightarrow$):** If $S$ is not an interval, some $a < c < b$ with $a, b \in S$, $c \notin S$. Then $S \cap (-\infty, c)$ and $S \cap (c, \infty)$ separate $S$. ∎

---

## 5. Theorems on Connected Spaces

### 5.1 Continuous Image of Connected Is Connected

> **Thm 1.4.** If $f: X \to Y$ is continuous and $X$ is connected, then $f(X)$ is connected.

**Proof:** If $f(X) = U \cup V$ is a separation, then $X = f^{-1}(U) \cup f^{-1}(V)$ is a separation of $X$. Contradiction. ∎

### 5.2 Intermediate Value Theorem

> **Cor 1.5 (IVT).** If $f: [a,b] \to \mathbb{R}$ is continuous and $f(a) < r < f(b)$, then there exists $c \in (a,b)$ with $f(c) = r$.

**Proof:** $f([a,b])$ is connected (continuous image of connected set) and hence an interval. Since $f(a)$ and $f(b)$ are in this interval, so is $r$. ∎

> **✦ Key Insight:** The IVT is really a theorem about connectedness!

### 5.3 Union of Connected Sets

> **Thm 1.6.** If $\{A_\alpha\}$ is a family of connected subsets of $X$ with $\bigcap_\alpha A_\alpha \neq \emptyset$, then $\bigcup_\alpha A_\alpha$ is connected.

More generally:

> **Thm 1.7.** If $A$ is connected and $A \subseteq B \subseteq \overline{A}$, then $B$ is connected.

In particular, the **closure of a connected set is connected**.

### 5.4 Finite Products

> **Thm 1.8.** $X \times Y$ is connected iff both $X$ and $Y$ are connected.

**Proof sketch ($\Leftarrow$):** Fix $(x_0, y_0) \in X \times Y$. For each $(x, y)$, the set $\{x_0\} \times Y \cup X \times \{y\}$ is connected (union of two connected sets sharing a point) and contains both $(x_0, y_0)$ and $(x, y)$. The union over all $(x,y)$ covers $X \times Y$ and is connected. ∎

### 5.5 Arbitrary Products

> **Thm 1.9.** The product $\prod_\alpha X_\alpha$ (product topology) is connected iff each $X_\alpha$ is connected.

---

## 6. Applications of Connectedness

| Application | How Connectedness Is Used |
|-------------|--------------------------|
| **IVT** | $f([a,b])$ is a connected subset of $\mathbb{R}$, hence an interval |
| **$\mathbb{R} \not\cong \mathbb{R}^2$** | Remove a point: $\mathbb{R}\setminus\{0\}$ disconnected, $\mathbb{R}^2\setminus\{0\}$ connected |
| **Fixed-point theorems** | Brouwer's theorem uses connectedness (and compactness) |
| **Classification** | Number of connected components is a topological invariant |

---

## 7. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Connected | No separation $X = U \sqcup V$ with $U, V$ open & nonempty |
| Equivalent | Only clopen sets: $\emptyset, X$ |
| Equivalent | Every $f: X \to \{0,1\}$ constant |
| Connected ⊂ ℝ | Iff it is an interval |
| Continuous image | Connected → connected |
| IVT | Consequence of connectedness of $[a,b]$ |
| Union | Connected sets with common point → union connected |
| Closure | Closure of connected is connected |
| Products | Connected iff all factors connected |

---

## 8. Quick Revision Questions

1. Prove that $\mathbb{Q}$ is not connected by exhibiting a separation.

2. Show that if $X$ is connected and $f: X \to \{0,1\}$ is continuous (discrete topology on $\{0,1\}$), then $f$ is constant.

3. Derive the Intermediate Value Theorem from the theorem that continuous images of connected sets are connected.

4. Prove: if $A$ is connected and $A \subseteq B \subseteq \overline{A}$, then $B$ is connected.

5. Is $\mathbb{R} \setminus \mathbb{Q}$ connected? Justify your answer.

6. Show that $X \times Y$ is connected when both $X$ and $Y$ are connected.

---

[← Previous: Quotient Maps](../03-Product-and-Quotient-Spaces/04-quotient-maps.md) | [Back to README](../README.md) | [Next: Path-Connected Spaces →](02-path-connected-spaces.md)
