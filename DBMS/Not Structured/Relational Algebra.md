---
tags:
  - database
  - relational-algebra
  - query-language
  - procedural-language
  - relational-model
---

# Relational Algebra

[[Relational Algebra]] is a **procedural** query language for the [[Relational Model]]. It was proposed by E.F. Codd in 1970 alongside the relational model itself. In relational algebra, the user specifies a sequence of operations to be performed on the database relations to obtain the desired result.

* **Nature:** Procedural (specifies *how* to get the result).
* **Operations:** It uses a set of well-defined operations that take one or two relations as input and produce a single new relation as output. These operations can be combined to form complex queries.

## Basic Structure of Relational Model (Formal View)

Relational algebra operates on relation instances, which formally are defined based on domains, attributes, and schemas (See [[Relational Model]]).

* **Domain:** A set of atomic values.
* **Attribute:** A role played by a domain, with a unique name within a relation.
* **Relation Schema:** Name of relation and set of attributes with their domains.
* **Tuple:** A collection of attribute values, one for each attribute.
* **Relation Instance:** A finite *set* of tuples conforming to the schema.

## Relational Algebra Operations

Relational Algebra operations are divided into Unary (operating on one relation) and Binary (operating on two relations).

### Unary Operations

* **Selection ($\sigma$):** Filters rows from a single relation based on a condition.
    * **Purpose:** Selects a subset of tuples from a relation that satisfy a given **selection predicate** ($\rho$).
    * **Symbol:** $\sigma$
    * **Notation:** $\sigma_{\rho}(r)$
    * **Definition:** $\{t \mid t \in r \land \rho(t)\}$ (The set of all tuples $t$ in relation $r$ such that the predicate $\rho$ evaluated on $t$ is true).
    * **Predicate ($\rho$):** A Boolean expression formed using attribute names, constants, comparison operators ($=, \ne, %3C, %3E, \le, \ge$), and logical operators ($\land$ (AND), $\lor$ (OR), $\neg$ (NOT)).

* **Projection ($\prod$):** Extracts specified columns from a single relation.
    * **Purpose:** Creates a new relation containing only a subset of the attributes (columns) of the original relation.
    * **Symbol:** $\prod$
    * **Notation:** $\prod_{A1, A2, ..., Ak}(r)$
    * **Definition:** The relation formed by taking relation $r$ and keeping only the attributes $A1, A2, ..., Ak$.
    * **Duplicate Removal:** Duplicate rows are automatically removed in the result because a relation is a set of unique tuples.

* **Rename ($\rho$):** Allows renaming relations or attributes.
    * **Purpose:** To give a new name to the result of a relational-algebra expression or to rename the attributes of a relation. Useful for complex queries or when performing operations like Cartesian product where attribute names might clash.
    * **Symbol:** $\rho$
    * **Notation:**
        * $\rho_x(E)$: Returns the result of expression $E$ under the name $X$.
        * $\rho_{x(A1, A2, ..., An)}(E)$: Returns the result of expression $E$ with the attributes renamed to $A1, A2, ..., An$. (Used when $E$ has arity $n$).

### Binary Operations

* **Union ($\cup$):** Combines tuples from two relations.
    * **Purpose:** Creates a new relation containing all tuples that are in either of the two input relations.
    * **Symbol:** $\cup$
    * **Notation:** $r \cup s$
    * **Definition:** $\{t \mid t \in r \lor t \in s\}$.
    * **Compatibility:** For the union operation to be valid, the two relations $r$ and $s$ must be **union-compatible**: they must have the same number of attributes and compatible attribute domains.

* **Set Difference ($-$):** Returns tuples present in one relation but not another.
    * **Purpose:** Creates a new relation containing all tuples that are in the first relation but *not* in the second relation.
    * **Symbol:** $-$
    * **Notation:** $r - s$
    * **Definition:** $\{t \mid t \in r \land t \notin s\}$.
    * **Compatibility:** Must be performed on union-compatible relations $r$ and $s$.

* **Cartesian Product ($\times$):** Combines every tuple of one relation with every tuple of another.
    * **Purpose:** Creates a new relation by concatenating each tuple of the first relation with each tuple of the second relation.
    * **Symbol:** $\times$
    * **Notation:** $r \times s$
    * **Definition:** $\{tq \mid t \in r \land q \in s\}$. (where $tq$ is the concatenation of tuple $t$ and tuple $q$).
    * **Attribute Naming:** If relations have attributes with the same name, renaming must be used on at least one before the product.
    * **Usefulness:** Fundamental for defining other operations like Join.

### Additional Relational-Algebra Operations

These operations can be expressed using the basic operations but are commonly used for convenience and clarity.

* **Set Intersection ($\cap$):** Returns tuples present in both relations.
    * **Purpose:** Creates a new relation containing only the tuples that are present in *both* input relations.
    * **Symbol:** $\cap$
    * **Notation:** $r \cap s$
    * **Definition:** $\{t \mid t \in r \land t \in s\}$.
    * **Compatibility:** Must be performed on union-compatible relations.
    * **Equivalent Basic Operations:** $r \cap s = r - (r - s)$.

* **Natural Join ($\bowtie$):** Combines tuples from two relations that have matching values on their common attributes.
    * **Purpose:** Joins two relations based on the equality of values in their shared attributes, combining related data from different tables.
    * **Symbol:** $\bowtie$
    * **Notation:** $r \bowtie s$
    * **Definition:** Combines tuples from $r$ and $s$ where values match on common attributes, projecting the result without duplicating common columns.

* **Division ($\div$):** Used for queries involving the phrase "for all".
    * **Purpose:** Finds entities in one set that are related to *all* entities in another set through a specific relationship.
    * **Symbol:** $\div$
    * **Notation:** $r \div s$
    * **Definition:** Let $r$ be on schema $R$, $s$ on schema $S \subseteq R$. $r \div s$ is on schema $R - S$. A tuple $t$ is in $r \div s$ if $\forall u \in s$, the tuple $tu$ (combination of $t$ and $u$) is in $r$.

* **Assignment ($\leftarrow$):** Allows breaking down complex queries into smaller steps using temporary relation variables.
    * **Purpose:** Improves readability and allows reusing intermediate results.
    * **Notation:** $temp\_relation \leftarrow E$

### Composition of Operations

Relational algebra operations can be nested; the output of one operation can be the input for another, allowing for the construction of complex queries.

### Extended Relational-Algebra Operations

These extend the basic set for more power or convenience.

* **Generalized Projection ($\prod_{F1, F2, ...}$):** Allows performing arithmetic calculations within the projection list.
    * **Notation:** $\prod_{F1, F2, ..., Fn}(E)$ where $F_i$ can be an attribute name or an arithmetic expression.

* **Aggregate Functions and Operations:** Allow summarizing data.
    * **Functions:** `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
    * **Operation:** Groups tuples and applies aggregate functions to groups.
    * **Symbol:** $\mathcal{G}$ (or $\gamma$)
    * **Notation:** $G_1, \dots \mathcal{G}_{\text{Agg1(Attr1), } \dots}(\text{Relation})$ (Group by $G_i$, aggregate $Attr_j$ using function $Agg_j$).

* **Outer Join:** Extends [[Relational Algebra#Natural Join ($\bowtie$)|Natural Join]] to include tuples without matches, using [[SQL NULL Values|Null Values]].
    * **Types:** Left Outer Join ($\enhancedisplaysci{\mathrel{\text{\textbf{⟕}}}}$), Right Outer Join ($\enhancedisplaysci{\mathrel{\text{\textbf{⟖}}}}$), Full Outer Join ($\enhancedisplaysci{\mathrel{\text{\textbf{⟗}}}}$).

## Relation to SQL

[[SQL]] is a practical query language based on the principles of relational algebra (and calculus). Many SQL operations (like `SELECT`, `WHERE`, `JOIN`, `GROUP BY`) directly correspond to or are implemented using relational algebra concepts.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Relational Model]]
* [[Relational Calculus]]
* [[SQL]]
* [[View of Data (Data Abstraction)|View of Data]]
* [[SQL Aggregate Functions and Grouping]]
* [[SQL Joins]]
* [[SQL NULL Values]]