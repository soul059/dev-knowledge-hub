# Chapter 3: Completeness Axiom

[← Previous: Order Axioms](02-order-axioms.md) | [Back to README](../README.md) | [Next: Supremum and Infimum →](04-supremum-and-infimum.md)

---

## Overview

The completeness axiom is the single property that **distinguishes ℝ from ℚ**. Both ℚ and ℝ are ordered fields, but ℝ has no "gaps." This axiom is the foundation of all limit-based arguments in analysis — convergence, continuity, integration — all depend on completeness.

---

## 3.1 The Problem with ℚ: Gaps in the Rational Numbers

Consider the set $A = \{r \in \mathbb{Q} : r^2 < 2\}$.

- $A$ is nonempty: $1 \in A$ since $1^2 = 1 < 2$.
- $A$ is bounded above: $2$ is an upper bound since $r \in A \implies r^2 < 2 \implies r < 2$.

**Question:** Does $A$ have a **least upper bound** in ℚ?

**Answer: No!** The least upper bound would have to be $\sqrt{2}$, but $\sqrt{2} \notin \mathbb{Q}$.

```
  The rational number line has GAPS:

  ◄──────┬──────┬──────┬──┬──────┬──────┬──────►
         1    1.4   1.41  ?   1.42   1.5    2
                          │
                          ▼
                       √2 ≈ 1.41421356...
                       is NOT in ℚ

  Set A = {r ∈ ℚ : r² < 2}

  ◄═══════════════════╗  ╔═══════════════════►
  ─────────────────────╫──╫─────────────────────
       members of A     ╚══╝    upper bounds
                        THE GAP
                     (√2 is missing!)
```

---

## 3.2 Upper and Lower Bounds

**Definition 3.1.** Let $S \subseteq \mathbb{R}$, $S \neq \emptyset$.

- An **upper bound** of $S$ is a number $M \in \mathbb{R}$ such that $s \leq M$ for all $s \in S$.
- A **lower bound** of $S$ is a number $m \in \mathbb{R}$ such that $m \leq s$ for all $s \in S$.
- $S$ is **bounded above** if it has an upper bound.
- $S$ is **bounded below** if it has a lower bound.
- $S$ is **bounded** if it is bounded above and below.

```
  ┌────────────────────────────────────────────────────┐
  │             Visualization of Bounds                 │
  │                                                     │
  │    lower bounds       S           upper bounds      │
  │   ◄═══════════╗ ┌─────────────┐ ╔═══════════►     │
  │   ─────────────╫─┤  s₁ s₂ s₃  ├─╫────────────     │
  │                ╚═╡             ╞═╝                  │
  │                  └─────────────┘                    │
  │                ▲ inf(S)    sup(S) ▲                 │
  │        greatest lower      least upper              │
  │            bound            bound                   │
  └────────────────────────────────────────────────────┘
```

---

## 3.3 The Completeness Axiom (Least Upper Bound Property)

**Axiom (Completeness / LUB Property).** Every nonempty subset of $\mathbb{R}$ that is bounded above has a **least upper bound** (supremum) in $\mathbb{R}$.

Formally: If $S \subseteq \mathbb{R}$, $S \neq \emptyset$, and $S$ is bounded above, then $\exists\, \alpha \in \mathbb{R}$ such that:
1. $\alpha$ is an upper bound of $S$: $s \leq \alpha$ for all $s \in S$
2. $\alpha$ is the **least** upper bound: if $M$ is any upper bound of $S$, then $\alpha \leq M$

We write $\alpha = \sup S$.

---

## 3.4 What Makes ℝ Special

```
  Comparison: ℚ vs ℝ
  ═══════════════════════════

  ┌───────────────────────┐     ┌───────────────────────┐
  │     ℚ (Rationals)     │     │     ℝ (Reals)         │
  ├───────────────────────┤     ├───────────────────────┤
  │ ✅ Field axioms       │     │ ✅ Field axioms       │
  │ ✅ Order axioms       │     │ ✅ Order axioms       │
  │ ❌ Completeness       │     │ ✅ COMPLETENESS       │
  ├───────────────────────┤     ├───────────────────────┤
  │ Has "gaps" — bounded  │     │ NO gaps — every       │
  │ sets may fail to have │     │ bounded-above set     │
  │ a least upper bound   │     │ has a supremum IN ℝ   │
  └───────────────────────┘     └───────────────────────┘

  The completeness axiom is THE defining feature of ℝ.
```

---

## 3.5 Equivalent Formulations of Completeness

The following are **logically equivalent** to the LUB property:

| # | Statement | Key Idea |
|---|-----------|----------|
| 1 | **LUB Property** | Bounded-above sets have a supremum |
| 2 | **GLB Property** | Bounded-below sets have an infimum |
| 3 | **Monotone Convergence** | Bounded monotone sequences converge |
| 4 | **Nested Interval Property** | Nested closed intervals have nonempty intersection |
| 5 | **Bolzano-Weierstrass** | Every bounded sequence has a convergent subsequence |
| 6 | **Cauchy Completeness** | Every Cauchy sequence converges |
| 7 | **Dedekind Completeness** | Every Dedekind cut is realized by a real number |

```
  Logical Equivalence Map:
  ════════════════════════

  LUB Property ◄═══► GLB Property
       │
       ▼
  Archimedean + Monotone Convergence
       │
       ▼
  Nested Intervals ◄═══► Bolzano-Weierstrass
       │
       ▼
  Cauchy Completeness
```

### Proof that LUB ⟹ GLB

**Theorem 3.1.** If ℝ has the LUB property, then it has the GLB property.

**Proof.** Let $S \subseteq \mathbb{R}$ be nonempty and bounded below. Let $L = \{l \in \mathbb{R} : l \text{ is a lower bound of } S\}$.

- $L \neq \emptyset$ (since $S$ is bounded below).  
- $L$ is bounded above (any $s \in S$ is an upper bound of $L$).

By the LUB property, $\alpha = \sup L$ exists. We claim $\alpha = \inf S$.

1. $\alpha$ is a lower bound of $S$: If $s \in S$, then $s$ is an upper bound of $L$, so $\alpha \leq s$.
2. $\alpha$ is the greatest lower bound: If $m$ is any lower bound of $S$, then $m \in L$, so $m \leq \alpha = \sup L$. $\blacksquare$

---

## 3.6 The Nested Interval Property

**Theorem 3.2 (Nested Interval Property).** Let $I_n = [a_n, b_n]$ be a sequence of closed, bounded intervals with $I_1 \supseteq I_2 \supseteq I_3 \supseteq \cdots$. Then:
$$\bigcap_{n=1}^{\infty} I_n \neq \emptyset$$

Moreover, if $\lim_{n \to \infty} (b_n - a_n) = 0$, then $\bigcap_{n=1}^{\infty} I_n$ contains exactly one point.

**Proof.** The sequence $(a_n)$ is increasing and bounded above (by any $b_k$). By completeness, $a = \sup\{a_n\}$ exists. Similarly, $b = \inf\{b_n\}$ exists. For all $m, n$: $a_m \leq b_n$ (since $I_{\max(m,n)}$ is nonempty). So $a \leq b$, and $[a, b] \subseteq \bigcap I_n$. $\blacksquare$

```
  Nested Intervals:

  I₁: [────────────────────────────────────]
       a₁                                 b₁

  I₂:    [──────────────────────────────]
          a₂                           b₂

  I₃:       [────────────────────────]
             a₃                     b₃

  I₄:          [──────────────────]
                a₄               b₄
          ...

  ∩ Iₙ:              [──]    or    { • }
                    a = sup{aₙ}   b = inf{bₙ}

  If lengths → 0:  a single point remains
```

**Important:** The intervals must be **closed**. Open intervals can fail:

**Counterexample:** $I_n = (0, 1/n)$. Then $I_1 \supset I_2 \supset \cdots$, but $\bigcap_{n=1}^{\infty} (0, 1/n) = \emptyset$.

---

## 3.7 Dedekind Cuts (Optional: Construction of ℝ)

A **Dedekind cut** is a partition of $\mathbb{Q}$ into sets $(A, B)$ where:
1. $A, B \neq \emptyset$ and $A \cup B = \mathbb{Q}$
2. If $a \in A$ and $b \in B$, then $a < b$
3. $A$ has no maximum element

Each Dedekind cut represents a real number. The cut $(\{r \in \mathbb{Q} : r < \sqrt{2}\},\; \{r \in \mathbb{Q} : r \geq \sqrt{2}\})$ doesn't work since $\sqrt{2} \notin \mathbb{Q}$. Instead: $A = \{r \in \mathbb{Q} : r < 0 \text{ or } r^2 < 2\}$, $B = \mathbb{Q} \setminus A$. This cut defines $\sqrt{2}$.

---

## 3.8 Real-World Application

Completeness is essential in:
- **Numerical analysis**: Iterative methods (Newton's method) converge because ℝ is complete
- **Fixed-point theorems**: The Banach fixed-point theorem requires completeness
- **Economics**: Existence of equilibrium prices uses completeness
- **Physics**: Solutions to differential equations exist by completeness arguments

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Upper bound | $M \geq s$ for all $s \in S$ |
| Bounded above | Has at least one upper bound |
| Supremum | Least upper bound (guaranteed to exist in ℝ) |
| Completeness axiom | Bounded-above nonempty sets have a sup in ℝ |
| ℚ is incomplete | $\{r \in \mathbb{Q} : r^2 < 2\}$ has no sup in ℚ |
| LUB ⟹ GLB | Proved by considering set of lower bounds |
| Nested intervals | Closed nested intervals have nonempty intersection |
| Dedekind cuts | One way to construct ℝ from ℚ |
| Equivalent forms | MCT, Bolzano-Weierstrass, Cauchy completeness, etc. |

---

## Quick Revision Questions

1. **State the Completeness Axiom precisely.** Why are the conditions "nonempty" and "bounded above" both necessary?

2. **Give an example of a bounded subset of ℚ** that has no least upper bound in ℚ.

3. **Prove that the GLB property follows from the LUB property.** What is the key construction?

4. **Why does the Nested Interval Property fail for open intervals?** Give a specific counterexample.

5. **List three equivalent formulations** of the completeness of ℝ.

6. **If $S = \{1 - 1/n : n \in \mathbb{N}\}$, what is $\sup S$?** Is the supremum attained (i.e., is $\sup S \in S$)?

---

[← Previous: Order Axioms](02-order-axioms.md) | [Back to README](../README.md) | [Next: Supremum and Infimum →](04-supremum-and-infimum.md)
