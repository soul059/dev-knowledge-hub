# Chapter 9.1: Tree Properties

[← Previous: Planar Graphs](../08-Graph-Theory/08-planar-graphs.md) | [Back to README](../README.md) | [Next: Binary Trees →](02-binary-trees.md)

---

## 📋 Chapter Overview

Trees are among the most important structures in discrete mathematics and computer science. This chapter focuses on the **properties, characterizations, and counting** of trees, extending the introduction from graph theory.

---

## 1. Recap: Tree Definition

A **tree** $T = (V, E)$ is an undirected graph that is:
- **Connected** (path between every pair of vertices)
- **Acyclic** (contains no cycles)

---

## 2. Fundamental Properties

| Property | Statement |
|----------|-----------|
| Edges | A tree with $n$ vertices has exactly $n - 1$ edges |
| Unique path | Exactly one path between any two vertices |
| Leaf existence | Every tree with $n \geq 2$ has at least 2 leaves |
| Adding an edge | Creates exactly one cycle |
| Removing an edge | Disconnects the tree |
| Removing a leaf | Produces a smaller tree |

### Proof: At Least 2 Leaves

By the Handshaking Theorem: $\sum \deg(v) = 2(n-1) = 2n-2$

If at most 1 vertex had degree 1, then at least $n-1$ vertices have degree $\geq 2$:

$$\sum \deg(v) \geq 1 + 2(n-1) = 2n - 1 > 2n - 2$$

Contradiction! So at least 2 vertices have degree 1 (leaves). ∎

---

## 3. Labeled vs. Unlabeled Trees

| Type | Question | $n = 4$ |
|------|----------|:-------:|
| **Labeled** | How many distinct trees on vertices $\{1,2,3,4\}$? | $4^2 = 16$ |
| **Unlabeled** | How many structurally distinct trees on 4 vertices? | 2 |

```
  The 2 unlabeled trees on 4 vertices:
  
  Type 1: Path          Type 2: Star
  *---*---*---*         *---*---*
                            |
                            *
```

**Cayley's Formula:** Number of labeled trees on $n$ vertices = $n^{n-2}$.

| $n$ | Labeled | Unlabeled |
|:---:|:-------:|:---------:|
| 1 | 1 | 1 |
| 2 | 1 | 1 |
| 3 | 3 | 1 |
| 4 | 16 | 2 |
| 5 | 125 | 3 |
| 6 | 1296 | 6 |

---

## 4. Prüfer Sequence

A **Prüfer sequence** provides a bijection between labeled trees on $n$ vertices and sequences of length $n-2$ over $\{1, 2, \ldots, n\}$. This proves Cayley's formula: $n^{n-2}$.

### Encoding (Tree → Sequence)

1. Find the leaf with the **smallest label**
2. Record the **neighbor** of this leaf in the sequence
3. Remove the leaf
4. Repeat until 2 vertices remain

### Example: Tree on $\{1,2,3,4,5,6\}$

```
  Tree:
    1 --- 4 --- 5
    |           |
    2           6
    |
    3
  
  Step 1: Smallest leaf = 3, neighbor = 2  → sequence: [2, ...]
          Remove 3.
  Step 2: Smallest leaf = 2, neighbor = 1  → sequence: [2, 1, ...]
          Remove 2.
  Step 3: Smallest leaf = 1, neighbor = 4  → sequence: [2, 1, 4, ...]
          Remove 1.
  Step 4: Smallest leaf = 5, neighbor = 4  → sequence: [2, 1, 4, 4]
          Remove 5.
  Remaining: {4, 6}
  
  Prüfer sequence: (2, 1, 4, 4)
```

### Decoding (Sequence → Tree)

The degree of vertex $v$ in the tree = (number of times $v$ appears in Prüfer sequence) + 1.

Leaves = vertices **not** in the sequence.

---

## 5. Center of a Tree

The **center** of a tree consists of the vertices with minimum **eccentricity**.

**Algorithm:** Repeatedly remove all leaves. The last 1 or 2 vertices remaining form the center.

```
  Step 0:     1 - 2 - 3 - 4 - 5 - 6 - 7
  Leaves: {1, 7}
  
  Step 1:         2 - 3 - 4 - 5 - 6
  Leaves: {2, 6}
  
  Step 2:             3 - 4 - 5
  Leaves: {3, 5}
  
  Step 3:                 4
  
  Center = {4}
```

**Theorem:** Every tree has a center of 1 or 2 vertices.

---

## 6. Degree Sequence Characterization

A sequence $d_1 \geq d_2 \geq \cdots \geq d_n$ is the degree sequence of a tree on $n$ vertices if and only if:

1. Each $d_i \geq 1$
2. $\sum d_i = 2(n-1)$

### Example

Is $(3, 2, 2, 1, 1, 1)$ a valid tree degree sequence?

- $n = 6$, sum = $3+2+2+1+1+1 = 10 = 2(5) = 2(n-1)$ ✓
- All $d_i \geq 1$ ✓

Yes! This is a valid tree.

---

## 7. Counting Leaves

If a tree has $n$ vertices and $k$ internal vertices (degree $\geq 2$), then:

$$\text{Leaves} = n - k$$

Also, from the degree sum:

$$\sum_{\text{internal}} \deg(v_i) + \text{Leaves} = 2(n-1)$$

---

## 8. Real-World Applications

```
  ┌─────────────────────────────────────────────────┐
  │           Tree Properties in Practice            │
  │                                                  │
  │  • Prüfer Sequences                              │
  │    Used in random tree generation algorithms      │
  │                                                  │
  │  • Tree Centers                                  │
  │    Optimal placement of facilities in networks    │
  │                                                  │
  │  • Labeled Tree Counting                         │
  │    Network reliability analysis                  │
  │                                                  │
  │  • Tree Isomorphism                              │
  │    Can be tested in linear time (unlike graphs)  │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| $n$ vertices → $n-1$ edges | Fundamental |
| Unique paths | Between every pair |
| At least 2 leaves | For $n \geq 2$ |
| Cayley's formula | $n^{n-2}$ labeled trees on $n$ vertices |
| Prüfer sequence | Bijection: trees ↔ sequences of length $n-2$ |
| Center | 1 or 2 vertices; found by leaf removal |
| Degree sum | $\sum \deg = 2(n-1)$ |

---

## ❓ Quick Revision Questions

1. **A tree has 12 vertices. How many edges?**

2. **Encode the path $1$-$2$-$3$-$4$-$5$ as a Prüfer sequence.**

3. **How many labeled trees are there on 7 vertices?**

4. **Find the center of the tree: 1-2-3-4-5-6 (path).**

5. **Is the sequence (4, 3, 1, 1, 1, 1, 1) a valid tree degree sequence?**

6. **A tree has 3 internal vertices with degrees 4, 3, and 2. How many leaves does it have?**

---

[← Previous: Planar Graphs](../08-Graph-Theory/08-planar-graphs.md) | [Back to README](../README.md) | [Next: Binary Trees →](02-binary-trees.md)
