# Chapter 10.2: Boolean Functions

[← Previous: Boolean Operations](01-boolean-operations.md) | [Back to README](../README.md) | [Next: Minimization with K-Maps →](03-minimization-kmaps.md)

---

## 📋 Chapter Overview

A **Boolean function** maps $n$ binary inputs to a single binary output. This chapter covers representing functions via truth tables, **minterms** and **maxterms**, and the two canonical forms — **Sum of Products (SOP)** and **Product of Sums (POS)** — which are the starting point for circuit minimization.

---

## 1. Boolean Function Definition

A Boolean function of $n$ variables is a mapping:

$$f: \{0,1\}^n \to \{0,1\}$$

```
  Example: f(a, b, c) = ab + c̄
  
  Inputs: 3 binary variables → 2³ = 8 rows in truth table
  Output: 1 binary value per row
```

---

## 2. Truth Table Representation

Every Boolean function can be **completely** specified by its truth table.

### Example: $f(a,b,c) = ab + \overline{c}$

```
  ┌───┬───┬───┬────┬────┬──────────┬─────────┐
  │ a │ b │ c │ ab │ c̄  │ ab + c̄   │ Row #   │
  ├───┼───┼───┼────┼────┼──────────┼─────────┤
  │ 0 │ 0 │ 0 │  0 │  1 │    1     │  m₀     │
  │ 0 │ 0 │ 1 │  0 │  0 │    0     │  m₁     │
  │ 0 │ 1 │ 0 │  0 │  1 │    1     │  m₂     │
  │ 0 │ 1 │ 1 │  0 │  0 │    0     │  m₃     │
  │ 1 │ 0 │ 0 │  0 │  1 │    1     │  m₄     │
  │ 1 │ 0 │ 1 │  0 │  0 │    0     │  m₅     │
  │ 1 │ 1 │ 0 │  1 │  1 │    1     │  m₆     │
  │ 1 │ 1 │ 1 │  1 │  0 │    1     │  m₇     │
  └───┴───┴───┴────┴────┴──────────┴─────────┘
  
  f = 1 at rows: 0, 2, 4, 6, 7
  f = 0 at rows: 1, 3, 5
```

---

## 3. Minterms

A **minterm** $m_i$ is a product (AND) term containing **every** variable exactly once, either complemented or uncomplemented, such that the term equals 1 for exactly **one** row of the truth table.

### For $n = 3$ variables ($a, b, c$):

```
  ┌──────┬───┬───┬───┬──────────────┬──────────┐
  │ Row  │ a │ b │ c │   Minterm     │ Symbol   │
  ├──────┼───┼───┼───┼──────────────┼──────────┤
  │  0   │ 0 │ 0 │ 0 │  ā · b̄ · c̄   │   m₀     │
  │  1   │ 0 │ 0 │ 1 │  ā · b̄ · c   │   m₁     │
  │  2   │ 0 │ 1 │ 0 │  ā · b · c̄   │   m₂     │
  │  3   │ 0 │ 1 │ 1 │  ā · b · c   │   m₃     │
  │  4   │ 1 │ 0 │ 0 │  a · b̄ · c̄   │   m₄     │
  │  5   │ 1 │ 0 │ 1 │  a · b̄ · c   │   m₅     │
  │  6   │ 1 │ 1 │ 0 │  a · b · c̄   │   m₆     │
  │  7   │ 1 │ 1 │ 1 │  a · b · c   │   m₇     │
  └──────┴───┴───┴───┴──────────────┴──────────┘
```

**Rule:** Variable appears **uncomplemented** if its value is 1, **complemented** if 0.

---

## 4. Maxterms

A **maxterm** $M_i$ is a sum (OR) term containing every variable exactly once, such that the term equals 0 for exactly **one** row.

### For $n = 3$ variables ($a, b, c$):

```
  ┌──────┬───┬───┬───┬──────────────┬──────────┐
  │ Row  │ a │ b │ c │   Maxterm     │ Symbol   │
  ├──────┼───┼───┼───┼──────────────┼──────────┤
  │  0   │ 0 │ 0 │ 0 │  a + b + c   │   M₀     │
  │  1   │ 0 │ 0 │ 1 │  a + b + c̄   │   M₁     │
  │  2   │ 0 │ 1 │ 0 │  a + b̄ + c   │   M₂     │
  │  3   │ 0 │ 1 │ 1 │  a + b̄ + c̄   │   M₃     │
  │  4   │ 1 │ 0 │ 0 │  ā + b + c   │   M₄     │
  │  5   │ 1 │ 0 │ 1 │  ā + b + c̄   │   M₅     │
  │  6   │ 1 │ 1 │ 0 │  ā + b̄ + c   │   M₆     │
  │  7   │ 1 │ 1 │ 1 │  ā + b̄ + c̄   │   M₇     │
  └──────┴───┴───┴───┴──────────────┴──────────┘
```

**Rule:** Variable appears **complemented** if its value is 1, **uncomplemented** if 0.

> **Key relationship:** $M_i = \overline{m_i}$

---

## 5. Canonical Sum of Products (SOP)

The canonical SOP (also called **disjunctive normal form**) expresses a function as the **OR of minterms** where $f = 1$.

### Notation

$$f(a,b,c) = \sum m(i_1, i_2, \ldots)$$

### Example

From the truth table in Section 2 where $f = 1$ at rows 0, 2, 4, 6, 7:

$$f(a,b,c) = \sum m(0, 2, 4, 6, 7)$$

$$= \overline{a}\,\overline{b}\,\overline{c} + \overline{a}\,b\,\overline{c} + a\,\overline{b}\,\overline{c} + ab\overline{c} + abc$$

```
  Visual:
  
  f = m₀ + m₂ + m₄ + m₆ + m₇
    = ā b̄ c̄ + ā b c̄ + a b̄ c̄ + a b c̄ + a b c
    
  Which simplifies to: c̄ + ab  (verified in next chapter)
```

---

## 6. Canonical Product of Sums (POS)

The canonical POS (also called **conjunctive normal form**) expresses a function as the **AND of maxterms** where $f = 0$.

### Notation

$$f(a,b,c) = \prod M(j_1, j_2, \ldots)$$

### Example

From the same truth table where $f = 0$ at rows 1, 3, 5:

$$f(a,b,c) = \prod M(1, 3, 5)$$

$$= (a + b + \overline{c})(a + \overline{b} + \overline{c})(\overline{a} + b + \overline{c})$$

---

## 7. SOP ↔ POS Conversion

The minterm numbers where $f = 1$ and maxterm numbers where $f = 0$ are **complements** of each other.

```
  Total rows for n=3: {0, 1, 2, 3, 4, 5, 6, 7}
  
  If f = Σm(0, 2, 4, 6, 7)  ← f=1 rows
  Then f = ΠM(1, 3, 5)       ← f=0 rows (the rest)
  
  Also: f̄ = Σm(1, 3, 5)  and  f̄ = ΠM(0, 2, 4, 6, 7)
```

---

## 8. Standard Forms (Simplified)

### Standard SOP

An expression that is a sum of product terms (not necessarily minterms):

$$f = ab + c\overline{d} + \overline{a}cd$$

Each term may have **fewer** variables than the full minterm.

### Standard POS

An expression that is a product of sum terms (not necessarily maxterms):

$$f = (a + b)(\overline{c} + d)(a + c + \overline{d})$$

### Converting Standard → Canonical SOP

Expand missing variables using $x + \overline{x} = 1$:

$$ab = ab(c + \overline{c}) = abc + ab\overline{c}$$

---

## 9. Obtaining SOP from Truth Table — Worked Example

**Problem:** Design a function that outputs 1 when the majority of 3 inputs are 1.

```
  Majority Function:
  ┌───┬───┬───┬──────┬─────────┐
  │ a │ b │ c │ f    │ Minterm │
  ├───┼───┼───┼──────┼─────────┤
  │ 0 │ 0 │ 0 │  0   │         │
  │ 0 │ 0 │ 1 │  0   │         │
  │ 0 │ 1 │ 0 │  0   │         │
  │ 0 │ 1 │ 1 │  1   │  m₃     │
  │ 1 │ 0 │ 0 │  0   │         │
  │ 1 │ 0 │ 1 │  1   │  m₅     │
  │ 1 │ 1 │ 0 │  1   │  m₆     │
  │ 1 │ 1 │ 1 │  1   │  m₇     │
  └───┴───┴───┴──────┴─────────┘
  
  Canonical SOP:
  f = Σm(3, 5, 6, 7)
    = ā b c + a b̄ c + a b c̄ + a b c
  
  Simplified (using K-map or algebra):
  f = ab + ac + bc
```

---

## 10. Functional Completeness

A set of operations is **functionally complete** if every Boolean function can be expressed using only those operations.

```
  Functionally complete sets:
  
  ┌─────────────────────────────────────────┐
  │  {AND, OR, NOT}    ← standard set       │
  │  {AND, NOT}        ← OR via De Morgan   │
  │  {OR, NOT}         ← AND via De Morgan  │
  │  {NAND}            ← universal gate     │
  │  {NOR}             ← universal gate     │
  └─────────────────────────────────────────┘
  
  NAND alone is sufficient:
    NOT a    = a NAND a
    a AND b  = (a NAND b) NAND (a NAND b)
    a OR b   = (a NAND a) NAND (b NAND b)
```

---

## 11. Literal Count and Cost

The **cost** of a Boolean expression is often measured by:

- **Literal count:** Total number of variable appearances
- **Gate count:** Number of logic gates needed
- **Gate inputs:** Total inputs across all gates

```
  f = ā b c + a b̄ c + a b c̄ + a b c
  
  Literal count: 12  (3 per term × 4 terms)
  
  Simplified: f = ab + ac + bc
  
  Literal count: 6   (2 per term × 3 terms) ← 50% reduction!
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Minterm $m_i$ | AND of all variables; equals 1 for row $i$ only |
| Maxterm $M_i$ | OR of all variables; equals 0 for row $i$ only |
| $M_i = \overline{m_i}$ | Maxterm is complement of corresponding minterm |
| Canonical SOP | $f = \sum m(\text{rows where } f=1)$ |
| Canonical POS | $f = \prod M(\text{rows where } f=0)$ |
| Standard SOP | Sum of product terms (not necessarily minterms) |
| Standard POS | Product of sum terms (not necessarily maxterms) |
| Functional completeness | {NAND} or {NOR} alone can represent any function |
| Literal count | Number of variable appearances (optimization metric) |

---

## ❓ Quick Revision Questions

1. **Write the canonical SOP and POS for the XOR function $f(a,b) = a \oplus b$.**

2. **A function has minterms $\sum m(1, 3, 5, 7)$ for 3 variables. What is the simplified expression?**

3. **Convert $f = a + \overline{b}c$ into canonical SOP form (3 variables: $a, b, c$).**

4. **Given $f(a,b,c) = \prod M(0, 2, 4)$, find the equivalent $\sum m$ form.**

5. **Why is {AND, OR} NOT functionally complete? What's missing?**

6. **Express NOT, AND, and OR using only NOR gates.**

---

[← Previous: Boolean Operations](01-boolean-operations.md) | [Back to README](../README.md) | [Next: Minimization with K-Maps →](03-minimization-kmaps.md)
