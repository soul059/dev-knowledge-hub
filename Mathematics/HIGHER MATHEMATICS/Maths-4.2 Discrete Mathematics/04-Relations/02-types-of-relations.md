# Chapter 4.2: Types of Relations

[← Previous: Definition and Representation](01-definition-and-representation.md) | [Back to README](../README.md) | [Next: Equivalence Relations and Partitions →](03-equivalence-relations-and-partitions.md)

---

## 📋 Chapter Overview

Relations on a set $A$ can have special properties that determine their behavior. These properties — **reflexive, symmetric, transitive**, and their variants — form the building blocks for classifying relations into important categories like equivalence relations and partial orders.

---

## 1. Reflexive and Irreflexive

### Reflexive
$R$ is **reflexive** if every element is related to itself:

$$\forall a \in A, \; (a, a) \in R$$

**Matrix:** All diagonal entries are 1.
**Digraph:** Every node has a self-loop.

### Irreflexive
$R$ is **irreflexive** if no element is related to itself:

$$\forall a \in A, \; (a, a) \notin R$$

**Matrix:** All diagonal entries are 0.

```
  Reflexive:              Irreflexive:
  
   (1)──↺                 (1)
    │                      │
    ↓                      ↓
   (2)──↺                 (2)
    │                      │
    ↓                      ↓
   (3)──↺                 (3)
  
  All self-loops           No self-loops
```

**Note:** A relation can be neither reflexive nor irreflexive (some self-loops, not all).

---

## 2. Symmetric and Antisymmetric

### Symmetric
$R$ is **symmetric** if:

$$\forall a, b \in A, \; (a, b) \in R \Rightarrow (b, a) \in R$$

**Matrix:** $M_R = M_R^T$ (symmetric matrix).
**Digraph:** Every edge has a matching reverse edge.

### Antisymmetric
$R$ is **antisymmetric** if:

$$\forall a, b \in A, \; [(a, b) \in R \text{ and } (b, a) \in R] \Rightarrow a = b$$

**Digraph:** No two distinct nodes have edges in both directions.

```
  Symmetric:              Antisymmetric:
  
  (1)⟵──⟶(2)             (1)────→(2)
                           
  (3)⟵──⟶(4)             (3)────→(4)
  
  All edges               No mutual edges
  are bidirectional       (except self-loops)
```

**Note:** A relation can be BOTH symmetric and antisymmetric! (e.g., the identity relation $\{(1,1),(2,2),(3,3)\}$)

---

## 3. Transitive

$R$ is **transitive** if:

$$\forall a, b, c \in A, \; [(a, b) \in R \text{ and } (b, c) \in R] \Rightarrow (a, c) \in R$$

```
  Transitive check:
  
  If  a ──→ b ──→ c   exists,
  then a ──────────→ c   must also exist.
  
  Example (transitive):
  1 → 2 → 3  and  1 → 3 ✓
  
  Example (NOT transitive):
  1 → 2 → 3  but  1 ↛ 3 ✗
```

---

## 4. Asymmetric

$R$ is **asymmetric** if:

$$\forall a, b \in A, \; (a, b) \in R \Rightarrow (b, a) \notin R$$

**Asymmetric = Antisymmetric + Irreflexive**

Example: The strict less-than relation $<$ on integers.

---

## 5. Complete Classification Table

| Property | Definition | Matrix Test | Digraph Test |
|----------|-----------|-------------|--------------|
| Reflexive | $\forall a: (a,a) \in R$ | All 1s on diagonal | All self-loops |
| Irreflexive | $\forall a: (a,a) \notin R$ | All 0s on diagonal | No self-loops |
| Symmetric | $(a,b) \in R \Rightarrow (b,a) \in R$ | $M = M^T$ | All edges bidirectional |
| Antisymmetric | $(a,b),(b,a) \in R \Rightarrow a=b$ | $M[i][j] \cdot M[j][i] = 0$ for $i \neq j$ | No mutual edges |
| Transitive | $(a,b),(b,c) \in R \Rightarrow (a,c) \in R$ | $M^2 \leq M$ (Boolean) | Shortcut edges exist |
| Asymmetric | $(a,b) \in R \Rightarrow (b,a) \notin R$ | $M[i][j] + M[j][i] \leq 1$ for all $i,j$ | No mutual edges, no self-loops |

---

## 6. Worked Examples

### Example 1: $R = \{(1,1),(1,2),(2,1),(2,2),(3,3)\}$ on $A = \{1,2,3\}$

```
  Matrix:          Digraph:
  
    1 2 3           (1)⟵──⟶(2)
  ┌───────┐          │↺      │↺
1 │ 1 1 0 │         
2 │ 1 1 0 │         (3)
3 │ 0 0 1 │          │↺
  └───────┘
```

| Property | Check | Result |
|----------|-------|--------|
| Reflexive | $(1,1),(2,2),(3,3) \in R$ | ✅ |
| Symmetric | $(1,2),(2,1) \in R$ | ✅ |
| Antisymmetric | $(1,2),(2,1) \in R$ but $1 \neq 2$ | ❌ |
| Transitive | $(1,2),(2,1) \in R$, $(1,1) \in R$ ✓; all paths checked | ✅ |

This is an **equivalence relation** (reflexive + symmetric + transitive).

### Example 2: $\leq$ on $\{1, 2, 3\}$

$$R = \{(1,1),(1,2),(1,3),(2,2),(2,3),(3,3)\}$$

```
  Matrix:          Digraph:
  
    1 2 3           (1)──→(2)──→(3)
  ┌───────┐          │↺    │↺    │↺
1 │ 1 1 1 │          │          │
2 │ 0 1 1 │          └──────────┘
3 │ 0 0 1 │             1 → 3
  └───────┘
```

| Property | Result |
|----------|--------|
| Reflexive | ✅ |
| Antisymmetric | ✅ |
| Transitive | ✅ |
| Symmetric | ❌ |

This is a **partial order** (reflexive + antisymmetric + transitive).

### Example 3: $<$ on $\{1, 2, 3\}$

$$R = \{(1,2),(1,3),(2,3)\}$$

| Property | Result |
|----------|--------|
| Irreflexive | ✅ |
| Antisymmetric | ✅ |
| Asymmetric | ✅ |
| Transitive | ✅ |

This is a **strict total order**.

---

## 7. Closures

The **closure** of $R$ with respect to a property $P$ is the smallest relation containing $R$ that has property $P$.

| Closure | Construction | Matrix |
|---------|-------------|--------|
| Reflexive closure | $R \cup \Delta$ where $\Delta = \{(a,a) : a \in A\}$ | Set diagonal to 1 |
| Symmetric closure | $R \cup R^{-1}$ | $M \lor M^T$ |
| Transitive closure | $R \cup R^2 \cup R^3 \cup \cdots$ | $(M \lor M^2 \lor M^3 \lor \cdots)$ until stable |

**Transitive closure** $R^+ = R \cup R^2 \cup R^3 \cup \cdots \cup R^n$ (for $|A| = n$).

**Reflexive-transitive closure** $R^* = R^+ \cup \Delta$.

```
  Original R:            Transitive Closure R⁺:
  
  (1)──→(2)──→(3)       (1)──→(2)──→(3)
                          │              ↑
                          └──────────────┘
                              1 → 3 added
```

---

## 8. Warshall's Algorithm for Transitive Closure

Efficient algorithm to compute $R^+$ in $O(n^3)$:

```
  Algorithm: Warshall(M, n)
  ──────────────────────────
  Input: n × n relation matrix M
  Output: Transitive closure matrix W
  
  W ← M
  for k ← 1 to n:
      for i ← 1 to n:
          for j ← 1 to n:
              W[i][j] ← W[i][j] OR (W[i][k] AND W[k][j])
  return W
```

**Trace for** $R = \{(1,2),(2,3),(3,1)\}$:

```
  Initial:     k=1:       k=2:       k=3:
  0 1 0        0 1 0      0 1 1      1 1 1
  0 0 1   →    0 0 1  →   0 0 1  →   1 1 1
  1 0 0        1 1 0      1 1 1      1 1 1
```

After Warshall's: every node reaches every other node (fully connected).

---

## 📝 Summary Table

| Concept | Key Definition |
|---------|---------------|
| Reflexive | $\forall a: a \, R \, a$ |
| Irreflexive | $\forall a: \neg(a \, R \, a)$ |
| Symmetric | $a \, R \, b \Rightarrow b \, R \, a$ |
| Antisymmetric | $a \, R \, b \land b \, R \, a \Rightarrow a = b$ |
| Transitive | $a \, R \, b \land b \, R \, c \Rightarrow a \, R \, c$ |
| Equivalence relation | Reflexive + Symmetric + Transitive |
| Partial order | Reflexive + Antisymmetric + Transitive |
| Transitive closure | Smallest transitive relation containing $R$ |
| Warshall's Algorithm | $O(n^3)$ method for transitive closure |

---

## ❓ Quick Revision Questions

1. **Determine which properties $R = \{(1,1),(1,2),(2,2),(2,3),(1,3),(3,3)\}$ has on $\{1,2,3\}$.**

2. **Give an example of a relation that is both symmetric and antisymmetric.**

3. **If $R$ is transitive and symmetric, is $R$ also reflexive? Why or why not?**

4. **Compute the reflexive closure and symmetric closure of $R = \{(1,2),(2,3)\}$ on $\{1,2,3\}$.**

5. **Apply Warshall's Algorithm to compute the transitive closure of $R = \{(1,2),(2,3)\}$.**

6. **Can a relation be both irreflexive and transitive? Give an example.**

---

[← Previous: Definition and Representation](01-definition-and-representation.md) | [Back to README](../README.md) | [Next: Equivalence Relations and Partitions →](03-equivalence-relations-and-partitions.md)
