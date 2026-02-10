# Chapter 2.4: Domain Relational Calculus

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**Domain Relational Calculus (DRC)** is another non-procedural query language based on predicate logic. While TRC uses tuple variables, DRC uses **domain variables** that range over attribute domains (values) rather than tuples.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand the difference between TRC and DRC                │
│  • Learn domain variable concept                                 │
│  • Write DRC queries using membership conditions                │
│  • Compare DRC with QBE (Query By Example)                      │
│  • Practice converting between query languages                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. TRC vs DRC

```
┌─────────────────────────────────────────────────────────────────────┐
│                       TRC vs DRC                                     │
└─────────────────────────────────────────────────────────────────────┘

    TUPLE RELATIONAL CALCULUS (TRC)
    ─────────────────────────────────
    
    • Variables range over TUPLES
    • Access attributes using: tuple_variable.attribute_name
    • Example: { t | STUDENT(t) AND t.Age > 20 }
    
    
    DOMAIN RELATIONAL CALCULUS (DRC)
    ─────────────────────────────────
    
    • Variables range over DOMAIN VALUES (individual values)
    • Each attribute has its own domain variable
    • Example: { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND a > 20) }
    
    
    VISUALIZATION:
    ───────────────
    
    STUDENT(Stud_ID, Name, Age, Dept_ID)
    ┌────────┬─────────┬─────┬─────────┐
    │  S101  │  Alice  │  20 │   D01   │
    │  S102  │   Bob   │  22 │   D02   │
    └────────┴─────────┴─────┴─────────┘
    
    TRC View:                    DRC View:
    ┌────────────────────────┐   ┌───────────────────────────────────┐
    │   t = (S101,Alice,20,D01)  │   i=S101, n=Alice, a=20, d=D01    │
    │   whole tuple variable │   │   separate domain variables       │
    └────────────────────────┘   └───────────────────────────────────┘
```

---

## 2. DRC Basics

### General Form

```
┌─────────────────────────────────────────────────────────────────────┐
│                      DRC GENERAL FORM                                │
└─────────────────────────────────────────────────────────────────────┘

    { <x1, x2, ..., xn> | P(x1, x2, ..., xn) }
    
    WHERE:
    ──────
    x1, x2, ..., xn  = Domain variables (range over attribute values)
    P                = Predicate/condition
    <x1, ..., xn>    = Requested output values
    
    
    RELATION MEMBERSHIP:
    ─────────────────────
    
    R(x1, x2, ..., xn)
    
    Means: "There exists a tuple in R with values x1, x2, ..., xn"
    
    
    EXAMPLE:
    ─────────
    
    STUDENT(Stud_ID, Name, Age, Dept_ID)
    
    STUDENT(i, n, a, d)  means:
    "There is a student tuple with Stud_ID=i, Name=n, Age=a, Dept_ID=d"
    
    
    ORDER MATTERS:
    ───────────────
    
    Variables must appear in same order as attributes in relation schema!
    
    STUDENT(i, n, a, d) is correct
    STUDENT(n, i, d, a) is WRONG (different meaning!)
```

### Domain Variables

```
┌─────────────────────────────────────────────────────────────────────┐
│                     DOMAIN VARIABLES                                 │
└─────────────────────────────────────────────────────────────────────┘

    Each domain variable ranges over the domain of ONE attribute.
    
    
    FOR STUDENT(Stud_ID, Name, Age, Dept_ID):
    ──────────────────────────────────────────
    
    Domain Variable     Ranges Over
    ───────────────     ────────────
         i              Domain of Stud_ID (student ID values)
         n              Domain of Name (name strings)
         a              Domain of Age (integer ages)
         d              Domain of Dept_ID (department IDs)
    
    
    USING IN FORMULA:
    ──────────────────
    
    ∃i,n,a,d (STUDENT(i,n,a,d) AND a > 20)
    
    Reads as: "There exist values i, n, a, d such that
               (i,n,a,d) is a tuple in STUDENT and a > 20"
```

---

## 3. DRC Query Examples

### Sample Relations

```
    STUDENT                         ENROLLMENT
    ┌────────┬─────────┬─────┬─────────┐  ┌────────┬───────────┬───────┐
    │ Stud_ID│  Name   │ Age │ Dept_ID │  │ Stud_ID│ Course_ID │ Grade │
    ├────────┼─────────┼─────┼─────────┤  ├────────┼───────────┼───────┤
    │  S101  │  Alice  │  20 │   D01   │  │  S101  │   CS101   │   A   │
    │  S102  │   Bob   │  22 │   D02   │  │  S101  │   CS102   │   B   │
    │  S103  │ Charlie │  21 │   D01   │  │  S102  │   CS101   │   A   │
    │  S104  │  Diana  │  23 │   D03   │  │  S103  │   MA101   │   B   │
    └────────┴─────────┴─────┴─────────┘  └────────┴───────────┴───────┘
    
    COURSE                          DEPARTMENT
    ┌───────────┬────────────┐      ┌─────────┬──────────────┐
    │ Course_ID │   Title    │      │ Dept_ID │  Dept_Name   │
    ├───────────┼────────────┤      ├─────────┼──────────────┤
    │   CS101   │  Database  │      │   D01   │     CS       │
    │   CS102   │  Networks  │      │   D02   │    Math      │
    │   MA101   │  Calculus  │      │   D03   │   Physics    │
    └───────────┴────────────┘      └─────────┴──────────────┘
```

### Example Queries

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DRC EXAMPLES                                  │
└─────────────────────────────────────────────────────────────────────┘


QUERY 1: Get all student names
──────────────────────────────

    { <n> | ∃i,a,d (STUDENT(i,n,a,d)) }
    
    "Get name n where there exist values i, a, d such that
     (i, n, a, d) is in STUDENT"
    
    Result: Alice, Bob, Charlie, Diana


QUERY 2: Get names of students older than 20
─────────────────────────────────────────────

    { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND a > 20) }
    
    Result: Bob, Charlie, Diana


QUERY 3: Get names of students in CS department (D01)
─────────────────────────────────────────────────────

    { <n> | ∃i,a (STUDENT(i,n,a,'D01')) }
    
    Note: 'D01' is a constant, not a variable
    
    Result: Alice, Charlie


QUERY 4: Get student IDs and names of students enrolled in CS101
─────────────────────────────────────────────────────────────────

    { <i,n> | ∃a,d (STUDENT(i,n,a,d) AND 
                    ∃g (ENROLLMENT(i,'CS101',g))) }
    
    Result: (S101, Alice), (S102, Bob)


QUERY 5: Get names of students NOT enrolled in any course
──────────────────────────────────────────────────────────

    { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND 
                    NOT ∃c,g (ENROLLMENT(i,c,g))) }
    
    Result: Diana


QUERY 6: Get names of students enrolled in ALL courses
───────────────────────────────────────────────────────

    { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND 
                    ∀c,t (COURSE(c,t) → 
                          ∃g (ENROLLMENT(i,c,g)))) }
    
    Result: (none if no student is in all courses)


QUERY 7: Get titles of courses that have at least one enrollment
─────────────────────────────────────────────────────────────────

    { <t> | ∃c (COURSE(c,t) AND 
                ∃s,g (ENROLLMENT(s,c,g))) }
    
    Result: Database, Networks, Calculus
```

---

## 4. DRC with Multiple Relations (Joins)

```
┌─────────────────────────────────────────────────────────────────────┐
│                  DRC JOINS - STEP BY STEP                            │
└─────────────────────────────────────────────────────────────────────┘


QUERY: Get names of students with their department names
─────────────────────────────────────────────────────────

    Step 1: Identify variables needed
    
    From STUDENT: i (Stud_ID), n (Name), a (Age), d (Dept_ID)
    From DEPARTMENT: d' (Dept_ID), dn (Dept_Name)
    
    Step 2: Write the join condition
    
    d = d' (common attribute)
    
    Step 3: Complete query
    
    { <n, dn> | ∃i,a,d (STUDENT(i,n,a,d) AND 
                        ∃dn (DEPARTMENT(d,dn))) }
    
    OR (using same variable d for join):
    
    { <n, dn> | ∃i,a,d (STUDENT(i,n,a,d) AND DEPARTMENT(d,dn)) }


VISUALIZATION:
───────────────

    STUDENT                    DEPARTMENT
    ┌────┬───────┬────┬─────┐  ┌─────┬──────────┐
    │ i  │   n   │ a  │  d  │  │  d  │    dn    │
    ├────┼───────┼────┼─────┤  ├─────┼──────────┤
    │S101│ Alice │ 20 │ D01 │──│ D01 │    CS    │
    │S102│  Bob  │ 22 │ D02 │──│ D02 │   Math   │
    │S103│Charlie│ 21 │ D01 │──│ D01 │    CS    │
    │S104│ Diana │ 23 │ D03 │──│ D03 │  Physics │
    └────┴───────┴────┴─────┘  └─────┴──────────┘
                    │              │
                    └──────────────┘
                    Join on Dept_ID (d)

    RESULT: <n, dn>
    ┌─────────┬──────────┐
    │    n    │    dn    │
    ├─────────┼──────────┤
    │  Alice  │    CS    │
    │   Bob   │   Math   │
    │ Charlie │    CS    │
    │  Diana  │ Physics  │
    └─────────┴──────────┘
```

---

## 5. Complex DRC Query Example

```
┌─────────────────────────────────────────────────────────────────────┐
│              COMPLEX QUERY: Multi-Relation Join                      │
└─────────────────────────────────────────────────────────────────────┘


QUERY: Find names of students who got grade 'A' in 'Database' course
─────────────────────────────────────────────────────────────────────

    RELATIONS INVOLVED:
    • STUDENT(Stud_ID, Name, Age, Dept_ID)
    • ENROLLMENT(Stud_ID, Course_ID, Grade)  
    • COURSE(Course_ID, Title)

    
    STEP-BY-STEP:
    
    1. Need: Student Name where Grade='A' and Title='Database'
    
    2. Variables:
       - STUDENT: i, n, a, d
       - ENROLLMENT: i', c, g
       - COURSE: c', t
       
    3. Join conditions:
       - i = i' (link STUDENT to ENROLLMENT)
       - c = c' (link ENROLLMENT to COURSE)
       
    4. Selection conditions:
       - g = 'A'
       - t = 'Database'


    DRC QUERY:
    ───────────
    
    { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND 
                    ∃c (ENROLLMENT(i,c,'A') AND 
                        COURSE(c,'Database'))) }
    
    OR more explicitly:
    
    { <n> | ∃i,a,d,c,g,t (STUDENT(i,n,a,d) AND 
                          ENROLLMENT(i,c,g) AND 
                          COURSE(c,t) AND 
                          g = 'A' AND 
                          t = 'Database') }


    TRC EQUIVALENT:
    ────────────────
    
    { s.Name | STUDENT(s) AND 
               ∃e(ENROLLMENT(e) AND 
                  e.Stud_ID = s.Stud_ID AND 
                  e.Grade = 'A' AND 
                  ∃c(COURSE(c) AND 
                     c.Course_ID = e.Course_ID AND 
                     c.Title = 'Database')) }
```

---

## 6. DRC and Query By Example (QBE)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DRC AND QBE CONNECTION                            │
└─────────────────────────────────────────────────────────────────────┘

    QBE is based on DRC principles!
    
    
    QUERY: Get names of students in CS department
    ───────────────────────────────────────────────
    
    DRC:
    { <n> | ∃i,a (STUDENT(i,n,a,'D01')) }
    
    
    QBE:
    ┌──────────┬──────────┬─────┬──────────┐
    │ Stud_ID  │   Name   │ Age │ Dept_ID  │
    ├──────────┼──────────┼─────┼──────────┤
    │          │   P.     │     │   D01    │
    └──────────┴──────────┴─────┴──────────┘
    
    • P. means "print this column" (like selecting n in DRC)
    • D01 is a condition (like the constant 'D01' in DRC)
    • Empty cells are "don't care" (like ∃ for those variables)
    
    
    MAPPING:
    ─────────
    
    ┌────────────────────────────────────────────────────────────────┐
    │  DRC Concept          │  QBE Equivalent                       │
    ├────────────────────────────────────────────────────────────────┤
    │  Domain variable      │  Column in table skeleton             │
    │  Output variables     │  P. (Print) marker                    │
    │  ∃ (exists)           │  Empty cell or _variable              │
    │  Constant             │  Explicit value in cell               │
    │  Condition            │  Value or comparison in cell          │
    │  Join                 │  Same example variable in cells       │
    └────────────────────────────────────────────────────────────────┘
    
    
    QBE JOIN EXAMPLE:
    ──────────────────
    
    "Find names of students with their department names"
    
    STUDENT:
    ┌──────────┬──────────┬─────┬──────────┐
    │ Stud_ID  │   Name   │ Age │ Dept_ID  │
    ├──────────┼──────────┼─────┼──────────┤
    │          │   P.     │     │   _d     │  ← _d is example variable
    └──────────┴──────────┴─────┴──────────┘
    
    DEPARTMENT:
    ┌──────────┬──────────────┐
    │ Dept_ID  │  Dept_Name   │
    ├──────────┼──────────────┤
    │   _d     │     P.       │  ← Same _d creates join
    └──────────┴──────────────┘
```

---

## 7. TRC vs DRC Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TRC vs DRC COMPARISON                             │
└─────────────────────────────────────────────────────────────────────┘


    SAME QUERY IN BOTH:
    ────────────────────
    
    "Get names of students older than 20"
    
    TRC: { s.Name | STUDENT(s) AND s.Age > 20 }
    
    DRC: { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND a > 20) }
    
    
    KEY DIFFERENCES:
    ─────────────────

    ┌────────────────────┬───────────────────────────────────────────┐
    │       Aspect       │          TRC          │       DRC         │
    ├────────────────────┼───────────────────────┼───────────────────┤
    │ Variable type      │ Tuple variables       │ Domain variables  │
    │ What ranges        │ Entire tuples         │ Individual values │
    │ Attribute access   │ t.AttributeName       │ Positional (i,n,a)│
    │ Notation           │ { t | P(t) }          │ { <x,y> | P }     │
    │ Verbosity          │ More concise          │ More verbose      │
    │ Relation membership│ R(t)                  │ R(x1,x2,...,xn)   │
    │ Basis for          │ SQL (conceptually)    │ QBE               │
    └────────────────────┴───────────────────────┴───────────────────┘
    
    
    EXPRESSIVENESS:
    ────────────────
    
    TRC ≡ DRC ≡ Relational Algebra
    
    All three have the SAME expressive power!
    (For safe expressions)
```

---

## 8. Converting Between TRC and DRC

```
┌─────────────────────────────────────────────────────────────────────┐
│                   TRC ↔ DRC CONVERSION                               │
└─────────────────────────────────────────────────────────────────────┘


    TRC TO DRC:
    ────────────
    
    1. Replace tuple variable with domain variables
    2. Replace t.Attr with corresponding domain variable
    3. Replace R(t) with R(x1, x2, ..., xn)
    
    EXAMPLE:
    TRC: { t.Name | STUDENT(t) AND t.Age > 20 }
    DRC: { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND a > 20) }
    
    Mapping:
    t.Stud_ID → i
    t.Name    → n  (this is our output)
    t.Age     → a
    t.Dept_ID → d


    DRC TO TRC:
    ────────────
    
    1. Introduce a tuple variable t
    2. Replace domain variables with t.Attr
    3. Replace R(x1,...) with R(t)
    
    EXAMPLE:
    DRC: { <n,d> | ∃i,a (STUDENT(i,n,a,d)) }
    TRC: { t.Name, t.Dept_ID | STUDENT(t) }


    COMPLEX CONVERSION:
    ────────────────────
    
    "Students in all courses"
    
    DRC:
    { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND 
                    ∀c,t (COURSE(c,t) → 
                          ∃g (ENROLLMENT(i,c,g)))) }
    
    TRC:
    { s.Name | STUDENT(s) AND 
               ∀c (COURSE(c) → 
                   ∃e (ENROLLMENT(e) AND 
                       e.Stud_ID = s.Stud_ID AND 
                       e.Course_ID = c.Course_ID)) }
```

---

## 9. Safe DRC Expressions

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SAFE DRC EXPRESSIONS                              │
└─────────────────────────────────────────────────────────────────────┘

    Same principle as TRC: must guarantee FINITE result
    
    
    UNSAFE EXAMPLE:
    ────────────────
    
    { <x> | NOT ∃y (R(x,y)) }
    
    Problem: x can be any value not appearing in first column of R
             This is potentially infinite!
    
    
    SAFE VERSION:
    ──────────────
    
    { <x> | ∃y (S(x,y)) AND NOT ∃z (R(x,z)) }
    
    Now x must come from relation S (finite)
    
    
    SAFETY RULES FOR DRC:
    ──────────────────────
    
    1. All free variables in output must appear in a positive
       relation membership in the formula
       
    2. Existentially quantified variables must appear in
       positive relation membership
       
    3. For ∀x(P → Q), x must appear in P as part of
       relation membership
       
    4. Domain of expression = all values appearing in
       relations or as constants
```

---

## 10. Relational Completeness

```
┌─────────────────────────────────────────────────────────────────────┐
│                   RELATIONAL COMPLETENESS                            │
└─────────────────────────────────────────────────────────────────────┘

    A query language is RELATIONALLY COMPLETE if it can express
    any query that can be expressed in relational algebra.
    
    
    ┌───────────────────────────────────────────────────────────────┐
    │                                                                │
    │   RELATIONAL ALGEBRA ≡ TRC ≡ DRC                              │
    │                                                                │
    │   All three are RELATIONALLY COMPLETE and have the            │
    │   SAME expressive power (for safe expressions)                │
    │                                                                │
    └───────────────────────────────────────────────────────────────┘
    
    
    SQL AND RELATIONAL COMPLETENESS:
    ─────────────────────────────────
    
    • SQL is at least as expressive as relational algebra
    • SQL actually has MORE features (aggregation, ordering, etc.)
    • SQL is therefore a SUPERSET of relational algebra
    
    
                    ┌─────────────────────────────────────┐
                    │              SQL                    │
                    │   ┌─────────────────────────────┐   │
                    │   │                             │   │
                    │   │    Relational Algebra      │   │
                    │   │         ≡ TRC              │   │
                    │   │         ≡ DRC              │   │
                    │   │                             │   │
                    │   └─────────────────────────────┘   │
                    │                                     │
                    │   Additional SQL features:          │
                    │   • Aggregation (SUM, AVG, etc.)   │
                    │   • Ordering (ORDER BY)            │
                    │   • Grouping (GROUP BY)            │
                    │   • NULL handling                  │
                    │   • Subqueries                     │
                    │                                     │
                    └─────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | TRC | DRC |
|---------|-----|-----|
| **Variable Type** | Tuple variable | Domain variable |
| **Ranges Over** | Entire tuples | Individual values |
| **Syntax** | { t \| P(t) } | { \<x,y\> \| P } |
| **Relation Check** | R(t) | R(x1,x2,...,xn) |
| **Attribute Access** | t.Attr | Positional variable |
| **Output** | t.Attr1, t.Attr2 | \<x1, x2\> |
| **Basis For** | SQL concept | QBE |
| **Expressive Power** | Same | Same |

| DRC Element | Description |
|-------------|-------------|
| **Domain Variable** | Variable ranging over domain values |
| **\<x,y,...\>** | Output specification |
| **R(x,y,z)** | Tuple (x,y,z) belongs to relation R |
| **∃x** | There exists value x |
| **∀x** | For all values x |
| **Safe Expression** | Guarantees finite result |

---

## ❓ Quick Revision Questions

1. **What is the main difference between TRC and DRC?**
   <details>
   <summary>Click for Answer</summary>
   TRC uses tuple variables that range over entire tuples. DRC uses domain variables that range over individual attribute values. TRC: { t | STUDENT(t) AND t.Age > 20 }. DRC: { <n> | ∃i,a,d (STUDENT(i,n,a,d) AND a > 20) }
   </details>

2. **Write DRC for: "Get Stud_ID and Name of students in department D01"**
   <details>
   <summary>Click for Answer</summary>
   { <i, n> | ∃a (STUDENT(i, n, a, 'D01')) }
   </details>

3. **How is QBE related to DRC?**
   <details>
   <summary>Click for Answer</summary>
   QBE is based on DRC principles. In QBE, columns correspond to domain variables, P. marks output variables (like listing in DRC output), empty cells represent existential quantification, and example variables (_x) create joins like shared domain variables in DRC.
   </details>

4. **Convert to DRC: { t.Name | STUDENT(t) AND t.Dept_ID = 'D01' }**
   <details>
   <summary>Click for Answer</summary>
   { <n> | ∃i, a (STUDENT(i, n, a, 'D01')) }
   </details>

5. **What does STUDENT(i, n, a, d) mean in DRC?**
   <details>
   <summary>Click for Answer</summary>
   It means there exists a tuple in the STUDENT relation with Stud_ID = i, Name = n, Age = a, and Dept_ID = d. The order of variables must match the attribute order in the relation schema.
   </details>

6. **What is relational completeness?**
   <details>
   <summary>Click for Answer</summary>
   A query language is relationally complete if it can express any query expressible in relational algebra. TRC, DRC, and SQL are all relationally complete. SQL is actually more powerful as it includes additional features like aggregation and ordering.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← Tuple Relational Calculus](03-tuple-relational-calculus.md) | [📚 Table of Contents](../README.md) | [Unit 3: DDL - Data Definition Language →](../03-SQL/01-ddl-data-definition-language.md) |

---

*End of Unit 2: Relational Model*

---

*Last Updated: January 2026*
