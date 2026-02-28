# Chapter 3.5: Cardinality

[← Previous: Power Set](04-power-set.md) | [Back to README](../README.md) | [Next: Countable and Uncountable Sets →](06-countable-and-uncountable-sets.md)

---

## 📋 Chapter Overview

**Cardinality** measures the "size" of a set. For finite sets, this is simply the number of elements. For infinite sets, the concept becomes much richer — leading to different "sizes of infinity."

---

## 1. Finite Cardinality

The **cardinality** of a finite set $A$, written $|A|$, is the number of distinct elements in $A$.

| Set | Cardinality |
|-----|:-----------:|
| $\{a, b, c\}$ | 3 |
| $\emptyset$ | 0 |
| $\{1, 1, 2\} = \{1, 2\}$ | 2 |
| $\{x \in \mathbb{Z} : 1 \leq x \leq 100\}$ | 100 |

---

## 2. Comparing Cardinalities via Bijections

Two sets $A$ and $B$ have the **same cardinality** ($|A| = |B|$) if there exists a **bijection** (one-to-one and onto function) $f: A \rightarrow B$.

```
  A = {a, b, c}        B = {x, y, z}
  
  a ──────────→ x
  b ──────────→ y       Bijection: |A| = |B| = 3
  c ──────────→ z
```

This definition also works for **infinite sets** — which is where it becomes profound.

---

## 3. Aleph Numbers

Georg Cantor introduced **cardinal numbers** for infinite sets:

$$\aleph_0 = |\mathbb{N}|$$

$\aleph_0$ ("aleph-null") is the cardinality of the natural numbers — the **smallest infinity**.

$$\mathfrak{c} = |\mathbb{R}| = 2^{\aleph_0}$$

$\mathfrak{c}$ is the **cardinality of the continuum** — a strictly larger infinity.

---

## 4. Important Cardinality Results

| Set | Cardinality | Equal to $|\mathbb{N}|$? |
|-----|:-----------:|:---:|
| $\mathbb{N} = \{0, 1, 2, \ldots\}$ | $\aleph_0$ | ✅ |
| $\mathbb{Z}$ (integers) | $\aleph_0$ | ✅ |
| $\mathbb{Q}$ (rationals) | $\aleph_0$ | ✅ |
| $\mathbb{N} \times \mathbb{N}$ | $\aleph_0$ | ✅ |
| $\mathbb{R}$ (reals) | $\mathfrak{c}$ | ❌ (strictly larger) |
| $(0, 1)$ interval | $\mathfrak{c}$ | ❌ |
| $\mathcal{P}(\mathbb{N})$ | $\mathfrak{c}$ | ❌ |

---

## 5. Cardinality Arithmetic (Finite)

For finite sets:

| Operation | Formula | Condition |
|-----------|---------|-----------|
| Union | $\|A \cup B\| = \|A\| + \|B\| - \|A \cap B\|$ | |
| Disjoint union | $\|A \cup B\| = \|A\| + \|B\|$ | if $A \cap B = \emptyset$ |
| Cartesian product | $\|A \times B\| = \|A\| \cdot \|B\|$ | |
| Power set | $\|\mathcal{P}(A)\| = 2^{\|A\|}$ | |
| Difference | $\|A \setminus B\| = \|A\| - \|A \cap B\|$ | |

---

## 6. Finite vs. Infinite Sets

A set is **finite** if $|A| = n$ for some $n \in \mathbb{N}_0$.

A set is **infinite** if it is not finite.

**Dedekind's definition:** A set is infinite iff it can be put in bijection with a proper subset of itself.

Example: $\mathbb{N} \leftrightarrow \mathbb{N} \setminus \{0\}$ via $f(n) = n + 1$. So $\mathbb{N}$ is infinite.

```
  ℕ:      0  1  2  3  4  5  ...
           ↓  ↓  ↓  ↓  ↓  ↓
  ℕ\{0}:  1  2  3  4  5  6  ...
  
  Bijection f(n) = n + 1
  ℕ has same cardinality as a proper subset!
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| $\|A\|$ | Number of elements (finite) or cardinal number (infinite) |
| Same cardinality | Bijection exists between the sets |
| $\aleph_0$ | Cardinality of $\mathbb{N}$ (smallest infinity) |
| $\mathfrak{c}$ | Cardinality of $\mathbb{R}$ (continuum) |
| Finite set | $\|A\| = n$ for some natural number $n$ |
| Infinite set | Can biject with a proper subset |

---

## ❓ Quick Revision Questions

1. **What is the cardinality of $\{x \in \mathbb{Z} : -5 \leq x \leq 5\}$?**

2. **If $|A| = 10$ and $|B| = 15$ and $|A \cap B| = 3$, find $|A \cup B|$.**

3. **Explain why $|\mathbb{Z}| = |\mathbb{N}|$ even though $\mathbb{N} \subset \mathbb{Z}$.**

4. **What does it mean for two infinite sets to have the "same size"?**

5. **Compute $|\mathcal{P}(\{1, 2, 3, 4, 5\})|$.**

6. **Is the set of even numbers the same cardinality as $\mathbb{N}$? Prove it.**

---

[← Previous: Power Set](04-power-set.md) | [Back to README](../README.md) | [Next: Countable and Uncountable Sets →](06-countable-and-uncountable-sets.md)
