# Topology — Comprehensive Study Notes

## Course Overview

**Topology** is the branch of mathematics concerned with the properties of geometric objects that are preserved under continuous deformations — stretching, twisting, crumpling, and bending, but **not** tearing or gluing. Often called "rubber-sheet geometry," topology abstracts the notion of closeness and continuity away from the rigid framework of metric spaces, providing a powerful and general language used throughout modern mathematics, physics, and data science.

These notes cover **point-set (general) topology** from first principles through advanced separation and countability axioms, metric space theory, and conclude with an **introduction to algebraic topology** (homotopy, fundamental groups, covering spaces).

---

## Prerequisites

- Naïve set theory (unions, intersections, complements, Cartesian products, power sets)
- Functions (injections, surjections, bijections, compositions, inverses)
- Familiarity with the real line ℝ and basic analysis (open intervals, convergence)
- Elementary group theory (for Unit 9)

---

## How to Use These Notes

| Symbol | Meaning |
|--------|---------|
| **Def** | Definition |
| **Thm** | Theorem |
| **Prop** | Proposition |
| **Lem** | Lemma |
| **Cor** | Corollary |
| **Ex** | Example |
| **⚠️** | Common pitfall / warning |
| **✦** | Key insight |

---

## Complete Table of Contents

### Unit 1 — Topological Spaces
| # | Topic | Link |
|---|-------|------|
| 1 | Definition of Topology | [01-definition-of-topology.md](01-Topological-Spaces/01-definition-of-topology.md) |
| 2 | Open and Closed Sets | [02-open-and-closed-sets.md](01-Topological-Spaces/02-open-and-closed-sets.md) |
| 3 | Basis and Subbasis | [03-basis-and-subbasis.md](01-Topological-Spaces/03-basis-and-subbasis.md) |
| 4 | Examples of Topologies | [04-examples-of-topologies.md](01-Topological-Spaces/04-examples-of-topologies.md) |
| 5 | Comparing Topologies | [05-comparing-topologies.md](01-Topological-Spaces/05-comparing-topologies.md) |

### Unit 2 — Continuity
| # | Topic | Link |
|---|-------|------|
| 1 | Continuous Functions | [01-continuous-functions.md](02-Continuity/01-continuous-functions.md) |
| 2 | Homeomorphisms | [02-homeomorphisms.md](02-Continuity/02-homeomorphisms.md) |
| 3 | Topological Properties | [03-topological-properties.md](02-Continuity/03-topological-properties.md) |
| 4 | Open and Closed Maps | [04-open-and-closed-maps.md](02-Continuity/04-open-and-closed-maps.md) |

### Unit 3 — Product and Quotient Spaces
| # | Topic | Link |
|---|-------|------|
| 1 | Product Topology | [01-product-topology.md](03-Product-and-Quotient-Spaces/01-product-topology.md) |
| 2 | Box Topology | [02-box-topology.md](03-Product-and-Quotient-Spaces/02-box-topology.md) |
| 3 | Quotient Topology | [03-quotient-topology.md](03-Product-and-Quotient-Spaces/03-quotient-topology.md) |
| 4 | Quotient Maps | [04-quotient-maps.md](03-Product-and-Quotient-Spaces/04-quotient-maps.md) |

### Unit 4 — Connectedness
| # | Topic | Link |
|---|-------|------|
| 1 | Connected Spaces | [01-connected-spaces.md](04-Connectedness/01-connected-spaces.md) |
| 2 | Path-Connected Spaces | [02-path-connected-spaces.md](04-Connectedness/02-path-connected-spaces.md) |
| 3 | Components | [03-components.md](04-Connectedness/03-components.md) |
| 4 | Local Connectedness | [04-local-connectedness.md](04-Connectedness/04-local-connectedness.md) |

### Unit 5 — Compactness
| # | Topic | Link |
|---|-------|------|
| 1 | Definition and Properties | [01-definition-and-properties.md](05-Compactness/01-definition-and-properties.md) |
| 2 | Compact Subsets of ℝⁿ | [02-compact-subsets-of-Rn.md](05-Compactness/02-compact-subsets-of-Rn.md) |
| 3 | Limit Point Compactness | [03-limit-point-compactness.md](05-Compactness/03-limit-point-compactness.md) |
| 4 | Local Compactness | [04-local-compactness.md](05-Compactness/04-local-compactness.md) |
| 5 | One-Point Compactification | [05-one-point-compactification.md](05-Compactness/05-one-point-compactification.md) |

### Unit 6 — Separation Axioms
| # | Topic | Link |
|---|-------|------|
| 1 | T₀, T₁, T₂ (Hausdorff) | [01-T0-T1-T2-hausdorff.md](06-Separation-Axioms/01-T0-T1-T2-hausdorff.md) |
| 2 | Regular and Normal Spaces | [02-regular-and-normal-spaces.md](06-Separation-Axioms/02-regular-and-normal-spaces.md) |
| 3 | Urysohn's Lemma | [03-urysohns-lemma.md](06-Separation-Axioms/03-urysohns-lemma.md) |
| 4 | Tietze Extension Theorem | [04-tietze-extension-theorem.md](06-Separation-Axioms/04-tietze-extension-theorem.md) |

### Unit 7 — Countability Axioms
| # | Topic | Link |
|---|-------|------|
| 1 | First and Second Countable | [01-first-and-second-countable.md](07-Countability-Axioms/01-first-and-second-countable.md) |
| 2 | Separable Spaces | [02-separable-spaces.md](07-Countability-Axioms/02-separable-spaces.md) |
| 3 | Lindelöf Spaces | [03-lindelof-spaces.md](07-Countability-Axioms/03-lindelof-spaces.md) |

### Unit 8 — Metric Spaces
| # | Topic | Link |
|---|-------|------|
| 1 | Metrics and Metric Topology | [01-metrics-and-metric-topology.md](08-Metric-Spaces/01-metrics-and-metric-topology.md) |
| 2 | Complete Metric Spaces | [02-complete-metric-spaces.md](08-Metric-Spaces/02-complete-metric-spaces.md) |
| 3 | Baire Category Theorem | [03-baire-category-theorem.md](08-Metric-Spaces/03-baire-category-theorem.md) |
| 4 | Compactness in Metric Spaces | [04-compactness-in-metric-spaces.md](08-Metric-Spaces/04-compactness-in-metric-spaces.md) |

### Unit 9 — Introduction to Algebraic Topology
| # | Topic | Link |
|---|-------|------|
| 1 | Homotopy | [01-homotopy.md](09-Algebraic-Topology-Intro/01-homotopy.md) |
| 2 | Fundamental Group | [02-fundamental-group.md](09-Algebraic-Topology-Intro/02-fundamental-group.md) |
| 3 | Covering Spaces | [03-covering-spaces.md](09-Algebraic-Topology-Intro/03-covering-spaces.md) |

---

## Recommended Textbooks

| Book | Author(s) | Level |
|------|-----------|-------|
| *Topology* (2nd ed.) | James R. Munkres | Standard undergraduate/graduate |
| *Introduction to Topological Manifolds* | John M. Lee | Graduate |
| *Algebraic Topology* | Allen Hatcher | Graduate |
| *General Topology* | Stephen Willard | Reference |
| *Counterexamples in Topology* | Steen & Seebach | Supplement |

---

## Key Notation Reference

| Notation | Meaning |
|----------|---------|
| $(X, \mathcal{T})$ | Topological space $X$ with topology $\mathcal{T}$ |
| $\mathcal{T}$ | A topology (collection of open sets) |
| $\text{Int}(A)$ or $A°$ | Interior of $A$ |
| $\overline{A}$ or $\text{Cl}(A)$ | Closure of $A$ |
| $\partial A$ or $\text{Bd}(A)$ | Boundary of $A$ |
| $A'$ | Set of limit points of $A$ |
| $f : X \to Y$ | Function from space $X$ to space $Y$ |
| $f^{-1}(U)$ | Preimage of $U$ under $f$ |
| $X \cong Y$ | $X$ is homeomorphic to $Y$ |
| $\prod X_\alpha$ | Product of spaces $X_\alpha$ |
| $X / \!\sim$ | Quotient space |
| $\pi_1(X, x_0)$ | Fundamental group of $X$ at basepoint $x_0$ |
| $[f]$ | Homotopy class of loop $f$ |
| $S^n$ | $n$-sphere |
| $D^n$ | $n$-disk (closed ball) |
| $\mathbb{R}^n$ | Euclidean $n$-space |

---

> *"Topology is precisely the mathematical discipline that allows the passage from local to global."* — René Thom

---

[Begin with **Unit 1: Topological Spaces** →](01-Topological-Spaces/01-definition-of-topology.md)
