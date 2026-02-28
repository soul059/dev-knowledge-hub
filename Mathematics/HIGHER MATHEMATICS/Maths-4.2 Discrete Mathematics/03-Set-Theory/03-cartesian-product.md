# Chapter 3.3: Cartesian Product

[← Previous: Venn Diagrams](02-venn-diagrams.md) | [Back to README](../README.md) | [Next: Power Set →](04-power-set.md)

---

## 📋 Chapter Overview

The **Cartesian product** creates ordered pairs from two sets. It is the foundation for defining relations, functions, coordinate systems, and database tables.

---

## 1. Definition

The **Cartesian product** of sets $A$ and $B$ is:

$$A \times B = \{(a, b) : a \in A \text{ and } b \in B\}$$

Each element $(a, b)$ is an **ordered pair** — the order matters: $(a, b) \neq (b, a)$ unless $a = b$.

### Example

$A = \{1, 2\}$, $B = \{x, y, z\}$

$$A \times B = \{(1,x), (1,y), (1,z), (2,x), (2,y), (2,z)\}$$

$$B \times A = \{(x,1), (x,2), (y,1), (y,2), (z,1), (z,2)\}$$

Note: $A \times B \neq B \times A$ (different ordered pairs!)

### Grid Visualization

```
  A × B:                    B × A:
  
  z │  (1,z)  (2,z)         2 │  (x,2)  (y,2)  (z,2)
  y │  (1,y)  (2,y)         1 │  (x,1)  (y,1)  (z,1)
  x │  (1,x)  (2,x)           └──────────────────────
    └──────────────            x      y      z
       1      2
```

---

## 2. Cardinality

$$|A \times B| = |A| \cdot |B|$$

If $|A| = m$ and $|B| = n$, then $|A \times B| = mn$.

### Extension to Multiple Sets

$$|A_1 \times A_2 \times \cdots \times A_n| = |A_1| \cdot |A_2| \cdots |A_n|$$

---

## 3. Properties

| Property | Statement |
|----------|-----------|
| Non-commutative | $A \times B \neq B \times A$ (in general) |
| Non-associative | $(A \times B) \times C \neq A \times (B \times C)$ (different tuple structure) |
| Distributive over $\cup$ | $A \times (B \cup C) = (A \times B) \cup (A \times C)$ |
| Distributive over $\cap$ | $A \times (B \cap C) = (A \times B) \cap (A \times C)$ |
| Empty set | $A \times \emptyset = \emptyset$ |

---

## 4. n-Tuples and Higher Products

$$A_1 \times A_2 \times \cdots \times A_n = \{(a_1, a_2, \ldots, a_n) : a_i \in A_i\}$$

**Special case:** $A^n = A \times A \times \cdots \times A$ ($n$ times)

**Examples:**
- $\mathbb{R}^2 = \mathbb{R} \times \mathbb{R}$ = the Cartesian plane
- $\mathbb{R}^3 = \mathbb{R} \times \mathbb{R} \times \mathbb{R}$ = 3D space
- $\{0, 1\}^n$ = set of all binary strings of length $n$ ($2^n$ elements)

---

## 5. Worked Examples

### Example 1: Listing all elements

$A = \{a, b\}$, $B = \{1, 2, 3\}$

$A \times B = \{(a,1), (a,2), (a,3), (b,1), (b,2), (b,3)\}$

$|A \times B| = 2 \times 3 = 6$ ✓

### Example 2: Cartesian product with itself

$A = \{0, 1\}$

$A \times A = A^2 = \{(0,0), (0,1), (1,0), (1,1)\}$

This represents all 2-bit binary strings.

### Example 3: Three sets

$A = \{0, 1\}$, $B = \{a\}$, $C = \{x, y\}$

$A \times B \times C = \{(0,a,x), (0,a,y), (1,a,x), (1,a,y)\}$

$|A \times B \times C| = 2 \times 1 \times 2 = 4$ ✓

---

## 6. Real-World Applications

| Application | Cartesian Product |
|-------------|------------------|
| **Coordinate systems** | $\mathbb{R} \times \mathbb{R}$ = 2D plane |
| **Database tables** | Each row is a tuple from domain products |
| **Playing cards** | Suits × Ranks = $\{♠,♥,♦,♣\} \times \{A,2,...,K\}$ |
| **Binary strings** | $\{0,1\}^n$ = all $n$-bit strings |
| **Relations** | Subsets of $A \times B$ |
| **State machines** | States × Inputs = transition possibilities |

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| $A \times B$ | Set of all ordered pairs $(a, b)$ |
| $\|A \times B\|$ | $\|A\| \cdot \|B\|$ |
| Ordered pair | $(a, b) \neq (b, a)$ unless $a = b$ |
| $n$-tuple | $(a_1, \ldots, a_n) \in A_1 \times \cdots \times A_n$ |
| $A^n$ | $A \times A \times \cdots \times A$ ($n$ copies) |
| Distributive | $A \times (B \cup C) = (A \times B) \cup (A \times C)$ |

---

## ❓ Quick Revision Questions

1. **If $A = \{1, 2, 3\}$ and $B = \{a, b\}$, list all elements of $A \times B$ and $B \times A$.**

2. **How many elements are in $\{1, 2, 3\} \times \{a, b, c, d\} \times \{0, 1\}$?**

3. **Is $A \times B = B \times A$ ever true? When?**

4. **Prove: $A \times (B \cup C) = (A \times B) \cup (A \times C)$.**

5. **What is $A \times \emptyset$? Explain why.**

6. **How does the Cartesian product relate to a relational database?**

---

[← Previous: Venn Diagrams](02-venn-diagrams.md) | [Back to README](../README.md) | [Next: Power Set →](04-power-set.md)
