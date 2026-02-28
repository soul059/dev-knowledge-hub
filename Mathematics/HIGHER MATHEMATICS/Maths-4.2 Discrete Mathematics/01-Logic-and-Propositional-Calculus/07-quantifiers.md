# Chapter 1.7: Quantifiers

[← Previous: Predicate Logic](06-predicate-logic.md) | [Back to README](../README.md) | [Next: Unit 2 — Direct Proof →](../02-Methods-of-Proof/01-direct-proof.md)

---

## 📋 Chapter Overview

Quantifiers turn predicates (open sentences) into propositions by specifying **how many** elements in the domain satisfy the predicate. The two primary quantifiers are the **universal quantifier** ($\forall$) and the **existential quantifier** ($\exists$).

---

## 1. Universal Quantifier — $\forall$

### Definition

$\forall x \, P(x)$ reads "**for all** $x$, $P(x)$" or "**for every** $x$, $P(x)$."

It is **true** if $P(x)$ is true for **every** element in the domain $D$.  
It is **false** if there exists **at least one** element $a \in D$ such that $P(a)$ is false.

### Truth Value Over Finite Domain

If $D = \{a_1, a_2, \ldots, a_n\}$, then:

$$\forall x \, P(x) \equiv P(a_1) \wedge P(a_2) \wedge \cdots \wedge P(a_n)$$

### Example

Let $P(x)$: "$x^2 \geq x$", $D = \{1, 2, 3\}$

| $x$ | $x^2$ | $x^2 \geq x$? |
|:---:|:---:|:---:|
| 1 | 1 | T ($1 \geq 1$) |
| 2 | 4 | T ($4 \geq 2$) |
| 3 | 9 | T ($9 \geq 3$) |

$$\forall x \, P(x) = T \wedge T \wedge T = T$$

### Counterexample

To **disprove** $\forall x \, P(x)$, find just ONE value where $P(x)$ is false. This value is called a **counterexample**.

**Example:** Disprove: $\forall x \in \mathbb{R}, \, x^2 > x$

**Counterexample:** $x = 0.5$, then $0.5^2 = 0.25 < 0.5$ → $P(0.5)$ is false ✗

---

## 2. Existential Quantifier — $\exists$

### Definition

$\exists x \, P(x)$ reads "**there exists** an $x$ such that $P(x)$" or "**for some** $x$, $P(x)$."

It is **true** if $P(x)$ is true for **at least one** element in the domain $D$.  
It is **false** if $P(x)$ is false for **every** element in $D$.

### Truth Value Over Finite Domain

$$\exists x \, P(x) \equiv P(a_1) \vee P(a_2) \vee \cdots \vee P(a_n)$$

### Example

Let $P(x)$: "$x$ is even", $D = \{3, 5, 6, 7\}$

| $x$ | Even? |
|:---:|:---:|
| 3 | F |
| 5 | F |
| 6 | **T** |
| 7 | F |

$$\exists x \, P(x) = F \vee F \vee T \vee F = T$$

---

## 3. Unique Existential Quantifier — $\exists!$

$\exists! x \, P(x)$ means "there exists **exactly one** $x$ such that $P(x)$."

$$\exists! x \, P(x) \equiv \exists x \, (P(x) \wedge \forall y \, (P(y) \rightarrow y = x))$$

**Example:** $\exists! x \in \mathbb{R}, \, x + 3 = 5$ → Only $x = 2$ satisfies this → **True**

---

## 4. Comparison Table

| Quantifier | Symbol | True When | False When |
|-----------|:------:|-----------|------------|
| Universal | $\forall x \, P(x)$ | ALL elements satisfy $P$ | At least one doesn't |
| Existential | $\exists x \, P(x)$ | At least one satisfies $P$ | NONE satisfy $P$ |
| Unique Existential | $\exists! x \, P(x)$ | Exactly one satisfies $P$ | Zero or more than one |

---

## 5. Negation of Quantifiers — De Morgan's Laws for Quantifiers

These are among the most important rules in predicate logic:

$$\neg \forall x \, P(x) \equiv \exists x \, \neg P(x)$$

> "NOT all satisfy $P$" ≡ "There exists one that does NOT satisfy $P$"

$$\neg \exists x \, P(x) \equiv \forall x \, \neg P(x)$$

> "There does NOT exist one satisfying $P$" ≡ "ALL fail to satisfy $P$"

### Visualization

```
  ┌────────────────────────────────────────────┐
  │        De Morgan's for Quantifiers         │
  │                                            │
  │   ¬∀x P(x)  ═══  ∃x ¬P(x)               │
  │                                            │
  │   "Not all are P"  =  "Some are not P"     │
  │                                            │
  │   ¬∃x P(x)  ═══  ∀x ¬P(x)               │
  │                                            │
  │   "None are P"  =  "All are not P"         │
  └────────────────────────────────────────────┘
```

### Examples of Negation

| Original Statement | Negation |
|-------------------|----------|
| "All students passed" $\forall x \, P(x)$ | "Some student did not pass" $\exists x \, \neg P(x)$ |
| "There exists an even prime > 2" $\exists x \, Q(x)$ | "No even prime > 2 exists" $\forall x \, \neg Q(x)$ |
| "Everyone likes ice cream" | "Someone does not like ice cream" |
| "Some dogs can fly" | "No dog can fly" |

---

## 6. Nested Quantifiers

When predicates have multiple variables, we use **nested quantifiers**.

### Order of Quantifiers

| Statement | Meaning | Example ($D = \mathbb{Z}^+$) |
|-----------|---------|------|
| $\forall x \forall y \, P(x,y)$ | For all $x$ and all $y$, $P(x,y)$ | All pairs satisfy $P$ |
| $\forall x \exists y \, P(x,y)$ | For every $x$, there exists a $y$ | Each $x$ has a matching $y$ |
| $\exists x \forall y \, P(x,y)$ | There exists an $x$ for which all $y$ | One $x$ works for all $y$ |
| $\exists x \exists y \, P(x,y)$ | There exist $x$ and $y$ | At least one pair satisfies $P$ |

> **Critical:** $\forall x \exists y \, P(x,y) \not\equiv \exists y \forall x \, P(x,y)$ in general!

### Example: $P(x,y)$: "$x + y = 0$", $D = \mathbb{Z}$

| Expression | Meaning | Truth Value |
|-----------|---------|:-----------:|
| $\forall x \forall y \, (x+y=0)$ | Every two integers sum to 0 | **F** |
| $\forall x \exists y \, (x+y=0)$ | For every $x$, there's a $y$ s.t. $x+y=0$ | **T** (take $y = -x$) |
| $\exists x \forall y \, (x+y=0)$ | Some $x$ makes $x+y=0$ for ALL $y$ | **F** |
| $\exists x \exists y \, (x+y=0)$ | Some pair sums to 0 | **T** (e.g., $x=3, y=-3$) |

### Visualization of Nested Quantifiers

```
  ∀x ∃y P(x,y):                    ∃x ∀y P(x,y):
  
  For EACH x →  find SOME y        Find ONE x →  works for ALL y
  
  x₁ → y₃  ✓                      x₂ → y₁ ✓
  x₂ → y₁  ✓   (different y       x₂ → y₂ ✓   (same x must
  x₃ → y₂  ✓    for each x)       x₂ → y₃ ✓    work for all y)
  
  WEAKER condition                  STRONGER condition
```

---

## 7. Negation of Nested Quantifiers

Apply De Morgan's rules from left to right, flipping each quantifier:

$$\neg \forall x \exists y \, P(x,y) \equiv \exists x \forall y \, \neg P(x,y)$$

$$\neg \exists x \forall y \exists z \, P(x,y,z) \equiv \forall x \exists y \forall z \, \neg P(x,y,z)$$

### General Rule

```
  ¬ ∀ ∃ ∀ ... P(...)
  ≡ ∃ ∀ ∃ ... ¬P(...)
  
  Rule: Flip each quantifier (∀↔∃) and negate the predicate
```

### Worked Example

Negate: "For every student $s$, there exists a course $c$ such that $s$ passed $c$."

$$\forall s \, \exists c \, \text{Passed}(s, c)$$

Negation:
$$\exists s \, \forall c \, \neg\text{Passed}(s, c)$$

English: "There exists a student $s$ who did not pass any course $c$."

---

## 8. Translating English to Predicate Logic

### Common Patterns

| English | Predicate Logic |
|---------|----------------|
| "All A are B" | $\forall x \, (A(x) \rightarrow B(x))$ |
| "Some A are B" | $\exists x \, (A(x) \wedge B(x))$ |
| "No A are B" | $\forall x \, (A(x) \rightarrow \neg B(x))$ |
| "Some A are not B" | $\exists x \, (A(x) \wedge \neg B(x))$ |

> **Warning:** Use $\rightarrow$ with $\forall$ and $\wedge$ with $\exists$!

**Why?**
- $\forall x (A(x) \wedge B(x))$ would wrongly claim everything is both A and B
- $\exists x (A(x) \rightarrow B(x))$ would be too weak — it's satisfied by any non-A

### Example Translations

| English | Predicate Logic |
|---------|----------------|
| "Every prime number greater than 2 is odd" | $\forall x \, ((P(x) \wedge x > 2) \rightarrow O(x))$ |
| "There is a student who likes both math and physics" | $\exists x \, (S(x) \wedge L(x, \text{math}) \wedge L(x, \text{physics}))$ |
| "No integer is both even and odd" | $\forall x \, \neg(E(x) \wedge O(x))$ |
| "Between any two reals there is a rational" | $\forall x \forall y \, (x < y \rightarrow \exists z \, (Q(z) \wedge x < z \wedge z < y))$ |

---

## 9. Rules of Inference for Quantified Statements

| Rule | Name | Form |
|------|------|------|
| Universal Instantiation | UI | $\forall x \, P(x) \therefore P(c)$ for any $c$ |
| Universal Generalization | UG | $P(c)$ for arbitrary $c$ $\therefore \forall x \, P(x)$ |
| Existential Instantiation | EI | $\exists x \, P(x) \therefore P(c)$ for some specific $c$ |
| Existential Generalization | EG | $P(c)$ for some $c$ $\therefore \exists x \, P(x)$ |

### Universal Modus Ponens

$$\forall x \, (P(x) \rightarrow Q(x))$$
$$P(a)$$
$$\therefore Q(a)$$

**Example:**
1. All humans are mortal: $\forall x \, (H(x) \rightarrow M(x))$
2. Socrates is human: $H(\text{Socrates})$
3. Therefore, Socrates is mortal: $M(\text{Socrates})$ ✅

---

## 10. Real-World Applications

| Application | Quantifier Usage |
|-------------|-----------------|
| **Software Specifications** | "For all inputs, the program terminates" |
| **Database Queries** | SQL EXISTS and FOR ALL (via NOT EXISTS NOT) |
| **Mathematical Definitions** | Continuity: $\forall \epsilon > 0, \exists \delta > 0, \ldots$ |
| **Legal Reasoning** | "No person shall..." = $\forall x \, \neg P(x)$ |
| **Type Systems** | Polymorphism: $\forall T, \text{List}(T) \rightarrow \text{Int}$ |

---

## 📝 Summary Table

| Concept | Symbol | Key Point |
|---------|:------:|-----------|
| Universal | $\forall x \, P(x)$ | True when ALL satisfy $P$ |
| Existential | $\exists x \, P(x)$ | True when at least ONE satisfies $P$ |
| Unique Existential | $\exists! x \, P(x)$ | Exactly one satisfies $P$ |
| Negation of $\forall$ | $\neg\forall \equiv \exists\neg$ | "Not all" = "Some don't" |
| Negation of $\exists$ | $\neg\exists \equiv \forall\neg$ | "None" = "All don't" |
| Nesting order | $\forall\exists \neq \exists\forall$ | Order matters! |
| $\forall$ with English | Use $\rightarrow$ | "All A are B" → $\forall x(A \rightarrow B)$ |
| $\exists$ with English | Use $\wedge$ | "Some A are B" → $\exists x(A \wedge B)$ |

---

## ❓ Quick Revision Questions

1. **Let $P(x)$: "$x^2 < 10$" with $D = \{1, 2, 3, 4\}$. Find the truth values of $\forall x\, P(x)$ and $\exists x\, P(x)$.**

2. **Negate: "For every real number $x$, if $x > 0$ then $x^2 > 0$."**

3. **Translate to predicate logic: "All birds can fly except penguins."**

4. **Why is $\forall x \exists y \, (x + y = 0)$ true over $\mathbb{Z}$ but $\exists y \forall x \, (x + y = 0)$ false?**

5. **Negate the statement: $\exists x \forall y \exists z \, (x + y = z)$**

6. **Using Universal Modus Ponens, derive a conclusion from:**
   - "All mammals have lungs"
   - "A whale is a mammal"

---

[← Previous: Predicate Logic](06-predicate-logic.md) | [Back to README](../README.md) | [Next: Unit 2 — Direct Proof →](../02-Methods-of-Proof/01-direct-proof.md)
