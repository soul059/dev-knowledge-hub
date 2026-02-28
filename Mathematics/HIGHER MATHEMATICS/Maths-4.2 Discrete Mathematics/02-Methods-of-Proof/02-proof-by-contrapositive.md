# Chapter 2.2: Proof by Contrapositive

[← Previous: Direct Proof](01-direct-proof.md) | [Back to README](../README.md) | [Next: Proof by Contradiction →](03-proof-by-contradiction.md)

---

## 📋 Chapter Overview

Proof by contrapositive leverages the logical equivalence $p \rightarrow q \equiv \neg q \rightarrow \neg p$. Instead of proving "if $p$ then $q$" directly, we prove "if not $q$ then not $p$" — which is logically equivalent but often easier.

---

## 1. The Contrapositive

For any implication $p \rightarrow q$:

| Statement | Form | Relation to Original |
|-----------|------|---------------------|
| **Original** | $p \rightarrow q$ | — |
| **Converse** | $q \rightarrow p$ | NOT equivalent |
| **Inverse** | $\neg p \rightarrow \neg q$ | NOT equivalent |
| **Contrapositive** | $\neg q \rightarrow \neg p$ | **EQUIVALENT** ✅ |

**Why it works:**

| $p$ | $q$ | $p \rightarrow q$ | $\neg q$ | $\neg p$ | $\neg q \rightarrow \neg p$ |
|:---:|:---:|:---:|:---:|:---:|:---:|
| T | T | T | F | F | T |
| T | F | F | T | F | F |
| F | T | T | F | T | T |
| F | F | T | T | T | T |

Columns 3 and 6 are identical — the original and contrapositive are logically equivalent.

---

## 2. Proof Strategy

To prove: $p \rightarrow q$

**Contrapositive method:**
1. **Assume** $\neg q$ (the negation of the conclusion)
2. **Derive** $\neg p$ (the negation of the hypothesis)
3. **Conclude** that $p \rightarrow q$ by contrapositive equivalence

```
  Direct:             Contrapositive:
  
  Assume p             Assume ¬q
     ↓                    ↓
  Derive q             Derive ¬p
     ↓                    ↓
  ∴ p → q              ∴ ¬q → ¬p ≡ p → q
```

---

## 3. When to Use Contrapositive

Use contrapositive when:
- The conclusion involves **"not"** or **"is not"**
- Working with the negation of the conclusion is **easier** than working with the hypothesis
- The statement involves **divisibility, parity**, or similar properties where the contrapositive gives you a more useful starting point
- Direct proof seems to **hit a dead end**

---

## 4. Worked Examples

### Example 1: If $n^2$ is even, then $n$ is even

**Direct proof is tricky** — starting from "$n^2$ is even" doesn't immediately tell us about $n$.

**Contrapositive:** If $n$ is NOT even (odd), then $n^2$ is NOT even (odd).

**Proof by Contrapositive:**
1. Assume $n$ is odd.
2. Then $n = 2k + 1$ for some integer $k$.
3. $n^2 = (2k+1)^2 = 4k^2 + 4k + 1 = 2(2k^2 + 2k) + 1$.
4. Since $2k^2 + 2k$ is an integer, $n^2 = 2m + 1$ where $m = 2k^2 + 2k$.
5. So $n^2$ is odd (not even).
6. By contrapositive, if $n^2$ is even, then $n$ is even. $\blacksquare$

---

### Example 2: If $3n + 2$ is odd, then $n$ is odd

**Contrapositive:** If $n$ is even, then $3n + 2$ is even.

**Proof:**
1. Assume $n$ is even. Then $n = 2k$ for some integer $k$.
2. $3n + 2 = 3(2k) + 2 = 6k + 2 = 2(3k + 1)$.
3. Since $3k + 1$ is an integer, $3n + 2$ is even.
4. By contrapositive, if $3n + 2$ is odd, then $n$ is odd. $\blacksquare$

---

### Example 3: If $n^2$ is divisible by 3, then $n$ is divisible by 3

**Contrapositive:** If $n$ is NOT divisible by 3, then $n^2$ is NOT divisible by 3.

**Proof:**
1. Assume $3 \nmid n$. Then $n$ leaves remainder 1 or 2 when divided by 3.

   **Case 1:** $n = 3k + 1$
   - $n^2 = 9k^2 + 6k + 1 = 3(3k^2 + 2k) + 1$
   - So $n^2$ has remainder 1 when divided by 3 → $3 \nmid n^2$ ✓

   **Case 2:** $n = 3k + 2$
   - $n^2 = 9k^2 + 12k + 4 = 3(3k^2 + 4k + 1) + 1$
   - So $n^2$ has remainder 1 when divided by 3 → $3 \nmid n^2$ ✓

2. In both cases, $3 \nmid n^2$.
3. By contrapositive, if $3 \mid n^2$, then $3 \mid n$. $\blacksquare$

---

### Example 4: If $a + b \geq 2$, then $a \geq 1$ or $b \geq 1$

**Contrapositive:** If $a < 1$ AND $b < 1$, then $a + b < 2$.

**Proof:**
1. Assume $a < 1$ and $b < 1$.
2. Adding the inequalities: $a + b < 1 + 1 = 2$.
3. By contrapositive, if $a + b \geq 2$, then $a \geq 1$ or $b \geq 1$. $\blacksquare$

> **Note:** $\neg(a \geq 1 \vee b \geq 1) \equiv (a < 1) \wedge (b < 1)$ by De Morgan's Law.

---

### Example 5: If $xy$ is even, then $x$ is even or $y$ is even

**Contrapositive:** If $x$ is odd AND $y$ is odd, then $xy$ is odd.

**Proof:**
1. Assume $x$ is odd and $y$ is odd.
2. $x = 2a + 1$ and $y = 2b + 1$ for integers $a$ and $b$.
3. $xy = (2a+1)(2b+1) = 4ab + 2a + 2b + 1 = 2(2ab + a + b) + 1$.
4. So $xy$ is odd.
5. By contrapositive, if $xy$ is even, then $x$ is even or $y$ is even. $\blacksquare$

---

## 5. Common Mistakes

| Mistake | Explanation |
|---------|-------------|
| Confusing contrapositive with converse | $\neg q \rightarrow \neg p$ (contrapositive) ≠ $q \rightarrow p$ (converse) |
| Confusing contrapositive with inverse | $\neg q \rightarrow \neg p$ ≠ $\neg p \rightarrow \neg q$ |
| Incorrect negation | $\neg(a \geq 1 \vee b \geq 1)$ is $a < 1 \wedge b < 1$, NOT $a < 1 \vee b < 1$ |
| Forgetting to negate BOTH parts | Must negate the conclusion AND must derive negation of hypothesis |

---

## 6. Direct vs. Contrapositive Comparison

| Feature | Direct Proof | Contrapositive |
|---------|:------------:|:--------------:|
| Assume | $p$ | $\neg q$ |
| Derive | $q$ | $\neg p$ |
| Best when | $p$ gives useful info | $\neg q$ gives useful info |
| Example | "If even, then $n^2$ is even" | "If $n^2$ is even, then $n$ is even" |

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Contrapositive of $p \rightarrow q$ | $\neg q \rightarrow \neg p$ |
| Logical basis | $p \rightarrow q \equiv \neg q \rightarrow \neg p$ |
| Proof steps | Assume $\neg q$, derive $\neg p$ |
| When to use | Negation of conclusion is easier to work with |
| Common domain | Parity, divisibility, inequalities |
| Key warning | Negate BOTH sides; use De Morgan's for compound statements |

---

## ❓ Quick Revision Questions

1. **Write the contrapositive of: "If it rains, then the ground is wet."**

2. **Prove by contrapositive: If $n^3$ is odd, then $n$ is odd.**

3. **Prove by contrapositive: If $5n + 3$ is even, then $n$ is odd.**

4. **What is the difference between the converse and contrapositive of $p \rightarrow q$? Are either logically equivalent to the original?**

5. **Prove by contrapositive: If $x^2 - 6x + 5 \neq 0$, then $x \neq 1$ and $x \neq 5$.**  
   *(Hint: What is the contrapositive?)*

6. **Why is proof by contrapositive preferred over direct proof for "If $n^2$ is divisible by $p$ (prime), then $n$ is divisible by $p$"?**

---

[← Previous: Direct Proof](01-direct-proof.md) | [Back to README](../README.md) | [Next: Proof by Contradiction →](03-proof-by-contradiction.md)
