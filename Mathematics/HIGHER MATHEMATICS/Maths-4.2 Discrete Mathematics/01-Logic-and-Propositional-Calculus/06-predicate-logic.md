# Chapter 1.6: Predicate Logic

[← Previous: Normal Forms](05-normal-forms.md) | [Back to README](../README.md) | [Next: Quantifiers →](07-quantifiers.md)

---

## 📋 Chapter Overview

Propositional logic deals with whole propositions. **Predicate logic** (also called **first-order logic**) extends this by analyzing the internal structure of propositions using **predicates**, **variables**, and **quantifiers**. This allows us to express statements like "All primes greater than 2 are odd" which propositional logic cannot handle.

---

## 1. Limitations of Propositional Logic

Consider these statements:
- "All humans are mortal"
- "Socrates is human"
- "Therefore, Socrates is mortal"

In propositional logic, we can only assign: $p$: "All humans are mortal", $q$: "Socrates is human", $r$: "Socrates is mortal"

We cannot express the *internal logical connection* between these statements. Predicate logic solves this.

---

## 2. Predicates

A **predicate** is a statement containing one or more variables that becomes a proposition when values are substituted for the variables.

### Notation

$P(x)$: predicate $P$ with variable $x$

### Examples

| Predicate | Variable | Instance | Truth Value |
|-----------|----------|----------|:-----------:|
| $P(x)$: "$x$ is even" | $x$ | $P(4)$: "4 is even" | T |
| $P(x)$: "$x$ is even" | $x$ | $P(7)$: "7 is even" | F |
| $Q(x,y)$: "$x + y > 10$" | $x, y$ | $Q(5, 8)$: "$5+8 > 10$" | T |
| $R(x)$: "$x$ is a prime" | $x$ | $R(9)$: "9 is a prime" | F |

### Predicate vs. Proposition

```
  ┌───────────────────────────────────────────────────┐
  │  Predicate (Open Sentence)    Proposition          │
  │                                                    │
  │  P(x): "x > 5"     ───→      P(7): "7 > 5" = T   │
  │       │                            │               │
  │  Contains free      Substitute     No free         │
  │  variable(s)        values         variables        │
  │                                                    │
  │  NOT true or false                 IS true or false │
  └───────────────────────────────────────────────────┘
```

---

## 3. Domain of Discourse (Universe of Discourse)

The **domain** $D$ specifies the set of values that variables can take.

The truth value of a predicate depends on the domain:

| Predicate | Domain | Truth |
|-----------|--------|:-----:|
| $P(x)$: "$x^2 \geq 0$" | $D = \mathbb{R}$ (all reals) | Always T |
| $P(x)$: "$x > 0$" | $D = \mathbb{Z}$ (all integers) | Sometimes T |
| $P(x)$: "$x > 0$" | $D = \mathbb{Z}^+$ (positive integers) | Always T |

> **Important:** Always specify the domain when working with predicates!

---

## 4. Multi-variable Predicates

Predicates can have **multiple variables** (also called **n-ary predicates** or **n-place predicates**):

- **Unary:** $P(x)$ — "$x$ is prime" (1 variable)
- **Binary:** $Q(x, y)$ — "$x$ divides $y$" (2 variables)
- **Ternary:** $R(x, y, z)$ — "$x + y = z$" (3 variables)
- **n-ary:** $S(x_1, x_2, \ldots, x_n)$ (n variables)

### Example: Binary Predicate

Let $L(x, y)$: "$x$ loves $y$" with domain = {Alice, Bob, Carl}

This defines a **relation** (covered in Unit 4):

| $L(x,y)$ | Alice | Bob | Carl |
|:---------:|:-----:|:---:|:----:|
| **Alice** | F | T | F |
| **Bob** | T | F | T |
| **Carl** | F | F | F |

Reading: $L(\text{Alice}, \text{Bob}) = T$ means "Alice loves Bob"

---

## 5. Propositional Functions

A **propositional function** $P(x_1, x_2, \ldots, x_n)$ is a generalization of predicates. It becomes a proposition when:

1. **All variables are assigned** specific values from the domain, OR
2. **Quantifiers are applied** to all variables (covered in next chapter)

### The Truth Set

The **truth set** of $P(x)$ over domain $D$ is:

$$\{x \in D \mid P(x) \text{ is true}\}$$

**Example:** $P(x)$: "$x^2 < 10$", $D = \mathbb{Z}^+$

$$\text{Truth set} = \{1, 2, 3\}$$

since $1^2 = 1 < 10$, $2^2 = 4 < 10$, $3^2 = 9 < 10$, but $4^2 = 16 \geq 10$.

---

## 6. Compound Predicates

We can combine predicates using logical connectives, just like propositions:

Let $P(x)$: "$x$ is prime" and $E(x)$: "$x$ is even", with $D = \mathbb{Z}^+$

| Compound Predicate | Meaning | Example |
|-------------------|---------|---------|
| $P(x) \wedge E(x)$ | $x$ is prime AND even | True only for $x = 2$ |
| $P(x) \vee E(x)$ | $x$ is prime OR even | True for 2, 3, 4, 5, 6, 7, 8, ... |
| $P(x) \wedge \neg E(x)$ | $x$ is prime AND odd | True for 3, 5, 7, 11, ... |
| $P(x) \rightarrow E(x)$ | If $x$ is prime then $x$ is even | False for $x = 3$ |

---

## 7. Predicate Logic Notation for Classic Arguments

### Socrates Argument (revisited)

Using predicate logic:

- $H(x)$: "$x$ is human"
- $M(x)$: "$x$ is mortal"

**Statements:**
1. $\forall x (H(x) \rightarrow M(x))$ — "All humans are mortal"
2. $H(\text{Socrates})$ — "Socrates is human"
3. $\therefore M(\text{Socrates})$ — "Therefore, Socrates is mortal"

This is **Universal Modus Ponens** — a valid argument form.

---

## 8. Free and Bound Variables

A variable is **free** if it is not bound by a quantifier. A variable is **bound** if a quantifier applies to it.

| Expression | Free Variables | Bound Variables |
|-----------|:--------------:|:---------------:|
| $P(x)$ | $x$ | — |
| $\forall x \, P(x)$ | — | $x$ |
| $\forall x \, (P(x) \rightarrow Q(x, y))$ | $y$ | $x$ |
| $\exists x \, \forall y \, R(x, y, z)$ | $z$ | $x, y$ |

> **A well-formed formula (wff)** with no free variables is a **sentence** — it has a definite truth value.

---

## 9. Predicate Logic vs. Propositional Logic

| Feature | Propositional Logic | Predicate Logic |
|---------|:-------------------:|:---------------:|
| Basic unit | Proposition ($p, q$) | Predicate ($P(x), Q(x,y)$) |
| Variables | No individual variables | Has individual variables |
| Quantifiers | No | $\forall$, $\exists$ |
| Expressive power | Limited | Much greater |
| Decidability | Decidable | Semi-decidable |
| Example | $p \wedge q$ | $\forall x (P(x) \rightarrow Q(x))$ |

---

## 10. Real-World Applications

| Application | How Predicate Logic is Used |
|-------------|---------------------------|
| **Databases** | SQL queries are predicate logic expressions |
| **Prolog** | Logic programming language based on predicate logic |
| **AI / Expert Systems** | Knowledge representation and inference |
| **Formal Verification** | Specifying system properties |
| **Mathematics** | Formal proofs and axiom systems |

**SQL Example:**
```sql
-- Predicate logic: ∃x (Student(x) ∧ Grade(x) > 90)
SELECT * FROM Students WHERE grade > 90;
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Predicate | Statement with variables: $P(x)$ |
| Domain | Set of possible values for variables |
| Propositional Function | Becomes proposition when all variables are assigned |
| Truth Set | $\{x \in D \mid P(x)\}$ |
| n-ary Predicate | Predicate with $n$ variables |
| Free Variable | Not bound by quantifier |
| Bound Variable | Under scope of a quantifier |
| Sentence | Formula with no free variables |

---

## ❓ Quick Revision Questions

1. **Which of the following are predicates and which are propositions?**
   - (a) "$x + 3 = 7$" (b) "$5 > 2$" (c) "$n$ is divisible by 3" (d) "Every cat has a tail"

2. **Given $P(x)$: "$x^2 = 4$" with domain $D = \mathbb{Z}$, what is the truth set?**

3. **Let $Q(x,y)$: "$x \leq y$" over $D = \{1, 2, 3\}$. List all pairs $(x,y)$ that make $Q$ true.**

4. **Why can propositional logic not express "All even numbers are divisible by 2"?**

5. **Identify free and bound variables in: $\forall x \exists y (P(x,y) \wedge Q(y,z))$**

6. **Express "There exists a student who passed all exams" using predicate logic notation.**

---

[← Previous: Normal Forms](05-normal-forms.md) | [Back to README](../README.md) | [Next: Quantifiers →](07-quantifiers.md)
