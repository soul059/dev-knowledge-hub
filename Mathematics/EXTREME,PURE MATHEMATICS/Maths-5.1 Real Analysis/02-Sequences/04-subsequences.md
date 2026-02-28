# Chapter 4: Subsequences

[← Previous: Monotone Convergence Theorem](03-monotone-convergence-theorem.md) | [Back to README](../README.md) | [Next: Bolzano-Weierstrass Theorem →](05-bolzano-weierstrass-theorem.md)

---

## Overview

A subsequence is obtained by selecting infinitely many terms from a sequence while preserving their order. Subsequences are central to analysis — they reveal hidden convergence behavior in divergent sequences and lead to the Bolzano-Weierstrass theorem.

---

## 4.1 Definition of Subsequence

**Definition 4.1.** Let $(a_n)$ be a sequence and let $n_1 < n_2 < n_3 < \cdots$ be a strictly increasing sequence of natural numbers. The sequence $(a_{n_k})_{k=1}^{\infty}$ is called a **subsequence** of $(a_n)$.

```
  Original sequence: a₁, a₂, a₃, a₄, a₅, a₆, a₇, a₈, a₉, ...
                      │       │           │       │
                      ▼       ▼           ▼       ▼
  Subsequence:       a₁,    a₃,         a₆,    a₈, ...
                    (n₁=1) (n₂=3)     (n₃=6) (n₄=8)

  KEY REQUIREMENTS:
  • Select infinitely many terms
  • Indices must be strictly increasing: n₁ < n₂ < n₃ < ...
  • Cannot rearrange the order
```

### Examples

| Sequence $(a_n)$ | Subsequence | Index sequence $(n_k)$ |
|-------------------|-------------|----------------------|
| $(1, 2, 3, 4, 5, \ldots)$ | $(2, 4, 6, 8, \ldots)$ | $n_k = 2k$ |
| $((-1)^n)$ | $(1, 1, 1, 1, \ldots)$ | $n_k = 2k$ (even terms) |
| $((-1)^n)$ | $(-1, -1, -1, \ldots)$ | $n_k = 2k - 1$ (odd terms) |
| $(1/n)$ | $(1/k^2)$ | $n_k = k^2$ |

---

## 4.2 Key Property of Index Sequences

**Lemma 4.2.** If $n_1 < n_2 < n_3 < \cdots$ is a strictly increasing sequence of natural numbers, then $n_k \geq k$ for all $k$.

**Proof.** By induction. $n_1 \geq 1$ ✓. If $n_k \geq k$, then $n_{k+1} > n_k \geq k$, so $n_{k+1} \geq k + 1$. $\blacksquare$

**Why this matters:** It guarantees that subsequences go "at least as far out" as their index, which is used in convergence proofs.

---

## 4.3 Subsequences and Convergence

**Theorem 4.3.** If $a_n \to L$, then every subsequence $(a_{n_k})$ also converges to $L$.

**Proof.** Let $\varepsilon > 0$. Since $a_n \to L$, $\exists\, N$ with $|a_n - L| < \varepsilon$ for $n \geq N$. For $k \geq N$: $n_k \geq k \geq N$, so $|a_{n_k} - L| < \varepsilon$. $\blacksquare$

### Contrapositive: Test for Divergence

**Corollary 4.4.** If a sequence has two subsequences converging to different limits, or a subsequence that diverges, then the original sequence diverges.

```
  DIVERGENCE TEST via subsequences:

  Sequence aₙ = (-1)ⁿ:

  Even subsequence: a₂, a₄, a₆, ... = 1, 1, 1, ... → 1
  Odd subsequence:  a₁, a₃, a₅, ... = -1, -1, -1, ... → -1

  Since 1 ≠ -1, the original sequence DIVERGES.

  ┌──────────────────────────────────────────────┐
  │ If ANY two subsequences have DIFFERENT limits,│
  │ the sequence does NOT converge.               │
  └──────────────────────────────────────────────┘
```

---

## 4.4 Subsequential Limits (Limit Points of a Sequence)

**Definition 4.5.** A number $L$ is a **subsequential limit** (or **limit point**) of $(a_n)$ if there exists a subsequence $(a_{n_k})$ with $a_{n_k} \to L$.

**Example:** For $a_n = \sin(n\pi/4)$, the set of subsequential limits is $\{0, \pm 1/\sqrt{2}, \pm 1\}$ — every value the sine function takes at multiples of $\pi/4$.

**Example:** For $a_n = (-1)^n(1 + 1/n)$:
- Even terms: $a_{2k} = 1 + 1/(2k) \to 1$
- Odd terms: $a_{2k-1} = -(1 + 1/(2k-1)) \to -1$
- Subsequential limits: $\{-1, 1\}$

---

## 4.5 Limit Superior and Limit Inferior

**Definition 4.6.** For a bounded sequence $(a_n)$:

$$\limsup_{n \to \infty} a_n = \lim_{n \to \infty} \sup_{k \geq n} a_k$$

$$\liminf_{n \to \infty} a_n = \lim_{n \to \infty} \inf_{k \geq n} a_k$$

**Theorem 4.7.** 
- $\limsup a_n = $ largest subsequential limit
- $\liminf a_n = $ smallest subsequential limit
- $a_n$ converges $\iff$ $\limsup a_n = \liminf a_n$ (and then both equal $\lim a_n$)

```
  lim sup and lim inf illustrated for aₙ = (-1)ⁿ/n + sin(nπ/3):

  Value
    ▲
    │  ●        ●                    sup{aₖ : k≥n}
    │     ●  ●     ●  ●  ●           decreases
    │                       ●  ●
  lim sup ─┼─── ─── ─── ─── ─── ──── → largest cluster point
    │
  lim inf ─┼─── ─── ─── ─── ─── ──── → smallest cluster point
    │●  ●  ●  ●  ●  ●  ●  ●
    │           ●        ●    ●      inf{aₖ : k≥n}
    │                                 increases
    └─────────────────────────────► n

  (aₙ) converges ⟺ lim sup = lim inf
```

---

## 4.6 Worked Examples

### Example 1: Finding All Subsequential Limits

$a_n$ cycles: $1, 2, 3, 1, 2, 3, 1, 2, 3, \ldots$ (i.e., $a_n = ((n-1) \bmod 3) + 1$)

Subsequential limits: $\{1, 2, 3\}$
- $a_{3k+1} = 1 \to 1$
- $a_{3k+2} = 2 \to 2$
- $a_{3k+3} = 3 \to 3$

$\limsup a_n = 3$, $\liminf a_n = 1$.

### Example 2: Every Real in $[0, 1]$ is a Subsequential Limit

The sequence that enumerates all rationals in $[0, 1]$ (possible since $\mathbb{Q} \cap [0, 1]$ is countable) has every point of $[0, 1]$ as a subsequential limit (by density of $\mathbb{Q}$).

---

## 4.7 The Diagonal Argument for Subsequences

When dealing with multiple sequences, we often need to extract a **common subsequence** that works for all. The **diagonal trick**:

```
  Given sequences with indices:

  Sequence 1: pick subsequence with indices n₁₁ < n₁₂ < n₁₃ < ...
  Sequence 2: from ^, pick sub-subsequence n₂₁ < n₂₂ < n₂₃ < ...
  Sequence 3: from ^, pick sub-sub-subseq  n₃₁ < n₃₂ < n₃₃ < ...
  ...

  Diagonal: nₖₖ  (the k-th element of the k-th subsequence)
  This is a subsequence that works for ALL original sequences.
```

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Subsequence | Infinite selection with increasing indices |
| $n_k \geq k$ | Indices grow at least as fast as $k$ |
| Convergent ⟹ subsequences converge | All subsequences share the same limit |
| Divergence test | Different subsequential limits ⟹ divergence |
| Subsequential limit | Limit of some subsequence |
| $\limsup a_n$ | Largest subsequential limit |
| $\liminf a_n$ | Smallest subsequential limit |
| Convergence criterion | $a_n$ converges $\iff$ $\limsup = \liminf$ |

---

## Quick Revision Questions

1. **Define "subsequence" precisely.** Why must the index sequence be strictly increasing?

2. **Prove:** If $a_n \to L$, then every subsequence converges to $L$.

3. **Find all subsequential limits** of $a_n = n \cdot \sin(n\pi/2)$. Does the sequence converge?

4. **Compute $\limsup$ and $\liminf$** for $a_n = (-1)^n + 1/n$.

5. **True or False:** If every subsequence of $(a_n)$ has a further subsequence converging to $L$, then $a_n \to L$.

6. **Explain the diagonal argument** for extracting a common subsequence. Give a context where it is used.

---

[← Previous: Monotone Convergence Theorem](03-monotone-convergence-theorem.md) | [Back to README](../README.md) | [Next: Bolzano-Weierstrass Theorem →](05-bolzano-weierstrass-theorem.md)
