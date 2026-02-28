# Chapter 2: Limit Theorems

[← Previous: Convergence Definition](01-convergence-definition.md) | [Back to README](../README.md) | [Next: Monotone Convergence Theorem →](03-monotone-convergence-theorem.md)

---

## Overview

Rather than proving convergence from the ε-N definition every time, we develop **algebraic limit theorems** that let us compute limits of combinations of sequences. These theorems — along with the Squeeze Theorem — are the primary tools for evaluating limits.

---

## 2.1 Algebraic Limit Theorems

**Theorem 2.1 (Algebraic Limit Theorem).** Suppose $a_n \to a$ and $b_n \to b$. Then:

| Rule | Statement | Formula |
|------|-----------|---------|
| (i) Sum | The sum converges | $\lim(a_n + b_n) = a + b$ |
| (ii) Scalar | Scalar multiples converge | $\lim(c \cdot a_n) = c \cdot a$ |
| (iii) Product | The product converges | $\lim(a_n \cdot b_n) = a \cdot b$ |
| (iv) Quotient | The quotient converges (if $b \neq 0$) | $\lim(a_n / b_n) = a / b$ |

---

### Proof of (i): Sum Rule

**Proof.** Let $\varepsilon > 0$. Since $a_n \to a$, $\exists\, N_1$ with $|a_n - a| < \varepsilon/2$ for $n \geq N_1$. Since $b_n \to b$, $\exists\, N_2$ with $|b_n - b| < \varepsilon/2$ for $n \geq N_2$.

Let $N = \max(N_1, N_2)$. For $n \geq N$:
$$|(a_n + b_n) - (a + b)| \leq |a_n - a| + |b_n - b| < \frac{\varepsilon}{2} + \frac{\varepsilon}{2} = \varepsilon$$
$\blacksquare$

```
  Key idea: Split ε into two halves

  ε-band for (aₙ+bₙ):     ◄────── ε ──────►
                           ◄── ε/2 ──►◄── ε/2 ──►
                           for aₙ-a    for bₙ-b

  Choose N = max(N₁, N₂) so BOTH conditions hold.
```

### Proof of (iii): Product Rule

**Proof.** Write $a_n b_n - ab = a_n b_n - a_n b + a_n b - ab = a_n(b_n - b) + b(a_n - a)$.

Since $(a_n)$ converges, it is bounded: $|a_n| \leq M$ for some $M > 0$.

Let $\varepsilon > 0$. Choose $N_1$ with $|b_n - b| < \frac{\varepsilon}{2M}$ for $n \geq N_1$, and $N_2$ with $|a_n - a| < \frac{\varepsilon}{2(|b|+1)}$ for $n \geq N_2$.

For $n \geq N = \max(N_1, N_2)$:
$$|a_n b_n - ab| \leq |a_n||b_n - b| + |b||a_n - a| < M \cdot \frac{\varepsilon}{2M} + |b| \cdot \frac{\varepsilon}{2(|b|+1)} < \varepsilon$$
$\blacksquare$

### Proof of (iv): Quotient Rule (Key Lemma)

**Lemma 2.2.** If $b_n \to b \neq 0$, then $1/b_n \to 1/b$.

**Proof.** Since $b \neq 0$, take $\varepsilon_0 = |b|/2 > 0$. There exists $N_0$ with $|b_n - b| < |b|/2$ for $n \geq N_0$. By reverse triangle inequality, $|b_n| > |b|/2$ for $n \geq N_0$.

Now:
$$\left|\frac{1}{b_n} - \frac{1}{b}\right| = \frac{|b_n - b|}{|b_n||b|} < \frac{|b_n - b|}{(|b|/2)|b|} = \frac{2|b_n - b|}{|b|^2}$$

Given $\varepsilon > 0$, choose $N_1$ with $|b_n - b| < \varepsilon |b|^2 / 2$ for $n \geq N_1$. Then for $n \geq \max(N_0, N_1)$: $|1/b_n - 1/b| < \varepsilon$. $\blacksquare$

---

## 2.2 The Squeeze Theorem

**Theorem 2.3 (Squeeze Theorem).** Suppose $a_n \leq b_n \leq c_n$ for all $n \geq N_0$, and $a_n \to L$ and $c_n \to L$. Then $b_n \to L$.

**Proof.** Let $\varepsilon > 0$. Choose $N_1$ with $|a_n - L| < \varepsilon$ for $n \geq N_1$, and $N_2$ with $|c_n - L| < \varepsilon$ for $n \geq N_2$. For $n \geq N = \max(N_0, N_1, N_2)$:
$$L - \varepsilon < a_n \leq b_n \leq c_n < L + \varepsilon$$
So $|b_n - L| < \varepsilon$. $\blacksquare$

```
  Squeeze Theorem — Visual:

  Value
    ▲
    │    cₙ
    │    ╲  ╱╲  ╱╲
    │     ╲╱  ╲╱  ╲────── → L
  L ├──────────bₙ──────────────────
    │     ╱╲  ╱╲  ╱────── → L
    │    ╱  ╲╱  ╲╱
    │   aₙ
    │
    └─────────────────────────────► n

  bₙ is "squeezed" between aₙ and cₙ,
  both of which converge to L.
  So bₙ must also converge to L.
```

### Example: $\lim_{n \to \infty} \frac{\sin n}{n} = 0$

Since $-1 \leq \sin n \leq 1$:
$$\frac{-1}{n} \leq \frac{\sin n}{n} \leq \frac{1}{n}$$

Both $-1/n \to 0$ and $1/n \to 0$, so by Squeeze: $\frac{\sin n}{n} \to 0$.

---

## 2.3 Order Limit Theorem

**Theorem 2.4 (Order Limit Theorem).** Suppose $a_n \to a$ and $b_n \to b$.

**(i)** If $a_n \geq 0$ for all $n \geq N_0$, then $a \geq 0$.

**(ii)** If $a_n \leq b_n$ for all $n \geq N_0$, then $a \leq b$.

**Warning:** $a_n < b_n$ does NOT imply $a < b$! Example: $0 < 1/n$ but $0 = 0$.

**Proof of (i).** By contradiction. If $a < 0$, take $\varepsilon = -a > 0$. Then $\exists\, N$ with $|a_n - a| < -a$, so $a_n < a + (-a) = 0$ for $n \geq N$, contradicting $a_n \geq 0$. $\blacksquare$

---

## 2.4 Useful Limit Results

| Sequence | Limit | Condition |
|----------|-------|-----------|
| $1/n^p$ | $0$ | $p > 0$ |
| $c^n$ | $0$ | $\|c\| < 1$ |
| $n^{1/n}$ | $1$ | — |
| $c^{1/n}$ | $1$ | $c > 0$ |
| $(1 + 1/n)^n$ | $e$ | — |
| $n!/n^n$ | $0$ | — |
| $n^p/c^n$ | $0$ | $c > 1$, any $p$ |

### Example: $c^n \to 0$ for $|c| < 1$

**Proof.** Let $|c| < 1$, so $1/|c| > 1$. Write $1/|c| = 1 + h$ with $h > 0$. By Bernoulli: $(1+h)^n \geq 1 + nh$. So:
$$|c^n| = |c|^n = \frac{1}{(1+h)^n} \leq \frac{1}{1 + nh} < \frac{1}{nh}$$

Given $\varepsilon > 0$, choose $N > 1/(h\varepsilon)$. For $n \geq N$: $|c^n| < 1/(nh) \leq 1/(Nh) < \varepsilon$. $\blacksquare$

---

## 2.5 Cesàro Means

**Theorem 2.5.** If $a_n \to L$, then the **Cesàro means** $\sigma_n = \frac{a_1 + a_2 + \cdots + a_n}{n} \to L$.

The converse is false: $a_n = (-1)^n$ has $\sigma_n \to 0$, but $(a_n)$ diverges.

---

## 2.6 Decision Chart for Computing Limits

```
  How to find lim aₙ:
  ════════════════════

  Is it a ratio of polynomials in n?
  │
  ├── YES → Divide top and bottom by highest power of n
  │         Example: (3n²+1)/(5n²-2) → 3/5
  │
  └── NO → Can you factor or simplify?
       │
       ├── YES → Simplify, then apply limit laws
       │
       └── NO → Try these strategies:
            │
            ├── Squeeze theorem (bound the sequence)
            ├── Algebraic manipulation (rationalize, etc.)
            ├── Monotone convergence (show bounded + monotone)
            └── Known limits table (above)
```

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Sum rule | $\lim(a_n + b_n) = \lim a_n + \lim b_n$ |
| Product rule | $\lim(a_n b_n) = \lim a_n \cdot \lim b_n$ |
| Quotient rule | $\lim(a_n/b_n) = \lim a_n / \lim b_n$ (if $\lim b_n \neq 0$) |
| Squeeze theorem | $a_n \leq b_n \leq c_n$, $a_n, c_n \to L$ ⟹ $b_n \to L$ |
| Order limit theorem | $a_n \leq b_n$ for large $n$ ⟹ $\lim a_n \leq \lim b_n$ |
| Strict inequality lost | $a_n < b_n$ does NOT give $\lim a_n < \lim b_n$ |
| Cesàro means | Averaging preserves convergence (but not vice versa) |

---

## Quick Revision Questions

1. **State and prove the Sum Rule** for limits of sequences.

2. **Why does the Product Rule proof require boundedness** of $(a_n)$? Where is this used?

3. **Use the Squeeze Theorem** to show $\lim_{n \to \infty} \frac{n!}{n^n} = 0$.

4. **True or False:** If $a_n < b_n$ for all $n$ and both converge, then $\lim a_n < \lim b_n$. Give a proof or counterexample.

5. **Compute** $\lim_{n \to \infty} \frac{3n^3 - 2n + 1}{5n^3 + 7n^2}$ using the Algebraic Limit Theorem.

6. **Give an example** where the Cesàro mean converges but the original sequence diverges.

---

[← Previous: Convergence Definition](01-convergence-definition.md) | [Back to README](../README.md) | [Next: Monotone Convergence Theorem →](03-monotone-convergence-theorem.md)
