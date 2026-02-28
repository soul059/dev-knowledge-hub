# Chapter 1.2: Truth Tables

[← Previous: Propositions and Logical Operators](01-propositions-and-logical-operators.md) | [Back to README](../README.md) | [Next: Logical Equivalence →](03-logical-equivalence.md)

---

## 📋 Chapter Overview

Truth tables are a systematic method for evaluating the truth value of compound propositions for every possible combination of truth values of their component propositions. They are fundamental tools for verifying logical equivalences, identifying tautologies, and designing digital logic circuits.

---

## 1. What is a Truth Table?

A **truth table** lists all possible combinations of truth values of the variables in a proposition, along with the resulting truth value of the compound proposition.

### Number of Rows

For a proposition with $n$ variables, the truth table has $2^n$ rows.

| Variables | Rows |
|:---------:|:----:|
| 1 | 2 |
| 2 | 4 |
| 3 | 8 |
| 4 | 16 |
| $n$ | $2^n$ |

### Systematic Listing of Truth Values

For $n$ variables, fill columns using the **binary counting pattern**:

**For 3 variables ($p$, $q$, $r$):**

| Row | $p$ | $q$ | $r$ |
|:---:|:---:|:---:|:---:|
| 1 | T | T | T |
| 2 | T | T | F |
| 3 | T | F | T |
| 4 | T | F | F |
| 5 | F | T | T |
| 6 | F | T | F |
| 7 | F | F | T |
| 8 | F | F | F |

**Pattern:** First variable alternates every $2^{n-1}$ rows, second every $2^{n-2}$, ..., last variable alternates every row.

---

## 2. Building Truth Tables Step by Step

### Method

1. **List all variables** and determine the number of rows ($2^n$)
2. **Fill in variable columns** using the binary pattern
3. **Add columns for sub-expressions** (inside-out, following precedence)
4. **Compute the final column**

---

### Example 1: $p \rightarrow (q \wedge r)$

**Step 1:** Identify sub-expressions: $q \wedge r$, then $p \rightarrow (q \wedge r)$

| $p$ | $q$ | $r$ | $q \wedge r$ | $p \rightarrow (q \wedge r)$ |
|:---:|:---:|:---:|:---:|:---:|
| T | T | T | T | **T** |
| T | T | F | F | **F** |
| T | F | T | F | **F** |
| T | F | F | F | **F** |
| F | T | T | T | **T** |
| F | T | F | F | **T** |
| F | F | T | F | **T** |
| F | F | F | F | **T** |

**Observation:** The proposition is false only when $p$ is true but $q \wedge r$ is false.

---

### Example 2: $(p \vee q) \rightarrow (p \wedge q)$

| $p$ | $q$ | $p \vee q$ | $p \wedge q$ | $(p \vee q) \rightarrow (p \wedge q)$ |
|:---:|:---:|:---:|:---:|:---:|
| T | T | T | T | **T** |
| T | F | T | F | **F** |
| F | T | T | F | **F** |
| F | F | F | F | **T** |

**Observation:** This is true when $p$ and $q$ have the same value — it's equivalent to $p \leftrightarrow q$.

---

### Example 3: $\neg(p \wedge q) \leftrightarrow (\neg p \vee \neg q)$

This demonstrates **De Morgan's Law:**

| $p$ | $q$ | $p \wedge q$ | $\neg(p \wedge q)$ | $\neg p$ | $\neg q$ | $\neg p \vee \neg q$ | $\neg(p \wedge q) \leftrightarrow (\neg p \vee \neg q)$ |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| T | T | T | F | F | F | F | **T** |
| T | F | F | T | F | T | T | **T** |
| F | T | F | T | T | F | T | **T** |
| F | F | F | T | T | T | T | **T** |

**Result:** All entries are T — this is a **tautology**, confirming De Morgan's Law.

---

## 3. Truth Tables for All Binary Operators

For two variables $p$ and $q$, there are $2^{2^2} = 16$ possible binary operators. Here are the most important ones:

| $p$ | $q$ | $p \wedge q$ | $p \vee q$ | $p \oplus q$ | $p \rightarrow q$ | $p \leftrightarrow q$ | $p \uparrow q$ (NAND) | $p \downarrow q$ (NOR) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| T | T | T | T | F | T | T | F | F |
| T | F | F | T | T | F | F | T | F |
| F | T | F | T | T | T | F | T | F |
| F | F | F | F | F | T | T | T | T |

### NAND and NOR — Universal Gates

**NAND** ($p \uparrow q$): negation of AND — $\neg(p \wedge q)$  
**NOR** ($p \downarrow q$): negation of OR — $\neg(p \vee q)$

```
  NAND Gate:                    NOR Gate:
       ┌─────┐                      ┌─────┐
  p ───┤     │                 p ───┤     │
       │NAND ├─── p ↑ q            │ NOR ├─── p ↓ q
  q ───┤     │                 q ───┤     │
       └─────┘                      └─────┘
```

> **Key Fact:** NAND and NOR are each **functionally complete** — any logical function can be built using only NAND gates (or only NOR gates).

---

## 4. Using Truth Tables to Verify Equivalence

Two propositions are **logically equivalent** if they have identical truth table columns.

### Example: Verify $p \rightarrow q \equiv \neg p \vee q$

| $p$ | $q$ | $p \rightarrow q$ | $\neg p$ | $\neg p \vee q$ |
|:---:|:---:|:---:|:---:|:---:|
| T | T | T | F | T |
| T | F | F | F | F |
| F | T | T | T | T |
| F | F | T | T | T |

**Result:** Columns 3 and 5 are identical ✅ → $p \rightarrow q \equiv \neg p \vee q$

---

## 5. Timing Diagram Representation

Truth tables can be visualized as **timing diagrams** showing how output changes with inputs:

```
  Time →   t₁   t₂   t₃   t₄
           ┌───┐     ┌───┐
  p:       │   │     │   │
       ────┘   └─────┘   └────

           ┌───┐─────┐
  q:       │   │     │
       ────┘   └─────┘────────

           ┌───┐
  p∧q:     │   │
       ────┘   └──────────────

           ┌───┐─────┐───┐
  p∨q:     │   │     │   │
       ────┘   └─────┘   └────
```

In this timing diagram:
- High (─┐ or ┌─) = True
- Low (─┘ or └─) = False
- At $t_1$: $p$=T, $q$=T → $p \wedge q$=T, $p \vee q$=T
- At $t_2$: $p$=T, $q$=F → $p \wedge q$=F, $p \vee q$=T
- At $t_3$: $p$=F, $q$=T → $p \wedge q$=F, $p \vee q$=T  (corrected from diagram)
- At $t_4$: $p$=F, $q$=F → $p \wedge q$=F, $p \vee q$=F

---

## 6. Shortcut: Determining Truth Value Without Full Table

### For Implication $p \rightarrow q$:
- Only check when $p$ is T. If $q$ is also T → whole thing is T.
- The implication is F **only** when we find T → F.

### For Tautology Check:
- Try to find a row where the expression is F.
- If no such row exists → Tautology.

### For Contradiction Check:
- Try to find a row where the expression is T.
- If no such row exists → Contradiction.

---

## 7. Design Example: Alarm System Logic

**Problem:** Design a truth table for an alarm system where:
- $p$: Door is open
- $q$: Motion is detected
- $r$: System is armed

The alarm rings when: **system is armed AND (door is open OR motion is detected)**

$$\text{Alarm} = r \wedge (p \vee q)$$

| $p$ | $q$ | $r$ | $p \vee q$ | $r \wedge (p \vee q)$ | Alarm |
|:---:|:---:|:---:|:---:|:---:|:---:|
| T | T | T | T | T | 🔔 ON |
| T | T | F | T | F | OFF |
| T | F | T | T | T | 🔔 ON |
| T | F | F | T | F | OFF |
| F | T | T | T | T | 🔔 ON |
| F | T | F | T | F | OFF |
| F | F | T | F | F | OFF |
| F | F | F | F | F | OFF |

**Circuit Implementation:**
```
  p ───┐
       ├──[OR]──┐
  q ───┘        ├──[AND]──── Alarm
  r ────────────┘

  Detailed:
       ┌────┐
  p ───┤    │    ┌─────┐
       │ OR ├────┤     │
  q ───┤    │    │ AND ├──── 🔔 Alarm
       └────┘    │     │
  r ─────────────┤     │
                 └─────┘
```

---

## 8. Real-World Applications

| Application | Use of Truth Tables |
|-------------|---------------------|
| **Digital Circuit Design** | Determine output for all input combinations |
| **Software Testing** | Decision table testing covers all condition combinations |
| **Database Design** | Verify integrity constraints |
| **Network Security** | Firewall rule verification |
| **Compiler Design** | Operator evaluation and short-circuit logic |

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Truth Table | Lists all $2^n$ combinations for $n$ variables |
| Binary Pattern | Fill columns by alternating T/F in powers of 2 |
| Sub-expressions | Evaluate inside-out following operator precedence |
| Equivalence Check | Two expressions are equivalent if their columns match |
| Tautology | All rows evaluate to T |
| Contradiction | All rows evaluate to F |
| Contingency | Some rows T, some F |

---

## ❓ Quick Revision Questions

1. **How many rows does a truth table with 5 variables have?**

2. **Construct the truth table for $(p \wedge q) \rightarrow p$. Is it a tautology?**

3. **What is the difference between NAND and NOR operations?**

4. **Using a truth table, verify that $p \oplus q \equiv (p \vee q) \wedge \neg(p \wedge q)$.**

5. **If a compound proposition has 4 variables and evaluates to T in exactly 10 rows, what fraction of inputs make it true?**

6. **Design a truth table for a voting system where a motion passes if at least 2 out of 3 members vote "Yes."**

---

[← Previous: Propositions and Logical Operators](01-propositions-and-logical-operators.md) | [Back to README](../README.md) | [Next: Logical Equivalence →](03-logical-equivalence.md)
