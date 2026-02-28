# Chapter 7: Completeness

[← Previous: Cauchy Sequences](06-cauchy-sequences.md) | [Back to README](../README.md) | [Next: Unit 3 — Convergence of Series →](../03-Series/01-convergence-of-series.md)

---

## Overview

Completeness is the unifying theme of real analysis. In this chapter, we bring together all the equivalent formulations of completeness, prove their equivalences, and understand why completeness is what makes analysis on ℝ possible.

---

## 7.1 The Equivalence Theorem

**Theorem 7.1.** The following statements are all equivalent for an ordered field $F$:

1. **(LUB)** Every nonempty subset of $F$ bounded above has a least upper bound in $F$
2. **(GLB)** Every nonempty subset of $F$ bounded below has a greatest lower bound in $F$
3. **(MCT)** Every bounded monotone sequence in $F$ converges
4. **(NIP)** Every nested sequence of closed bounded intervals has nonempty intersection
5. **(BW)** Every bounded sequence has a convergent subsequence
6. **(CC)** Every Cauchy sequence converges

```
  The Equivalence Web:

            LUB
           ╱   ╲
          ╱     ╲
       GLB       MCT
                ╱   ╲
               ╱     ╲
            NIP       BW
               ╲     ╱
                ╲   ╱
                 CC

  All six are EQUIVALENT — proving any one allows
  you to derive all the others.
  
  All six FAIL in ℚ.
  All six HOLD in ℝ.
```

---

## 7.2 Proof Chain

We have already proved most of these implications in previous chapters. Here is the complete chain:

### LUB ⟹ GLB
*Proved in Chapter 3.* Consider the set of lower bounds; its supremum is the infimum.

### LUB ⟹ MCT
*Proved in Chapter 3 (MCT).* Bounded increasing sequence: sup of range is the limit.

### MCT ⟹ NIP
**Proof.** Let $[a_n, b_n]$ be nested. Then $(a_n)$ is increasing and bounded above (by $b_1$), so $a_n \to a$ by MCT. Similarly $b_n \to b$. Since $a_n \leq b_n$ for all $n$, the Order Limit Theorem gives $a \leq b$. So $a \in \bigcap [a_n, b_n]$. $\blacksquare$

### NIP ⟹ BW
*Proved in Chapter 5.* Bisection argument constructs nested intervals with infinitely many terms.

### BW ⟹ CC
*Proved in Chapter 6.* Cauchy + bounded + BW gives convergent subsequence, then full convergence.

### CC ⟹ LUB
**Proof.** Let $S \subseteq F$ be nonempty and bounded above. We construct a Cauchy sequence converging to $\sup S$.

Pick any upper bound $b_1$ and any $a_1 \in S$. Bisect $[a_1, b_1]$: let $m = (a_1+b_1)/2$.
- If $m$ is an upper bound of $S$, set $b_2 = m$, $a_2 = a_1$.
- Otherwise, $\exists\, s \in S$ with $s > m$; set $a_2 = s$, $b_2 = b_1$.

Continue to get $(a_n)$ and $(b_n)$ with $a_n \in S$, $b_n$ upper bounds, $b_n - a_n = (b_1 - a_1)/2^{n-1}$.

Both sequences are Cauchy (since $|a_m - a_n| \leq (b_1-a_1)/2^{\min(m,n)-1}$). By CC, they converge to the same limit $\alpha$. One verifies $\alpha = \sup S$. $\blacksquare$

---

## 7.3 Summary of the Complete Chain

| Implication | Chapter | Key Technique |
|-------------|---------|---------------|
| LUB ⟹ GLB | Ch. 3 | Set of lower bounds |
| LUB ⟹ MCT | Ch. 3 | Sup of range = limit |
| MCT ⟹ NIP | This chapter | Monotone sequences of endpoints |
| NIP ⟹ BW | Ch. 5 | Bisection method |
| BW ⟹ CC | Ch. 6 | Cauchy + convergent subseq |
| CC ⟹ LUB | This chapter | Bisection to construct Cauchy seq |

---

## 7.4 Completeness vs Incompleteness: ℝ vs ℚ

| Property | $\mathbb{R}$ | $\mathbb{Q}$ |
|----------|:---:|:---:|
| Field | ✅ | ✅ |
| Ordered field | ✅ | ✅ |
| Archimedean | ✅ | ✅ |
| LUB property | ✅ | ❌ |
| MCT | ✅ | ❌ |
| NIP | ✅ | ❌ |
| BW | ✅ | ❌ |
| Cauchy completeness | ✅ | ❌ |

### Counterexample for each failure in ℚ

| Property | Counterexample in ℚ |
|----------|-------------------|
| LUB | $\{r \in \mathbb{Q} : r^2 < 2\}$ — no sup in ℚ |
| MCT | $a_n = \sum_{k=0}^n 1/k!$ — bounded, increasing, but limit is $e \notin \mathbb{Q}$ |
| NIP | $I_n = [3.14..., 3.15...]$ narrowing to $\pi$ — intersection empty in ℚ |
| BW | Rational approx. of $\sqrt{2}$ — bounded but no convergent subseq in ℚ |
| CC | Same Cauchy sequence — doesn't converge in ℚ |

---

## 7.5 Constructing ℝ from ℚ

There are two classical constructions:

### Method 1: Dedekind Cuts
- A cut is a partition $(A, B)$ of $\mathbb{Q}$ with $A$ downward closed and without maximum
- Each cut "is" a real number
- $\mathbb{Q}$ embeds into cuts, and the resulting field is complete

### Method 2: Cauchy Sequences
- Consider all Cauchy sequences in $\mathbb{Q}$
- Define $(a_n) \sim (b_n)$ if $|a_n - b_n| \to 0$
- $\mathbb{R} := \{\text{Cauchy sequences in } \mathbb{Q}\} / \sim$
- Define arithmetic and ordering on equivalence classes
- The resulting field is complete (Cauchy sequences of equivalence classes converge)

```
  Construction of ℝ from ℚ:

  ┌─────────────────────────┐
  │     ℚ (incomplete)      │
  │   "has gaps"            │
  └──────────┬──────────────┘
             │
     ┌───────┴───────┐
     │               │
  Dedekind        Cauchy
  Cuts            Sequences
     │               │
     └───────┬───────┘
             │
  ┌──────────▼──────────────┐
  │     ℝ (complete)        │
  │   "no gaps"             │
  │   Every Cauchy seq      │
  │   converges             │
  └─────────────────────────┘
```

---

## 7.6 Completeness in Other Contexts (Preview)

| Space | Complete? | Why/Why Not |
|-------|-----------|-------------|
| $\mathbb{R}$ | ✅ | Completeness axiom |
| $\mathbb{Q}$ | ❌ | $\sqrt{2}$ gap |
| $\mathbb{R}^n$ | ✅ | Complete in each coordinate |
| $C([0,1])$ with sup norm | ✅ | Uniform limits of continuous functions are continuous |
| $C([0,1])$ with $L^1$ norm | ❌ | Cauchy sequences can converge to discontinuous functions |
| $\ell^2$ (square-summable sequences) | ✅ | A Hilbert space |

---

## 7.7 Real-World Application

- **Numerical analysis**: Completeness guarantees that well-behaved iterative methods converge to a solution
- **Fixed-point theory**: Banach's theorem requires completeness of the underlying space
- **Functional analysis**: Completeness of $L^2$ spaces is essential for quantum mechanics
- **Economics**: Existence of Nash equilibria uses fixed-point theorems that require completeness

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Six equivalent properties | LUB, GLB, MCT, NIP, BW, CC |
| All require completeness | All fail in ℚ |
| LUB ⟹ MCT ⟹ NIP ⟹ BW ⟹ CC ⟹ LUB | Circular equivalence chain |
| Construction of ℝ | Via Dedekind cuts or Cauchy sequences |
| Characterization of ℝ | Unique complete ordered field (up to isomorphism) |
| Extends to metric spaces | "Complete metric space" = Cauchy sequences converge |

---

## Quick Revision Questions

1. **List all six equivalent formulations** of completeness. Which two are used most frequently in proofs?

2. **Prove MCT ⟹ NIP.** What role does the Order Limit Theorem play?

3. **Give a counterexample** showing the Nested Interval Property fails in ℚ.

4. **Sketch the proof chain** LUB ⟹ MCT ⟹ NIP ⟹ BW ⟹ CC ⟹ LUB. For each arrow, state the key idea.

5. **Briefly describe** how ℝ can be constructed from ℚ using Cauchy sequences.

6. **True or False:** $\mathbb{R}$ is the only complete ordered field. In what sense?

---

[← Previous: Cauchy Sequences](06-cauchy-sequences.md) | [Back to README](../README.md) | [Next: Unit 3 — Convergence of Series →](../03-Series/01-convergence-of-series.md)
