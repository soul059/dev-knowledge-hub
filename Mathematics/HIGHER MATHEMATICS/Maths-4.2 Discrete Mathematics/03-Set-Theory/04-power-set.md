# Chapter 3.4: Power Set

[← Previous: Cartesian Product](03-cartesian-product.md) | [Back to README](../README.md) | [Next: Cardinality →](05-cardinality.md)

---

## 📋 Chapter Overview

The **power set** of a set $A$ is the set of **all subsets** of $A$, including the empty set and $A$ itself. It plays a crucial role in combinatorics, topology, and the study of set cardinalities.

---

## 1. Definition

$$\mathcal{P}(A) = \{S : S \subseteq A\}$$

The power set $\mathcal{P}(A)$ (also written $2^A$) is the collection of ALL subsets of $A$.

---

## 2. Examples

### Example 1: $A = \{1, 2\}$

$$\mathcal{P}(A) = \{\emptyset, \{1\}, \{2\}, \{1, 2\}\}$$

$|\mathcal{P}(A)| = 4 = 2^2$

### Example 2: $A = \{a, b, c\}$

$$\mathcal{P}(A) = \{\emptyset, \{a\}, \{b\}, \{c\}, \{a,b\}, \{a,c\}, \{b,c\}, \{a,b,c\}\}$$

$|\mathcal{P}(A)| = 8 = 2^3$

### Example 3: $A = \emptyset$

$$\mathcal{P}(\emptyset) = \{\emptyset\}$$

$|\mathcal{P}(\emptyset)| = 1 = 2^0$

> **Note:** $\mathcal{P}(\emptyset) \neq \emptyset$. The power set of the empty set has ONE element (the empty set itself).

---

## 3. Cardinality of Power Set

$$|\mathcal{P}(A)| = 2^{|A|}$$

**Proof (by counting):** For each of the $|A|$ elements, we have 2 choices — include it or exclude it. By the multiplication principle: $2 \times 2 \times \cdots \times 2 = 2^{|A|}$.

### Correspondence with Binary Strings

Each subset of $A = \{a_1, a_2, \ldots, a_n\}$ corresponds to a binary string of length $n$:

For $A = \{a, b, c\}$:

| Binary String | Subset |
|:---:|:---:|
| 000 | $\emptyset$ |
| 001 | $\{c\}$ |
| 010 | $\{b\}$ |
| 011 | $\{b, c\}$ |
| 100 | $\{a\}$ |
| 101 | $\{a, c\}$ |
| 110 | $\{a, b\}$ |
| 111 | $\{a, b, c\}$ |

Each bit represents: 1 = include, 0 = exclude.

---

## 4. Growth of Power Sets

| $|A|$ | $|\mathcal{P}(A)|$ | Growth |
|:---:|:---:|------|
| 0 | 1 | |
| 1 | 2 | |
| 2 | 4 | |
| 3 | 8 | |
| 5 | 32 | |
| 10 | 1,024 | |
| 20 | 1,048,576 | ~1 million |
| 30 | ~1 billion | |

> Power sets grow **exponentially** — this is why they are fundamental in complexity theory.

---

## 5. Properties

| Property | Statement |
|----------|-----------|
| $\emptyset \in \mathcal{P}(A)$ | Always (empty set is subset of any set) |
| $A \in \mathcal{P}(A)$ | Always (any set is subset of itself) |
| $A \subseteq B \Rightarrow \mathcal{P}(A) \subseteq \mathcal{P}(B)$ | Power set preserves subset relation |
| $\mathcal{P}(A \cap B) = \mathcal{P}(A) \cap \mathcal{P}(B)$ | Intersection distributes |
| $\mathcal{P}(A) \cup \mathcal{P}(B) \subseteq \mathcal{P}(A \cup B)$ | Union: only subset relation |
| $|\mathcal{P}(\mathcal{P}(A))| = 2^{2^{|A|}}$ | Power set of power set |

---

## 6. Iterated Power Sets

$$\mathcal{P}(\emptyset) = \{\emptyset\}$$
$$\mathcal{P}(\mathcal{P}(\emptyset)) = \mathcal{P}(\{\emptyset\}) = \{\emptyset, \{\emptyset\}\}$$
$$\mathcal{P}(\mathcal{P}(\mathcal{P}(\emptyset))) = \{\emptyset, \{\emptyset\}, \{\{\emptyset\}\}, \{\emptyset, \{\emptyset\}\}\}$$

Sizes: $1 \rightarrow 2 \rightarrow 4 \rightarrow 16 \rightarrow 65536 \rightarrow \cdots$ (tower of powers of 2)

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| $\mathcal{P}(A)$ | Set of all subsets of $A$ |
| $\|\mathcal{P}(A)\|$ | $2^{\|A\|}$ |
| Always contains | $\emptyset$ and $A$ |
| Binary correspondence | Each subset ↔ binary string |
| Growth | Exponential ($2^n$) |

---

## ❓ Quick Revision Questions

1. **List all elements of $\mathcal{P}(\{1, 2, 3, 4\})$. How many are there?**

2. **Is $\{1, 2\} \in \mathcal{P}(\{1, 2, 3\})$? Is $\{1, 2\} \subseteq \mathcal{P}(\{1, 2, 3\})$?**

3. **Prove by induction: $|\mathcal{P}(A)| = 2^{|A|}$ for finite sets.**

4. **What is $\mathcal{P}(A) \cap \mathcal{P}(B)$? Prove it equals $\mathcal{P}(A \cap B)$.**

5. **How many subsets of $\{1, 2, \ldots, 10\}$ contain exactly 3 elements?**

6. **If $|A| = 5$, how many elements does $\mathcal{P}(\mathcal{P}(A))$ have?**

---

[← Previous: Cartesian Product](03-cartesian-product.md) | [Back to README](../README.md) | [Next: Cardinality →](05-cardinality.md)
