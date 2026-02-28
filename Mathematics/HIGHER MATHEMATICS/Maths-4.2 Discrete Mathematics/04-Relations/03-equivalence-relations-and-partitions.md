# Chapter 4.3: Equivalence Relations and Partitions

[← Previous: Types of Relations](02-types-of-relations.md) | [Back to README](../README.md) | [Next: Partial Ordering →](04-partial-ordering.md)

---

## 📋 Chapter Overview

An **equivalence relation** groups elements that are "alike" into classes. This chapter explores the deep connection between equivalence relations and **partitions** of a set — two sides of the same coin.

---

## 1. Definition of Equivalence Relation

A relation $R$ on set $A$ is an **equivalence relation** if it is:

1. **Reflexive:** $\forall a \in A, \; a \, R \, a$
2. **Symmetric:** $\forall a, b \in A, \; a \, R \, b \Rightarrow b \, R \, a$
3. **Transitive:** $\forall a, b, c \in A, \; a \, R \, b \land b \, R \, c \Rightarrow a \, R \, c$

We write $a \sim b$ to denote $a$ is equivalent to $b$.

---

## 2. Common Examples

| Relation | Set | Reflexive | Symmetric | Transitive | Equiv? |
|----------|-----|:---------:|:---------:|:----------:|:------:|
| $a \equiv b \pmod{n}$ | $\mathbb{Z}$ | ✅ | ✅ | ✅ | ✅ |
| $=$ (equality) | Any set | ✅ | ✅ | ✅ | ✅ |
| "same birthday" | People | ✅ | ✅ | ✅ | ✅ |
| $\leq$ | $\mathbb{Z}$ | ✅ | ❌ | ✅ | ❌ |
| "is friend of" | People | ✅ | ✅ | ❌ | ❌ |

---

## 3. Equivalence Classes

If $\sim$ is an equivalence relation on $A$, the **equivalence class** of $a$ is:

$$[a] = \{x \in A : x \sim a\}$$

**Key Properties:**

1. Every element belongs to its own class: $a \in [a]$
2. Two classes are **equal or disjoint**: $[a] = [b]$ or $[a] \cap [b] = \emptyset$
3. The classes **cover** $A$: $\bigcup_{a \in A} [a] = A$

---

## 4. Detailed Example: Congruence Modulo 3

On $\mathbb{Z}$, define $a \sim b$ iff $3 \mid (a - b)$.

The equivalence classes are:

$$[0] = \{\ldots, -6, -3, 0, 3, 6, 9, \ldots\}$$
$$[1] = \{\ldots, -5, -2, 1, 4, 7, 10, \ldots\}$$
$$[2] = \{\ldots, -4, -1, 2, 5, 8, 11, \ldots\}$$

```
  ┌─────────────────────────────────────────────────────┐
  │   Equivalence Classes for ≡ (mod 3)                 │
  │                                                      │
  │   ...─6──┬──3──┬──0──┬──3──┬──6──┬──9──...  [0]     │
  │          │     │     │     │     │                    │
  │   ...─5──┤─2───┤──1──┤──4──┤──7──┤─10──...  [1]     │
  │          │     │     │     │     │                    │
  │   ...─4──┴─1───┴──2──┴──5──┴──8──┴─11──...  [2]     │
  │                                                      │
  │   Every integer falls into exactly ONE class         │
  └─────────────────────────────────────────────────────┘
```

---

## 5. Definition of Partition

A **partition** of a set $A$ is a collection of non-empty subsets $\{A_1, A_2, \ldots\}$ such that:

1. **Pairwise disjoint:** $A_i \cap A_j = \emptyset$ for $i \neq j$
2. **Union is all of $A$:** $\bigcup_i A_i = A$

```
  Partition of {1, 2, 3, 4, 5, 6}:
  
  ┌─────────────────────────────────┐
  │  ┌───────┐ ┌─────┐ ┌─────────┐ │
  │  │ 1   4 │ │ 2 5 │ │ 3     6 │ │
  │  │  A₁   │ │ A₂  │ │   A₃    │ │
  │  └───────┘ └─────┘ └─────────┘ │
  │                                 │
  │  Every element in exactly one   │
  │  block. No overlaps. No gaps.   │
  └─────────────────────────────────┘
```

---

## 6. The Fundamental Theorem: Equivalence Relations ↔ Partitions

**Theorem:** There is a one-to-one correspondence between:
- Equivalence relations on $A$
- Partitions of $A$

**Direction 1: Equivalence Relation → Partition**
The equivalence classes form a partition.

**Direction 2: Partition → Equivalence Relation**
Define $a \sim b$ iff $a$ and $b$ are in the same block.

```
  ┌────────────────────────────┐
  │   Equivalence Relation     │
  │   a ~ b iff 3 | (a-b)     │
  │          ↕                 │
  │   Partition                │
  │   {[0], [1], [2]}         │
  │                            │
  │   Two views of the same   │
  │   mathematical structure! │
  └────────────────────────────┘
```

---

## 7. Quotient Set

The **quotient set** $A / \sim$ is the set of all equivalence classes:

$$A / \sim \; = \{[a] : a \in A\}$$

**Examples:**

| Relation | Set | Quotient Set |
|----------|-----|-------------|
| $\equiv \pmod{3}$ | $\mathbb{Z}$ | $\mathbb{Z}/3\mathbb{Z} = \{[0], [1], [2]\}$ |
| $\equiv \pmod{2}$ | $\mathbb{Z}$ | $\mathbb{Z}/2\mathbb{Z} = \{[0], [1]\}$ = \{evens, odds\} |
| Equality | Any $A$ | $A$ itself (each class is a singleton) |
| "All equivalent" | $A$ | $\{A\}$ (single class) |

---

## 8. Counting Partitions: Bell Numbers

The number of partitions of a set with $n$ elements is the **Bell number** $B_n$:

| $n$ | $B_n$ | Partitions |
|:---:|:-----:|-----------|
| 0 | 1 | $\{\}$ |
| 1 | 1 | $\{\{1\}\}$ |
| 2 | 2 | $\{\{1,2\}\}, \{\{1\},\{2\}\}$ |
| 3 | 5 | 5 ways to partition $\{1,2,3\}$ |
| 4 | 15 | |
| 5 | 52 | |

**The 5 partitions of $\{1, 2, 3\}$:**

```
  1. {{1, 2, 3}}        ← all together
  2. {{1, 2}, {3}}      ← group 1,2
  3. {{1, 3}, {2}}      ← group 1,3
  4. {{2, 3}, {1}}      ← group 2,3
  5. {{1}, {2}, {3}}    ← all separate
```

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Equivalence Relations in Practice         │
  │                                                   │
  │  1. Modular Arithmetic                            │
  │     Clock arithmetic: 15:00 ≡ 3:00 (mod 12)     │
  │                                                   │
  │  2. Data Deduplication                            │
  │     Group identical files → store one copy        │
  │                                                   │
  │  3. Network Clusters                              │
  │     "Connected to" forms equivalence classes      │
  │     ≡ connected components                        │
  │                                                   │
  │  4. Type Checking                                 │
  │     Types form equivalence classes of values      │
  │                                                   │
  │  5. Hash Tables                                   │
  │     Keys with same hash → same bucket             │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Equivalence relation | Reflexive + Symmetric + Transitive |
| Equivalence class $[a]$ | $\{x : x \sim a\}$ |
| Partition | Disjoint, exhaustive division of $A$ |
| Fundamental Theorem | Equiv. relations ↔ Partitions (bijection) |
| Quotient set $A/\sim$ | Set of all equivalence classes |
| Bell number $B_n$ | Number of partitions of an $n$-element set |

---

## ❓ Quick Revision Questions

1. **Verify that congruence modulo 5 is an equivalence relation on $\mathbb{Z}$.**

2. **Find all equivalence classes of $\{1,2,3,4,5,6\}$ under the relation "same remainder when divided by 3."**

3. **State the Fundamental Theorem connecting equivalence relations and partitions.**

4. **Given the partition $\{\{a,c\}, \{b,d,e\}\}$ of $\{a,b,c,d,e\}$, write the corresponding equivalence relation.**

5. **How many equivalence relations exist on a set with 4 elements?**

6. **Give a real-world example of an equivalence relation and identify its equivalence classes.**

---

[← Previous: Types of Relations](02-types-of-relations.md) | [Back to README](../README.md) | [Next: Partial Ordering →](04-partial-ordering.md)
