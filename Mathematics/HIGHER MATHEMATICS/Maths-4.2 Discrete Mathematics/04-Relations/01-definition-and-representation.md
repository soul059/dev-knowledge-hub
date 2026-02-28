# Chapter 4.1: Definition and Representation of Relations

[← Previous: Cantor's Theorem](../03-Set-Theory/07-cantors-theorem.md) | [Back to README](../README.md) | [Next: Types of Relations →](02-types-of-relations.md)

---

## 📋 Chapter Overview

A **relation** is a fundamental concept that formalizes the idea of elements being "related" to each other. Relations generalize the notion of functions and describe connections between objects in sets.

---

## 1. Definition of a Relation

A **binary relation** $R$ from set $A$ to set $B$ is a subset of the Cartesian product $A \times B$:

$$R \subseteq A \times B$$

If $(a, b) \in R$, we write $a \, R \, b$ (read "$a$ is related to $b$").

A **relation on a set** $A$ is a relation from $A$ to $A$, i.e., $R \subseteq A \times A$.

**Example:** Let $A = \{1, 2, 3, 4\}$ and $R = \{(a, b) : a \mid b\}$ (divides).

$$R = \{(1,1), (1,2), (1,3), (1,4), (2,2), (2,4), (3,3), (4,4)\}$$

---

## 2. Representation as a Matrix (Relation Matrix)

For $A = \{a_1, \ldots, a_m\}$ and $B = \{b_1, \ldots, b_n\}$, the **relation matrix** $M_R$ is:

$$M_R[i][j] = \begin{cases} 1 & \text{if } (a_i, b_j) \in R \\ 0 & \text{otherwise} \end{cases}$$

**Example:** $A = \{1, 2, 3, 4\}$, $R = \{(a,b) : a \mid b\}$

```
           b=1  b=2  b=3  b=4
         ┌────┬────┬────┬────┐
  a = 1  │  1 │  1 │  1 │  1 │
         ├────┼────┼────┼────┤
  a = 2  │  0 │  1 │  0 │  1 │
         ├────┼────┼────┼────┤
  a = 3  │  0 │  0 │  1 │  0 │
         ├────┼────┼────┼────┤
  a = 4  │  0 │  0 │  0 │  1 │
         └────┴────┴────┴────┘
```

$$M_R = \begin{pmatrix} 1 & 1 & 1 & 1 \\ 0 & 1 & 0 & 1 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}$$

---

## 3. Representation as a Directed Graph (Digraph)

Each element is a **node** and each pair $(a, b) \in R$ is a **directed edge** from $a$ to $b$.

**Example:** $R = \{(1,1),(1,2),(2,4),(3,3),(4,4)\}$ on $\{1,2,3,4\}$

```
      ┌──┐         ┌──┐
      │  ↓         │  ↓
     (1)────→────(2)
      
      
     (3)         (4)
      │  ↑         │  ↑
      └──┘         └──┘
      
  Self-loops at 1, 3, 4
  Edge from 1 → 2 and 2 → 4
```

---

## 4. Representation as a Set of Ordered Pairs

The most direct representation:

$$R = \{(1,2), (2,3), (3,1)\}$$

```
  Arrow Diagram:
  
  A            B
  ┌──┐      ┌──┐
  │ 1│──────│2 │
  │  │      │  │
  │ 2│──────│3 │
  │  │      │  │
  │ 3│──────│1 │
  └──┘      └──┘
```

---

## 5. Operations on Relations

### 5.1 Inverse Relation

$$R^{-1} = \{(b, a) : (a, b) \in R\}$$

If $R = \{(1,2), (2,3), (3,1)\}$, then $R^{-1} = \{(2,1), (3,2), (1,3)\}$.

**Matrix:** $M_{R^{-1}} = M_R^T$ (transpose).

### 5.2 Complement

$$\bar{R} = (A \times B) \setminus R$$

**Matrix:** $M_{\bar{R}}[i][j] = 1 - M_R[i][j]$

### 5.3 Union and Intersection

$$M_{R_1 \cup R_2}[i][j] = \max(M_{R_1}[i][j], M_{R_2}[i][j])$$
$$M_{R_1 \cap R_2}[i][j] = \min(M_{R_1}[i][j], M_{R_2}[i][j])$$

---

## 6. Composition of Relations

If $R_1 \subseteq A \times B$ and $R_2 \subseteq B \times C$, then:

$$R_2 \circ R_1 = \{(a, c) : \exists b \in B, (a, b) \in R_1 \text{ and } (b, c) \in R_2\}$$

**Matrix:** $M_{R_2 \circ R_1} = M_{R_1} \odot M_{R_2}$ (Boolean matrix multiplication)

**Example:**

$$R_1 = \{(1,2), (2,3)\}, \quad R_2 = \{(2,1), (3,4)\}$$

$$R_2 \circ R_1 = \{(1,1), (2,4)\}$$

```
  Composition Visualized:
  
  A ──R₁──→ B ──R₂──→ C
  
  1 ──→ 2 ──→ 1    ⟹  1 ──→ 1
  2 ──→ 3 ──→ 4    ⟹  2 ──→ 4
```

---

## 7. Powers of a Relation

$$R^1 = R, \quad R^{n+1} = R^n \circ R$$

$R^n$ connects $a$ to $b$ iff there is a **path of length $n$** from $a$ to $b$ in the digraph.

```
  R = {(1,2), (2,3), (3,1)}
  
  R¹: 1→2,  2→3,  3→1     (paths of length 1)
  R²: 1→3,  2→1,  3→2     (paths of length 2)
  R³: 1→1,  2→2,  3→3     (paths of length 3 = back to start)
```

---

## 8. Number of Relations

For $|A| = m$ and $|B| = n$:

$$\text{Number of relations from } A \text{ to } B = 2^{m \cdot n}$$

| $\|A\|$ | $\|B\|$ | Relations from $A$ to $B$ |
|:---:|:---:|:---:|
| 2 | 2 | $2^4 = 16$ |
| 2 | 3 | $2^6 = 64$ |
| 3 | 3 | $2^9 = 512$ |
| 4 | 4 | $2^{16} = 65536$ |

---

## 9. Real-World Examples

```
  ┌──────────────────────────────────────────────────┐
  │         Relations in the Real World               │
  │                                                   │
  │  1. Database: Table linking Students ↔ Courses    │
  │     R = {(Alice, Math), (Bob, CS), (Alice, CS)}  │
  │                                                   │
  │  2. Social Network: "follows" relation            │
  │     R ⊆ Users × Users                             │
  │                                                   │
  │  3. File System: "is subdirectory of"             │
  │     R ⊆ Directories × Directories                 │
  │                                                   │
  │  4. Web: Hyperlink relation                       │
  │     R = {(page_a, page_b) : a links to b}        │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Binary relation | $R \subseteq A \times B$ |
| $a \, R \, b$ | $(a,b) \in R$ |
| Relation matrix | Boolean $m \times n$ matrix |
| Digraph | Nodes + directed edges |
| $R^{-1}$ | Swap all pairs; transpose matrix |
| $R_2 \circ R_1$ | Boolean matrix multiplication |
| $R^n$ | Paths of length $n$ |
| Count | $2^{m \cdot n}$ relations from $A$ ($m$ elts) to $B$ ($n$ elts) |

---

## ❓ Quick Revision Questions

1. **Define a binary relation. How is it different from a function?**

2. **Given $R = \{(1,2),(2,3),(3,1)\}$ on $\{1,2,3\}$, write the relation matrix.**

3. **Compute $R^{-1}$ for $R = \{(a,1),(b,2),(c,1)\}$.**

4. **If $R_1 = \{(1,2),(2,3)\}$ and $R_2 = \{(2,a),(3,b)\}$, find $R_2 \circ R_1$.**

5. **How many relations are there on a set with 5 elements?**

6. **Draw the digraph for $R = \{(1,1),(1,2),(2,3),(3,3),(3,1)\}$ on $\{1,2,3\}$.**

---

[← Previous: Cantor's Theorem](../03-Set-Theory/07-cantors-theorem.md) | [Back to README](../README.md) | [Next: Types of Relations →](02-types-of-relations.md)
