# Chapter 6: Extreme Value Theorem

[← Previous: Intermediate Value Theorem](05-intermediate-value-theorem.md) | [Back to README](../README.md) | [Next: Uniform Continuity →](07-uniform-continuity.md)

---

## Overview

The **Extreme Value Theorem (EVT)** guarantees that a continuous function on a compact set attains its maximum and minimum. This is essential for optimization and is a direct consequence of the preservation of compactness.

---

## 6.1 Statement

**Theorem 6.1 (Extreme Value Theorem).** If $f: K \to \mathbb{R}$ is continuous and $K$ is compact (in $\mathbb{R}$: closed and bounded), then $f$ attains its supremum and infimum on $K$:

$$\exists\, x_M, x_m \in K: \quad f(x_m) \leq f(x) \leq f(x_M) \quad \forall\, x \in K$$

```
  EVT on [a, b]:

  f(x) ▲
       │    ● ← maximum f(x_M) attained!
       │   ╱ ╲
       │  ╱   ╲     ╱ ╲
       │ ╱     ╲   ╱   ╲
       │╱       ╲ ╱     ╲
       │         ●       ╲
       │    minimum ←      ╲ ←(also check endpoints)
       └──┼────────────────┼──► x
          a                b

  ✅ Guaranteed: max and min exist and are achieved.
  ❌ Fails if domain not compact: 1/x on (0,1) — no max!
```

---

## 6.2 Proof

**Proof.** $K$ compact + $f$ continuous $\Rightarrow$ $f(K)$ is compact (Theorem 4.3) $\Rightarrow$ $f(K)$ is closed and bounded.

Since $f(K)$ is bounded, $M = \sup f(K)$ exists. Since $f(K)$ is closed, $M \in f(K)$ (closed sets contain their limit points; $M$ is a limit point of $f(K)$ since it's the supremum of a bounded set). So $\exists\, x_M \in K$ with $f(x_M) = M$.

Similarly for the infimum. $\blacksquare$

**Alternatively (sequential proof):** Let $M = \sup\{f(x) : x \in K\}$. Choose $(x_n) \subseteq K$ with $f(x_n) \to M$. By sequential compactness, $(x_n)$ has a subsequence $x_{n_k} \to x_M \in K$. By continuity, $f(x_M) = \lim f(x_{n_k}) = M$. $\blacksquare$

---

## 6.3 Why Each Hypothesis is Needed

| Hypothesis Dropped | Counterexample | What fails |
|-------------------|----------------|------------|
| Continuity | $f(x) = \begin{cases} x & x \in [0,1) \\ 0 & x = 1 \end{cases}$ | $\sup = 1$ not attained |
| Bounded | $f(x) = x$ on $[0, \infty)$ | No supremum |
| Closed | $f(x) = x$ on $(0, 1)$ | $\sup = 1$, $\inf = 0$ not attained |

```
  Failures when hypotheses are violated:

  Not bounded:           Not closed:          Not continuous:
  f(x) = x on [0,∞)    f(x) = 1/x on (0,1]  f with "hole" at max

  ▲ ╱                    ▲                     ▲    ○ ← not attained
    ╱                    │╲                       ╱╲
   ╱  no max!            │ ╲  no max!            ╱  ╲
  ╱                      │  ╲──── 1             ╱  ● ╲
  ►                      └──────►              └──────►
```

---

## 6.4 Applications

**Application 1: Boundedness.** Continuous functions on $[a,b]$ are bounded (immediate corollary).

**Application 2: Optimization.** To find the maximum of $f$ on $[a,b]$:
1. Find critical points (where $f' = 0$ or DNE) in $(a,b)$
2. Evaluate $f$ at critical points and endpoints
3. The largest value is the maximum (EVT guarantees it exists)

**Application 3: Norm attainment.** On any compact set, the distance function $d(x, p) = |x - p|$ attains its minimum (closest point) and maximum (farthest point).

---

## 6.5 Weierstrass' Theorem (Generalized EVT)

**Theorem 6.2.** If $K$ is a compact metric space, $f: K \to \mathbb{R}$ is continuous, and $K \neq \emptyset$, then $f$ is bounded and attains its bounds.

This extends EVT to any compact metric space.

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| EVT | Continuous $f$ on compact $K$ attains max and min |
| Proof | $f(K)$ compact → closed + bounded → sup/inf attained |
| Needs compactness | $(0,1)$ fails; $[0,\infty)$ fails |
| Needs continuity | Discontinuous functions can fail to attain bounds |
| Application | Guarantees existence of global max/min in optimization |

---

## Quick Revision Questions

1. **State and prove** the Extreme Value Theorem.

2. **Give** three examples showing each hypothesis (continuous, closed, bounded) is necessary.

3. **Use EVT** to show: if $f: [a,b] \to \mathbb{R}$ is continuous, then $f$ is bounded.

4. **Does EVT** hold for $f(x) = x^2$ on $(-1, 1)$? On $[-1, 1]$? Explain.

5. **Prove** that every continuous function $f: [0,1] \to \mathbb{R}$ attains its maximum, using the sequential characterization.

---

[← Previous: Intermediate Value Theorem](05-intermediate-value-theorem.md) | [Back to README](../README.md) | [Next: Uniform Continuity →](07-uniform-continuity.md)
