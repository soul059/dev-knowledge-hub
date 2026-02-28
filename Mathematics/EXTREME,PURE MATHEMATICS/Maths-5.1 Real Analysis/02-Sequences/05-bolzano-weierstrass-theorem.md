# Chapter 5: Bolzano-Weierstrass Theorem

[← Previous: Subsequences](04-subsequences.md) | [Back to README](../README.md) | [Next: Cauchy Sequences →](06-cauchy-sequences.md)

---

## Overview

The Bolzano-Weierstrass theorem is one of the crown jewels of real analysis. It guarantees that every bounded sequence has a convergent subsequence. This theorem is equivalent to completeness and is the engine behind many existence proofs in analysis.

---

## 5.1 Statement

**Theorem 5.1 (Bolzano-Weierstrass).** Every bounded sequence in $\mathbb{R}$ has a convergent subsequence.

That is: if $(a_n)$ is bounded, then there exists a subsequence $(a_{n_k})$ and a real number $L$ such that $a_{n_k} \to L$.

---

## 5.2 Proof via Nested Intervals (Bisection Method)

**Proof.** Let $(a_n)$ be bounded, say $a_n \in [a, b]$ for all $n$. We construct a nested sequence of intervals by bisection.

**Step 1.** Set $I_1 = [a, b]$. Bisect $I_1$ into $[a, (a+b)/2]$ and $[(a+b)/2, b]$. At least one half contains infinitely many terms of $(a_n)$. Call it $I_2$.

**Step 2.** Repeat: bisect $I_k$ and let $I_{k+1}$ be a half containing infinitely many terms.

```
  Bisection Method:

  I₁: [══════════════════════════════════]
       a                                b

  I₂: [════════════════]
       a            (a+b)/2
       (contains ∞ many terms)

  I₃:          [════════]
               (some subinterval with ∞ many terms)

  I₄:             [════]
                   ...

  • Each Iₖ has length (b-a)/2^(k-1) → 0
  • Each Iₖ contains infinitely many terms
  • Nested Interval Property gives a single point L
```

**Step 3.** By the Nested Interval Property, $\bigcap_{k=1}^{\infty} I_k = \{L\}$ for some $L \in \mathbb{R}$.

**Step 4.** Construct the subsequence: Pick $n_1$ so that $a_{n_1} \in I_1$. Given $n_k$, pick $n_{k+1} > n_k$ with $a_{n_{k+1}} \in I_{k+1}$ (possible since $I_{k+1}$ contains infinitely many terms).

**Step 5.** Since $a_{n_k} \in I_k$ and $I_k$ has length $(b-a)/2^{k-1} \to 0$, and $L \in I_k$ for all $k$:
$$|a_{n_k} - L| \leq \text{length}(I_k) = \frac{b-a}{2^{k-1}} \to 0$$

So $a_{n_k} \to L$. $\blacksquare$

---

## 5.3 Alternative Proof via Monotone Subsequence Lemma

**Lemma 5.2 (Monotone Subsequence Lemma).** Every sequence in $\mathbb{R}$ has a monotone subsequence.

**Proof.** Call index $m$ a **peak** if $a_m \geq a_n$ for all $n > m$ (i.e., $a_m$ is larger than everything after it).

**Case 1:** Infinitely many peaks $m_1 < m_2 < m_3 < \cdots$. By definition, $a_{m_1} \geq a_{m_2} \geq a_{m_3} \geq \cdots$. This is a decreasing (monotone) subsequence.

**Case 2:** Finitely many peaks — say the last peak is at index $M$. For $n_1 = M + 1$: since $n_1$ is not a peak, $\exists\, n_2 > n_1$ with $a_{n_2} > a_{n_1}$. Since $n_2$ is not a peak, $\exists\, n_3 > n_2$ with $a_{n_3} > a_{n_2}$. Continuing gives a strictly increasing subsequence. $\blacksquare$

```
  Peak indices — visualization:

  Case 1: Infinitely many peaks (●)

  Value
    ▲
    │ ● 
    │   ○   ●
    │     ○    ○   ●
    │               ○   ○   ●
    │                         ○   ●
    └─────────────────────────────────► n
  
  The peaks form a DECREASING subsequence.

  Case 2: Finitely many peaks

  Value
    ▲
    │ ●              last peak
    │   ○  ●───────────────
    │         ○                    ○
    │           ○               ○
    │              ○          ○
    │                 ○    ○
    │                   ○
    └─────────────────────────────────► n
  
  After the last peak, build an INCREASING subsequence.
```

**Proof of BW via Monotone Subsequence:**
By Lemma 5.2, $(a_n)$ has a monotone subsequence $(a_{n_k})$. Since $(a_n)$ is bounded, $(a_{n_k})$ is bounded and monotone. By MCT, $(a_{n_k})$ converges. $\blacksquare$

---

## 5.4 Why BW Matters

The BW theorem is the foundation for:

| Application | How BW is Used |
|-------------|---------------|
| Extreme Value Theorem | Existence of max/min on compact sets |
| Heine-Borel Theorem | Characterization of compact sets |
| Cauchy completeness | BW ⟹ Cauchy sequences converge |
| Uniform continuity | Proof on compact sets uses BW |
| Arzelà-Ascoli Theorem | Subsequences of function sequences |

```
  Logical Dependency:

  Completeness (LUB)
       │
       ▼
  Monotone Convergence Theorem
       │
       ▼
  Bolzano-Weierstrass ────► Cauchy Completeness
       │
       ├──► Extreme Value Theorem
       ├──► Heine-Borel
       └──► Uniform Continuity on Compact Sets
```

---

## 5.5 BW in Higher Dimensions (Preview)

**Theorem 5.3.** Every bounded sequence in $\mathbb{R}^n$ has a convergent subsequence.

**Proof idea (for $\mathbb{R}^2$).** Let $(\mathbf{x}_k) = ((x_k, y_k))$ be bounded. By BW in ℝ, $(x_k)$ has a convergent subsequence $(x_{k_j})$. By BW again, $(y_{k_j})$ has a convergent subsequence $(y_{k_{j_i}})$. Then $(x_{k_{j_i}}, y_{k_{j_i}})$ converges in $\mathbb{R}^2$.

This uses the **diagonal argument** from the previous chapter.

---

## 5.6 Equivalence with Completeness

```
  The following are all EQUIVALENT:

  ┌─────────────────────────┐
  │ 1. LUB Property         │
  │ 2. MCT                  │◄── All equivalent
  │ 3. Nested Intervals     │    formulations of
  │ 4. Bolzano-Weierstrass  │    COMPLETENESS
  │ 5. Cauchy Completeness  │    of ℝ
  └─────────────────────────┘

  Removing ANY one of these makes ℝ collapse to something
  like ℚ (incomplete).
```

---

## 5.7 Counterexample: BW Fails in ℚ

The sequence $a_n = $ rational approximation of $\sqrt{2}$ to $n$ decimal places. This is bounded in $\mathbb{Q}$, but no subsequence converges *in* $\mathbb{Q}$ (it would need to converge to $\sqrt{2} \notin \mathbb{Q}$).

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Bolzano-Weierstrass | Bounded sequence ⟹ convergent subsequence |
| Bisection proof | Nested intervals with infinitely many terms |
| Monotone subsequence lemma | Every sequence has a monotone subsequence |
| Alternative BW proof | Monotone subseq + bounded + MCT |
| Peak index | $a_m \geq a_n$ for all $n > m$ |
| Equivalent to completeness | BW ⟺ LUB ⟺ MCT ⟺ Cauchy completeness |
| Higher dimensions | Apply BW coordinate-by-coordinate |

---

## Quick Revision Questions

1. **State and prove the Bolzano-Weierstrass theorem** using the bisection (nested intervals) method.

2. **Prove the Monotone Subsequence Lemma.** What is a "peak index"?

3. **Give an alternative proof of BW** using the Monotone Subsequence Lemma and MCT.

4. **Does BW hold in $\mathbb{Q}$?** Give a specific counterexample.

5. **True or False:** Every sequence has a monotone subsequence. Every sequence has a convergent subsequence.

6. **Use BW to prove:** Every sequence in $[-5, 5]$ has a subsequence converging to some point in $[-5, 5]$.

---

[← Previous: Subsequences](04-subsequences.md) | [Back to README](../README.md) | [Next: Cauchy Sequences →](06-cauchy-sequences.md)
