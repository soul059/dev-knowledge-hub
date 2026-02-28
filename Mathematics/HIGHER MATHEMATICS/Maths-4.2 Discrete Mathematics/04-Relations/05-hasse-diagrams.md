# Chapter 4.5: Hasse Diagrams

[← Previous: Partial Ordering](04-partial-ordering.md) | [Back to README](../README.md) | [Next: Lattices →](06-lattices.md)

---

## 📋 Chapter Overview

A **Hasse diagram** is a simplified graphical representation of a finite poset. By removing redundant information (self-loops and transitively-implied edges), Hasse diagrams reveal the essential structure of a partial order at a glance.

---

## 1. Construction Rules

To draw a Hasse diagram of a poset $(A, \preceq)$:

1. **Remove self-loops** (reflexivity is assumed)
2. **Remove transitive edges** (if $a \prec b \prec c$, remove $a \prec c$)
3. **Draw upward** (if $a \prec b$, place $a$ below $b$; no arrowheads needed)

```
  Full Digraph of ≤ on {1,2,3}:    Hasse Diagram:
  
       (3)──↺                           (3)
       ↑  ↑                              │
      /    \                              │
    /       \                            (2)
  (1)──→──(2)──↺                         │
    ↺                                     │
                                         (1)
  
  9 edges                            2 edges!
  (with loops & transitive edges)   (much simpler)
```

---

## 2. Reading a Hasse Diagram

- $a \preceq b$ iff there is an **upward path** from $a$ to $b$
- $a \prec b$ (covering) iff there is a **single upward edge** from $a$ to $b$
- Elements at the same height level are **incomparable** (unless connected)

---

## 3. Example: Divisibility on $\{1, 2, 3, 6\}$

$a \preceq b$ iff $a \mid b$.

```
          (6)
         / \
        /   \
      (2)   (3)
        \   /
         \ /
          (1)
```

Reading: $1 \mid 2$, $1 \mid 3$, $2 \mid 6$, $3 \mid 6$ (direct divisors).
Also: $1 \mid 6$ (by following the path $1 \to 2 \to 6$ or $1 \to 3 \to 6$).

---

## 4. Example: Power Set $\mathcal{P}(\{a, b, c\})$ under $\subseteq$

```
                    {a,b,c}
                   /   |   \
                  /    |    \
              {a,b}  {a,c}  {b,c}
                \   \ | /   /
                 \   \|/   /
                  {a} {b} {c}
                   \   |   /
                    \  |  /
                      ∅
```

**Structure:** 8 elements (= $2^3$), 4 levels (0 to 3 elements).

Reading:
- $\emptyset \subseteq \{a\} \subseteq \{a,b\} \subseteq \{a,b,c\}$ ✓
- $\{a\}$ and $\{b\}$ are **incomparable** (no edge between them)

---

## 5. Example: Divisors of 30

$(\{1, 2, 3, 5, 6, 10, 15, 30\}, \mid)$

```
                    (30)
                   / | \
                  /  |  \
               (6) (10) (15)
              / |    |    | \
             /  |    |    |  \
           (2) (3)  (5)  (3) (5)
             \  |    |  /
              \ |    | /
                (1)
```

Simplified (each element appears once):

```
                     30
                   / | \
                  /  |  \
                6   10   15
               / \   |   / \
              2   3  5  /   \
               \  | /  /     |
                \ |/  /      |
                 1 ──────────┘
```

Proper Hasse diagram:

```
                  (30)
                /  |  \
              /    |    \
           (6)   (10)   (15)
           / \    |     / \
         (2) (3) (5)--+   |
           \  |  /        |
            \ | /         |
              (1)---------+
```

---

## 6. Identifying Special Elements from Hasse Diagrams

```
  ┌────────────────────────────────────────────────┐
  │   Visual Identification:                       │
  │                                                │
  │   Maximum:  Single top element                 │
  │   Minimum:  Single bottom element              │
  │   Maximal:  Elements with no upward edges      │
  │   Minimal:  Elements with no downward edges    │
  │                                                │
  │   Upper bounds of {a,b}: Elements above        │
  │     both a and b                               │
  │   LUB: Lowest such upper bound                 │
  │   Lower bounds of {a,b}: Elements below        │
  │     both a and b                               │
  │   GLB: Highest such lower bound                │
  └────────────────────────────────────────────────┘
```

**Example** using divisors of 30:

For $B = \{6, 10\}$:
- Upper bounds: elements above both 6 and 10 → $\{30\}$
- LUB = $30$ = $\text{lcm}(6, 10)$
- Lower bounds: elements below both 6 and 10 → $\{1, 2\}$
- GLB = $2$ = $\gcd(6, 10)$

---

## 7. Chains and Antichains in Hasse Diagrams

A **chain** is a set of mutually comparable elements (a path in the Hasse diagram).
An **antichain** is a set of mutually incomparable elements (no element above/below another).

```
  Example: P({a,b,c})
  
  Chain:    ∅ ⊂ {a} ⊂ {a,b} ⊂ {a,b,c}    (length 4)
  
  Antichain: {a}, {b}, {c}                  (size 3)
  Antichain: {a,b}, {a,c}, {b,c}           (size 3)
```

**Dilworth's Theorem:** The minimum number of chains needed to cover a poset equals the maximum size of an antichain.

---

## 8. Topological Sorting

A **topological sort** of a poset is a total order extending the partial order.

For the divisibility poset $\{1, 2, 3, 6\}$:
- Valid topological sorts: $1, 2, 3, 6$ or $1, 3, 2, 6$
- Invalid: $2, 1, 3, 6$ (because $1 \preceq 2$, so 1 must come before 2)

```
  Algorithm (Kahn's):
  1. Find a minimal element
  2. Output it and remove it
  3. Repeat
  
  Step 1: Minimal = {1} → output 1, remove 1
  Step 2: Minimal = {2, 3} → pick 2, output 2
  Step 3: Minimal = {3} → output 3
  Step 4: Minimal = {6} → output 6
  
  Result: 1, 2, 3, 6
```

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Hasse Diagrams in Practice                │
  │                                                   │
  │  1. Course Prerequisites                         │
  │     Courses as nodes, prereq as partial order    │
  │     Topological sort = valid semester schedule   │
  │                                                   │
  │  2. Build Systems (Make, Gradle)                 │
  │     Dependencies between compilation targets     │
  │     Build order = topological sort               │
  │                                                   │
  │  3. Organizational Hierarchy                     │
  │     Reporting structure in a company             │
  │                                                   │
  │  4. Inheritance Hierarchies (OOP)                │
  │     Class hierarchies with multiple inheritance  │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Hasse diagram | Simplified poset graph (no self-loops, no transitive edges) |
| Covering relation | $a \prec b$ with nothing between |
| Reading | Upward path = related |
| Chain | Totally ordered subset |
| Antichain | Mutually incomparable subset |
| Dilworth's Theorem | Min chains to cover = max antichain size |
| Topological sort | Linear extension of partial order |

---

## ❓ Quick Revision Questions

1. **Draw the Hasse diagram for $(\{1, 2, 4, 8, 16\}, \mid)$.**

2. **From the Hasse diagram of $\mathcal{P}(\{1,2,3\})$, identify all maximal chains.**

3. **Find the LUB and GLB of $\{2, 5\}$ in $(\{1, 2, 3, 5, 6, 10, 15, 30\}, \mid)$.**

4. **List all possible topological sorts of $\{1, 2, 3, 6\}$ under divisibility.**

5. **What is the longest chain and largest antichain in the divisors of 12 poset?**

6. **Explain why Hasse diagrams cannot have cycles.**

---

[← Previous: Partial Ordering](04-partial-ordering.md) | [Back to README](../README.md) | [Next: Lattices →](06-lattices.md)
