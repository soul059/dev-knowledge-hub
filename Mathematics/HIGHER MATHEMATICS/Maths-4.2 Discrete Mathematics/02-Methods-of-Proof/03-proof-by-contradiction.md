# Chapter 2.3: Proof by Contradiction

[← Previous: Proof by Contrapositive](02-proof-by-contrapositive.md) | [Back to README](../README.md) | [Next: Mathematical Induction →](04-mathematical-induction.md)

---

## 📋 Chapter Overview

**Proof by contradiction** (also called *reductio ad absurdum*) works by assuming the **negation** of what we want to prove, then showing this assumption leads to a logical impossibility. Since the assumption causes a contradiction, it must be false — meaning the original statement is true.

---

## 1. Logical Foundation

The method is based on the tautology:

$$(\neg p \rightarrow F) \rightarrow p$$

If assuming $\neg p$ leads to a contradiction (falsehood), then $p$ must be true.

For implications $p \rightarrow q$: we assume $p \wedge \neg q$ and derive a contradiction.

$$\neg(p \rightarrow q) \equiv p \wedge \neg q$$

If $p \wedge \neg q$ leads to contradiction → $p \rightarrow q$ must be true.

---

## 2. Proof Strategy

### For general statements:
1. **Assume** $\neg p$ (the statement is false)
2. **Reason logically** from this assumption
3. **Arrive at a contradiction** (something like $r \wedge \neg r$)
4. **Conclude** $p$ is true

### For implications $p \rightarrow q$:
1. **Assume** $p$ is true **AND** $q$ is false
2. **Derive** a contradiction
3. **Conclude** $p \rightarrow q$

```
  ┌──────────────────────────────┐
  │  Assume ¬(statement)          │
  │         ↓                     │
  │  Logical reasoning...        │
  │         ↓                     │
  │  Arrive at: A ∧ ¬A  (boom!)  │
  │         ↓                     │
  │  ¬(statement) is impossible   │
  │         ↓                     │
  │  ∴ statement is TRUE          │
  └──────────────────────────────┘
```

---

## 3. Worked Examples

### Example 1: $\sqrt{2}$ is irrational (Classic!)

**Theorem:** $\sqrt{2}$ is irrational.

**Proof:**
1. **Assume for contradiction** that $\sqrt{2}$ is rational.
2. Then $\sqrt{2} = \frac{a}{b}$ where $a, b \in \mathbb{Z}$, $b \neq 0$, and $\gcd(a, b) = 1$ (reduced form).
3. Squaring: $2 = \frac{a^2}{b^2}$, so $a^2 = 2b^2$.
4. This means $a^2$ is even, so $a$ is even (proved by contrapositive earlier).
5. Let $a = 2k$. Then $(2k)^2 = 2b^2 \Rightarrow 4k^2 = 2b^2 \Rightarrow b^2 = 2k^2$.
6. So $b^2$ is even, meaning $b$ is even.
7. **Contradiction!** Both $a$ and $b$ are even, but we assumed $\gcd(a, b) = 1$.
8. Therefore, $\sqrt{2}$ is irrational. $\blacksquare$

---

### Example 2: There are infinitely many primes (Euclid)

**Theorem:** There are infinitely many prime numbers.

**Proof:**
1. **Assume for contradiction** that there are only finitely many primes: $p_1, p_2, \ldots, p_n$.
2. Consider $N = p_1 \cdot p_2 \cdot \ldots \cdot p_n + 1$.
3. $N > 1$, so $N$ has a prime divisor $p$.
4. $p$ must be one of $p_1, p_2, \ldots, p_n$ (they are ALL the primes).
5. But $N = p_1 p_2 \cdots p_n + 1$, so $N / p_i$ leaves remainder 1 for every $p_i$.
6. So $p$ does NOT divide $N$ — **contradiction** with step 4.
7. Therefore, there are infinitely many primes. $\blacksquare$

---

### Example 3: No largest integer

**Theorem:** There is no largest integer.

**Proof:**
1. **Assume** there exists a largest integer $M$.
2. Consider $M + 1$. This is also an integer (integers are closed under addition).
3. $M + 1 > M$.
4. **Contradiction:** $M$ was supposed to be the largest integer, but $M + 1$ is larger.
5. Therefore, there is no largest integer. $\blacksquare$

---

### Example 4: If $a^2$ is even, then $a$ is even (by contradiction)

**Proof:**
1. **Assume** $a^2$ is even AND $a$ is odd (for contradiction).
2. Since $a$ is odd, $a = 2k + 1$.
3. $a^2 = (2k+1)^2 = 4k^2 + 4k + 1 = 2(2k^2 + 2k) + 1$.
4. So $a^2$ is odd.
5. **Contradiction:** We assumed $a^2$ is even (step 1) but derived $a^2$ is odd.
6. Therefore, if $a^2$ is even, then $a$ is even. $\blacksquare$

---

### Example 5: $\log_2 3$ is irrational

**Proof:**
1. **Assume** $\log_2 3 = \frac{p}{q}$ where $p, q \in \mathbb{Z}^+$.
2. Then $2^{p/q} = 3$, so $2^p = 3^q$.
3. But $2^p$ is even (always) and $3^q$ is odd (always).
4. **Contradiction:** An even number cannot equal an odd number.
5. Therefore, $\log_2 3$ is irrational. $\blacksquare$

---

## 4. Contradiction vs. Contrapositive

| Feature | Contrapositive | Contradiction |
|---------|:--------------:|:-------------:|
| Proves | $p \rightarrow q$ | Any statement |
| Assumes | $\neg q$ only | $\neg p$ (or $p \wedge \neg q$) |
| Derives | $\neg p$ | Any contradiction $r \wedge \neg r$ |
| Number of assumptions | 1 | 1 or 2 |
| Preferred when | Result is an implication | Non-existence, irrationality |

> **Rule of thumb:** If you're proving $p \rightarrow q$ and contrapositive works, use it — it's cleaner. Use contradiction for existential/non-existence statements.

---

## 5. Common Contradictions to Look For

| Type | Example |
|------|---------|
| $n$ is both even and odd | $n = 2k$ and $n = 2m + 1$ |
| $a = b$ and $a \neq b$ | Two different values for the same thing |
| $x > 0$ and $x \leq 0$ | Contradictory inequalities |
| $\gcd = 1$ but both divisible by $p$ | Common factor contradicts coprimality |
| A finite set contains more elements | Exceeds stated count |

---

## 6. Proof Template

```
  Theorem: [Statement P]
  
  Proof (by contradiction):
  Assume, for the sake of contradiction, that ¬P.
  [i.e., assume P is false]
  ...
  [Logical reasoning based on ¬P]
  ...
  This gives us [A], but we also know [¬A].
  This is a contradiction.
  Therefore, our assumption is false, and P is true. ■
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Method | Assume negation, derive contradiction |
| Logical basis | $(\neg p \rightarrow F) \rightarrow p$ |
| For implications | Assume $p \wedge \neg q$, find contradiction |
| Classic example | $\sqrt{2}$ is irrational |
| When to use | Non-existence, irrationality, uniqueness proofs |
| Key step | Identify what contradicts what |

---

## ❓ Quick Revision Questions

1. **Prove by contradiction: If $n^2$ is odd, then $n$ is odd.**

2. **Prove: There is no rational number $r$ such that $r^2 = 3$.**

3. **What is the key difference between proof by contrapositive and proof by contradiction?**

4. **Prove by contradiction: If $a$ and $b$ are positive reals with $a + b = 1$, then $ab \leq \frac{1}{4}$.**

5. **Why does proof by contradiction work as a valid proof technique? What logical tautology underpins it?**

6. **Prove: $\sqrt{3}$ is irrational.**

---

[← Previous: Proof by Contrapositive](02-proof-by-contrapositive.md) | [Back to README](../README.md) | [Next: Mathematical Induction →](04-mathematical-induction.md)
