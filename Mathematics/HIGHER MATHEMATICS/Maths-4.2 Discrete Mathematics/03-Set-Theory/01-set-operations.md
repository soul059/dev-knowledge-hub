# Chapter 3.1: Set Operations

[← Previous: Well-Ordering Principle](../02-Methods-of-Proof/06-well-ordering-principle.md) | [Back to README](../README.md) | [Next: Venn Diagrams →](02-venn-diagrams.md)

---

## 📋 Chapter Overview

Sets are the most fundamental objects in mathematics. This chapter covers the basic operations on sets — union, intersection, complement, and difference — along with their properties and algebraic laws.

---

## 1. What is a Set?

A **set** is an unordered collection of distinct objects called **elements** or **members**.

### Notation

| Notation | Meaning |
|----------|---------|
| $a \in A$ | $a$ is an element of $A$ |
| $a \notin A$ | $a$ is not an element of $A$ |
| $\{1, 2, 3\}$ | Roster (listing) notation |
| $\{x : P(x)\}$ or $\{x \mid P(x)\}$ | Set-builder notation |
| $\emptyset$ or $\{\}$ | Empty set |
| $U$ | Universal set |

### Examples

| Set | Description |
|-----|-------------|
| $\mathbb{N} = \{0, 1, 2, 3, \ldots\}$ | Natural numbers |
| $\mathbb{Z} = \{\ldots, -2, -1, 0, 1, 2, \ldots\}$ | Integers |
| $\mathbb{Q} = \{p/q : p, q \in \mathbb{Z}, q \neq 0\}$ | Rational numbers |
| $\mathbb{R}$ | Real numbers |

---

## 2. Subset and Equality

### Subset
$A \subseteq B$ iff every element of $A$ is in $B$:
$$A \subseteq B \iff \forall x (x \in A \rightarrow x \in B)$$

### Proper Subset
$A \subset B$ iff $A \subseteq B$ and $A \neq B$.

### Set Equality
$A = B$ iff $A \subseteq B$ and $B \subseteq A$.

### Properties
- $\emptyset \subseteq A$ for every set $A$ (vacuously true)
- $A \subseteq A$ for every set $A$ (reflexive)
- If $A \subseteq B$ and $B \subseteq C$, then $A \subseteq C$ (transitive)

---

## 3. Set Operations

### 3.1 Union — $A \cup B$

$$A \cup B = \{x : x \in A \text{ or } x \in B\}$$

| Element in $A$? | Element in $B$? | In $A \cup B$? |
|:---:|:---:|:---:|
| ✅ | ✅ | ✅ |
| ✅ | ❌ | ✅ |
| ❌ | ✅ | ✅ |
| ❌ | ❌ | ❌ |

### 3.2 Intersection — $A \cap B$

$$A \cap B = \{x : x \in A \text{ and } x \in B\}$$

| Element in $A$? | Element in $B$? | In $A \cap B$? |
|:---:|:---:|:---:|
| ✅ | ✅ | ✅ |
| ✅ | ❌ | ❌ |
| ❌ | ✅ | ❌ |
| ❌ | ❌ | ❌ |

### 3.3 Complement — $\overline{A}$ or $A'$ or $A^c$

$$\overline{A} = \{x \in U : x \notin A\}$$

### 3.4 Difference — $A \setminus B$ or $A - B$

$$A \setminus B = \{x : x \in A \text{ and } x \notin B\} = A \cap \overline{B}$$

### 3.5 Symmetric Difference — $A \triangle B$ or $A \oplus B$

$$A \triangle B = (A \setminus B) \cup (B \setminus A) = (A \cup B) \setminus (A \cap B)$$

Elements in exactly one of $A$ or $B$.

---

## 4. Analogy: Set Operations as Logic Operations

| Set Operation | Logic Operation | Circuit Gate |
|:---:|:---:|:---:|
| $A \cup B$ | $p \vee q$ | OR |
| $A \cap B$ | $p \wedge q$ | AND |
| $\overline{A}$ | $\neg p$ | NOT |
| $A \triangle B$ | $p \oplus q$ | XOR |

---

## 5. Laws of Set Algebra

These mirror the laws of propositional logic:

### Identity Laws
$$A \cup \emptyset = A \qquad A \cap U = A$$

### Domination Laws
$$A \cup U = U \qquad A \cap \emptyset = \emptyset$$

### Idempotent Laws
$$A \cup A = A \qquad A \cap A = A$$

### Complement Laws
$$A \cup \overline{A} = U \qquad A \cap \overline{A} = \emptyset$$
$$\overline{\overline{A}} = A \qquad \overline{U} = \emptyset \qquad \overline{\emptyset} = U$$

### Commutative Laws
$$A \cup B = B \cup A \qquad A \cap B = B \cap A$$

### Associative Laws
$$(A \cup B) \cup C = A \cup (B \cup C)$$
$$(A \cap B) \cap C = A \cap (B \cap C)$$

### Distributive Laws
$$A \cup (B \cap C) = (A \cup B) \cap (A \cup C)$$
$$A \cap (B \cup C) = (A \cap B) \cup (A \cap C)$$

### De Morgan's Laws
$$\overline{A \cup B} = \overline{A} \cap \overline{B}$$
$$\overline{A \cap B} = \overline{A} \cup \overline{B}$$

### Absorption Laws
$$A \cup (A \cap B) = A \qquad A \cap (A \cup B) = A$$

---

## 6. Proving Set Identities

### Method 1: Element Argument (most rigorous)

Prove $A \cup (B \cap C) = (A \cup B) \cap (A \cup C)$:

$x \in A \cup (B \cap C)$
$\iff x \in A \text{ or } x \in B \cap C$
$\iff x \in A \text{ or } (x \in B \text{ and } x \in C)$
$\iff (x \in A \text{ or } x \in B) \text{ and } (x \in A \text{ or } x \in C)$ [Distributive law of logic]
$\iff x \in (A \cup B) \text{ and } x \in (A \cup C)$
$\iff x \in (A \cup B) \cap (A \cup C)$ $\blacksquare$

### Method 2: Algebraic (using known set laws)

### Method 3: Membership Table

| $A$ | $B$ | $C$ | $B \cap C$ | $A \cup (B \cap C)$ | $A \cup B$ | $A \cup C$ | $(A \cup B) \cap (A \cup C)$ |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 1 | 1 | 0 | 0 | 1 | 1 | 1 | 1 |
| 1 | 0 | 1 | 0 | 1 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 | 1 | 1 | 1 | 1 |
| 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 0 | 1 | 0 | 0 | 0 | 1 | 0 | 0 |
| 0 | 0 | 1 | 0 | 0 | 0 | 1 | 0 |
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

Columns 5 and 8 match ✅

---

## 7. Disjoint Sets

Two sets are **disjoint** if $A \cap B = \emptyset$.

A family of sets $A_1, A_2, \ldots, A_n$ is **pairwise disjoint** if $A_i \cap A_j = \emptyset$ for all $i \neq j$.

---

## 8. Generalized Operations

For a collection of sets $A_1, A_2, \ldots, A_n$:

$$\bigcup_{i=1}^{n} A_i = A_1 \cup A_2 \cup \cdots \cup A_n$$

$$\bigcap_{i=1}^{n} A_i = A_1 \cap A_2 \cap \cdots \cap A_n$$

**Generalized De Morgan's:**
$$\overline{\bigcup_{i=1}^{n} A_i} = \bigcap_{i=1}^{n} \overline{A_i}$$
$$\overline{\bigcap_{i=1}^{n} A_i} = \bigcup_{i=1}^{n} \overline{A_i}$$

---

## 📝 Summary Table

| Operation | Notation | Definition |
|-----------|:--------:|-----------|
| Union | $A \cup B$ | Elements in $A$ or $B$ (or both) |
| Intersection | $A \cap B$ | Elements in both $A$ and $B$ |
| Complement | $\overline{A}$ | Elements not in $A$ |
| Difference | $A \setminus B$ | Elements in $A$ but not $B$ |
| Symmetric Diff | $A \triangle B$ | Elements in exactly one of $A, B$ |
| Disjoint | $A \cap B = \emptyset$ | No common elements |
| Subset | $A \subseteq B$ | Every element of $A$ is in $B$ |

---

## ❓ Quick Revision Questions

1. **If $A = \{1, 2, 3, 4\}$ and $B = \{3, 4, 5, 6\}$, find $A \cup B$, $A \cap B$, $A \setminus B$, and $A \triangle B$.**

2. **Prove using element argument: $A \setminus (B \cup C) = (A \setminus B) \cap (A \setminus C)$.**

3. **State and prove De Morgan's Law for sets.**

4. **Are $\{1, 2\}$ and $\{3, 4\}$ disjoint? Are $\{1, 2\}$, $\{2, 3\}$, $\{3, 4\}$ pairwise disjoint?**

5. **Simplify: $A \cap (A \cup B)$ using set laws.**

6. **Prove or disprove: $A \triangle B = A \triangle C$ implies $B = C$.**

---

[← Previous: Well-Ordering Principle](../02-Methods-of-Proof/06-well-ordering-principle.md) | [Back to README](../README.md) | [Next: Venn Diagrams →](02-venn-diagrams.md)
