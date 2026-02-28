# Chapter 1: Field Axioms

[← Back to README](../README.md) | [Next: Order Axioms →](02-order-axioms.md)

---

## Overview

The real numbers ℝ are characterized by three groups of axioms: **field axioms**, **order axioms**, and the **completeness axiom**. In this chapter we study the field axioms — the algebraic foundation that tells us how addition and multiplication behave.

A **field** is a set $F$ equipped with two binary operations, addition $(+)$ and multiplication $(\cdot)$, satisfying the axioms below.

---

## 1.1 The Field Axioms

Let $F$ be a set with operations $+ : F \times F \to F$ and $\cdot : F \times F \to F$.

### Addition Axioms (A1–A5)

| Axiom | Name | Statement |
|-------|------|-----------|
| A1 | Closure | $\forall\, a, b \in F:\; a + b \in F$ |
| A2 | Commutativity | $\forall\, a, b \in F:\; a + b = b + a$ |
| A3 | Associativity | $\forall\, a, b, c \in F:\; (a + b) + c = a + (b + c)$ |
| A4 | Additive Identity | $\exists\, 0 \in F\;\text{such that}\; \forall\, a \in F:\; a + 0 = a$ |
| A5 | Additive Inverse | $\forall\, a \in F,\; \exists\, (-a) \in F:\; a + (-a) = 0$ |

### Multiplication Axioms (M1–M5)

| Axiom | Name | Statement |
|-------|------|-----------|
| M1 | Closure | $\forall\, a, b \in F:\; a \cdot b \in F$ |
| M2 | Commutativity | $\forall\, a, b \in F:\; a \cdot b = b \cdot a$ |
| M3 | Associativity | $\forall\, a, b, c \in F:\; (a \cdot b) \cdot c = a \cdot (b \cdot c)$ |
| M4 | Multiplicative Identity | $\exists\, 1 \in F,\; 1 \neq 0,\;\text{such that}\; \forall\, a \in F:\; a \cdot 1 = a$ |
| M5 | Multiplicative Inverse | $\forall\, a \in F,\; a \neq 0,\; \exists\, a^{-1} \in F:\; a \cdot a^{-1} = 1$ |

### Distributive Law (D)

| Axiom | Name | Statement |
|-------|------|-----------|
| D | Distributivity | $\forall\, a, b, c \in F:\; a \cdot (b + c) = a \cdot b + a \cdot c$ |

---

## 1.2 Logical Structure of the Axioms

```
                    FIELD AXIOMS
                    ════════════
                         │
          ┌──────────────┼──────────────┐
          │              │              │
    ┌─────▼─────┐  ┌────▼────┐  ┌──────▼──────┐
    │ ADDITION  │  │ DISTRI- │  │ MULTIPLI-   │
    │  AXIOMS   │  │ BUTIVE  │  │ CATION      │
    │ (A1–A5)   │  │  LAW    │  │ AXIOMS      │
    │           │  │  (D)    │  │ (M1–M5)     │
    └─────┬─────┘  └────┬────┘  └──────┬──────┘
          │              │              │
          │    Connects addition        │
          │    and multiplication       │
          │              │              │
          ▼              ▼              ▼
    (ℝ, +) is      Links the      (ℝ \ {0}, ·)
    an abelian      two group      is an abelian
    group           structures     group
```

---

## 1.3 Consequences of the Field Axioms

These are **theorems**, not axioms — they are proved from the axioms above.

### Theorem 1.1: Uniqueness of Additive Identity
> The element $0$ in axiom A4 is unique.

**Proof.** Suppose $0$ and $0'$ are both additive identities. Then:
$$0' = 0' + 0 = 0 + 0' = 0$$
The first equality uses A4 (with identity $0$), the second uses A2, and the result follows. $\blacksquare$

### Theorem 1.2: Uniqueness of Additive Inverse
> For each $a \in F$, the element $-a$ is unique.

**Proof.** Suppose $b$ and $c$ are both additive inverses of $a$. Then:
$$b = b + 0 = b + (a + c) = (b + a) + c = 0 + c = c$$
using A4, assumption on $c$, A3, assumption on $b$, and A4. $\blacksquare$

### Theorem 1.3: $0 \cdot a = 0$ for all $a \in F$

**Proof.**
$$0 \cdot a = (0 + 0) \cdot a = 0 \cdot a + 0 \cdot a$$
Adding $-(0 \cdot a)$ to both sides:
$$0 = 0 \cdot a$$
$\blacksquare$

### Theorem 1.4: $(-1) \cdot a = -a$ for all $a \in F$

**Proof.**
$$a + (-1) \cdot a = 1 \cdot a + (-1) \cdot a = (1 + (-1)) \cdot a = 0 \cdot a = 0$$
By uniqueness of additive inverse, $(-1) \cdot a = -a$. $\blacksquare$

### Theorem 1.5: $(-a)(-b) = ab$ for all $a, b \in F$

**Proof.**
$$(-a)(-b) = (-1)(a) \cdot (-1)(b) = (-1)(-1) \cdot ab$$
Now $(-1)(-1) + (-1) = (-1)(-1) + (-1)(1) = (-1)(-1 + 1) = (-1)(0) = 0$, so $(-1)(-1) = 1$.
Thus $(-a)(-b) = 1 \cdot ab = ab$. $\blacksquare$

### Theorem 1.6: Cancellation Law
> If $a + b = a + c$, then $b = c$.

**Proof.** Add $-a$ to both sides and use associativity. $\blacksquare$

---

## 1.4 Examples and Non-Examples

### Examples of Fields

| Set | Addition | Multiplication | Field? |
|-----|----------|---------------|--------|
| $\mathbb{R}$ | Usual | Usual | ✅ Yes |
| $\mathbb{Q}$ | Usual | Usual | ✅ Yes |
| $\mathbb{C}$ | Component-wise | $(a+bi)(c+di) = (ac-bd)+(ad+bc)i$ | ✅ Yes |
| $\mathbb{Z}_p$ ($p$ prime) | mod $p$ | mod $p$ | ✅ Yes |

### Non-Examples

| Set | Why Not a Field? |
|-----|-----------------|
| $\mathbb{Z}$ | No multiplicative inverses (e.g., $2^{-1} \notin \mathbb{Z}$) |
| $\mathbb{N}$ | No additive inverses, no multiplicative inverses |
| $\mathbb{Z}_6$ | $2 \cdot 3 = 0$ (zero divisors), so no multiplicative inverses for $2$ or $3$ |

---

## 1.5 Why the Field Axioms Matter for Analysis

The field axioms alone give us algebra — we can add, subtract, multiply, and divide (by nonzero elements). But they do **not** give us:

```
  FIELD AXIOMS alone provide:          They do NOT provide:
  ┌────────────────────────┐          ┌─────────────────────────┐
  │ • Arithmetic rules     │          │ • Notion of "less than" │
  │ • Cancellation laws    │          │ • Absolute value        │
  │ • Unique identities    │          │ • Limits                │
  │ • Unique inverses      │          │ • Supremum              │
  │ • Distributivity       │          │ • Completeness          │
  └────────────────────────┘          └─────────────────────────┘
          ▲                                     ▲
          │                                     │
     This chapter                    Next chapters (Order + 
                                     Completeness axioms)
```

To do analysis, we need **order** (Chapter 2) and **completeness** (Chapter 3).

---

## 1.6 Real-World Application: Error-Correcting Codes

Finite fields $\mathbb{Z}_p$ are used in:
- **Reed-Solomon codes** (used in CDs, DVDs, QR codes)
- **Cryptography** (RSA, elliptic curves operate over fields)
- **Digital signal processing**

The field axioms guarantee that encoding/decoding operations are reversible — every nonzero element has a multiplicative inverse, enabling error correction.

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Field | A set with $+$ and $\cdot$ satisfying 11 axioms |
| Addition axioms | $(F, +)$ is an abelian group |
| Multiplication axioms | $(F \setminus \{0\}, \cdot)$ is an abelian group |
| Distributive law | Links addition and multiplication |
| $0 \cdot a = 0$ | Theorem, not an axiom |
| $(-1)a = -a$ | Follows from distributivity |
| Cancellation | $a + b = a + c \Rightarrow b = c$ |
| $\mathbb{R}, \mathbb{Q}, \mathbb{C}$ | Examples of fields |
| $\mathbb{Z}, \mathbb{N}$ | Not fields |

---

## Quick Revision Questions

1. **State the distributive law** and explain why it is essential for connecting addition and multiplication.

2. **Prove that the additive identity is unique.** Where does commutativity enter the proof?

3. **Is $\mathbb{Z}$ a field?** If not, which axiom(s) does it fail?

4. **Prove that $0 \cdot a = 0$** for all $a$ in a field, using only the field axioms.

5. **True or False:** Every field has at least two distinct elements. Justify your answer.

6. **Why is the requirement $1 \neq 0$ included** in axiom M4? What would go wrong without it?

---

[← Back to README](../README.md) | [Next: Order Axioms →](02-order-axioms.md)
