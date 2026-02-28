# Chapter 6.4 — Tietze Extension Theorem

[← Previous: Urysohn's Lemma](03-urysohns-lemma.md) | [Back to README](../README.md) | [Next: Unit 7 — First and Second Countable →](../07-Countability-Axioms/01-first-and-second-countable.md)

---

## 1. Overview

The **Tietze Extension Theorem** generalises Urysohn's Lemma: any continuous real-valued function on a closed subspace of a normal space can be extended to the entire space. This is indispensable in analysis, geometry, and applied mathematics.

---

## 2. Statement

> **Theorem 4.1 (Tietze Extension Theorem).** Let $X$ be a normal space and $A \subseteq X$ a closed subspace. If $f : A \to [a,b]$ (or $f : A \to \mathbb{R}$) is continuous, then there exists a continuous extension $F : X \to [a,b]$ (resp. $F : X \to \mathbb{R}$) with $F|_A = f$.

```
         f
    A ──────→ [a, b]  (or ℝ)
    │          ↗
    │ incl.  F
    ↓       /
    X ─────
    
    F|_A = f  (F extends f)
```

---

## 3. Proof Sketch (Bounded Case: $f : A \to [-1, 1]$)

The proof uses an iterative approximation via Urysohn's Lemma.

### Step 1: First Approximation

Define closed sets:
$$A_1 = f^{-1}([-1, -\tfrac{1}{3}]), \qquad B_1 = f^{-1}([\tfrac{1}{3}, 1])$$

These are disjoint closed sets in $X$ (since $A$ is closed in $X$). By Urysohn's Lemma, $\exists\, g_1 : X \to [-\tfrac{1}{3}, \tfrac{1}{3}]$ with $g_1|_{A_1} = -\tfrac{1}{3}$ and $g_1|_{B_1} = \tfrac{1}{3}$.

Then $|f(x) - g_1(x)| \leq \tfrac{2}{3}$ for all $x \in A$.

### Step 2: Iterate

Apply the same argument to $f - g_1$ (scaled), obtaining $g_2$ with
$$|f - g_1 - g_2| \leq \left(\tfrac{2}{3}\right)^2$$

### Step $n$:

$$g_n : X \to \left[-\tfrac{1}{3}\left(\tfrac{2}{3}\right)^{n-1}, \tfrac{1}{3}\left(\tfrac{2}{3}\right)^{n-1}\right]$$

$$\left|f - \sum_{k=1}^n g_k\right| \leq \left(\tfrac{2}{3}\right)^n \to 0$$

### Step 3: Sum the Series

$$F(x) = \sum_{n=1}^{\infty} g_n(x)$$

This converges uniformly (by comparison with geometric series $\sum \frac{1}{3}(\frac{2}{3})^{n-1} = 1$), hence $F$ is continuous, and $F|_A = f$.

```
Error reduction:
Step 1:  |f - g₁| ≤ 2/3
Step 2:  |f - g₁ - g₂| ≤ (2/3)²
Step 3:  |f - g₁ - g₂ - g₃| ≤ (2/3)³
  ⋮                  ⋮
Step n:  |f - Σgₖ| ≤ (2/3)ⁿ → 0
```

∎

---

## 4. Unbounded Case

For $f : A \to \mathbb{R}$: compose with a homeomorphism $\mathbb{R} \cong (-1,1)$ (e.g., $t \mapsto t/(1+|t|)$), extend, then compose back. Care is needed to ensure the range stays in $\mathbb{R}$ (not $[-1,1]$).

---

## 5. Converse

> **Thm 4.2.** If $X$ is T₁ and every continuous $f : A \to [0,1]$ (for any closed $A$) extends to $X$, then $X$ is normal.

*Proof:* Given disjoint closed $A, B$, define $f : A \cup B \to [0,1]$ by $f|_A = 0$, $f|_B = 1$. Then $A \cup B$ is closed, so $f$ extends to $F : X \to [0,1]$. Then $F^{-1}([0, 1/2))$ and $F^{-1}((1/2, 1])$ are the desired separating open sets. ∎

---

## 6. Applications

### 6.1 Extending Bump Functions

On a normal space, if $f$ is defined on a closed set, we can extend it to the whole space. This is heavily used in:
- Differential topology (extending smooth functions on submanifolds)
- PDE theory (extending boundary data)
- Embedding theorems

### 6.2 Dugundji Extension Theorem

For **metric** spaces, a stronger result holds: any continuous function $f : A \to E$ from a closed subset to a locally convex topological vector space extends continuously. The extension is even a **linear operator** on the function space.

---

## 7. Relationship to Urysohn's Lemma

| | Urysohn's Lemma | Tietze Extension |
|--|:-:|:-:|
| Input | Two disjoint closed sets | Continuous function on closed set |
| Output | Function $f : X \to [0,1]$ | Extension $F : X \to [a,b]$ |
| Hypothesis | Normality | Normality |
| Urysohn is special case? | — | Yes: $f(A)=0, f(B)=1$ |
| Proof uses | Dyadic nesting | Urysohn's Lemma iteratively |

Tietze is strictly stronger: it implies Urysohn (define $f$ on $A \cup B$ as above), but Urysohn is used to prove Tietze.

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| Tietze Extension | Continuous $f : A \to [a,b]$ extends to $F : X \to [a,b]$ |
| Hypothesis | $X$ normal, $A$ closed |
| Proof | Iterative Urysohn; geometric series convergence |
| Converse | Extension property ⟹ normality |
| Unbounded case | Via homeomorphism $\mathbb{R} \cong (-1,1)$ |
| Stronger variant | Dugundji (metric spaces, values in LCTVS) |

---

## 9. Quick Revision Questions

1. State the Tietze Extension Theorem precisely (both bounded and unbounded versions).

2. Outline the proof for $f : A \to [-1, 1]$. Why does $\sum g_n$ converge?

3. Why is the hypothesis "$A$ closed" necessary? Give a counterexample if $A$ is open.

4. Prove: Tietze Extension implies Urysohn's Lemma.

5. State the converse: the extension property characterises normality.

6. How does the Dugundji Extension Theorem generalise Tietze?

---

[← Previous: Urysohn's Lemma](03-urysohns-lemma.md) | [Back to README](../README.md) | [Next: Unit 7 — First and Second Countable →](../07-Countability-Axioms/01-first-and-second-countable.md)
