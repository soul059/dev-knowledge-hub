# Chapter 5: Archimedean Property

[← Previous: Supremum and Infimum](04-supremum-and-infimum.md) | [Back to README](../README.md) | [Next: Density of Rationals and Irrationals →](06-density-of-rationals-and-irrationals.md)

---

## Overview

The Archimedean property says that no real number is "infinitely large" or "infinitely small" — the natural numbers are unbounded and can eventually exceed any real number. This seemingly obvious fact requires the completeness axiom for its proof and is the workhorse behind many ε-arguments.

---

## 5.1 Statement of the Archimedean Property

**Theorem 5.1 (Archimedean Property).** For every $x \in \mathbb{R}$, there exists $n \in \mathbb{N}$ such that $n > x$.

Equivalently: $\mathbb{N}$ is **not bounded above** in $\mathbb{R}$.

### Equivalent Formulations

| Form | Statement |
|------|-----------|
| **(i)** | $\forall\, x \in \mathbb{R},\; \exists\, n \in \mathbb{N}:\; n > x$ |
| **(ii)** | $\forall\, \varepsilon > 0,\; \exists\, n \in \mathbb{N}:\; 1/n < \varepsilon$ |
| **(iii)** | $\forall\, x > 0,\; \forall\, y \in \mathbb{R},\; \exists\, n \in \mathbb{N}:\; nx > y$ |
| **(iv)** | $\inf\{1/n : n \in \mathbb{N}\} = 0$ |

```
  Archimedean Property Visualized:

  Given ANY real number x, the natural numbers
  eventually surpass it:

  ◄──┬──┬──┬──┬──┬──┬──┬──┬──┬───────┬──────►
     1  2  3  4  5  6  7  8  9  ...   n > x
                                       ▲
                                       │
                              ┌────────┴────────┐
                              │ x = 8.7         │
                              │ Take n = 9 > x  │
                              └─────────────────┘

  No matter how large x is, ℕ is NOT bounded above.
```

---

## 5.2 Proof (Using Completeness)

**Proof.** By contradiction. Suppose $\mathbb{N}$ is bounded above. Since $\mathbb{N} \neq \emptyset$ and $\mathbb{N}$ is bounded above, the completeness axiom gives $\alpha = \sup \mathbb{N}$.

By the ε-characterization (with $\varepsilon = 1$), there exists $n \in \mathbb{N}$ with $n > \alpha - 1$.

Then $n + 1 > \alpha$. But $n + 1 \in \mathbb{N}$, contradicting $\alpha = \sup \mathbb{N}$. $\blacksquare$

```
  Proof by contradiction — visual summary:

  Assume ℕ is bounded above
          │
          ▼
  α = sup ℕ exists (by completeness)
          │
          ▼
  ∃ n ∈ ℕ with n > α - 1  (ε-characterization, ε = 1)
          │
          ▼
  n + 1 ∈ ℕ and n + 1 > α
          │
          ▼
  Contradiction! (α was supposed to be an upper bound)
          │
          ▼
  ∴ ℕ is unbounded above ✓
```

---

## 5.3 Corollaries

### Corollary 5.2: $1/n \to 0$
For every $\varepsilon > 0$, there exists $n \in \mathbb{N}$ such that $0 < 1/n < \varepsilon$.

**Proof.** By Archimedean property, $\exists\, n \in \mathbb{N}$ with $n > 1/\varepsilon$, so $1/n < \varepsilon$. $\blacksquare$

### Corollary 5.3: Floor Function
For every $x \in \mathbb{R}$, there exists a unique $n \in \mathbb{Z}$ with $n \leq x < n + 1$. This $n$ is denoted $\lfloor x \rfloor$ (the floor of $x$).

**Proof.** The set $\{m \in \mathbb{Z} : m \leq x\}$ is nonempty (by Archimedean property applied to $-x$) and bounded above (by $x$). Let $n$ be its maximum (which exists since it's a bounded-above set of integers). $\blacksquare$

### Corollary 5.4: Powers of 2 Grow Without Bound
For every $M > 0$, there exists $n \in \mathbb{N}$ with $2^n > M$.

**Proof.** Since $2^n \geq n$ for all $n \geq 1$ (by induction), the Archimedean property gives $n > M$, hence $2^n \geq n > M$. $\blacksquare$

---

## 5.4 The Archimedean Property in ε-Arguments

The Archimedean property is used in virtually every convergence proof. Here is the pattern:

```
  Standard Pattern in Analysis:

  Goal: Show |xₙ - L| < ε for all n ≥ N

  Step 1: Establish |xₙ - L| ≤ C/n for some constant C
                                          │
  Step 2: Given ε > 0, CHOOSE N such    │
          that C/N < ε                    │
                                          ▼
  Step 3: By Archimedean property,  ┌──────────────┐
          ∃ N ∈ ℕ with N > C/ε      │ This is where │
                                     │ Archimedean   │
                                     │ property is   │
                                     │ used!         │
                                     └──────────────┘
  Step 4: For n ≥ N:
          |xₙ - L| ≤ C/n ≤ C/N < ε  ✓
```

---

## 5.5 Non-Archimedean Fields (Why This Matters)

Not every ordered field is Archimedean! There exist **non-Archimedean ordered fields** containing elements that are "infinitely large" or "infinitely small."

| Field | Archimedean? | Notes |
|-------|-------------|-------|
| $\mathbb{R}$ | ✅ Yes | By completeness |
| $\mathbb{Q}$ | ✅ Yes | Inherits from ℝ (or direct proof) |
| $\mathbb{Q}(x)$ (rational functions) | ❌ No | $x$ is larger than every rational — it's "infinitely large" |
| Hyperreal numbers ${}^*\mathbb{R}$ | ❌ No | Contains infinitesimals; used in nonstandard analysis |
| $p$-adic numbers $\mathbb{Q}_p$ | ❌ No | Different absolute value; used in number theory |

```
  In ℝ (Archimedean):          In *ℝ (Non-Archimedean):

  No infinitesimals:            Infinitesimals exist:
  if 0 < x < 1/n for all n,    ε > 0 but ε < 1/n
  then x = 0                   for ALL n ∈ ℕ

  ◄──┬──┬──┬──┬──►             ◄──┬──┬──┬──┬──────┬►
     0  1  2  3                   0  1  2  3  ...  ω
                                  ▲                ▲
                               infinitesimal    infinite
                               neighborhood     element
```

**Key insight:** The Archimedean property is a **consequence of completeness** in ℝ. It cannot be derived from the field and order axioms alone!

---

## 5.6 The Well-Ordering Principle

**Theorem 5.5 (Well-Ordering Principle).** Every nonempty subset of $\mathbb{N}$ has a minimum element.

This is equivalent to mathematical induction and is often used alongside the Archimedean property.

**Application:** If $S = \{n \in \mathbb{N} : n > x\}$ for some $x \in \mathbb{R}$, then $S \neq \emptyset$ (by Archimedean property) and $S$ has a minimum element (by well-ordering).

---

## 5.7 Real-World Application

- **Computer arithmetic**: Floating-point numbers satisfy a "discrete Archimedean" property — there's a smallest positive representable number (machine epsilon)
- **Approximation theory**: We can approximate any real number to arbitrary precision using rationals with bounded denominators
- **Numerical methods**: Iterative algorithms converge because $1/n \to 0$, which rests on the Archimedean property

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Archimedean property | $\mathbb{N}$ is unbounded in $\mathbb{R}$ |
| Equivalent: $1/n \to 0$ | For any $\varepsilon > 0$, some $1/n < \varepsilon$ |
| Proof uses completeness | Contradiction via $\sup \mathbb{N}$ |
| Floor function | $\lfloor x \rfloor = \max\{n \in \mathbb{Z} : n \leq x\}$ |
| Role in ε-proofs | Choosing $N$ large enough that $C/N < \varepsilon$ |
| Non-Archimedean fields | Exist! (hyperreals, $p$-adics) — contain infinitesimals |
| Well-ordering | Every nonempty $S \subseteq \mathbb{N}$ has a minimum |

---

## Quick Revision Questions

1. **State the Archimedean property** in three equivalent ways.

2. **Prove the Archimedean property** using the completeness axiom. Why can't we prove it from field + order axioms alone?

3. **Given $\varepsilon = 0.0037$**, find explicitly an $n \in \mathbb{N}$ with $1/n < \varepsilon$.

4. **Prove that $\inf\{1/n : n \in \mathbb{N}\} = 0$.** Use the ε-characterization of infimum.

5. **What is a non-Archimedean ordered field?** Give an example and explain what goes wrong with the Archimedean property in it.

6. **Use the Archimedean property to prove:** For any $a, b \in \mathbb{R}$ with $a < b$, there exists $n \in \mathbb{N}$ with $1/n < b - a$.

---

[← Previous: Supremum and Infimum](04-supremum-and-infimum.md) | [Back to README](../README.md) | [Next: Density of Rationals and Irrationals →](06-density-of-rationals-and-irrationals.md)
