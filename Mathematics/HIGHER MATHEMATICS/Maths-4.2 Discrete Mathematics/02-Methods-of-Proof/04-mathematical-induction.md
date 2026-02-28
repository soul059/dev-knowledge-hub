# Chapter 2.4: Mathematical Induction

[← Previous: Proof by Contradiction](03-proof-by-contradiction.md) | [Back to README](../README.md) | [Next: Strong Induction →](05-strong-induction.md)

---

## 📋 Chapter Overview

**Mathematical induction** is a powerful proof technique for statements involving natural numbers. It works like a chain of dominoes — proving the first one falls and that each one knocks down the next guarantees they all fall.

---

## 1. The Principle of Mathematical Induction (PMI)

To prove: $\forall n \geq n_0, \, P(n)$ is true

**Two steps:**

| Step | Name | What to Do |
|------|------|-----------|
| **Step 1** | Base Case | Prove $P(n_0)$ is true |
| **Step 2** | Inductive Step | Prove: if $P(k)$ is true, then $P(k+1)$ is true |

If both steps hold, then $P(n)$ is true for all $n \geq n_0$.

### The Domino Analogy

```
  Base Case        Inductive Step
  (first falls)    (each knocks next)
  
  ┌─┐  ┌─┐  ┌─┐  ┌─┐  ┌─┐  ┌─┐  ┌─┐
  │ │  │ │  │ │  │ │  │ │  │ │  │ │  ...
  │ │\ │ │\ │ │\ │ │\ │ │\ │ │\ │ │\
  │ │ \│ │ \│ │ \│ │ \│ │ \│ │ \│ │ \
  └─┘  └─┘  └─┘  └─┘  └─┘  └─┘  └─┘
  n₀   n₀+1 n₀+2 n₀+3  ...
  
  P(n₀) → P(n₀+1) → P(n₀+2) → P(n₀+3) → ...
```

---

## 2. Why Induction Works

Induction is based on the axiom: if a set $S \subseteq \mathbb{N}$ satisfies:
1. $n_0 \in S$
2. $k \in S \Rightarrow k+1 \in S$

Then $S = \{n_0, n_0+1, n_0+2, \ldots\} = \{n \in \mathbb{N} : n \geq n_0\}$.

---

## 3. Worked Examples

### Example 1: Sum of first $n$ natural numbers

**Theorem:** $\displaystyle\sum_{i=1}^{n} i = \frac{n(n+1)}{2}$ for all $n \geq 1$.

**Base Case ($n = 1$):**
$$\text{LHS} = 1, \quad \text{RHS} = \frac{1 \cdot 2}{2} = 1 \quad \checkmark$$

**Inductive Step:** Assume $P(k)$: $\sum_{i=1}^{k} i = \frac{k(k+1)}{2}$ (Inductive Hypothesis)

Prove $P(k+1)$: $\sum_{i=1}^{k+1} i = \frac{(k+1)(k+2)}{2}$

$$\sum_{i=1}^{k+1} i = \underbrace{\sum_{i=1}^{k} i}_{\text{IH: } \frac{k(k+1)}{2}} + (k+1)$$
$$= \frac{k(k+1)}{2} + (k+1) = \frac{k(k+1) + 2(k+1)}{2} = \frac{(k+1)(k+2)}{2} \quad \checkmark$$

By PMI, the formula holds for all $n \geq 1$. $\blacksquare$

---

### Example 2: Sum of first $n$ squares

**Theorem:** $\displaystyle\sum_{i=1}^{n} i^2 = \frac{n(n+1)(2n+1)}{6}$ for all $n \geq 1$.

**Base Case ($n = 1$):**
$$\text{LHS} = 1, \quad \text{RHS} = \frac{1 \cdot 2 \cdot 3}{6} = 1 \quad \checkmark$$

**Inductive Step:** Assume $\sum_{i=1}^{k} i^2 = \frac{k(k+1)(2k+1)}{6}$.

$$\sum_{i=1}^{k+1} i^2 = \frac{k(k+1)(2k+1)}{6} + (k+1)^2$$
$$= \frac{k(k+1)(2k+1) + 6(k+1)^2}{6}$$
$$= \frac{(k+1)[k(2k+1) + 6(k+1)]}{6}$$
$$= \frac{(k+1)(2k^2 + k + 6k + 6)}{6}$$
$$= \frac{(k+1)(2k^2 + 7k + 6)}{6}$$
$$= \frac{(k+1)(k+2)(2k+3)}{6}$$
$$= \frac{(k+1)((k+1)+1)(2(k+1)+1)}{6} \quad \checkmark$$

$\blacksquare$

---

### Example 3: Divisibility — $6 \mid (n^3 - n)$ for all $n \geq 1$

**Base Case ($n = 1$):** $1^3 - 1 = 0 = 6 \cdot 0$ ✓

**Inductive Step:** Assume $6 \mid (k^3 - k)$, i.e., $k^3 - k = 6m$ for some integer $m$.

$$(k+1)^3 - (k+1) = k^3 + 3k^2 + 3k + 1 - k - 1$$
$$= (k^3 - k) + 3k^2 + 3k$$
$$= (k^3 - k) + 3k(k + 1)$$

Now, $k(k+1)$ is always even (product of consecutive integers), so $k(k+1) = 2t$.
Thus $3k(k+1) = 6t$.

So $(k+1)^3 - (k+1) = 6m + 6t = 6(m + t)$.

Therefore $6 \mid ((k+1)^3 - (k+1))$. $\blacksquare$

---

### Example 4: Inequality — $2^n > n$ for all $n \geq 1$

**Base Case ($n = 1$):** $2^1 = 2 > 1$ ✓

**Inductive Step:** Assume $2^k > k$ for some $k \geq 1$.

$$2^{k+1} = 2 \cdot 2^k > 2 \cdot k = k + k \geq k + 1$$

(since $k \geq 1$ implies $k \geq 1$, so $k + k \geq k + 1$)

Therefore $2^{k+1} > k + 1$. $\blacksquare$

---

### Example 5: Geometric series

**Theorem:** $\displaystyle\sum_{i=0}^{n} r^i = \frac{r^{n+1} - 1}{r - 1}$ for $r \neq 1$ and $n \geq 0$.

**Base Case ($n = 0$):**
$$\text{LHS} = r^0 = 1, \quad \text{RHS} = \frac{r - 1}{r - 1} = 1 \quad \checkmark$$

**Inductive Step:** Assume the formula holds for $k$:

$$\sum_{i=0}^{k+1} r^i = \sum_{i=0}^{k} r^i + r^{k+1} = \frac{r^{k+1} - 1}{r - 1} + r^{k+1}$$
$$= \frac{r^{k+1} - 1 + r^{k+1}(r - 1)}{r - 1} = \frac{r^{k+1} - 1 + r^{k+2} - r^{k+1}}{r - 1} = \frac{r^{k+2} - 1}{r - 1} \quad \checkmark$$

$\blacksquare$

---

## 4. Common Mistakes in Induction

| Mistake | Why it's Wrong |
|---------|---------------|
| **Forgetting base case** | Inductive step alone proves nothing |
| **Wrong base case** | Must match the starting value $n_0$ |
| **Not using the inductive hypothesis** | Must explicitly use $P(k)$ to prove $P(k+1)$ |
| **Proving $P(k)$ instead of $P(k+1)$** | Must prove the NEXT case, not the current one |
| **Circular reasoning** | Cannot assume $P(k+1)$ while proving it |

### Classic Trap: "All horses are the same color"

The inductive step fails at the base case $n = 1 \rightarrow n = 2$: when $n = 2$, removing one horse from each end gives two single-horse sets that DON'T overlap. The argument requires overlap, which fails at this critical step.

---

## 5. Proof Template

```
  Theorem: P(n) for all n ≥ n₀.
  
  Proof (by mathematical induction):
  
  Base Case: [Show P(n₀) is true]
  
  Inductive Hypothesis: Assume P(k) is true for some 
  arbitrary k ≥ n₀. That is, assume [state P(k)].
  
  Inductive Step: We must show P(k+1). That is, [state P(k+1)].
  [Derivation using P(k)]
  This completes the inductive step.
  
  By the Principle of Mathematical Induction, P(n) is true
  for all n ≥ n₀. ■
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Base Case | Verify $P(n_0)$ directly |
| Inductive Hypothesis | Assume $P(k)$ for arbitrary $k \geq n_0$ |
| Inductive Step | Prove $P(k) \Rightarrow P(k+1)$ |
| Result | $P(n)$ true for all $n \geq n_0$ |
| Analogy | Domino effect |
| Key requirement | Must USE the inductive hypothesis |
| Common uses | Sums, divisibility, inequalities |

---

## ❓ Quick Revision Questions

1. **Prove by induction: $\sum_{i=1}^{n} (2i - 1) = n^2$.**

2. **Prove: $n! > 2^n$ for all $n \geq 4$.**

3. **What goes wrong if we skip the base case?** Give an example of a false statement where the inductive step holds.

4. **Prove: $3 \mid (4^n - 1)$ for all $n \geq 1$.**

5. **Prove: $1 + 2 + 4 + \cdots + 2^n = 2^{n+1} - 1$.**

6. **Why is it essential to use the inductive hypothesis in the inductive step?**

---

[← Previous: Proof by Contradiction](03-proof-by-contradiction.md) | [Back to README](../README.md) | [Next: Strong Induction →](05-strong-induction.md)
