# Chapter 3: Monotone Convergence Theorem

[← Previous: Limit Theorems](02-limit-theorems.md) | [Back to README](../README.md) | [Next: Subsequences →](04-subsequences.md)

---

## Overview

The Monotone Convergence Theorem (MCT) is one of the most powerful tools in analysis. It says that a **bounded monotone sequence must converge** — and it tells us what the limit is (the supremum or infimum). This theorem is a direct consequence of the completeness of ℝ.

---

## 3.1 Monotone Sequences

**Definition.** A sequence $(a_n)$ is:
- **Increasing** (or non-decreasing) if $a_n \leq a_{n+1}$ for all $n$
- **Strictly increasing** if $a_n < a_{n+1}$ for all $n$
- **Decreasing** (or non-increasing) if $a_n \geq a_{n+1}$ for all $n$
- **Strictly decreasing** if $a_n > a_{n+1}$ for all $n$
- **Monotone** if it is either increasing or decreasing

```
  Increasing sequence:              Decreasing sequence:

  Value                             Value
    ▲       ●  ●  ●                   ▲ ●
    │    ●                            │    ●
    │   ●                             │       ●
    │  ●                              │         ●  ●
    │ ●                               │              ●  ●  ●
    │●                                │
    └──────────────► n                └──────────────────────► n
```

### How to Prove Monotonicity

| Method | Technique |
|--------|-----------|
| Direct comparison | Show $a_{n+1} - a_n \geq 0$ (or $\leq 0$) |
| Ratio test | Show $a_{n+1}/a_n \geq 1$ (if $a_n > 0$) |
| Induction | Prove $a_n \leq a_{n+1}$ by induction on $n$ |
| Derivative | If $a_n = f(n)$ and $f'(x) \geq 0$, then increasing |

---

## 3.2 The Monotone Convergence Theorem

**Theorem 3.1 (MCT).** 
- If $(a_n)$ is increasing and bounded above, then $(a_n)$ converges and $\lim_{n \to \infty} a_n = \sup\{a_n : n \in \mathbb{N}\}$.
- If $(a_n)$ is decreasing and bounded below, then $(a_n)$ converges and $\lim_{n \to \infty} a_n = \inf\{a_n : n \in \mathbb{N}\}$.

**Proof (increasing case).** Let $S = \{a_n : n \in \mathbb{N}\}$. Since $S$ is nonempty and bounded above, $\alpha = \sup S$ exists (by completeness).

We show $a_n \to \alpha$. Let $\varepsilon > 0$. By the ε-characterization of supremum, $\exists\, N$ with $a_N > \alpha - \varepsilon$.

Since $(a_n)$ is increasing, for $n \geq N$:
$$\alpha - \varepsilon < a_N \leq a_n \leq \alpha$$
So $|a_n - \alpha| = \alpha - a_n < \varepsilon$. $\blacksquare$

```
  Visual Proof:

  Value
    ▲
  α ─┤─────────────────────────────── sup{aₙ}
    │                     ● ● ● ● ●  ← aₙ approaching α
  α-ε┤· · · · · · ● · · · · · · · ·
    │          ●
    │       ●
    │    ●
    │  ●
    │ ●
    │●
    └──────────────────────────────► n
              ▲ N
              │
    From N onward, aₙ is between α-ε and α
    (it's ≥ aₙ by monotonicity, ≤ α by definition of sup)
```

---

## 3.3 Why Completeness is Necessary

In $\mathbb{Q}$, the MCT is **false**:

Consider $a_n = $ decimal expansion of $\sqrt{2}$ truncated to $n$ digits:
$$a_1 = 1,\quad a_2 = 1.4,\quad a_3 = 1.41,\quad a_4 = 1.414,\quad \ldots$$

This sequence is increasing and bounded above (by 2) in $\mathbb{Q}$, but it does NOT converge in $\mathbb{Q}$ since $\sqrt{2} \notin \mathbb{Q}$.

---

## 3.4 Worked Examples

### Example 1: $a_n = \left(1 + \frac{1}{n}\right)^n \to e$

**Step 1: Monotonicity.** Using AM-GM or the binomial theorem, one can show $(a_n)$ is increasing.

**Step 2: Boundedness.** By the binomial theorem:
$$a_n = \sum_{k=0}^{n} \binom{n}{k} \frac{1}{n^k} \leq 1 + \sum_{k=1}^{n} \frac{1}{k!} \leq 1 + \sum_{k=1}^{n} \frac{1}{2^{k-1}} < 1 + 2 = 3$$

**Step 3: By MCT,** $(a_n)$ converges to some $e = \sup\{a_n\} \leq 3$.

### Example 2: Recursively Defined Sequences

Let $a_1 = 1$ and $a_{n+1} = \sqrt{2 + a_n}$.

**Claim:** $(a_n)$ is increasing and bounded above by 2.

**Proof by induction.**

*Base:* $a_1 = 1$, $a_2 = \sqrt{3} \approx 1.73$. So $a_1 < a_2$ ✓ and $a_1 < 2$ ✓.

*Inductive step (bounded):* If $a_n < 2$, then $a_{n+1} = \sqrt{2 + a_n} < \sqrt{2 + 2} = 2$. ✓

*Inductive step (increasing):* If $a_n < a_{n+1}$, then $2 + a_n < 2 + a_{n+1}$, so $\sqrt{2 + a_n} < \sqrt{2 + a_{n+1}}$, i.e., $a_{n+1} < a_{n+2}$. ✓

By MCT, $L = \lim a_n$ exists. Taking limits in $a_{n+1} = \sqrt{2 + a_n}$:
$$L = \sqrt{2 + L} \implies L^2 = 2 + L \implies L^2 - L - 2 = 0 \implies (L-2)(L+1) = 0$$

Since $a_n > 0$, we have $L \geq 0$, so $L = 2$.

```
  Recursive sequence aₙ₊₁ = √(2 + aₙ):

  Value
    ▲
  2 ─┤──────────────── ● ● ● ● ● ── limit
    │              ●
    │           ●
    │        ●
    │     ●
    │  ●
  1 ─┤●
    │
    └──────────────────────────────► n
       1  2  3  4  5  6  7  8  ...

  Strategy: (1) Prove bounded  (2) Prove monotone
            (3) MCT gives convergence
            (4) Solve for limit using the recursion
```

### Example 3: Nested Radicals

$$a_n = \sqrt{2 + \sqrt{2 + \sqrt{2 + \cdots}}} \quad (n \text{ nested radicals})$$

This is exactly the recursion $a_1 = \sqrt{2}$, $a_{n+1} = \sqrt{2 + a_n}$, which converges to 2 by the analysis above.

---

## 3.5 General Strategy for Recursive Sequences

```
  Given: a₁ = c, aₙ₊₁ = f(aₙ)

  Step 1: GUESS the limit L
          Solve L = f(L) to find candidates

  Step 2: PROVE boundedness
          Show aₙ ≤ M (or aₙ ≥ m) by induction

  Step 3: PROVE monotonicity
          Show aₙ₊₁ - aₙ ≥ 0 (or ≤ 0) by induction

  Step 4: APPLY MCT
          Conclude (aₙ) converges

  Step 5: IDENTIFY the limit
          Take limits in the recursion:
          L = f(L), choose the correct root
```

---

## 3.6 Euler's Number $e$

The number $e$ can be characterized as:

$$e = \lim_{n \to \infty} \left(1 + \frac{1}{n}\right)^n = \sum_{n=0}^{\infty} \frac{1}{n!} \approx 2.71828\ldots$$

| $n$ | $(1 + 1/n)^n$ | Approximation |
|-----|---------------|---------------|
| 1 | 2.000000 | |
| 2 | 2.250000 | |
| 5 | 2.488320 | |
| 10 | 2.593742 | |
| 100 | 2.704814 | |
| 1000 | 2.716924 | |
| $\infty$ | $e$ | 2.718281828... |

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Monotone sequence | Increasing or decreasing |
| MCT | Bounded + monotone ⟹ convergent |
| Limit = sup or inf | Increasing: limit = sup; Decreasing: limit = inf |
| Requires completeness | MCT fails in ℚ |
| Recursive sequences | Prove bounded + monotone by induction, then apply MCT |
| Finding the limit | Solve $L = f(L)$ from the recursion |
| Euler's number | $e = \lim(1 + 1/n)^n$, proved via MCT |

---

## Quick Revision Questions

1. **State and prove the Monotone Convergence Theorem.** Where exactly does completeness enter?

2. **Give an example** showing MCT fails in $\mathbb{Q}$.

3. **Prove that $a_1 = 2$, $a_{n+1} = \frac{1}{2}(a_n + 3/a_n)$** converges and find its limit.

4. **Let $a_1 = 0$, $a_{n+1} = \sqrt{2a_n + 3}$.** Show $(a_n)$ is increasing and bounded, find the limit.

5. **True or False:** Every bounded sequence is convergent. Every monotone sequence is convergent.

6. **Prove that $(1 + 1/n)^n$ is bounded above by 3** using the binomial theorem.

---

[← Previous: Limit Theorems](02-limit-theorems.md) | [Back to README](../README.md) | [Next: Subsequences →](04-subsequences.md)
