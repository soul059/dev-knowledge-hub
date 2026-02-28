# Chapter 5.2 — Compact Subsets of ℝⁿ

[← Previous: Definition and Properties](01-definition-and-properties.md) | [Back to README](../README.md) | [Next: Limit Point Compactness →](03-limit-point-compactness.md)

---

## 1. Overview

The **Heine-Borel theorem** gives a complete characterization of compact subsets of $\mathbb{R}^n$: they are exactly the **closed and bounded** sets. This is one of the most important results in analysis and provides the bridge between the abstract definition of compactness and concrete calculations.

---

## 2. Heine-Borel Theorem

> **Thm 2.1 (Heine-Borel).** A subset $K \subseteq \mathbb{R}^n$ is compact if and only if $K$ is **closed** and **bounded**.

Here "bounded" means $K \subseteq B_R(0)$ for some $R > 0$.

### Proof Outline

**($\Rightarrow$ Compact ⟹ closed and bounded):**
- Compact ⊂ Hausdorff ⟹ closed (Thm 1.6 from previous chapter).
- Bounded: $\mathbb{R}^n = \bigcup_{n=1}^{\infty} B_n(0)$ is an open cover. Finite subcover ⟹ $K \subseteq B_N(0)$ for some $N$.

**($\Leftarrow$ Closed and bounded ⟹ compact):**
- $K$ bounded ⟹ $K \subseteq [-M, M]^n$ for some $M$.
- $[-M, M]^n$ is compact (by Tychonoff or direct proof for $[a,b]$).
- $K$ is closed in a compact space ⟹ $K$ is compact. ∎

---

## 3. Compactness of $[a,b]$

> **Thm 2.2.** The closed interval $[a,b]$ is compact.

**Proof (direct, using least upper bound property):**

Let $\{U_\alpha\}$ be an open cover of $[a,b]$. Let:

$$S = \{x \in [a,b] : [a,x] \text{ is covered by finitely many } U_\alpha\}$$

- $S \neq \emptyset$ since $a \in S$.
- Let $s = \sup S$. We show $s \in S$ and $s = b$.
- $s \in U_{\alpha_0}$ for some $\alpha_0$. Since $U_{\alpha_0}$ is open, $(s - \epsilon, s + \epsilon) \cap [a,b] \subseteq U_{\alpha_0}$.
- By definition of sup, $s - \epsilon < x_0$ for some $x_0 \in S$. So $[a, x_0]$ has a finite subcover, and adding $U_{\alpha_0}$ covers $[a, s]$ (and beyond).
- If $s < b$, we can extend past $s$, contradicting $s = \sup S$.
- So $s = b$ and $[a,b]$ is compact. ∎

---

## 4. Compact Subsets of $\mathbb{R}$: Quick Reference

| Set | Closed? | Bounded? | Compact? |
|-----|:---:|:---:|:---:|
| $[0,1]$ | ✓ | ✓ | ✓ |
| $(0,1)$ | ✗ | ✓ | ✗ |
| $[0,\infty)$ | ✓ | ✗ | ✗ |
| $\{1/n : n \in \mathbb{N}\}$ | ✗ | ✓ | ✗ |
| $\{0\} \cup \{1/n : n \in \mathbb{N}\}$ | ✓ | ✓ | ✓ |
| $\mathbb{Z}$ | ✓ | ✗ | ✗ |
| Cantor set | ✓ | ✓ | ✓ |
| $\emptyset$ | ✓ | ✓ | ✓ |

---

## 5. Extreme Value Theorem

> **Thm 2.3 (Extreme Value Theorem).** If $f: K \to \mathbb{R}$ is continuous and $K$ is compact, then $f$ attains its maximum and minimum: there exist $x_*, x^* \in K$ with $f(x_*) \leq f(x) \leq f(x^*)$ for all $x \in K$.

**Proof:** $f(K)$ is compact (continuous image of compact). In $\mathbb{R}$, compact = closed + bounded. So $f(K)$ has a supremum $M$ and infimum $m$, and both belong to $f(K)$ (closed). ∎

---

## 6. Bolzano-Weierstrass Theorem

> **Thm 2.4 (Bolzano-Weierstrass).** Every bounded sequence in $\mathbb{R}^n$ has a convergent subsequence.

This is equivalent to: every closed and bounded (= compact) subset of $\mathbb{R}^n$ is **sequentially compact**.

---

## 7. Lebesgue Number Lemma

> **Lem 2.5 (Lebesgue Number Lemma).** Let $(X, d)$ be a compact metric space and $\{U_\alpha\}$ an open cover. Then there exists $\delta > 0$ (the **Lebesgue number**) such that every subset of $X$ with diameter $< \delta$ is contained in some $U_\alpha$.

$$\forall A \subseteq X,\; \text{diam}(A) < \delta \;\Rightarrow\; \exists \alpha : A \subseteq U_\alpha$$

---

## 8. The Cantor Set: A Compact Oddity

The **Cantor set** $C$ is constructed by iteratively removing middle thirds from $[0,1]$:

$$C_0 = [0,1], \quad C_1 = [0,\tfrac{1}{3}] \cup [\tfrac{2}{3}, 1], \quad C_2 = [0,\tfrac{1}{9}] \cup [\tfrac{2}{9}, \tfrac{1}{3}] \cup \cdots$$

$$C = \bigcap_{n=0}^{\infty} C_n$$

```
Cantor set construction:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
C₀: ████████████████████████████████████████
C₁: ████████████            ████████████
C₂: ████    ████            ████    ████
C₃: ██  ██  ██  ██          ██  ██  ██  ██
 ...
    0        1/3       2/3        1
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Properties of $C$:**
| Property | Value |
|----------|-------|
| Compact | ✓ (closed + bounded) |
| Uncountable | ✓ (bijection with $\{0,2\}^\mathbb{N}$) |
| Lebesgue measure | 0 |
| Perfect | ✓ (no isolated points) |
| Totally disconnected | ✓ |
| Homeomorphic to | $\{0,1\}^\mathbb{N}$ (product topology) |

---

## 9. Summary Table

| Concept | Key Point |
|---------|-----------|
| Heine-Borel | In $\mathbb{R}^n$: compact ⟺ closed + bounded |
| $[a,b]$ compact | Proved via LUB property |
| EVT | Continuous $f$ on compact $K$ attains max and min |
| Bolzano-Weierstrass | Bounded sequence in $\mathbb{R}^n$ has convergent subsequence |
| Lebesgue number | Compact metric space covers have uniform refinement size |
| Cantor set | Compact, uncountable, measure zero, totally disconnected |

---

## 10. Quick Revision Questions

1. State and prove the Heine-Borel theorem for $\mathbb{R}$.

2. Is $\{1/n : n \in \mathbb{N}\}$ compact? What about $\{0\} \cup \{1/n\}$? Explain.

3. Prove the Extreme Value Theorem using compactness.

4. Explain why the Bolzano-Weierstrass theorem is a consequence of compactness.

5. Describe the Cantor set construction and list three topological properties it has.

6. Does Heine-Borel hold in an arbitrary metric space? (Hint: consider infinite-dimensional spaces.)

---

[← Previous: Definition and Properties](01-definition-and-properties.md) | [Back to README](../README.md) | [Next: Limit Point Compactness →](03-limit-point-compactness.md)
