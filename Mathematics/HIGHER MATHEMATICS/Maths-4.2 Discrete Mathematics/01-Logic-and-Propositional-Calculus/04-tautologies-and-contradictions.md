# Chapter 1.4: Tautologies and Contradictions

[← Previous: Logical Equivalence](03-logical-equivalence.md) | [Back to README](../README.md) | [Next: Normal Forms →](05-normal-forms.md)

---

## 📋 Chapter Overview

This chapter classifies compound propositions based on their truth values: those that are **always true** (tautologies), **always false** (contradictions), and **sometimes true** (contingencies). These concepts are central to validating logical arguments and understanding logical structure.

---

## 1. Definitions

### Tautology
A compound proposition that is **true for every possible assignment** of truth values to its variables.

$$\text{Notation: } \top \text{ or } T$$

**Example:** $p \vee \neg p$ is always true.

| $p$ | $\neg p$ | $p \vee \neg p$ |
|:---:|:---:|:---:|
| T | F | **T** |
| F | T | **T** |

### Contradiction
A compound proposition that is **false for every possible assignment** of truth values to its variables.

$$\text{Notation: } \bot \text{ or } F$$

**Example:** $p \wedge \neg p$ is always false.

| $p$ | $\neg p$ | $p \wedge \neg p$ |
|:---:|:---:|:---:|
| T | F | **F** |
| F | T | **F** |

### Contingency
A compound proposition that is **neither a tautology nor a contradiction** — it is true for some assignments and false for others.

**Example:** $p \rightarrow q$

| $p$ | $q$ | $p \rightarrow q$ |
|:---:|:---:|:---:|
| T | T | T |
| T | F | **F** |
| F | T | T |
| F | F | T |

---

## 2. Visual Classification

```
  ┌───────────────────────────────────────────────┐
  │         All Compound Propositions              │
  │                                                │
  │   ┌──────────┐  ┌──────────────┐  ┌─────────┐ │
  │   │Tautology │  │ Contingency  │  │Contra-  │ │
  │   │          │  │              │  │diction  │ │
  │   │ Always T │  │ Sometimes T  │  │Always F │ │
  │   │          │  │ Sometimes F  │  │         │ │
  │   └──────────┘  └──────────────┘  └─────────┘ │
  └───────────────────────────────────────────────┘
```

---

## 3. Important Tautologies

### 3.1 Law of Excluded Middle
$$p \vee \neg p$$

Every proposition is either true or false — there is no middle ground.

### 3.2 Modus Ponens
$$(p \wedge (p \rightarrow q)) \rightarrow q$$

"If $p$ is true and $p$ implies $q$, then $q$ is true."

| $p$ | $q$ | $p \rightarrow q$ | $p \wedge (p \rightarrow q)$ | Full expression |
|:---:|:---:|:---:|:---:|:---:|
| T | T | T | T | **T** |
| T | F | F | F | **T** |
| F | T | T | F | **T** |
| F | F | T | F | **T** |

### 3.3 Modus Tollens
$$(\neg q \wedge (p \rightarrow q)) \rightarrow \neg p$$

"If $q$ is false and $p$ implies $q$, then $p$ must be false."

### 3.4 Hypothetical Syllogism
$$((p \rightarrow q) \wedge (q \rightarrow r)) \rightarrow (p \rightarrow r)$$

"If $p$ implies $q$ and $q$ implies $r$, then $p$ implies $r$."

### 3.5 Disjunctive Syllogism
$$((p \vee q) \wedge \neg p) \rightarrow q$$

"If $p$ or $q$ is true and $p$ is false, then $q$ must be true."

### 3.6 Resolution
$$((p \vee q) \wedge (\neg p \vee r)) \rightarrow (q \vee r)$$

This is the foundation of the **resolution principle** used in automated theorem proving.

---

## 4. Properties of Tautologies and Contradictions

| Property | Formula |
|----------|---------|
| Negation of tautology is contradiction | $\neg T \equiv F$ |
| Negation of contradiction is tautology | $\neg F \equiv T$ |
| AND with tautology = identity | $p \wedge T \equiv p$ |
| OR with tautology = tautology | $p \vee T \equiv T$ |
| AND with contradiction = contradiction | $p \wedge F \equiv F$ |
| OR with contradiction = identity | $p \vee F \equiv p$ |
| Tautology implies anything | $T \rightarrow p \equiv p$ |
| Anything implies tautology | $p \rightarrow T \equiv T$ |
| Contradiction implies anything | $F \rightarrow p \equiv T$ |

---

## 5. Testing for Tautology, Contradiction, or Contingency

### Method 1: Complete Truth Table
Build the full truth table and check the final column:
- All T → Tautology
- All F → Contradiction
- Mix → Contingency

### Method 2: Short-Circuit Evaluation

**To check if NOT a tautology:**
- Try to find an assignment that makes the expression **false**
- If found → Not a tautology

**To check if NOT a contradiction:**
- Try to find an assignment that makes the expression **true**
- If found → Not a contradiction

### Method 3: Algebraic Simplification
Simplify using logical equivalences:
- If simplifies to $T$ → Tautology
- If simplifies to $F$ → Contradiction

---

## 6. Worked Examples

### Example 1: Classify $(p \rightarrow q) \vee (q \rightarrow p)$

| $p$ | $q$ | $p \rightarrow q$ | $q \rightarrow p$ | $(p \rightarrow q) \vee (q \rightarrow p)$ |
|:---:|:---:|:---:|:---:|:---:|
| T | T | T | T | **T** |
| T | F | F | T | **T** |
| F | T | T | F | **T** |
| F | F | T | T | **T** |

**Result:** Tautology ✅

**Algebraic proof:**
$$(p \rightarrow q) \vee (q \rightarrow p)$$
$$\equiv (\neg p \vee q) \vee (\neg q \vee p)$$
$$\equiv (\neg p \vee p) \vee (q \vee \neg q) \quad \text{(rearranging)}$$
$$\equiv T \vee T \equiv T$$

### Example 2: Classify $(p \rightarrow q) \wedge (p \wedge \neg q)$

$$(p \rightarrow q) \wedge (p \wedge \neg q)$$
$$\equiv (\neg p \vee q) \wedge p \wedge \neg q$$
$$\equiv ((\neg p \wedge p) \vee (q \wedge p)) \wedge \neg q \quad \text{(Distributive)}$$
$$\equiv (F \vee (p \wedge q)) \wedge \neg q$$
$$\equiv (p \wedge q) \wedge \neg q$$
$$\equiv p \wedge (q \wedge \neg q)$$
$$\equiv p \wedge F \equiv F$$

**Result:** Contradiction ✅

### Example 3: Classify $p \rightarrow (q \rightarrow p)$

$p \rightarrow (q \rightarrow p)$
$\equiv p \rightarrow (\neg q \vee p)$
$\equiv \neg p \vee (\neg q \vee p)$
$\equiv (\neg p \vee p) \vee \neg q$
$\equiv T \vee \neg q \equiv T$

**Result:** Tautology ✅  
**Interpretation:** A true proposition is implied by anything.

---

## 7. Logical Implication vs. Material Implication

A proposition $P$ **logically implies** $Q$ (written $P \Rightarrow Q$) if $P \rightarrow Q$ is a tautology.

| Concept | Symbol | Meaning |
|---------|:------:|---------|
| Material implication | $\rightarrow$ | A connective (can be T or F) |
| Logical implication | $\Rightarrow$ | $P \rightarrow Q$ is always true (a meta-statement) |
| Logical equivalence | $\equiv$ | $P \leftrightarrow Q$ is always true |

**Example:**
- $p \wedge q \Rightarrow p$ because $(p \wedge q) \rightarrow p$ is a tautology
- But $p \not\Rightarrow p \wedge q$ because $p \rightarrow (p \wedge q)$ is NOT a tautology

---

## 8. Satisfiability

A compound proposition is **satisfiable** if there exists **at least one** assignment of truth values that makes it true.

- Tautology → Always satisfiable
- Contradiction → Not satisfiable (unsatisfiable)
- Contingency → Satisfiable

**SAT Problem:** Determining whether a given proposition is satisfiable is one of the most important problems in computer science (NP-complete).

```
  Classification Flowchart:
  
  Is the proposition true       YES ──→ Is it true for
  for some assignment? ──────┐          ALL assignments?
                             │              │
                          NO │         YES──┤──NO
                             │              │    │
                             ▼              ▼    ▼
                        Contradiction  Tautology  Contingency
                       (Unsatisfiable)            (Satisfiable)
```

---

## 9. Real-World Applications

| Application | Concept Used |
|-------------|--------------|
| **Formal Verification** | Checking if a system specification is a tautology (always holds) |
| **Automated Theorem Proving** | Proving formulas are tautologies using resolution |
| **Constraint Satisfaction** | Checking satisfiability of logical constraints |
| **Circuit Testing** | Verifying that a safety condition is always true |
| **Cryptography** | SAT solvers used in cryptanalysis |

---

## 📝 Summary Table

| Classification | Definition | Example |
|----------------|------------|---------|
| Tautology | True for all assignments | $p \vee \neg p$ |
| Contradiction | False for all assignments | $p \wedge \neg p$ |
| Contingency | True for some, false for others | $p \rightarrow q$ |
| Satisfiable | True for at least one assignment | Any non-contradiction |
| Valid argument | Premises → Conclusion is a tautology | Modus Ponens |

---

## ❓ Quick Revision Questions

1. **Classify each of the following: (a) $(p \wedge q) \rightarrow (p \vee q)$ (b) $p \wedge \neg p \wedge q$ (c) $p \oplus q$**

2. **Prove algebraically that $(p \rightarrow q) \vee (q \rightarrow r)$ is a tautology.**

3. **Is the proposition $(p \rightarrow q) \rightarrow (\neg q \rightarrow \neg p)$ a tautology? What logical equivalence does it represent?**

4. **What is the difference between $\rightarrow$ and $\Rightarrow$?**

5. **A proposition with 3 variables is satisfiable. What is the maximum number of rows where it could be false?**

6. **State Modus Ponens and Modus Tollens. Give a real-life example of each.**

---

[← Previous: Logical Equivalence](03-logical-equivalence.md) | [Back to README](../README.md) | [Next: Normal Forms →](05-normal-forms.md)
