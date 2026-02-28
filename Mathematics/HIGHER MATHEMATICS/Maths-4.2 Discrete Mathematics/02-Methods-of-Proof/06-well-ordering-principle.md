# Chapter 2.6: Well-Ordering Principle

[← Previous: Strong Induction](05-strong-induction.md) | [Back to README](../README.md) | [Next: Unit 3 — Set Operations →](../03-Set-Theory/01-set-operations.md)

---

## 📋 Chapter Overview

The **Well-Ordering Principle** (WOP) states that every non-empty set of non-negative integers has a **least element**. Though it seems obvious, this principle is logically equivalent to mathematical induction and provides an alternative proof technique, particularly for existence and divisibility results.

---

## 1. Statement of the Well-Ordering Principle

> **Well-Ordering Principle (WOP):** Every non-empty subset of $\mathbb{N}_0 = \{0, 1, 2, 3, \ldots\}$ has a **smallest (least) element**.

Equivalently: Every non-empty subset of $\mathbb{Z}^+ = \{1, 2, 3, \ldots\}$ has a smallest element.

### What This Means

If $S \subseteq \mathbb{N}_0$ and $S \neq \emptyset$, then $\exists m \in S$ such that $m \leq s$ for all $s \in S$.

### What This Does NOT Apply To

| Set | Has least element? | Why? |
|-----|:---:|------|
| $\{3, 7, 15, 22\}$ | ✅ Yes (3) | Finite subset of $\mathbb{N}$ |
| $\{n \in \mathbb{N} : n > 100\}$ | ✅ Yes (101) | Non-empty subset of $\mathbb{N}$ |
| $\mathbb{Z}$ (all integers) | ❌ No | Not a subset of $\mathbb{N}$ |
| $\mathbb{Q}^+$ (positive rationals) | ❌ No | Not well-ordered |
| $\mathbb{R}^+$ (positive reals) | ❌ No | $(0, 1)$ has no minimum |
| $\emptyset$ | ❌ No | Must be non-empty |

---

## 2. Equivalence of WOP, Induction, and Strong Induction

These three principles are logically equivalent — any one can be used to prove the other two:

```
     Well-Ordering Principle
            ↕
    Mathematical Induction
            ↕
      Strong Induction
```

All three are axioms about $\mathbb{N}$ — accepting any one gives you the others.

---

## 3. Proof Technique Using WOP

### General Strategy

To prove $\forall n \geq n_0, P(n)$:

1. **Assume for contradiction** that $P(n)$ fails for some $n$.
2. Let $S = \{n \geq n_0 : \neg P(n)\}$ be the set of **counterexamples**.
3. By our assumption, $S \neq \emptyset$.
4. By WOP, $S$ has a **least element** $m$.
5. Show that $m$ being the least counterexample leads to a **contradiction**.
6. Therefore $S = \emptyset$, so $P(n)$ holds for all $n \geq n_0$.

---

## 4. Worked Examples

### Example 1: Every positive integer ≥ 2 has a prime factorization

**Theorem:** Every integer $n \geq 2$ can be written as a product of primes.

**Proof using WOP:**
1. Let $S = \{n \in \mathbb{Z} : n \geq 2 \text{ and } n \text{ cannot be written as a product of primes}\}$.
2. Assume $S \neq \emptyset$ (for contradiction).
3. By WOP, $S$ has a least element $m$.
4. $m$ is not prime (otherwise $m$ is a product of one prime, contradicting $m \in S$).
5. So $m$ is composite: $m = a \cdot b$ where $2 \leq a, b < m$.
6. Since $a, b < m$ and $m$ is the **least** element of $S$, we have $a, b \notin S$.
7. Therefore $a$ and $b$ CAN be written as products of primes.
8. So $m = a \cdot b$ is also a product of primes.
9. **Contradiction:** $m \in S$ but $m$ can be factored into primes.
10. Therefore $S = \emptyset$. $\blacksquare$

---

### Example 2: Division Algorithm

**Theorem:** For any integers $a$ and $d > 0$, there exist unique integers $q$ (quotient) and $r$ (remainder) such that $a = dq + r$ and $0 \leq r < d$.

**Proof (Existence, using WOP):**
1. Let $S = \{a - dq : q \in \mathbb{Z}, \, a - dq \geq 0\}$.
2. $S$ is non-empty: choose $q$ sufficiently negative to make $a - dq > 0$.
3. By WOP, $S$ has a least element $r = a - dq_0$ for some $q_0$.
4. By construction, $r \geq 0$.
5. **Claim:** $r < d$.
6. If $r \geq d$, then $a - d(q_0 + 1) = r - d \geq 0$, so $r - d \in S$.
7. But $r - d < r$, contradicting $r$ being the least element of $S$.
8. Therefore $r < d$, and $a = dq_0 + r$ with $0 \leq r < d$. $\blacksquare$

---

### Example 3: Sum formula via WOP

**Theorem:** $\sum_{i=1}^{n} i = \frac{n(n+1)}{2}$ for all $n \geq 1$.

**Proof using WOP:**
1. Let $S = \{n \geq 1 : \sum_{i=1}^{n} i \neq \frac{n(n+1)}{2}\}$.
2. Assume $S \neq \emptyset$.
3. By WOP, let $m$ be the least element of $S$.
4. $m \neq 1$ since $\sum_{i=1}^{1} i = 1 = \frac{1 \cdot 2}{2}$.
5. So $m \geq 2$, and $m - 1 \notin S$ (since $m$ is least).
6. Therefore $\sum_{i=1}^{m-1} i = \frac{(m-1)m}{2}$.
7. $\sum_{i=1}^{m} i = \sum_{i=1}^{m-1} i + m = \frac{(m-1)m}{2} + m = \frac{m(m+1)}{2}$.
8. But this means $m \notin S$ — **contradiction!**
9. Therefore $S = \emptyset$. $\blacksquare$

---

### Example 4: GCD as a linear combination

**Theorem:** For positive integers $a, b$, $\gcd(a, b) = sa + tb$ for some integers $s, t$.

**Proof using WOP:**
1. Let $S = \{sa + tb : s, t \in \mathbb{Z}, \, sa + tb > 0\}$.
2. $S$ is non-empty (e.g., $a = 1 \cdot a + 0 \cdot b \in S$ if $a > 0$).
3. By WOP, let $d$ be the least element of $S$. So $d = s_0 a + t_0 b > 0$.
4. By the division algorithm: $a = qd + r$ where $0 \leq r < d$.
5. $r = a - qd = a - q(s_0 a + t_0 b) = (1 - qs_0)a + (-qt_0)b$.
6. If $r > 0$, then $r \in S$ and $r < d$, contradicting $d$ being least.
7. So $r = 0$, meaning $d \mid a$. Similarly, $d \mid b$.
8. If $c \mid a$ and $c \mid b$, then $c \mid (s_0 a + t_0 b) = d$.
9. Therefore $d = \gcd(a, b)$. $\blacksquare$

---

## 5. Comparison of Proof Methods

| Method | Approach | Best For |
|--------|----------|----------|
| **Direct** | Assume $p$, derive $q$ | Algebraic identities |
| **Contrapositive** | Assume $\neg q$, derive $\neg p$ | Parity/divisibility |
| **Contradiction** | Assume $\neg p$, find absurdity | Irrationality, non-existence |
| **Weak Induction** | Base case + $P(k) \Rightarrow P(k+1)$ | Sequential formulas |
| **Strong Induction** | Base + all previous → next | Recursive structures |
| **Well-Ordering** | Least counterexample → contradiction | Existence, Division Alg. |

---

## 6. Well-Ordering in Other Domains

| Set | Well-Ordered? | Reason |
|-----|:---:|------|
| $\mathbb{N}$ | ✅ | WOP axiom |
| $\mathbb{Z}^+$ | ✅ | Same as $\mathbb{N} \setminus \{0\}$ |
| $\{0, 1, 2, \ldots, n\}$ | ✅ | Finite subset of $\mathbb{N}$ |
| $\mathbb{Z}$ | ❌ | $\{\ldots, -3, -2, -1\}$ has no least element |
| $\mathbb{Q}^+$ | ❌ | Between any two, there's another |
| $\mathbb{R}^+$ | ❌ | $(0,1)$ has infimum 0 but no minimum |

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| WOP Statement | Every non-empty subset of $\mathbb{N}$ has a least element |
| Proof strategy | Assume counterexample set $S$ is non-empty; find contradiction via least element |
| Equivalence | WOP ↔ Induction ↔ Strong Induction |
| Applies to | $\mathbb{N}$, $\mathbb{Z}^+$, and their subsets |
| Does NOT apply to | $\mathbb{Z}$, $\mathbb{Q}$, $\mathbb{R}$ |
| Classic application | Division Algorithm, GCD existence |

---

## ❓ Quick Revision Questions

1. **State the Well-Ordering Principle. Does it apply to the set of negative integers?**

2. **Using WOP, prove that $\sqrt{3}$ is irrational.**

3. **Explain why the set $\{x \in \mathbb{Q} : x > 0\}$ is NOT well-ordered.**

4. **How does a WOP proof relate to proof by contradiction?**

5. **Use WOP to prove: If $a \mid bc$ and $\gcd(a, b) = 1$, then $a \mid c$.**

6. **Are strong induction and WOP logically equivalent? Explain briefly.**

---

[← Previous: Strong Induction](05-strong-induction.md) | [Back to README](../README.md) | [Next: Unit 3 — Set Operations →](../03-Set-Theory/01-set-operations.md)
