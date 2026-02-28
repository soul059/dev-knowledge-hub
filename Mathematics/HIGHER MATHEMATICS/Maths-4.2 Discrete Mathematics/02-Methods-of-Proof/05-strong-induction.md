# Chapter 2.5: Strong Induction

[← Previous: Mathematical Induction](04-mathematical-induction.md) | [Back to README](../README.md) | [Next: Well-Ordering Principle →](06-well-ordering-principle.md)

---

## 📋 Chapter Overview

**Strong induction** (also called **complete induction**) is a variant of mathematical induction where the inductive hypothesis assumes the statement is true for **all** values from the base case up to $k$, not just for $k$ alone. This extra power is needed for proofs where $P(k+1)$ depends on values earlier than $P(k)$.

---

## 1. Strong vs. Weak (Ordinary) Induction

| Feature | Weak Induction | Strong Induction |
|---------|:---:|:---:|
| Inductive Hypothesis | Assume $P(k)$ | Assume $P(n_0), P(n_0+1), \ldots, P(k)$ |
| Uses | Only $P(k)$ to prove $P(k+1)$ | Any/all of $P(n_0), \ldots, P(k)$ |
| Power | Equivalent (same theorems) | Equivalent (same theorems) |
| Preferred when | $P(k+1)$ depends only on $P(k)$ | $P(k+1)$ depends on earlier cases |

### Analogy

```
  Weak Induction:              Strong Induction:
  
  Each domino knocked          Each domino knocked
  by PREVIOUS one only         by ANY/ALL previous ones
  
  ┌─┐  ┌─┐  ┌─┐  ┌─┐         ┌─┐  ┌─┐  ┌─┐  ┌─┐
  │ │→ │ │→ │ │→ │ │         │ │╲ │ │╲ │ │╲ │ │
  └─┘  └─┘  └─┘  └─┘         │ │ ╲│ │ ╲│ │ ╲│ │
                               │ │──╲──╲──╲──→│ │
                               └─┘  └─┘  └─┘  └─┘
```

---

## 2. Formal Statement

**Principle of Strong Induction:**

To prove $P(n)$ for all $n \geq n_0$:

1. **Base Case:** Prove $P(n_0)$ (sometimes multiple base cases needed)
2. **Inductive Step:** For $k \geq n_0$, assume $P(n_0), P(n_0+1), \ldots, P(k)$ are ALL true. Prove $P(k+1)$.

---

## 3. Worked Examples

### Example 1: Every integer $\geq 2$ is divisible by a prime

**Theorem:** Every integer $n \geq 2$ has a prime divisor.

**Proof (Strong Induction):**

**Base Case ($n = 2$):** $2$ is prime, so it is divisible by itself (a prime). ✓

**Inductive Step:** Assume every integer from 2 to $k$ has a prime divisor. Prove it for $k + 1$.

**Case 1:** $k + 1$ is prime → It is divisible by itself. ✓

**Case 2:** $k + 1$ is composite → $k + 1 = a \cdot b$ where $2 \leq a, b < k + 1$.
- Since $2 \leq a \leq k$, by the **strong inductive hypothesis**, $a$ has a prime divisor $p$.
- Since $p \mid a$ and $a \mid (k+1)$, we have $p \mid (k+1)$. ✓

By strong induction, every integer $\geq 2$ has a prime divisor. $\blacksquare$

> **Note:** Weak induction would fail here because $P(k+1)$ doesn't depend on $P(k)$ — it depends on $P(a)$ where $a$ could be much smaller.

---

### Example 2: Fundamental Theorem of Arithmetic (Existence Part)

**Theorem:** Every integer $n \geq 2$ can be written as a product of primes.

**Proof (Strong Induction):**

**Base Case ($n = 2$):** $2 = 2$ (product of one prime). ✓

**Inductive Step:** Assume every integer from 2 to $k$ can be written as a product of primes.

**Case 1:** $k + 1$ is prime → $k + 1$ is itself a product of one prime. ✓

**Case 2:** $k + 1$ is composite → $k + 1 = a \cdot b$ where $2 \leq a, b \leq k$.
- By strong IH, $a = p_1 p_2 \cdots p_r$ and $b = q_1 q_2 \cdots q_s$.
- So $k + 1 = p_1 p_2 \cdots p_r \cdot q_1 q_2 \cdots q_s$, a product of primes. ✓

$\blacksquare$

---

### Example 3: Fibonacci numbers — $F_n < 2^n$

**Recall:** $F_1 = 1, F_2 = 1, F_n = F_{n-1} + F_{n-2}$ for $n \geq 3$.

**Theorem:** $F_n < 2^n$ for all $n \geq 1$.

**Base Cases:**
- $F_1 = 1 < 2 = 2^1$ ✓
- $F_2 = 1 < 4 = 2^2$ ✓

**Inductive Step:** Assume $F_j < 2^j$ for all $1 \leq j \leq k$ where $k \geq 2$.

$$F_{k+1} = F_k + F_{k-1} < 2^k + 2^{k-1} \quad \text{(by strong IH)}$$
$$< 2^k + 2^k = 2 \cdot 2^k = 2^{k+1}$$

So $F_{k+1} < 2^{k+1}$. $\blacksquare$

> **Why strong induction?** $F_{k+1}$ depends on BOTH $F_k$ and $F_{k-1}$.

---

### Example 4: Postage stamps — $n \geq 12$ can be formed with 4¢ and 5¢ stamps

**Theorem:** Every integer $n \geq 12$ can be written as $4a + 5b$ for non-negative integers $a, b$.

**Base Cases:**
- $12 = 4 \cdot 3 + 5 \cdot 0$ ✓
- $13 = 4 \cdot 2 + 5 \cdot 1$ ✓
- $14 = 4 \cdot 1 + 5 \cdot 2$ ✓
- $15 = 4 \cdot 0 + 5 \cdot 3$ ✓

**Inductive Step:** For $k \geq 15$, assume $P(j)$ for all $12 \leq j \leq k$.

Since $k + 1 \geq 16$, we have $k + 1 - 4 = k - 3 \geq 12$.

By strong IH, $P(k - 3)$ is true: $k - 3 = 4a + 5b$.

So $k + 1 = (k - 3) + 4 = 4a + 5b + 4 = 4(a + 1) + 5b$.

Since $a + 1 \geq 1$ and $b \geq 0$, this is valid. $\blacksquare$

---

### Example 5: Binary representation exists

**Theorem:** Every positive integer has a binary (base-2) representation.

**Proof (Strong Induction):**

**Base Case ($n = 1$):** $1 = (1)_2$ ✓

**Inductive Step:** Assume every integer from 1 to $k$ has a binary representation.

**Case 1:** $k + 1$ is even → $k + 1 = 2m$ where $1 \leq m \leq k$.
- By strong IH, $m$ has binary representation $b_r b_{r-1} \cdots b_1 b_0$.
- Then $k + 1 = 2m$ has representation $b_r b_{r-1} \cdots b_1 b_0 0$ (append 0). ✓

**Case 2:** $k + 1$ is odd → $k + 1 = 2m + 1$ where $1 \leq m \leq k$.
- By strong IH, $m$ has binary representation.
- Then $k + 1$ has representation with last bit 1. ✓

$\blacksquare$

---

## 4. When to Use Strong Induction

| Situation | Use |
|-----------|-----|
| $P(k+1)$ depends only on $P(k)$ | Weak induction |
| $P(k+1)$ depends on $P(k)$ and $P(k-1)$ | Strong induction |
| $P(k+1)$ depends on $P(j)$ for some unknown $j < k+1$ | Strong induction |
| Recursive definitions (like Fibonacci) | Strong induction |
| Prime factorization arguments | Strong induction |
| Division / splitting into smaller parts | Strong induction |

---

## 5. Multiple Base Cases

Strong induction often requires **more than one base case**, especially when the inductive step uses $P(k-1)$ or earlier values.

| Statement | Base Cases Needed |
|-----------|:-----------------:|
| Uses $P(k)$ only | 1 base case |
| Uses $P(k)$ and $P(k-1)$ | 2 base cases |
| Uses $P(k)$, $P(k-1)$, $P(k-2)$ | 3 base cases |
| Uses $P(k-c)$ | $c+1$ base cases |

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Strong Induction | Assume $P(n_0)$ through $P(k)$, prove $P(k+1)$ |
| Vs. Weak Induction | Uses ALL previous cases, not just $P(k)$ |
| Equivalence | Strong and weak induction prove the same theorems |
| Multiple base cases | Often needed when step looks back more than 1 |
| Common uses | Prime factorization, Fibonacci, recursive structures |
| Key advantage | Can use $P(j)$ for any $j \leq k$ in the proof |

---

## ❓ Quick Revision Questions

1. **Why does ordinary induction fail for proving "every integer ≥ 2 can be factored into primes"?**

2. **Prove by strong induction: Every amount of postage ≥ 8¢ can be made using 3¢ and 5¢ stamps.**

3. **Prove: $F_n \geq \left(\frac{3}{2}\right)^{n-2}$ for $n \geq 1$ (where $F_n$ is the $n$-th Fibonacci number).**

4. **How many base cases are needed if the inductive step uses $P(k-3)$?**

5. **What is the relationship between strong induction and weak induction in terms of expressive power?**

6. **Prove by strong induction that every integer $n \geq 2$ is either prime or a product of two smaller integers both $\geq 2$.**

---

[← Previous: Mathematical Induction](04-mathematical-induction.md) | [Back to README](../README.md) | [Next: Well-Ordering Principle →](06-well-ordering-principle.md)
