# Chapter 1.3: Logical Equivalence

[← Previous: Truth Tables](02-truth-tables.md) | [Back to README](../README.md) | [Next: Tautologies and Contradictions →](04-tautologies-and-contradictions.md)

---

## 📋 Chapter Overview

Two compound propositions are **logically equivalent** if they always have the same truth value regardless of the truth values of their variables. Logical equivalences are the "algebraic rules" of logic — they let us simplify expressions, prove theorems, and optimize circuits.

---

## 1. Definition

Two propositions $P$ and $Q$ are **logically equivalent** (written $P \equiv Q$) if they have identical truth values for every possible assignment of truth values to their variables.

Equivalently, $P \equiv Q$ iff $P \leftrightarrow Q$ is a **tautology**.

---

## 2. Fundamental Logical Equivalences (Laws of Logic)

### 2.1 Identity Laws

$$p \wedge T \equiv p$$
$$p \vee F \equiv p$$

### 2.2 Domination Laws

$$p \vee T \equiv T$$
$$p \wedge F \equiv F$$

### 2.3 Idempotent Laws

$$p \vee p \equiv p$$
$$p \wedge p \equiv p$$

### 2.4 Double Negation Law

$$\neg(\neg p) \equiv p$$

### 2.5 Commutative Laws

$$p \vee q \equiv q \vee p$$
$$p \wedge q \equiv q \wedge p$$

### 2.6 Associative Laws

$$(p \vee q) \vee r \equiv p \vee (q \vee r)$$
$$(p \wedge q) \wedge r \equiv p \wedge (q \wedge r)$$

### 2.7 Distributive Laws

$$p \vee (q \wedge r) \equiv (p \vee q) \wedge (p \vee r)$$
$$p \wedge (q \vee r) \equiv (p \wedge q) \vee (p \wedge r)$$

### 2.8 De Morgan's Laws

$$\neg(p \wedge q) \equiv \neg p \vee \neg q$$
$$\neg(p \vee q) \equiv \neg p \wedge \neg q$$

> **Memory Aid:** "Break the bar, change the sign"  
> The negation distributes by switching AND ↔ OR

### 2.9 Absorption Laws

$$p \vee (p \wedge q) \equiv p$$
$$p \wedge (p \vee q) \equiv p$$

### 2.10 Negation Laws

$$p \vee \neg p \equiv T \quad \text{(Law of Excluded Middle)}$$
$$p \wedge \neg p \equiv F \quad \text{(Law of Contradiction)}$$

---

## 3. Complete Reference Table

| # | Law Name | Equivalence |
|---|----------|-------------|
| 1 | Identity | $p \wedge T \equiv p$, $p \vee F \equiv p$ |
| 2 | Domination | $p \vee T \equiv T$, $p \wedge F \equiv F$ |
| 3 | Idempotent | $p \vee p \equiv p$, $p \wedge p \equiv p$ |
| 4 | Double Negation | $\neg\neg p \equiv p$ |
| 5 | Commutative | $p \vee q \equiv q \vee p$, $p \wedge q \equiv q \wedge p$ |
| 6 | Associative | $(p \vee q) \vee r \equiv p \vee (q \vee r)$ |
| 7 | Distributive | $p \wedge (q \vee r) \equiv (p \wedge q) \vee (p \wedge r)$ |
| 8 | De Morgan's | $\neg(p \wedge q) \equiv \neg p \vee \neg q$ |
| 9 | Absorption | $p \vee (p \wedge q) \equiv p$ |
| 10 | Negation | $p \vee \neg p \equiv T$, $p \wedge \neg p \equiv F$ |

---

## 4. Equivalences Involving Implication

These are crucial for proof techniques:

$$p \rightarrow q \equiv \neg p \vee q$$

$$p \rightarrow q \equiv \neg q \rightarrow \neg p \quad \text{(Contrapositive)}$$

$$p \leftrightarrow q \equiv (p \rightarrow q) \wedge (q \rightarrow p)$$

$$\neg(p \rightarrow q) \equiv p \wedge \neg q$$

---

## 5. Proving Equivalences

### Method 1: Truth Table

**Example:** Prove $p \rightarrow q \equiv \neg p \vee q$

| $p$ | $q$ | $p \rightarrow q$ | $\neg p$ | $\neg p \vee q$ |
|:---:|:---:|:---:|:---:|:---:|
| T | T | T | F | T |
| T | F | F | F | F |
| F | T | T | T | T |
| F | F | T | T | T |

Columns match ✅

### Method 2: Algebraic Manipulation (Chain of Equivalences)

**Example:** Prove $\neg(p \rightarrow q) \equiv p \wedge \neg q$

$$\neg(p \rightarrow q)$$
$$\equiv \neg(\neg p \vee q) \quad \text{(implication equivalence)}$$
$$\equiv \neg(\neg p) \wedge \neg q \quad \text{(De Morgan's Law)}$$
$$\equiv p \wedge \neg q \quad \text{(Double Negation)}$$

---

## 6. Detailed Worked Examples

### Example 1: Simplify $\neg(p \vee (\neg p \wedge q))$

**Step-by-step algebraic simplification:**

$$\neg(p \vee (\neg p \wedge q))$$
$$\equiv \neg p \wedge \neg(\neg p \wedge q) \quad \text{(De Morgan)}$$
$$\equiv \neg p \wedge (\neg\neg p \vee \neg q) \quad \text{(De Morgan)}$$
$$\equiv \neg p \wedge (p \vee \neg q) \quad \text{(Double Negation)}$$
$$\equiv (\neg p \wedge p) \vee (\neg p \wedge \neg q) \quad \text{(Distributive)}$$
$$\equiv F \vee (\neg p \wedge \neg q) \quad \text{(Negation Law)}$$
$$\equiv \neg p \wedge \neg q \quad \text{(Identity Law)}$$

**Verification by truth table:**

| $p$ | $q$ | $\neg p \wedge q$ | $p \vee (\neg p \wedge q)$ | $\neg(p \vee (\neg p \wedge q))$ | $\neg p \wedge \neg q$ |
|:---:|:---:|:---:|:---:|:---:|:---:|
| T | T | F | T | F | F |
| T | F | F | T | F | F |
| F | T | T | T | F | F |
| F | F | F | F | **T** | **T** |

Columns match ✅

### Example 2: Show $(p \wedge q) \rightarrow p$ is a tautology

**Method:** Algebraic

$$(p \wedge q) \rightarrow p$$
$$\equiv \neg(p \wedge q) \vee p \quad \text{(implication equivalence)}$$
$$\equiv (\neg p \vee \neg q) \vee p \quad \text{(De Morgan)}$$
$$\equiv (\neg p \vee p) \vee \neg q \quad \text{(Associative, Commutative)}$$
$$\equiv T \vee \neg q \quad \text{(Negation Law)}$$
$$\equiv T \quad \text{(Domination Law)}$$

Therefore $(p \wedge q) \rightarrow p$ is a **tautology** ✅

### Example 3: Simplify $(p \rightarrow q) \wedge (p \rightarrow \neg q)$

$$(p \rightarrow q) \wedge (p \rightarrow \neg q)$$
$$\equiv (\neg p \vee q) \wedge (\neg p \vee \neg q) \quad \text{(implication equivalence)}$$
$$\equiv \neg p \vee (q \wedge \neg q) \quad \text{(Distributive — factoring } \neg p \text{)}$$
$$\equiv \neg p \vee F \quad \text{(Negation Law)}$$
$$\equiv \neg p \quad \text{(Identity Law)}$$

**Interpretation:** If $p$ implies both $q$ and $\neg q$, then $p$ must be false!

---

## 7. Circuit Simplification Using Equivalences

Logical equivalences directly translate to **circuit optimization**:

**Original Expression:** $\neg(p \wedge q) \wedge (p \vee q)$

**Original Circuit (4 gates):**
```
               ┌─────┐
  p ───┬───────┤     │
       │       │ AND ├──[NOT]──┐
  q ───┬───────┤     │        │    ┌─────┐
       │       └─────┘        ├────┤     │
       │                      │    │ AND ├─── Output
       │       ┌─────┐        │    │     │
       ├───────┤     │    ┌───┘    └─────┘
       │       │ OR  ├────┘
       └───────┤     │
               └─────┘
```

**Simplify algebraically:**
$$\neg(p \wedge q) \wedge (p \vee q)$$
$$\equiv (\neg p \vee \neg q) \wedge (p \vee q)$$

This is actually $p \oplus q$ (XOR). Let's verify:

| $p$ | $q$ | Original | $p \oplus q$ |
|:---:|:---:|:---:|:---:|
| T | T | F | F |
| T | F | T | T |
| F | T | T | T |
| F | F | F | F |

**Simplified Circuit (1 gate):**
```
       ┌─────┐
  p ───┤     │
       │ XOR ├──── Output
  q ───┤     │
       └─────┘
```

**Savings:** From 4 gates to 1 gate!

---

## 8. Duality Principle

Every logical equivalence has a **dual** obtained by swapping:
- $\wedge \leftrightarrow \vee$
- $T \leftrightarrow F$

If an equivalence is valid, its dual is also valid.

| Original | Dual |
|----------|------|
| $p \wedge T \equiv p$ | $p \vee F \equiv p$ |
| $p \vee T \equiv T$ | $p \wedge F \equiv F$ |
| $\neg(p \wedge q) \equiv \neg p \vee \neg q$ | $\neg(p \vee q) \equiv \neg p \wedge \neg q$ |

---

## 📝 Summary Table

| Topic | Key Formula / Concept |
|-------|----------------------|
| Logical Equivalence | $P \equiv Q$ iff $P \leftrightarrow Q$ is a tautology |
| De Morgan's Laws | $\neg(p \wedge q) \equiv \neg p \vee \neg q$ |
| Implication Equivalence | $p \rightarrow q \equiv \neg p \vee q$ |
| Distributive Law | $p \wedge (q \vee r) \equiv (p \wedge q) \vee (p \wedge r)$ |
| Absorption | $p \vee (p \wedge q) \equiv p$ |
| Contrapositive | $p \rightarrow q \equiv \neg q \rightarrow \neg p$ |
| Proving Methods | Truth table or algebraic manipulation |
| Duality | Swap $\wedge/\vee$ and $T/F$ |

---

## ❓ Quick Revision Questions

1. **State De Morgan's Laws and prove one of them using a truth table.**

2. **Simplify: $\neg(\neg p \wedge q) \vee q$**

3. **Are $p \rightarrow q$ and $q \rightarrow p$ logically equivalent? Justify.**

4. **Using logical equivalences, show that $(p \rightarrow q) \wedge (q \rightarrow r) \rightarrow (p \rightarrow r)$ is a tautology (Hypothetical Syllogism).**

5. **What is the dual of the absorption law $p \wedge (p \vee q) \equiv p$?**

6. **Simplify the circuit expression: $(\neg p \wedge q) \vee (p \wedge q)$ using algebraic laws.**

---

[← Previous: Truth Tables](02-truth-tables.md) | [Back to README](../README.md) | [Next: Tautologies and Contradictions →](04-tautologies-and-contradictions.md)
