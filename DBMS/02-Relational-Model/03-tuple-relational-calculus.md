# Chapter 2.3: Tuple Relational Calculus

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**Tuple Relational Calculus (TRC)** is a non-procedural query language where we specify WHAT data we want without specifying HOW to retrieve it. It's based on first-order predicate logic and uses tuple variables.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand the difference between procedural and declarative │
│  • Learn tuple variable concept and quantifiers                 │
│  • Write TRC queries for various scenarios                      │
│  • Understand safe expressions                                  │
│  • Compare TRC with Relational Algebra                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. Relational Calculus vs Relational Algebra

```
┌─────────────────────────────────────────────────────────────────────┐
│              ALGEBRA vs CALCULUS COMPARISON                          │
└─────────────────────────────────────────────────────────────────────┘

    RELATIONAL ALGEBRA                RELATIONAL CALCULUS
    ──────────────────                ────────────────────
    
    • PROCEDURAL                      • NON-PROCEDURAL (Declarative)
    • Specifies HOW to retrieve       • Specifies WHAT to retrieve
    • Sequence of operations          • Describes properties of result
    • Closer to implementation        • Closer to natural language
    
    
    EXAMPLE: "Get names of CS students"
    ────────────────────────────────────
    
    RELATIONAL ALGEBRA:
    π_Name (σ_Dept='CS' (STUDENT))
    
    Step 1: Select students from CS dept
    Step 2: Project their names
    
    
    TUPLE RELATIONAL CALCULUS:
    { t.Name | STUDENT(t) AND t.Dept = 'CS' }
    
    "Give me names from STUDENT tuples where Dept is CS"
    (No steps specified - just what we want)


    EQUIVALENCE:
    ─────────────

    ┌───────────────────────────────────────────────────────────────┐
    │                                                                │
    │  For every relational algebra expression, there exists an     │
    │  equivalent relational calculus expression, and vice versa.   │
    │                                                                │
    │  They have the SAME expressive power.                         │
    │                                                                │
    └───────────────────────────────────────────────────────────────┘
```

---

## 2. TRC Basics

### General Form

```
┌─────────────────────────────────────────────────────────────────────┐
│                      TRC GENERAL FORM                                │
└─────────────────────────────────────────────────────────────────────┘

    { t | P(t) }
    
    OR for specific attributes:
    
    { t.A1, t.A2, ... | P(t) }
    
    
    WHERE:
    ──────
    t       = Tuple variable (ranges over tuples)
    P(t)    = Predicate (condition) that t must satisfy
    t.A     = Attribute A of tuple t
    
    
    MEANING:
    ─────────
    "The set of all tuples t such that predicate P(t) is TRUE"


    EXAMPLE:
    ─────────
    
    { t | STUDENT(t) AND t.Age > 20 }
    
    Components:
    • t is a tuple variable
    • STUDENT(t) means t is a tuple from STUDENT relation
    • t.Age > 20 is the condition
    • Result: All STUDENT tuples where Age > 20
```

### Tuple Variables

```
┌─────────────────────────────────────────────────────────────────────┐
│                      TUPLE VARIABLES                                 │
└─────────────────────────────────────────────────────────────────────┘

    A tuple variable RANGES OVER tuples of a relation.
    
    
    DECLARATION (Implicit):
    ────────────────────────
    
    STUDENT(t)  →  t ranges over tuples in STUDENT relation
    COURSE(c)   →  c ranges over tuples in COURSE relation
    
    
    ACCESSING ATTRIBUTES:
    ──────────────────────
    
    If t ranges over STUDENT(Stud_ID, Name, Age, Dept_ID):
    
    t.Stud_ID   →  Student ID of tuple t
    t.Name      →  Name of tuple t
    t.Age       →  Age of tuple t
    t.Dept_ID   →  Department ID of tuple t
    
    
    MULTIPLE TUPLE VARIABLES:
    ─────────────────────────
    
    { s.Name | STUDENT(s) AND ENROLLMENT(e) AND s.Stud_ID = e.Stud_ID }
    
    • s ranges over STUDENT
    • e ranges over ENROLLMENT
    • Condition links them together
```

---

## 3. Predicates and Formulas

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PREDICATE COMPONENTS                              │
└─────────────────────────────────────────────────────────────────────┘

    ATOMS (Basic building blocks):
    ───────────────────────────────

    1. RELATION MEMBERSHIP
       R(t)
       "t is a tuple in relation R"
       
       Example: STUDENT(s)
    
    2. COMPARISON
       t.A θ u.B   or   t.A θ constant
       where θ ∈ {=, ≠, <, >, ≤, ≥}
       
       Examples: 
       s.Age > 20
       s.Dept_ID = d.Dept_ID
    
    
    FORMULAS (Built from atoms):
    ─────────────────────────────

    If P and Q are formulas:
    
    ┌────────────────┬─────────────────────────────────────┐
    │   Formula      │           Meaning                   │
    ├────────────────┼─────────────────────────────────────┤
    │   P AND Q      │   Both P and Q must be true        │
    │   P OR Q       │   At least one must be true        │
    │   NOT P        │   P must be false                  │
    │   ∃t(P(t))     │   There exists at least one t      │
    │   ∀t(P(t))     │   For all t, P(t) is true         │
    └────────────────┴─────────────────────────────────────┘
```

---

## 4. Existential Quantifier (∃)

```
┌─────────────────────────────────────────────────────────────────────┐
│                 EXISTENTIAL QUANTIFIER (∃)                           │
└─────────────────────────────────────────────────────────────────────┘

    ∃t(P(t)) means "There EXISTS at least one tuple t such that P(t)"
    
    
    EXAMPLE 1: Students enrolled in at least one course
    ─────────────────────────────────────────────────────
    
    { s | STUDENT(s) AND ∃e(ENROLLMENT(e) AND e.Stud_ID = s.Stud_ID) }
    
    Translation:
    "Get students s where there EXISTS an enrollment e 
     with the same student ID"
    
    
    EXAMPLE 2: Students enrolled in 'CS101'
    ─────────────────────────────────────────
    
    { s | STUDENT(s) AND ∃e(ENROLLMENT(e) AND 
                           e.Stud_ID = s.Stud_ID AND 
                           e.Course_ID = 'CS101') }
    
    
    VISUALIZATION:
    ───────────────
    
    STUDENT                    ENROLLMENT
    ┌────────┬─────────┐      ┌────────┬───────────┐
    │ Stud_ID│  Name   │      │ Stud_ID│ Course_ID │
    ├────────┼─────────┤      ├────────┼───────────┤
    │  S101  │  Alice  │◄─────│  S101  │   CS101   │ ✓ Alice in result
    │  S102  │   Bob   │◄─────│  S101  │   CS102   │
    │  S103  │ Charlie │      │  S102  │   CS101   │ ✓ Bob in result
    │  S104  │  Diana  │      └────────┴───────────┘
    └────────┴─────────┘
    
    S103 and S104 NOT in result (no matching enrollment)
```

---

## 5. Universal Quantifier (∀)

```
┌─────────────────────────────────────────────────────────────────────┐
│                  UNIVERSAL QUANTIFIER (∀)                            │
└─────────────────────────────────────────────────────────────────────┘

    ∀t(P(t)) means "For ALL tuples t, P(t) is true"
    
    
    IMPORTANT RELATIONSHIP:
    ────────────────────────
    
    ∀t(P(t))  ≡  NOT ∃t(NOT P(t))
    
    "All t satisfy P" ≡ "There is no t that doesn't satisfy P"
    
    
    EXAMPLE: Students enrolled in ALL courses
    ───────────────────────────────────────────
    
    { s | STUDENT(s) AND 
          ∀c(COURSE(c) → ∃e(ENROLLMENT(e) AND 
                           e.Stud_ID = s.Stud_ID AND 
                           e.Course_ID = c.Course_ID)) }
    
    Translation:
    "Get students s where FOR ALL courses c,
     there EXISTS an enrollment e linking s to c"
    
    
    USING EQUIVALENCE (Alternative form):
    ──────────────────────────────────────
    
    { s | STUDENT(s) AND 
          NOT ∃c(COURSE(c) AND 
                 NOT ∃e(ENROLLMENT(e) AND 
                        e.Stud_ID = s.Stud_ID AND 
                        e.Course_ID = c.Course_ID)) }
    
    Translation:
    "Get students s where there is NO course c
     that s is NOT enrolled in"


    IMPLICATION (→):
    ─────────────────
    
    P → Q  ≡  NOT P OR Q
    
    "If P then Q" ≡ "Either P is false or Q is true"
```

---

## 6. TRC Examples

### Sample Relations

```
    STUDENT                         ENROLLMENT
    ┌────────┬─────────┬─────────┐  ┌────────┬───────────┬───────┐
    │ Stud_ID│  Name   │ Dept_ID │  │ Stud_ID│ Course_ID │ Grade │
    ├────────┼─────────┼─────────┤  ├────────┼───────────┼───────┤
    │  S101  │  Alice  │   D01   │  │  S101  │   CS101   │   A   │
    │  S102  │   Bob   │   D02   │  │  S101  │   CS102   │   B   │
    │  S103  │ Charlie │   D01   │  │  S102  │   CS101   │   A   │
    │  S104  │  Diana  │   D03   │  │  S103  │   MA101   │   B   │
    └────────┴─────────┴─────────┘  └────────┴───────────┴───────┘
    
    COURSE                          DEPARTMENT
    ┌───────────┬────────────┐      ┌─────────┬──────────────┐
    │ Course_ID │   Title    │      │ Dept_ID │  Dept_Name   │
    ├───────────┼────────────┤      ├─────────┼──────────────┤
    │   CS101   │  Database  │      │   D01   │     CS       │
    │   CS102   │  Networks  │      │   D02   │    Math      │
    │   MA101   │  Calculus  │      │   D03   │   Physics    │
    └───────────┴────────────┘      └─────────┴──────────────┘
```

### Query Examples

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TRC EXAMPLES                                 │
└─────────────────────────────────────────────────────────────────────┘


QUERY 1: All students with Age > 20
────────────────────────────────────

    { s | STUDENT(s) AND s.Age > 20 }
    
    Result: Bob (22), Charlie (21), Diana (23)


QUERY 2: Names of students in CS department (D01)
─────────────────────────────────────────────────

    { s.Name | STUDENT(s) AND s.Dept_ID = 'D01' }
    
    Result: Alice, Charlie


QUERY 3: Students enrolled in 'Database' course
───────────────────────────────────────────────

    { s | STUDENT(s) AND 
          ∃e(ENROLLMENT(e) AND 
             e.Stud_ID = s.Stud_ID AND 
             ∃c(COURSE(c) AND 
                c.Course_ID = e.Course_ID AND 
                c.Title = 'Database')) }
    
    OR (simplified with direct join):
    
    { s | STUDENT(s) AND 
          ∃e(ENROLLMENT(e) AND ∃c(COURSE(c) AND 
             s.Stud_ID = e.Stud_ID AND 
             e.Course_ID = c.Course_ID AND 
             c.Title = 'Database')) }
    
    Result: Alice, Bob


QUERY 4: Students NOT enrolled in any course
─────────────────────────────────────────────

    { s | STUDENT(s) AND 
          NOT ∃e(ENROLLMENT(e) AND e.Stud_ID = s.Stud_ID) }
    
    Result: Diana


QUERY 5: Students enrolled in BOTH CS101 AND CS102
──────────────────────────────────────────────────

    { s | STUDENT(s) AND 
          ∃e1(ENROLLMENT(e1) AND 
              e1.Stud_ID = s.Stud_ID AND 
              e1.Course_ID = 'CS101') AND 
          ∃e2(ENROLLMENT(e2) AND 
              e2.Stud_ID = s.Stud_ID AND 
              e2.Course_ID = 'CS102') }
    
    Result: Alice


QUERY 6: Students enrolled in ALL courses
──────────────────────────────────────────

    { s | STUDENT(s) AND 
          ∀c(COURSE(c) → 
             ∃e(ENROLLMENT(e) AND 
                e.Stud_ID = s.Stud_ID AND 
                e.Course_ID = c.Course_ID)) }
    
    Equivalent (without →):
    
    { s | STUDENT(s) AND 
          NOT ∃c(COURSE(c) AND 
                 NOT ∃e(ENROLLMENT(e) AND 
                        e.Stud_ID = s.Stud_ID AND 
                        e.Course_ID = c.Course_ID)) }
    
    Result: None (no student is in all 3 courses)


QUERY 7: Courses with no enrollments
─────────────────────────────────────

    { c | COURSE(c) AND 
          NOT ∃e(ENROLLMENT(e) AND e.Course_ID = c.Course_ID) }
    
    Result: (none based on sample data, but would return 
             courses that have no matching enrollment)
```

---

## 7. Free and Bound Variables

```
┌─────────────────────────────────────────────────────────────────────┐
│                  FREE AND BOUND VARIABLES                            │
└─────────────────────────────────────────────────────────────────────┘

    BOUND VARIABLE: Appears within scope of a quantifier (∃ or ∀)
    FREE VARIABLE:  Not bound by any quantifier
    
    
    EXAMPLE:
    ─────────
    
    { s | STUDENT(s) AND ∃e(ENROLLMENT(e) AND e.Stud_ID = s.Stud_ID) }
          ─┬─               ─┬─              ─┬─          ─┬─
           │                 │                │            │
         FREE              BOUND            BOUND        FREE
    (not under ∃)     (under ∃)        (under ∃)    (reference to s)
    
    
    RULES:
    ───────
    
    1. In { t | P(t) }, t is the FREE variable (defines result)
    
    2. Variables introduced by ∃ or ∀ are BOUND within their scope
    
    3. A well-formed expression has exactly ONE free variable
       in the result specification
    
    
    SCOPE VISUALIZATION:
    ─────────────────────
    
    { s | STUDENT(s) AND ∃e(ENROLLMENT(e) AND e.Stud_ID = s.Stud_ID) }
          │                 └─────────────────────────────┘
          │                          Scope of ∃e
          │
          └─── s is free throughout the formula
```

---

## 8. Safe Expressions

```
┌─────────────────────────────────────────────────────────────────────┐
│                      SAFE EXPRESSIONS                                │
└─────────────────────────────────────────────────────────────────────┘

    A TRC expression is SAFE if it guarantees a FINITE result.
    
    
    UNSAFE EXPRESSION EXAMPLE:
    ───────────────────────────
    
    { t | NOT STUDENT(t) }
    
    Problem: This asks for ALL tuples that are NOT in STUDENT.
             This is an INFINITE set (all possible tuples in universe)!
    
    
    ANOTHER UNSAFE EXAMPLE:
    ────────────────────────
    
    { s.Name | s.Age > 20 }
    
    Problem: s is not restricted to any relation!
             Where do we look for tuples?
    
    
    SAFE VERSION:
    ───────────────
    
    { s.Name | STUDENT(s) AND s.Age > 20 }
    
    Now s ranges only over STUDENT relation - finite!
    
    
    SAFETY RULES:
    ──────────────
    
    ┌───────────────────────────────────────────────────────────────┐
    │                                                                │
    │  1. Every free variable must appear in a positive relation    │
    │     membership clause (e.g., STUDENT(s))                      │
    │                                                                │
    │  2. For ∃t(P(t)), t must appear in a positive relation       │
    │     membership within P                                        │
    │                                                                │
    │  3. For ∀t(P → Q), t must appear in P as relation member     │
    │                                                                │
    │  4. Result must be a subset of existing relation tuples       │
    │                                                                │
    └───────────────────────────────────────────────────────────────┘


    DOMAIN OF EXPRESSION:
    ──────────────────────
    
    The domain (dom) of an expression includes all values that 
    appear in relations or as constants in the expression.
    
    A safe expression only produces tuples with values from this domain.
```

---

## 9. TRC vs Relational Algebra Equivalence

```
┌─────────────────────────────────────────────────────────────────────┐
│              TRC ↔ RELATIONAL ALGEBRA MAPPING                        │
└─────────────────────────────────────────────────────────────────────┘


    SELECTION (σ):
    ────────────────
    
    RA: σ_condition (R)
    TRC: { t | R(t) AND condition }
    
    Example:
    RA:  σ_Age>20 (STUDENT)
    TRC: { s | STUDENT(s) AND s.Age > 20 }


    PROJECTION (π):
    ─────────────────
    
    RA: π_A1,A2 (R)
    TRC: { t.A1, t.A2 | R(t) }
    
    Example:
    RA:  π_Name,Age (STUDENT)
    TRC: { s.Name, s.Age | STUDENT(s) }


    UNION (∪):
    ───────────
    
    RA: R ∪ S
    TRC: { t | R(t) OR S(t) }


    INTERSECTION (∩):
    ───────────────────
    
    RA: R ∩ S
    TRC: { t | R(t) AND S(t) }


    DIFFERENCE (−):
    ─────────────────
    
    RA: R − S
    TRC: { t | R(t) AND NOT S(t) }


    CARTESIAN PRODUCT (×):
    ────────────────────────
    
    RA: R × S
    TRC: { t | ∃r(R(r) AND ∃s(S(s) AND t = (r, s))) }
    
    (More complex - need to combine attributes)


    NATURAL JOIN (⋈):
    ───────────────────
    
    RA: R ⋈ S (on common attribute A)
    TRC: { t | ∃r(R(r) AND ∃s(S(s) AND r.A = s.A AND 
                               t is combination of r,s)) }


    DIVISION (÷):
    ───────────────
    
    RA: R ÷ S
    TRC: { t | ∃r(R(r) AND t = r[A]) AND 
               ∀s(S(s) → ∃u(R(u) AND u[A] = t AND u[B] = s[B])) }
    
    (Division naturally maps to universal quantifier!)
```

---

## 10. Comparison Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│          RELATIONAL ALGEBRA vs TUPLE RELATIONAL CALCULUS             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ASPECT              │  ALGEBRA         │  TRC                      │
│  ────────────────────┼──────────────────┼─────────────────────────  │
│  Type                │  Procedural      │  Declarative              │
│  Specifies           │  HOW to get data │  WHAT data is needed      │
│  Operations          │  Operators       │  Formulas/Predicates      │
│  Variables           │  Relations       │  Tuple variables          │
│  Expressiveness      │  Same            │  Same                     │
│  Closer to           │  Implementation  │  Natural language         │
│  Used in             │  Query plans     │  Theory/proofs            │
│                                                                      │
│  EXAMPLE (Names of CS students):                                     │
│                                                                      │
│  ALGEBRA: π_Name (σ_Dept='CS' (STUDENT))                            │
│                                                                      │
│  TRC:     { s.Name | STUDENT(s) AND s.Dept = 'CS' }                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **TRC** | Non-procedural query language using predicates |
| **Tuple Variable** | Variable ranging over tuples of a relation |
| **∃ (Existential)** | "There exists at least one" |
| **∀ (Universal)** | "For all" |
| **Free Variable** | Not bound by any quantifier |
| **Bound Variable** | Within scope of ∃ or ∀ |
| **Safe Expression** | Guarantees finite result |
| **Implication** | P → Q ≡ NOT P OR Q |
| **Quantifier Relation** | ∀t(P) ≡ NOT ∃t(NOT P) |

---

## ❓ Quick Revision Questions

1. **What is the difference between Relational Algebra and Tuple Relational Calculus?**
   <details>
   <summary>Click for Answer</summary>
   Relational Algebra is PROCEDURAL - it specifies HOW to retrieve data using a sequence of operations. TRC is DECLARATIVE - it specifies WHAT data is needed using predicates without specifying how to get it. Both have equivalent expressive power.
   </details>

2. **Write TRC for: "Find names of students older than 21"**
   <details>
   <summary>Click for Answer</summary>
   { s.Name | STUDENT(s) AND s.Age > 21 }
   </details>

3. **What is the relationship between ∃ and ∀?**
   <details>
   <summary>Click for Answer</summary>
   ∀t(P(t)) ≡ NOT ∃t(NOT P(t)). "For all t, P is true" is equivalent to "There does not exist any t where P is false."
   </details>

4. **What makes a TRC expression "unsafe"? Give an example.**
   <details>
   <summary>Click for Answer</summary>
   An unsafe expression can produce an infinite result. Example: { t | NOT STUDENT(t) } asks for all tuples NOT in STUDENT, which is infinite. Safe expressions restrict tuple variables to range over existing relations.
   </details>

5. **Write TRC for: "Find students enrolled in ALL courses"**
   <details>
   <summary>Click for Answer</summary>
   { s | STUDENT(s) AND ∀c(COURSE(c) → ∃e(ENROLLMENT(e) AND e.Stud_ID = s.Stud_ID AND e.Course_ID = c.Course_ID)) }
   </details>

6. **What is the difference between free and bound variables?**
   <details>
   <summary>Click for Answer</summary>
   A bound variable appears within the scope of a quantifier (∃ or ∀). A free variable is not bound by any quantifier. In { s | STUDENT(s) AND ∃e(...) }, s is free and e is bound.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← Relational Algebra](02-relational-algebra.md) | [📚 Table of Contents](../README.md) | [Domain Relational Calculus →](04-domain-relational-calculus.md) |

---

*Last Updated: January 2026*
