# Chapter 10.1: Boolean Operations

[← Previous: Minimum Spanning Tree Algorithms](../09-Trees/05-minimum-spanning-tree-algorithms.md) | [Back to README](../README.md) | [Next: Boolean Functions →](02-boolean-functions.md)

---

## 📋 Chapter Overview

**Boolean algebra** is the mathematical foundation of digital logic and computer science. Developed by George Boole in 1854, it operates on **binary values** (0 and 1, or True and False) using a small set of operations. This chapter covers Boolean algebra axioms, fundamental operations, key theorems, and duality.

---

## 1. Boolean Algebra: Definition

A **Boolean algebra** is a set $B$ with two binary operations $+$ (OR) and $\cdot$ (AND), a unary operation $\overline{\phantom{x}}$ (complement), and two distinguished elements $0$ and $1$, satisfying the following axioms:

### Huntington's Axioms

| # | Axiom | OR form ($+$) | AND form ($\cdot$) |
|:-:|-------|:------------:|:-----------------:|
| 1 | **Closure** | $a + b \in B$ | $a \cdot b \in B$ |
| 2 | **Identity** | $a + 0 = a$ | $a \cdot 1 = a$ |
| 3 | **Commutative** | $a + b = b + a$ | $a \cdot b = b \cdot a$ |
| 4 | **Distributive** | $a \cdot (b + c) = ab + ac$ | $a + (b \cdot c) = (a+b)(a+c)$ |
| 5 | **Complement** | $a + \overline{a} = 1$ | $a \cdot \overline{a} = 0$ |

> **Note:** Boolean algebra has **two** distributive laws — the second ($a + bc = (a+b)(a+c)$) does **not** hold in ordinary algebra!

---

## 2. Fundamental Boolean Operations

### AND (Boolean Product / Conjunction)

$$a \cdot b = 1 \text{ only when both } a = 1 \text{ and } b = 1$$

```
  Truth Table:
  ┌───┬───┬───────┐
  │ a │ b │ a · b │
  ├───┼───┼───────┤
  │ 0 │ 0 │   0   │
  │ 0 │ 1 │   0   │
  │ 1 │ 0 │   0   │
  │ 1 │ 1 │   1   │
  └───┴───┴───────┘
  
  Notation: a · b, ab, a ∧ b
```

### OR (Boolean Sum / Disjunction)

$$a + b = 0 \text{ only when both } a = 0 \text{ and } b = 0$$

```
  Truth Table:
  ┌───┬───┬───────┐
  │ a │ b │ a + b │
  ├───┼───┼───────┤
  │ 0 │ 0 │   0   │
  │ 0 │ 1 │   1   │
  │ 1 │ 0 │   1   │
  │ 1 │ 1 │   1   │
  └───┴───┴───────┘
  
  Notation: a + b, a ∨ b
```

### NOT (Complement / Negation)

$$\overline{a} = \begin{cases} 1 & \text{if } a = 0 \\ 0 & \text{if } a = 1 \end{cases}$$

```
  Truth Table:
  ┌───┬──────┐
  │ a │  ā   │
  ├───┼──────┤
  │ 0 │  1   │
  │ 1 │  0   │
  └───┴──────┘
  
  Notation: ā, a', ¬a, NOT a
```

### XOR (Exclusive OR)

$$a \oplus b = a\overline{b} + \overline{a}b = 1 \text{ when } a \neq b$$

```
  Truth Table:
  ┌───┬───┬───────┐
  │ a │ b │ a ⊕ b │
  ├───┼───┼───────┤
  │ 0 │ 0 │   0   │
  │ 0 │ 1 │   1   │
  │ 1 │ 0 │   1   │
  │ 1 │ 1 │   0   │
  └───┴───┴───────┘
```

### XNOR (Equivalence)

$$a \odot b = ab + \overline{a}\,\overline{b} = 1 \text{ when } a = b$$

---

## 3. Operator Precedence

$$\text{NOT} > \text{AND} > \text{XOR} > \text{OR}$$

```
  Example: ā · b + c
  
  Step 1: Complement a → ā
  Step 2: AND with b  → ā · b
  Step 3: OR with c   → (ā · b) + c
  
  So ā · b + c = (ā · b) + c   (NOT: ā · (b + c))
```

---

## 4. Key Boolean Theorems

All theorems come in **dual pairs** (swap $+\leftrightarrow\cdot$ and $0\leftrightarrow1$):

| # | Theorem | OR form | AND form |
|:-:|---------|:-------:|:--------:|
| T1 | **Idempotent** | $a + a = a$ | $a \cdot a = a$ |
| T2 | **Null/Domination** | $a + 1 = 1$ | $a \cdot 0 = 0$ |
| T3 | **Involution** | $\overline{\overline{a}} = a$ | — |
| T4 | **Absorption** | $a + ab = a$ | $a(a + b) = a$ |
| T5 | **De Morgan's** | $\overline{a + b} = \overline{a} \cdot \overline{b}$ | $\overline{a \cdot b} = \overline{a} + \overline{b}$ |
| T6 | **Consensus** | $ab + \overline{a}c + bc = ab + \overline{a}c$ | $(a+b)(\overline{a}+c)(b+c) = (a+b)(\overline{a}+c)$ |

---

## 5. De Morgan's Laws — Detailed

### Statement

$$\overline{a + b} = \overline{a} \cdot \overline{b}$$
$$\overline{a \cdot b} = \overline{a} + \overline{b}$$

### Proof by Truth Table

```
  ┌───┬───┬──────────┬──────┬──────┬──────────────┐
  │ a │ b │ a + b    │ (a+b)' │  a'  │ a' · b'      │
  ├───┼───┼──────────┼────────┼──────┼──────────────┤
  │ 0 │ 0 │    0     │   1    │  1·1 │      1       │ ✓
  │ 0 │ 1 │    1     │   0    │  1·0 │      0       │ ✓
  │ 1 │ 0 │    1     │   0    │  0·1 │      0       │ ✓
  │ 1 │ 1 │    1     │   0    │  0·0 │      0       │ ✓
  └───┴───┴──────────┴────────┴──────┴──────────────┘
  
  (a+b)' = a' · b'  ✓  (columns match)
```

### Generalized De Morgan's

$$\overline{x_1 + x_2 + \cdots + x_n} = \overline{x_1} \cdot \overline{x_2} \cdots \overline{x_n}$$

$$\overline{x_1 \cdot x_2 \cdots x_n} = \overline{x_1} + \overline{x_2} + \cdots + \overline{x_n}$$

---

## 6. Duality Principle

The **dual** of a Boolean expression is obtained by:

- Swapping $+$ and $\cdot$
- Swapping $0$ and $1$
- Keeping complements unchanged

```
  Expression:         a + (b · c) = (a + b)(a + c)
  Dual:               a · (b + c) = ab + ac
  
  Expression:         a + 0 = a
  Dual:               a · 1 = a
  
  Expression:         a + ā = 1
  Dual:               a · ā = 0
```

**Duality Principle:** If a Boolean identity is valid, its **dual** is also valid.

---

## 7. Simplification Using Boolean Algebra

### Example 1

Simplify $F = a\overline{b}c + abc + \overline{a}bc$

$$F = a\overline{b}c + abc + \overline{a}bc$$
$$= ac(\overline{b} + b) + \overline{a}bc \quad \text{(factor } ac\text{)}$$
$$= ac \cdot 1 + \overline{a}bc \quad \text{(complement)}$$
$$= ac + \overline{a}bc$$
$$= c(a + \overline{a}b) \quad \text{(factor } c\text{)}$$
$$= c(a + b) \quad \text{(by absorption: } a + \overline{a}b = a + b\text{)}$$

### Example 2

Simplify $F = (a + b)(a + \overline{b})$

$$F = a \cdot a + a\overline{b} + ba + b\overline{b}$$
$$= a + a\overline{b} + ab + 0$$
$$= a + a(\overline{b} + b)$$
$$= a + a = a$$

Or by **distributive law** directly: $(a + b)(a + \overline{b}) = a + b\overline{b} = a + 0 = a$

### Example 3

Simplify $F = \overline{(\overline{a}b + c)}$

$$F = \overline{(\overline{a}b)} \cdot \overline{c} \quad \text{(De Morgan's)}$$
$$= (a + \overline{b}) \cdot \overline{c} \quad \text{(De Morgan's on } \overline{a}b\text{)}$$
$$= a\overline{c} + \overline{b}\,\overline{c}$$

---

## 8. Boolean Functions of $n$ Variables

For $n$ Boolean variables, there are:

- $2^n$ possible input combinations
- $2^{2^n}$ possible Boolean functions

```
  n     Input combinations     Functions
  ──────────────────────────────────
  1           2                    4
  2           4                   16
  3           8                  256
  4          16                65,536
```

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │          Boolean Algebra Applications              │
  │                                                    │
  │  1. Digital Circuit Design                        │
  │     Logic gates implement Boolean operations      │
  │                                                    │
  │  2. Search Engines                                │
  │     Boolean queries: "cat AND dog NOT fish"       │
  │                                                    │
  │  3. Database Queries                              │
  │     SQL WHERE clauses use Boolean logic           │
  │                                                    │
  │  4. Programming                                   │
  │     if/else conditions evaluated as Boolean       │
  │                                                    │
  │  5. Set Operations                                │
  │     Union = OR, Intersection = AND, Comp = NOT    │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| AND ($a \cdot b$) | 1 only when both inputs are 1 |
| OR ($a + b$) | 0 only when both inputs are 0 |
| NOT ($\overline{a}$) | Flips the bit |
| XOR ($a \oplus b$) | 1 when inputs differ |
| Idempotent | $a + a = a$, $a \cdot a = a$ |
| Absorption | $a + ab = a$, $a(a+b) = a$ |
| De Morgan's | $\overline{a+b} = \overline{a} \cdot \overline{b}$ |
| Duality | Swap $+/\cdot$ and $0/1$ preserves identity |
| $n$-variable functions | $2^{2^n}$ possible Boolean functions |

---

## ❓ Quick Revision Questions

1. **Simplify $F = a + \overline{a}b + \overline{a}\,\overline{b}$ using Boolean algebra.**

2. **Apply De Morgan's law to $\overline{(a + b)(c + d)}$.**

3. **How many distinct Boolean functions of 3 variables exist?**

4. **Find the dual of the expression: $a(b + \overline{c}) + 0$.**

5. **Prove the absorption law $a + ab = a$ using the axioms.**

6. **Simplify $F = \overline{a}b + ab + b\overline{c}$ to a minimal form.**

---

[← Previous: Minimum Spanning Tree Algorithms](../09-Trees/05-minimum-spanning-tree-algorithms.md) | [Back to README](../README.md) | [Next: Boolean Functions →](02-boolean-functions.md)
