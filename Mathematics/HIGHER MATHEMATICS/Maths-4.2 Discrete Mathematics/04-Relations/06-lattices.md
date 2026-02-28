# Chapter 4.6: Lattices

[← Previous: Hasse Diagrams](05-hasse-diagrams.md) | [Back to README](../README.md) | [Next: Types of Functions →](../05-Functions/01-types-of-functions.md)

---

## 📋 Chapter Overview

A **lattice** is a special type of poset where every pair of elements has both a least upper bound (join) and a greatest lower bound (meet). Lattices appear throughout mathematics and computer science, from logic to programming language theory.

---

## 1. Definition

A poset $(L, \preceq)$ is a **lattice** if for every pair $a, b \in L$:

1. The **join** (LUB / supremum) $a \lor b$ exists
2. The **meet** (GLB / infimum) $a \land b$ exists

| Operation | Symbol | Meaning |
|-----------|:------:|---------|
| Join | $a \lor b$ | Least upper bound of $\{a, b\}$ |
| Meet | $a \land b$ | Greatest lower bound of $\{a, b\}$ |

---

## 2. Examples of Lattices

### 2.1 $(\mathcal{P}(S), \subseteq)$ — Power Set Lattice

$$A \lor B = A \cup B, \quad A \land B = A \cap B$$

```
                   {a,b,c}
                  /   |   \
                /     |     \
           {a,b}  {a,c}  {b,c}
              \    \|/    /
               {a} {b} {c}
                \   |   /
                  \ | /
                    ∅
                    
  Meet: {a,b} ∧ {b,c} = {a,b} ∩ {b,c} = {b}
  Join: {a,b} ∨ {b,c} = {a,b} ∪ {b,c} = {a,b,c}
```

### 2.2 $(\{1, 2, 3, 5, 6, 10, 15, 30\}, \mid)$ — Divisor Lattice

$$a \lor b = \text{lcm}(a, b), \quad a \land b = \gcd(a, b)$$

```
                  30
                / | \
              6  10  15
             /\ / \ / \
            2  3   5
             \ |  /
               1
               
  Meet: 6 ∧ 10 = gcd(6,10) = 2
  Join: 6 ∨ 10 = lcm(6,10) = 30
```

### 2.3 Not-a-Lattice Example

```
  The "diamond problem":
  
      d
     / \
    b   c
     \ /
      a
      
  This IS a lattice ✓
  
  But this is NOT:
  
    c   d
    |\ /|
    | X |
    |/ \|
    a   b
    
  a ∨ b = ? Both c and d are upper bounds,
  but neither c ≤ d nor d ≤ c.
  No LEAST upper bound exists → NOT a lattice.
```

---

## 3. Properties of Meet and Join

For all $a, b, c \in L$:

| Property | Meet ($\land$) | Join ($\lor$) |
|----------|:---:|:---:|
| **Commutative** | $a \land b = b \land a$ | $a \lor b = b \lor a$ |
| **Associative** | $(a \land b) \land c = a \land (b \land c)$ | $(a \lor b) \lor c = a \lor (b \lor c)$ |
| **Idempotent** | $a \land a = a$ | $a \lor a = a$ |
| **Absorption** | $a \land (a \lor b) = a$ | $a \lor (a \land b) = a$ |

**Connecting order and operations:**

$$a \preceq b \iff a \land b = a \iff a \lor b = b$$

---

## 4. Bounded Lattices

A lattice is **bounded** if it has:

- A **least element** (bottom) $\mathbf{0}$: $\forall a, \; \mathbf{0} \preceq a$
- A **greatest element** (top) $\mathbf{1}$: $\forall a, \; a \preceq \mathbf{1}$

| Property | Formula |
|----------|---------|
| $a \lor \mathbf{0} = a$ | Join with bottom = identity |
| $a \land \mathbf{1} = a$ | Meet with top = identity |
| $a \lor \mathbf{1} = \mathbf{1}$ | Join with top = top |
| $a \land \mathbf{0} = \mathbf{0}$ | Meet with bottom = bottom |

---

## 5. Complemented Lattices

In a bounded lattice, an element $b$ is a **complement** of $a$ if:

$$a \lor b = \mathbf{1} \quad \text{and} \quad a \land b = \mathbf{0}$$

A lattice is **complemented** if every element has at least one complement.

**Example:** $\mathcal{P}(\{1,2,3\})$ is complemented: the complement of $A$ is $A^c$.

```
  In P({1,2,3}):
  
  Complement of {1}   = {2,3}
  Complement of {1,2} = {3}
  Complement of ∅     = {1,2,3}
  
  Check: {1} ∨ {2,3} = {1} ∪ {2,3} = {1,2,3} = 1 ✓
         {1} ∧ {2,3} = {1} ∩ {2,3} = ∅ = 0         ✓
```

---

## 6. Distributive Lattices

A lattice is **distributive** if:

$$a \land (b \lor c) = (a \land b) \lor (a \land c)$$
$$a \lor (b \land c) = (a \lor b) \land (a \lor c)$$

(Either one implies the other in a lattice.)

```
  ┌──────────────────────────────────────────────────┐
  │   Distributive vs. Non-Distributive               │
  │                                                   │
  │   Distributive:         Non-distributive:         │
  │                                                   │
  │       1                     1                     │
  │      / \                  / | \                   │
  │     a   b                a  b  c                  │
  │      \ /                  \ | /                   │
  │       0                     0                     │
  │                                                   │
  │   "Diamond" M₃             "Pentagon" N₅         │
  │   is distributive       is NOT distributive       │
  └──────────────────────────────────────────────────┘
```

**Theorem (Birkhoff):** A lattice is non-distributive iff it contains a sublattice isomorphic to $M_3$ (diamond) or $N_5$ (pentagon).

---

## 7. Boolean Lattice (Boolean Algebra)

A **Boolean lattice** is a complemented distributive lattice.

$$\text{Boolean Lattice} = \text{Distributive} + \text{Complemented} + \text{Bounded}$$

**Key example:** $(\mathcal{P}(S), \subseteq)$ with $\cup, \cap, ^c$ is a Boolean lattice.

| Property | Status in Boolean Lattice |
|----------|:---:|
| Bounded | ✅ |
| Complemented | ✅ (unique complements) |
| Distributive | ✅ |

In a Boolean lattice, complements are **unique**.

---

## 8. Lattice Homomorphisms

A function $f: L_1 \to L_2$ is a **lattice homomorphism** if:

$$f(a \land b) = f(a) \land f(b)$$
$$f(a \lor b) = f(a) \lor f(b)$$

A **lattice isomorphism** is a bijective homomorphism.

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Lattices in Practice                      │
  │                                                   │
  │  1. Type Systems                                 │
  │     Types form a lattice under subtyping          │
  │     Meet = most specific common supertype         │
  │                                                   │
  │  2. Access Control                               │
  │     Security clearance levels form a lattice      │
  │     (Top Secret > Secret > Classified > Public)  │
  │                                                   │
  │  3. Digital Logic                                │
  │     Boolean algebra ≅ Boolean lattice             │
  │     AND = meet, OR = join, NOT = complement       │
  │                                                   │
  │  4. Concept Analysis                             │
  │     Formal concept analysis uses concept lattices │
  │                                                   │
  │  5. Abstract Interpretation                       │
  │     Program analysis domains form lattices        │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Lattice | Poset where every pair has a meet and join |
| Meet ($\land$) | Greatest lower bound |
| Join ($\lor$) | Least upper bound |
| Bounded | Has top ($\mathbf{1}$) and bottom ($\mathbf{0}$) |
| Complemented | Every element has a complement |
| Distributive | $\land$ distributes over $\lor$ (and vice versa) |
| Boolean lattice | Complemented + Distributive + Bounded |
| Key example | $(\mathcal{P}(S), \subseteq, \cup, \cap, ^c)$ |

---

## ❓ Quick Revision Questions

1. **Verify that $(\{1, 2, 3, 6\}, \mid)$ is a lattice. Compute $2 \land 3$ and $2 \lor 3$.**

2. **Is $(\{1, 2, 3, 4\}, \mid)$ a lattice? Why or why not?**

3. **In $\mathcal{P}(\{a, b\})$, find the complement of $\{a\}$.**

4. **State the absorption laws for lattices.**

5. **What makes a lattice a Boolean algebra? Give an example.**

6. **Show that the diamond lattice $M_3$ is non-distributive by finding a counterexample to the distributive law.**

---

[← Previous: Hasse Diagrams](05-hasse-diagrams.md) | [Back to README](../README.md) | [Next: Types of Functions →](../05-Functions/01-types-of-functions.md)
