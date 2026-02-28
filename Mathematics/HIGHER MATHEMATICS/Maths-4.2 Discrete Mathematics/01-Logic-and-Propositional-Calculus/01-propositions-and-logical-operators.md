# Chapter 1.1: Propositions and Logical Operators

[← Back to README](../README.md) | [Next: Truth Tables →](02-truth-tables.md)

---

## 📋 Chapter Overview

This chapter introduces the fundamental building blocks of mathematical logic: **propositions** and **logical operators** (connectives). Understanding these concepts is essential for constructing valid arguments, designing digital circuits, and programming.

---

## 1. What is a Proposition?

A **proposition** (or **statement**) is a declarative sentence that is either **true (T)** or **false (F)**, but **not both**.

### Examples of Propositions

| Statement | Proposition? | Truth Value |
|-----------|:---:|:---:|
| "2 + 3 = 5" | ✅ Yes | T |
| "Paris is the capital of Germany" | ✅ Yes | F |
| "7 is a prime number" | ✅ Yes | T |
| "x + 5 = 10" | ❌ No | — |
| "Close the door!" | ❌ No | — |
| "Is it raining?" | ❌ No | — |

### Why "x + 5 = 10" is NOT a Proposition

The sentence "x + 5 = 10" contains a **free variable** `x`. Its truth value depends on what value `x` takes:
- If x = 5 → True
- If x = 3 → False

Since it doesn't have a **definite** truth value, it is called an **open sentence** or **propositional function**, not a proposition.

### Notation

Propositions are typically denoted by lowercase letters: $p, q, r, s, \ldots$

$$p: \text{"The sun rises in the east"} \quad (T)$$
$$q: \text{"3 > 5"} \quad (F)$$

---

## 2. Logical Operators (Connectives)

Logical operators combine simple propositions into **compound propositions**. There are five primary logical operators:

### 2.1 Negation (NOT) — $\neg p$

The **negation** of a proposition $p$ reverses its truth value.

| $p$ | $\neg p$ |
|:---:|:---:|
| T | F |
| F | T |

**Example:**
- $p$: "It is raining" → $\neg p$: "It is **not** raining"

**Circuit Analogy — NOT Gate:**
```
         ┌─────┐
  p ─────┤ NOT ├──── ¬p
         └─────┘

  Input   Output
    1   →   0
    0   →   1
```

---

### 2.2 Conjunction (AND) — $p \wedge q$

The **conjunction** $p \wedge q$ is true **only when both** $p$ and $q$ are true.

| $p$ | $q$ | $p \wedge q$ |
|:---:|:---:|:---:|
| T | T | **T** |
| T | F | F |
| F | T | F |
| F | F | F |

**Example:**
- $p$: "It is sunny" and $q$: "I have an umbrella"
- $p \wedge q$: "It is sunny **and** I have an umbrella"

**Circuit Analogy — AND Gate:**
```
  p ────┐
        ├───[AND]──── p ∧ q
  q ────┘

       ┌─────┐
  p ───┤     │
       │ AND ├──── p ∧ q
  q ───┤     │
       └─────┘
```

**Key Insight:** AND is like series circuit — both switches must be ON for current to flow.

```
  Battery ──[Switch p]──[Switch q]──💡 Bulb
  
  Both ON  → 💡 Lights up (T)
  Any OFF  → 💡 Stays off (F)
```

---

### 2.3 Disjunction (OR) — $p \vee q$

The **disjunction** $p \vee q$ is true **when at least one** of $p$ or $q$ is true.

| $p$ | $q$ | $p \vee q$ |
|:---:|:---:|:---:|
| T | T | **T** |
| T | F | **T** |
| F | T | **T** |
| F | F | F |

**Example:**
- $p$: "I will study math" or $q$: "I will study physics"
- $p \vee q$: "I will study math **or** physics (or both)"

> **Note:** This is **inclusive OR** — it includes the case where both are true.

**Circuit Analogy — OR Gate:**
```
       ┌─────┐
  p ───┤     │
       │ OR  ├──── p ∨ q
  q ───┤     │
       └─────┘
```

**Key Insight:** OR is like parallel circuit — either switch being ON lets current flow.

```
        ┌──[Switch p]──┐
  Bat ──┤              ├── 💡 Bulb
        └──[Switch q]──┘
  
  Any ON  → 💡 Lights up (T)
  Both OFF → 💡 Stays off (F)
```

---

### 2.4 Implication (IF...THEN) — $p \rightarrow q$

The **implication** $p \rightarrow q$ reads "if $p$ then $q$". It is false **only when** $p$ is true but $q$ is false.

| $p$ | $q$ | $p \rightarrow q$ |
|:---:|:---:|:---:|
| T | T | **T** |
| T | F | **F** |
| F | T | **T** |
| F | F | **T** |

**Terminology:**
- $p$ is called the **hypothesis** (antecedent, premise)
- $q$ is called the **conclusion** (consequent)

**Example:**
- "If it rains ($p$), then the ground is wet ($q$)"
- If it doesn't rain, the statement says nothing — so it's vacuously true

**Why is F → T considered True?**

Think of a promise: "If you pass the exam, I'll buy you a gift."
- You pass and get a gift → Promise kept (T → T = T)
- You pass but no gift → Promise broken (T → F = F)  
- You didn't pass but still got a gift → No violation (F → T = T)
- You didn't pass and no gift → No violation (F → F = T)

**Related Statements:**
| Statement | Form | Name |
|-----------|------|------|
| If $p$ then $q$ | $p \rightarrow q$ | Implication |
| If $q$ then $p$ | $q \rightarrow p$ | Converse |
| If $\neg p$ then $\neg q$ | $\neg p \rightarrow \neg q$ | Inverse |
| If $\neg q$ then $\neg p$ | $\neg q \rightarrow \neg p$ | Contrapositive |

> **Important:** An implication and its **contrapositive** are logically equivalent.  
> The converse and inverse are NOT equivalent to the original.

---

### 2.5 Biconditional (IF AND ONLY IF) — $p \leftrightarrow q$

The **biconditional** $p \leftrightarrow q$ is true when both $p$ and $q$ have the **same truth value**.

| $p$ | $q$ | $p \leftrightarrow q$ |
|:---:|:---:|:---:|
| T | T | **T** |
| T | F | F |
| F | T | F |
| F | F | **T** |

**Example:**
- "You pass the course **if and only if** your score ≥ 50"
- This means: pass ↔ score ≥ 50 (both conditions match)

**Note:** $p \leftrightarrow q \equiv (p \rightarrow q) \wedge (q \rightarrow p)$

---

## 3. Exclusive OR (XOR) — $p \oplus q$

While not always listed as a primary connective, **XOR** is important:

| $p$ | $q$ | $p \oplus q$ |
|:---:|:---:|:---:|
| T | T | F |
| T | F | **T** |
| F | T | **T** |
| F | F | F |

- XOR is true when $p$ and $q$ have **different** truth values
- $p \oplus q \equiv (p \vee q) \wedge \neg(p \wedge q)$

**Circuit Analogy — XOR Gate:**
```
       ┌─────┐
  p ───┤     │
       │ XOR ├──── p ⊕ q
  q ───┤     │
       └─────┘
```

---

## 4. Operator Precedence

When multiple operators appear in an expression, use this precedence (highest to lowest):

| Priority | Operator | Name |
|:---:|:---:|------|
| 1 (highest) | $\neg$ | Negation |
| 2 | $\wedge$ | Conjunction |
| 3 | $\vee$ | Disjunction |
| 4 | $\rightarrow$ | Implication |
| 5 (lowest) | $\leftrightarrow$ | Biconditional |

**Example:**
$$\neg p \wedge q \vee r \equiv ((\neg p) \wedge q) \vee r$$

> **Tip:** When in doubt, use parentheses to make the order explicit.

---

## 5. Compound Propositions — Worked Example

**Problem:** Construct the truth table for $\neg p \wedge (q \vee r)$

**Step-by-step:**

| $p$ | $q$ | $r$ | $\neg p$ | $q \vee r$ | $\neg p \wedge (q \vee r)$ |
|:---:|:---:|:---:|:---:|:---:|:---:|
| T | T | T | F | T | F |
| T | T | F | F | T | F |
| T | F | T | F | T | F |
| T | F | F | F | F | F |
| F | T | T | T | T | **T** |
| F | T | F | T | T | **T** |
| F | F | T | T | T | **T** |
| F | F | F | T | F | F |

---

## 6. Real-World Applications

| Application | How Logic is Used |
|-------------|-------------------|
| **Digital Circuits** | Logic gates (AND, OR, NOT) implement Boolean operations in hardware |
| **Programming** | `if-else` statements, `&&`, `||`, `!` operators |
| **Database Queries** | SQL `WHERE` clauses use AND, OR, NOT |
| **Artificial Intelligence** | Knowledge representation and inference engines |
| **Mathematical Proofs** | Building valid logical arguments |

---

## 📝 Summary Table

| Concept | Symbol | Description |
|---------|:------:|-------------|
| Proposition | $p, q, r$ | Declarative sentence with definite truth value |
| Negation | $\neg p$ | Reverses truth value |
| Conjunction | $p \wedge q$ | True only when both are true |
| Disjunction | $p \vee q$ | True when at least one is true |
| Implication | $p \rightarrow q$ | False only when T → F |
| Biconditional | $p \leftrightarrow q$ | True when both have same truth value |
| XOR | $p \oplus q$ | True when exactly one is true |

---

## ❓ Quick Revision Questions

1. **Which of the following are propositions?**
   - (a) "5 + 3 = 8" (b) "What time is it?" (c) "x² = 9" (d) "The Earth is flat"
   
2. **If $p$ is true and $q$ is false, what is the truth value of $p \rightarrow q$?**

3. **Write the negation of "All students passed the exam."**

4. **Why is the statement "If 2 + 2 = 5, then the moon is made of cheese" considered true?**

5. **What is the difference between inclusive OR ($\vee$) and exclusive OR ($\oplus$)?**

6. **Given $p$: "It is cold" and $q$: "I wear a jacket", express "I wear a jacket only if it is cold" using logical notation.**

---

[← Back to README](../README.md) | [Next: Truth Tables →](02-truth-tables.md)
