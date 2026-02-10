---
tags:
  - database
  - relational-calculus
  - query-language
  - non-procedural-language
  - relational-model
---

# Relational Calculus

[[Relational Calculus]] is a family of **non-procedural** (declarative) query languages for the [[Relational Model]]. Unlike [[Relational Algebra]], which specifies *how* to obtain a result by defining a sequence of operations, relational calculus specifies *what* data is needed based on a mathematical predicate (a condition or property). The database system is then responsible for figuring out the most efficient way to retrieve the data that satisfies the predicate.

* **Nature:** Non-procedural / Declarative (specifies *what* is needed).
* **Basis:** Uses mathematical logic and predicates.

There are two primary forms of relational calculus:

1.  [[#Tuple Relational Calculus (TRC)|Tuple Relational Calculus (TRC)]]
2.  [[#Domain Relational Calculus (DRC)|Domain Relational Calculus (DRC)]]

## Tuple Relational Calculus (TRC)

TRC is a non-procedural language where queries are expressed as finding a set of tuples that satisfy a given predicate.

* **Basis:** Uses **tuple variables** that range over the tuples of database relations.
* **Query Form:** $\{t \mid P(t)\}$
    * $t$: A tuple variable, representing a tuple in the result relation.
    * $P(t)$: A predicate formula involving the tuple variable $t$, specifying the conditions that $t$ must satisfy to be included in the result set.
* **Predicate Formulas:** Built using:
    * **Atomic formulas:**
        * $t \in R$: Tuple variable $t$ is in relation $R$.
        * $t[A] \theta s[B]$: The value of attribute $A$ in tuple $t$ compared ($\theta$) to the value of attribute $B$ in tuple $s$. ($\theta$ is a comparison operator like $=, \ne, %3C, %3E, \le, \ge$)
        * $t[A] \theta c$: The value of attribute $A$ in tuple $t$ compared to a constant $c$.
    * **Logical connectives:** $\land$ (AND), $\lor$ (OR), $\neg$ (NOT).
    * **Implication:** $\implies$.
    * **Quantifiers:**
        * $\exists s (P(s))$: "There exists a tuple variable $s$ such that predicate $P(s)$ is true." (Existential Quantifier)
        * $\forall s (P(s))$: "For all tuple variables $s$, predicate $P(s)$ is true." (Universal Quantifier)

* **Example (Conceptual):** Find the names of all students in the 'CE' department.
    * $\{ s.Name \mid s \in \text{Student} \land s.\text{Dept} = \text{'CE'} \}$
    * (This reads: "The set of Name values of tuples $s$, such that $s$ is in the Student relation AND the value of $s$'s Dept attribute is 'CE'.")

## Domain Relational Calculus (DRC)

DRC is another non-procedural language where queries are expressed as finding a set of attribute values (components of tuples) that satisfy a given predicate.

* **Basis:** Uses **domain variables** that range over the values within the domains of attributes.
* **Query Form:** $\{ <x_1, x_2, ..., x_n> \mid P(x_1, x_2, ..., x_n) \}$
    * $<x_1, x_2, ..., x_n>$: A list of domain variables representing the components (values) of the tuples in the result relation.
    * $P(x_1, x_2, ..., x_n)$: A predicate formula involving the domain variables $x_1, \dots, x_n$, specifying the conditions that these values must satisfy.
* **Predicate Formulas:** Built using domain variables, relation membership, comparison operators, logical connectives, and quantifiers ($\exists, \forall$).
    * **Atomic formulas:**
        * $<v_1, v_2, ..., v_m> \in R$: The tuple formed by values $v_1, ..., v_m$ (where $v_i$ are domain variables or constants) is in relation $R$. The variables/constants must correspond to the attributes of $R$.
        * $x_i \theta x_j$, $x_i \theta c$.

* **Example (Conceptual):** Find the names of all students in the 'CE' department.
    * $\{ <s\_name> \mid \exists s\_id, s\_dept, s\_cpi (<s\_id, s\_name, s\_dept, s\_cpi> \in \text{Student} \land s\_dept = \text{'CE'}) \}$
    * (This reads: "The set of $s\_name$ values, such that there exist values $s\_id, s\_dept, s\_cpi$ where the tuple formed by these values is in the Student relation AND $s\_dept$ is 'CE'.")

## Safety of Expressions (TRC and DRC)

* **Problem:** Without restrictions, relational calculus allows expressions that define infinite relations (e.g., asking for all values that *don't* appear in the database).
* **Solution:** **Safe expressions** are defined to avoid this. A safe expression guarantees that all values in the resulting tuples are taken from the finite set of values present in the database relations or the constants used in the query.
* Specific conditions on the use of quantifiers and variables within predicate formulas ensure safety.

## Relation to SQL

[[SQL]] is influenced by both relational algebra and relational calculus. While its structure and syntax are more procedural-like (using keywords like `SELECT`, `FROM`, `WHERE`), its underlying nature is largely declarative (you specify *what* you want in the `WHERE` clause, not the step-by-step process). The database system's query optimizer converts the declarative SQL query into an execution plan, often based on relational algebra operations.

Relational calculus provides a formal basis for understanding the expressive power of query languages – any query that can be expressed in one of the safe relational calculi can also be expressed in relational algebra, and vice versa.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Relational Model]]
* [[Relational Algebra]]
* [[SQL]]
* [[Query Languages]] (Link to section in another note if needed)
* [[Safe Expressions]] (Create if needed)