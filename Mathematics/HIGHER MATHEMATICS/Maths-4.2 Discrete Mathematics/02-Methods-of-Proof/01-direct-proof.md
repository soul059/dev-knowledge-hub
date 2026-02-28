# Chapter 2.1: Direct Proof

[← Previous: Quantifiers](../01-Logic-and-Propositional-Calculus/07-quantifiers.md) | [Back to README](../README.md) | [Next: Proof by Contrapositive →](02-proof-by-contrapositive.md)

---

## 📋 Chapter Overview

A **direct proof** is the most straightforward proof technique: to prove $p \rightarrow q$, we assume $p$ is true and use logical steps, definitions, and previously proven results to show that $q$ must also be true.

---

## 1. What is a Proof?

A **mathematical proof** is a valid argument that establishes the truth of a mathematical statement. It consists of:
- **Hypotheses** (premises, given information)
- **Logical steps** (each justified by a definition, axiom, or theorem)
- **Conclusion** (the statement being proved)

```
  Structure of a Proof:
  
  ┌──────────────────────────────┐
  │  Hypothesis (Assume p)       │
  │  ────────────────────────    │
  │  Step 1: ... (by definition) │
  │  Step 2: ... (by theorem X)  │
  │  Step 3: ... (by algebra)    │
  │  ...                         │
  │  ────────────────────────    │
  │  Conclusion (Therefore q)    │
  └──────────────────────────────┘
```

---

## 2. Direct Proof Structure

To prove: **If $p$, then $q$** (i.e., $p \rightarrow q$)

**Steps:**
1. **Assume** $p$ is true
2. **Derive** $q$ through a chain of logical implications
3. **Conclude** $q$ is true

> A direct proof builds a chain: $p \Rightarrow s_1 \Rightarrow s_2 \Rightarrow \cdots \Rightarrow q$

---

## 3. Key Definitions (for number theory proofs)

| Term | Definition |
|------|-----------|
| **Even** | $n$ is even iff $n = 2k$ for some integer $k$ |
| **Odd** | $n$ is odd iff $n = 2k + 1$ for some integer $k$ |
| **Divides** | $a \mid b$ iff $b = ak$ for some integer $k$ |
| **Rational** | $r$ is rational iff $r = \frac{a}{b}$ where $a, b \in \mathbb{Z}$, $b \neq 0$ |
| **Prime** | $p > 1$ is prime iff its only divisors are 1 and $p$ |

---

## 4. Worked Examples

### Example 1: Sum of two even numbers is even

**Theorem:** If $m$ and $n$ are even integers, then $m + n$ is even.

**Proof:**
1. Assume $m$ and $n$ are even integers. *(hypothesis)*
2. By definition, $m = 2a$ for some integer $a$, and $n = 2b$ for some integer $b$.
3. Then $m + n = 2a + 2b = 2(a + b)$.
4. Since $a + b$ is an integer (integers are closed under addition), let $k = a + b$.
5. So $m + n = 2k$, which is even by definition. $\blacksquare$

---

### Example 2: Product of two odd numbers is odd

**Theorem:** If $m$ and $n$ are odd integers, then $mn$ is odd.

**Proof:**
1. Assume $m$ and $n$ are odd.
2. Then $m = 2a + 1$ and $n = 2b + 1$ for some integers $a$ and $b$.
3. $mn = (2a + 1)(2b + 1) = 4ab + 2a + 2b + 1 = 2(2ab + a + b) + 1$.
4. Let $k = 2ab + a + b$, which is an integer.
5. So $mn = 2k + 1$, which is odd by definition. $\blacksquare$

---

### Example 3: Divisibility is transitive

**Theorem:** If $a \mid b$ and $b \mid c$, then $a \mid c$.

**Proof:**
1. Assume $a \mid b$ and $b \mid c$.
2. By definition, $b = ak$ for some integer $k$, and $c = bm$ for some integer $m$.
3. Substituting: $c = bm = (ak)m = a(km)$.
4. Since $km$ is an integer, we have $c = a \cdot (km)$, so $a \mid c$ by definition. $\blacksquare$

---

### Example 4: Sum of a rational and rational is rational

**Theorem:** If $r$ and $s$ are rational numbers, then $r + s$ is rational.

**Proof:**
1. Assume $r$ and $s$ are rational.
2. Then $r = \frac{a}{b}$ and $s = \frac{c}{d}$ where $a, b, c, d \in \mathbb{Z}$ and $b, d \neq 0$.
3. $r + s = \frac{a}{b} + \frac{c}{d} = \frac{ad + bc}{bd}$.
4. Since $ad + bc \in \mathbb{Z}$ and $bd \in \mathbb{Z}$ with $bd \neq 0$, $r + s$ is rational. $\blacksquare$

---

### Example 5: If $n$ is odd, then $n^2$ is odd

**Theorem:** For any integer $n$, if $n$ is odd, then $n^2$ is odd.

**Proof:**
1. Assume $n$ is odd.
2. Then $n = 2k + 1$ for some integer $k$.
3. $n^2 = (2k+1)^2 = 4k^2 + 4k + 1 = 2(2k^2 + 2k) + 1$.
4. Let $m = 2k^2 + 2k$, which is an integer.
5. Then $n^2 = 2m + 1$, which is odd. $\blacksquare$

---

## 5. Proof Writing Guidelines

### Do's ✅
| Guideline | Example |
|-----------|---------|
| State what you assume | "Assume $n$ is an even integer" |
| Use definitions precisely | "By definition, $n = 2k$ for some integer $k$" |
| Justify each step | "Since integers are closed under multiplication..." |
| Use proper notation | $\Rightarrow$, $\therefore$, $\blacksquare$ |
| End clearly | "Therefore, $q$ holds. $\blacksquare$" |

### Don'ts ❌
| Mistake | Why it's Wrong |
|---------|---------------|
| Working backward | Start from hypothesis, not conclusion |
| Using specific examples | Proof must work for ALL cases |
| Assuming what you're proving | Circular reasoning |
| Skipping justification | Each step needs a reason |

---

## 6. When to Use Direct Proof

Direct proof works best when:
- There is a clear **algebraic path** from $p$ to $q$
- The definitions of terms lead naturally from hypothesis to conclusion
- The implication is **constructive** (you can build the object)

Direct proof may be **difficult** when:
- The statement involves negation or "not exists"
- The relationship between $p$ and $q$ is not straightforward
- In these cases, consider contrapositive or contradiction

---

## 7. Proof Template

```
  Theorem: If [hypothesis], then [conclusion].
  
  Proof:
  Let [variables] be [type/conditions].
  Assume [hypothesis].
  [State definitions applied to hypothesis.]
  [Algebraic/logical manipulations.]
  [Each step justified by definition/theorem/axiom.]
  Therefore, [conclusion]. ■
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Direct Proof | Assume $p$, derive $q$ |
| Structure | $p \Rightarrow s_1 \Rightarrow \cdots \Rightarrow q$ |
| Key tool | Algebraic manipulation with definitions |
| When to use | Clear algebraic path from $p$ to $q$ |
| Common domain | Number theory, algebra |
| End marker | $\blacksquare$ (QED, "quod erat demonstrandum") |

---

## ❓ Quick Revision Questions

1. **Prove directly: If $n$ is even, then $n^2$ is even.**

2. **Prove: The sum of any two odd numbers is even.**

3. **Prove: If $a \mid b$, then $a \mid (bc)$ for any integer $c$.**

4. **Why can't we use a single numerical example (e.g., $n = 4$) as a proof?**

5. **What is wrong with the following "proof" that if $n^2$ is even then $n$ is even?**
   - "Assume $n$ is even. Then $n = 2k$. Then $n^2 = 4k^2 = 2(2k^2)$, which is even." ■

6. **Prove: If $x$ is rational and $y$ is rational, then $xy$ is rational.**

---

[← Previous: Quantifiers](../01-Logic-and-Propositional-Calculus/07-quantifiers.md) | [Back to README](../README.md) | [Next: Proof by Contrapositive →](02-proof-by-contrapositive.md)
