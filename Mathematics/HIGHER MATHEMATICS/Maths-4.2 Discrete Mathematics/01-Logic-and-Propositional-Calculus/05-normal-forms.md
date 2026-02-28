# Chapter 1.5: Normal Forms (CNF and DNF)

[← Previous: Tautologies and Contradictions](04-tautologies-and-contradictions.md) | [Back to README](../README.md) | [Next: Predicate Logic →](06-predicate-logic.md)

---

## 📋 Chapter Overview

Normal forms provide a **standardized way** to write any logical expression. They are essential for automated reasoning, circuit design, and simplification. The two main normal forms are **Conjunctive Normal Form (CNF)** and **Disjunctive Normal Form (DNF)**.

---

## 1. Literals, Clauses, and Terms

### Literal
A **literal** is a variable or its negation: $p$, $\neg p$, $q$, $\neg q$

### Clause (for CNF)
A **clause** is a disjunction of literals: $(p \vee \neg q \vee r)$

### Term / Minterm / Maxterm
A **term** (for DNF) is a conjunction of literals: $(p \wedge \neg q \wedge r)$

---

## 2. Disjunctive Normal Form (DNF)

A proposition is in **DNF** if it is a **disjunction of conjunctive terms** (OR of ANDs).

$$\text{DNF: } (l_{11} \wedge l_{12} \wedge \cdots) \vee (l_{21} \wedge l_{22} \wedge \cdots) \vee \cdots$$

### Structure:
```
  DNF = Term₁ ∨ Term₂ ∨ ... ∨ Termₖ
  
  where each Term = literal₁ ∧ literal₂ ∧ ... ∧ literalₙ
```

### Examples

| Expression | In DNF? |
|-----------|:-------:|
| $(p \wedge q) \vee (\neg p \wedge r)$ | ✅ Yes |
| $p \vee (q \wedge r)$ | ✅ Yes |
| $p \wedge (q \vee r)$ | ❌ No (AND of OR) |
| $\neg(p \wedge q)$ | ❌ No (negation of compound) |

---

## 3. Conjunctive Normal Form (CNF)

A proposition is in **CNF** if it is a **conjunction of disjunctive clauses** (AND of ORs).

$$\text{CNF: } (l_{11} \vee l_{12} \vee \cdots) \wedge (l_{21} \vee l_{22} \vee \cdots) \wedge \cdots$$

### Structure:
```
  CNF = Clause₁ ∧ Clause₂ ∧ ... ∧ Clauseₖ
  
  where each Clause = literal₁ ∨ literal₂ ∨ ... ∨ literalₙ
```

### Examples

| Expression | In CNF? |
|-----------|:-------:|
| $(p \vee q) \wedge (\neg p \vee r)$ | ✅ Yes |
| $(p \vee q) \wedge r$ | ✅ Yes |
| $(p \wedge q) \vee r$ | ❌ No (OR of AND) |
| $p \vee (q \wedge r)$ | ❌ No |

---

## 4. Converting to DNF

### Algorithm:
1. **Eliminate implications:** $p \rightarrow q \equiv \neg p \vee q$, $p \leftrightarrow q \equiv (p \rightarrow q) \wedge (q \rightarrow p)$
2. **Push negations inward** using De Morgan's Laws
3. **Apply distribution:** $p \wedge (q \vee r) \equiv (p \wedge q) \vee (p \wedge r)$ — distribute $\wedge$ over $\vee$
4. **Simplify** using idempotent, negation, and identity laws

### Worked Example: Convert $(p \rightarrow q) \wedge r$ to DNF

**Step 1:** Eliminate implication
$$(\neg p \vee q) \wedge r$$

**Step 2:** Distribute $\wedge$ over $\vee$
$$(\neg p \wedge r) \vee (q \wedge r)$$

**Result:** $(\neg p \wedge r) \vee (q \wedge r)$ ← **DNF** ✅

---

### Worked Example: Convert $\neg(p \vee q) \vee (p \wedge \neg r)$ to DNF

**Step 1:** Apply De Morgan's to $\neg(p \vee q)$
$$(\neg p \wedge \neg q) \vee (p \wedge \neg r)$$

**Result:** Already in DNF ✅ (disjunction of conjunctive terms)

---

## 5. Converting to CNF

### Algorithm:
1. **Eliminate implications**
2. **Push negations inward** using De Morgan's Laws
3. **Apply distribution:** $p \vee (q \wedge r) \equiv (p \vee q) \wedge (p \vee r)$ — distribute $\vee$ over $\wedge$
4. **Simplify**

### Worked Example: Convert $(p \wedge q) \vee r$ to CNF

**Step 1:** Distribute $\vee$ over $\wedge$
$$(p \vee r) \wedge (q \vee r)$$

**Result:** $(p \vee r) \wedge (q \vee r)$ ← **CNF** ✅

---

### Worked Example: Convert $p \rightarrow (q \wedge r)$ to CNF

**Step 1:** Eliminate implication
$$\neg p \vee (q \wedge r)$$

**Step 2:** Distribute $\vee$ over $\wedge$
$$(\neg p \vee q) \wedge (\neg p \vee r)$$

**Result:** $(\neg p \vee q) \wedge (\neg p \vee r)$ ← **CNF** ✅

---

## 6. Minterms and Maxterms

For $n$ variables, each **row** of the truth table corresponds to a unique minterm and maxterm.

### Minterms (for DNF)

A **minterm** is a conjunction of all variables where each appears exactly once (complemented or not), corresponding to a row where the function is **True**.

### Maxterms (for CNF)

A **maxterm** is a disjunction of all variables where each appears exactly once, corresponding to a row where the function is **False**.

### For 2 variables $p, q$:

| Row | $p$ | $q$ | Minterm ($m_i$) | Maxterm ($M_i$) |
|:---:|:---:|:---:|:---:|:---:|
| 0 | F | F | $\neg p \wedge \neg q$ ($m_0$) | $p \vee q$ ($M_0$) |
| 1 | F | T | $\neg p \wedge q$ ($m_1$) | $p \vee \neg q$ ($M_1$) |
| 2 | T | F | $p \wedge \neg q$ ($m_2$) | $\neg p \vee q$ ($M_2$) |
| 3 | T | T | $p \wedge q$ ($m_3$) | $\neg p \vee \neg q$ ($M_3$) |

> **Key relationship:** $M_i = \neg m_i$

---

## 7. Canonical Forms from Truth Tables

### Canonical DNF (Sum of Minterms)

Take the **OR of all minterms** where the function is TRUE.

### Canonical CNF (Product of Maxterms)

Take the **AND of all maxterms** where the function is FALSE.

### Example: Find canonical DNF and CNF

Given truth table:

| $p$ | $q$ | $r$ | $f(p,q,r)$ |
|:---:|:---:|:---:|:---:|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 |

**Canonical DNF** (rows where $f = 1$: rows 1, 3, 4, 6, 7):
$$f = (\neg p \wedge \neg q \wedge r) \vee (\neg p \wedge q \wedge r) \vee (p \wedge \neg q \wedge \neg r) \vee (p \wedge q \wedge \neg r) \vee (p \wedge q \wedge r)$$

$$f = \sum m(1, 3, 4, 6, 7)$$

**Canonical CNF** (rows where $f = 0$: rows 0, 2, 5):
$$f = (p \vee q \vee r) \wedge (p \vee \neg q \vee r) \wedge (\neg p \vee q \vee \neg r)$$

$$f = \prod M(0, 2, 5)$$

---

## 8. Relationship Between CNF and DNF

```
  ┌─────────────────────────────────────────────┐
  │            Any Logical Expression            │
  │                                              │
  │     ┌──────────────┐  ┌──────────────┐       │
  │     │     DNF      │  │     CNF      │       │
  │     │              │  │              │       │
  │     │  OR of ANDs  │  │  AND of ORs  │       │
  │     │              │  │              │       │
  │     │  Σ minterms  │  │  Π maxterms  │       │
  │     │  (f = 1)     │  │  (f = 0)     │       │
  │     └──────────────┘  └──────────────┘       │
  │                                              │
  │  DNF of f  ←→  CNF of ¬f (after De Morgan)  │
  └─────────────────────────────────────────────┘
```

---

## 9. Comparison: CNF vs DNF

| Feature | DNF | CNF |
|---------|-----|-----|
| Structure | OR of ANDs | AND of ORs |
| Built from | Minterms (f = 1 rows) | Maxterms (f = 0 rows) |
| Notation | $\sum m(...)$ | $\prod M(...)$ |
| Tautology detection | Hard (must check all terms) | Easy (empty clause → not tautology) |
| Contradiction detection | Easy (empty → contradiction) | Hard |
| Used in | Circuit design (SOP form) | SAT solvers, resolution |

---

## 10. Real-World Applications

| Application | Normal Form Used |
|-------------|-----------------|
| **Digital Circuit Design** | DNF → Sum of Products (SOP) circuits |
| **SAT Solvers** | CNF is the standard input format |
| **Automated Theorem Proving** | CNF for resolution-based provers |
| **Database Query Optimization** | CNF/DNF for optimizing WHERE clauses |
| **Knowledge Representation** | CNF in logic programming (Prolog) |

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Literal | Variable or its negation: $p$ or $\neg p$ |
| DNF | Disjunction of conjunctions (OR of ANDs) |
| CNF | Conjunction of disjunctions (AND of ORs) |
| Minterm | Conjunction of all variables (one per truth table row) |
| Maxterm | Disjunction of all variables (one per truth table row) |
| To DNF | Distribute $\wedge$ over $\vee$ |
| To CNF | Distribute $\vee$ over $\wedge$ |
| Canonical DNF | $\sum$ minterms where $f = 1$ |
| Canonical CNF | $\prod$ maxterms where $f = 0$ |

---

## ❓ Quick Revision Questions

1. **Convert $(p \rightarrow q) \rightarrow r$ to (a) DNF and (b) CNF.**

2. **Write the canonical DNF for $f(p, q) = p \oplus q$.**

3. **If $f = \sum m(0, 2, 5, 7)$ for 3 variables, write the equivalent CNF using $\prod M$.**

4. **Is $(\neg p \vee q) \wedge (p \vee \neg r) \wedge (q \vee r)$ in CNF, DNF, or neither?**

5. **Why is CNF preferred over DNF in SAT solvers?**

6. **Given $f = (p \wedge \neg q) \vee (\neg p \wedge q)$, verify this is in DNF and convert to CNF.**

---

[← Previous: Tautologies and Contradictions](04-tautologies-and-contradictions.md) | [Back to README](../README.md) | [Next: Predicate Logic →](06-predicate-logic.md)
