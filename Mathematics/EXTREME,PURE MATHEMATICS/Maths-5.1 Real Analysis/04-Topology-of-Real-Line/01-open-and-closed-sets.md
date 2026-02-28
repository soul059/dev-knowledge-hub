# Chapter 1: Open and Closed Sets

[← Previous: Unit 3 — Rearrangement of Series](../03-Series/07-rearrangement-of-series.md) | [Back to README](../README.md) | [Next: Limit Points and Closure →](02-limit-points-and-closure.md)

---

## Overview

Point-set topology on $\mathbb{R}$ provides the language for continuity, convergence, and compactness. The central objects are **open sets** (every point has "breathing room") and **closed sets** (contain all their limit points). These concepts generalize interval endpoints to arbitrary subsets.

---

## 1.1 Neighborhoods and Interior Points

**Definition 1.1.** The **$\varepsilon$-neighborhood** of $a \in \mathbb{R}$ is:

$$V_\varepsilon(a) = (a - \varepsilon, a + \varepsilon) = \{x \in \mathbb{R} : |x - a| < \varepsilon\}$$

**Definition 1.2.** A point $a$ is an **interior point** of $A \subseteq \mathbb{R}$ if $\exists\, \varepsilon > 0$ such that $V_\varepsilon(a) \subseteq A$.

The **interior** of $A$ is $A^\circ = \text{int}(A) = \{a \in A : a \text{ is an interior point of } A\}$.

```
  Interior point of A:

  ◄────────────────── A ──────────────────►
  ═══════════════════════════════════════════
         ▲
         a
      (a-ε, a+ε) ⊆ A
      ◄───────────►
         ε    ε
  
  There's room to move left and right and stay in A.

  NON-interior point (boundary):
  ═══════════════════|
                     ▲
                     a
                  (a-ε, a+ε) ⊄ A for any ε > 0
```

---

## 1.2 Open Sets

**Definition 1.3.** A set $O \subseteq \mathbb{R}$ is **open** if every point of $O$ is an interior point: $O = O^\circ$.

Equivalently: $\forall\, x \in O$, $\exists\, \varepsilon > 0$ such that $(x - \varepsilon, x + \varepsilon) \subseteq O$.

**Examples:**
| Set | Open? | Reason |
|-----|:-----:|--------|
| $(a, b)$ | ✅ | For $x \in (a,b)$, take $\varepsilon = \min(x-a, b-x)$ |
| $\mathbb{R}$ | ✅ | Any $\varepsilon$ works for any $x$ |
| $\emptyset$ | ✅ | Vacuously true (no points to check) |
| $[a, b]$ | ❌ | $a$ is not an interior point |
| $(a, b]$ | ❌ | $b$ is not an interior point |
| $\{a\}$ | ❌ | No neighborhood of $a$ fits in $\{a\}$ |
| $\mathbb{Q}$ | ❌ | Every neighborhood contains irrationals |
| $(0,1) \cup (2,3)$ | ✅ | Union of open sets |

---

## 1.3 Properties of Open Sets

**Theorem 1.4.** Open sets satisfy:

1. $\mathbb{R}$ and $\emptyset$ are open
2. **Arbitrary unions** of open sets are open: if $\{O_\alpha\}_{\alpha \in I}$ is any collection of open sets, then $\bigcup_{\alpha \in I} O_\alpha$ is open
3. **Finite intersections** of open sets are open: if $O_1, \ldots, O_N$ are open, then $\bigcap_{k=1}^N O_k$ is open

**Proof of (2).** Let $x \in \bigcup O_\alpha$. Then $x \in O_\beta$ for some $\beta$. Since $O_\beta$ is open, $\exists\, \varepsilon > 0$ with $V_\varepsilon(x) \subseteq O_\beta \subseteq \bigcup O_\alpha$. $\blacksquare$

**Proof of (3).** Let $x \in \bigcap_{k=1}^N O_k$. For each $k$, $\exists\, \varepsilon_k > 0$ with $V_{\varepsilon_k}(x) \subseteq O_k$. Take $\varepsilon = \min(\varepsilon_1, \ldots, \varepsilon_N) > 0$. Then $V_\varepsilon(x) \subseteq O_k$ for all $k$, so $V_\varepsilon(x) \subseteq \bigcap O_k$. $\blacksquare$

```
  Why FINITE intersections, not arbitrary?

  Counterexample: Oₙ = (-1/n, 1/n)  (each is open)

  ∩ Oₙ = {0}  ← a single point, NOT open!
   n=1
   ∞

  The ε = min(ε₁,...,εₙ) trick fails with infinitely many εₖ
  because inf{εₖ} could be 0.
```

---

## 1.4 Closed Sets

**Definition 1.5.** A set $F \subseteq \mathbb{R}$ is **closed** if its complement $F^c = \mathbb{R} \setminus F$ is open.

**Examples:**
| Set | Closed? | Reason |
|-----|:------:|--------|
| $[a, b]$ | ✅ | $(-\infty, a) \cup (b, \infty)$ is open |
| $\{a\}$ | ✅ | $(-\infty, a) \cup (a, \infty)$ is open |
| $\mathbb{R}$ | ✅ | $\emptyset$ is open |
| $\emptyset$ | ✅ | $\mathbb{R}$ is open |
| $(a, b)$ | ❌ | $(-\infty, a] \cup [b, \infty)$ not open |
| $\mathbb{Z}$ | ✅ | Complement is union of open intervals |
| $\mathbb{Q}$ | ❌ | Complement $\mathbb{R}\setminus\mathbb{Q}$ not open |

> **Warning:** "Not open" does NOT mean "closed". Many sets are neither open nor closed (e.g., $[0,1)$), and both $\mathbb{R}$ and $\emptyset$ are **both open and closed** ("clopen").

```
  ┌──────────────────────────────────────────┐
  │            Sets in ℝ                     │
  │                                          │
  │     ┌─────────────┐                     │
  │     │   Open      │  ← (a,b), ℝ, ∅     │
  │     │   ┌───┐     │                     │
  │     │   │BO │     │  ← ℝ, ∅ (clopen)   │
  │     │   └───┘     │                     │
  │     └─────────────┘                     │
  │     ┌─────────────┐                     │
  │     │  Closed     │  ← [a,b], {a}, ℤ   │
  │     └─────────────┘                     │
  │                                          │
  │     Neither: [0,1), (0,1], ℚ            │
  └──────────────────────────────────────────┘
  
  BO = Both Open and Closed (clopen)
```

---

## 1.5 Properties of Closed Sets (De Morgan Duality)

**Theorem 1.6.** By complementation from Theorem 1.4:

1. $\mathbb{R}$ and $\emptyset$ are closed
2. **Arbitrary intersections** of closed sets are closed
3. **Finite unions** of closed sets are closed

```
  De Morgan Duality:

  Open sets                     Closed sets
  ─────────                     ───────────
  Arbitrary unions    ←→    Arbitrary intersections
  Finite intersections ←→    Finite unions

  (complement swaps union ↔ intersection)
```

---

## 1.6 Characterization of Closed Sets via Sequences

**Theorem 1.7.** $F$ is closed $\iff$ whenever $(x_n)$ is a sequence in $F$ with $x_n \to x$, we have $x \in F$.

In other words: **closed sets contain all limits of their convergent sequences.**

**Proof ($\Rightarrow$).** Suppose $F$ is closed, $(x_n) \subseteq F$, $x_n \to x$. If $x \notin F$, then $x \in F^c$ (open), so $\exists\, \varepsilon > 0$ with $V_\varepsilon(x) \subseteq F^c$. But $x_n \to x$ means $\exists\, N$ with $x_N \in V_\varepsilon(x) \subseteq F^c$, contradicting $x_N \in F$.

**Proof ($\Leftarrow$).** Suppose $F$ contains all its sequence limits. We show $F^c$ is open. Let $x \in F^c$. If $x$ is not an interior point of $F^c$, then $\forall\, n$, $V_{1/n}(x) \cap F \neq \emptyset$, so pick $x_n \in V_{1/n}(x) \cap F$. Then $x_n \to x$ and $x_n \in F$, so $x \in F$ — contradiction. $\blacksquare$

---

## 1.7 Boundary

**Definition 1.8.** The **boundary** of $A$ is:
$$\partial A = \{x \in \mathbb{R} : \forall\, \varepsilon > 0,\; V_\varepsilon(x) \cap A \neq \emptyset \text{ and } V_\varepsilon(x) \cap A^c \neq \emptyset\}$$

| Set $A$ | $\partial A$ |
|---------|-------------|
| $(a, b)$ | $\{a, b\}$ |
| $[a, b]$ | $\{a, b\}$ |
| $\mathbb{Q}$ | $\mathbb{R}$ |
| $\mathbb{R}$ | $\emptyset$ |

**Fact:** $A$ is closed $\iff$ $\partial A \subseteq A$. $A$ is open $\iff$ $\partial A \cap A = \emptyset$.

---

## 1.8 Structure of Open Sets in ℝ

**Theorem 1.9.** Every non-empty open set in $\mathbb{R}$ is a countable union of disjoint open intervals.

**Proof idea.** For each $x \in O$, let $I_x$ be the largest open interval containing $x$ and contained in $O$. The intervals $\{I_x\}$ are either disjoint or equal. Each contains a rational, so the collection is countable. $\blacksquare$

```
  Example: O = (0,1) ∪ (2,3) ∪ (3,5)

  ◄──┤     ├──┤     ├┤         ├──►
     0     1  2     3          5

  Disjoint open intervals: (0,1), (2,3), (3,5)
  (Even complicated open sets decompose this way)
```

---

## Summary Table

| Concept | Definition / Key Property |
|---------|--------------------------|
| $\varepsilon$-neighborhood | $V_\varepsilon(a) = (a-\varepsilon, a+\varepsilon)$ |
| Interior point | $\exists\, \varepsilon > 0$: $V_\varepsilon(a) \subseteq A$ |
| Open set | Every point is interior; $O = O^\circ$ |
| Closed set | Complement is open; contains all sequence limits |
| Clopen | Both open and closed; only $\mathbb{R}$ and $\emptyset$ in $\mathbb{R}$ |
| Open: unions/intersections | Arbitrary $\cup$, finite $\cap$ |
| Closed: unions/intersections | Finite $\cup$, arbitrary $\cap$ |
| Boundary | $\partial A$: every nbhd meets both $A$ and $A^c$ |
| Structure theorem | Open in $\mathbb{R}$ = countable $\cup$ of disjoint intervals |

---

## Quick Revision Questions

1. **Prove** that $(a, b)$ is open by finding a suitable $\varepsilon$ for each $x \in (a, b)$.

2. **Show** that an arbitrary intersection of open sets need not be open. Give an explicit example.

3. **Prove** that $F$ is closed if and only if it contains all limits of convergent sequences in $F$.

4. **Is** $\mathbb{Q}$ open, closed, both, or neither? Prove your answer.

5. **Find** the interior, closure, and boundary of (a) $(0,1] \cup \{2\}$ (b) $\mathbb{Z}$.

6. **State** the structure theorem for open sets in $\mathbb{R}$. Why is countability important?

---

[← Previous: Unit 3 — Rearrangement of Series](../03-Series/07-rearrangement-of-series.md) | [Back to README](../README.md) | [Next: Limit Points and Closure →](02-limit-points-and-closure.md)
