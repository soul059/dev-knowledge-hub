# Chapter 4.2: Normalization

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**Normalization** is the process of organizing database tables to minimize redundancy and dependency anomalies. It uses functional dependencies to decompose tables into smaller, well-structured relations.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand why normalization is needed                       │
│  • Learn about data anomalies                                   │
│  • Master normal forms (1NF through BCNF)                       │
│  • Apply normalization step by step                             │
│  • Understand denormalization trade-offs                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. Why Normalization?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PROBLEMS WITH BAD DESIGN                          │
└─────────────────────────────────────────────────────────────────────┘

    Consider this UNNORMALIZED table:
    
    Student_Course
    ┌─────────┬───────┬─────────┬────────────┬─────────┬───────┐
    │ Stud_ID │ Name  │ Dept_ID │ Dept_Name  │Course_ID│ Grade │
    ├─────────┼───────┼─────────┼────────────┼─────────┼───────┤
    │  S101   │ Alice │   D01   │     CS     │  CS101  │   A   │
    │  S101   │ Alice │   D01   │     CS     │  MA101  │   B   │
    │  S102   │  Bob  │   D02   │   Math     │  CS101  │   A   │
    │  S103   │ Carol │   D01   │     CS     │  MA101  │   C   │
    └─────────┴───────┴─────────┴────────────┴─────────┴───────┘
    
    
    REDUNDANCY:
    ────────────
    • Alice's name appears twice
    • D01 → CS mapping repeated
    • Same data stored multiple times = wasted space
    
    
    ANOMALIES (Problems):
    ──────────────────────

    1. INSERTION ANOMALY
    ─────────────────────
    Cannot add a new department without a student.
    Want to add D03 = "Physics"? Need a student first!
    
    
    2. UPDATE ANOMALY
    ──────────────────
    Change "CS" to "Computer Science"?
    Must update MULTIPLE rows. Miss one = inconsistency!
    
    ┌─────────┬───────┬─────────┬──────────────────┐
    │  S101   │ Alice │   D01   │ Computer Science │  Updated ✓
    │  S103   │ Carol │   D01   │        CS        │  Forgot! ✗
    └─────────┴───────┴─────────┴──────────────────┘
    Inconsistent data!
    
    
    3. DELETION ANOMALY
    ────────────────────
    Delete Bob (S102)?
    We lose the information that D02 = "Math"!
    
    
    VISUALIZATION:
    ───────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   UNNORMALIZED                    NORMALIZED                    │
    │   ────────────                    ──────────                    │
    │                                                                  │
    │   ┌───────────────────────┐       ┌─────────────┐               │
    │   │  Mixed data           │       │   Student   │               │
    │   │  Redundancy           │  →    ├─────────────┤               │
    │   │  Anomalies            │       │  Enrollment │               │
    │   └───────────────────────┘       ├─────────────┤               │
    │                                   │  Department │               │
    │                                   └─────────────┘               │
    │                                                                  │
    │   Problems                        Clean, separate tables        │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 2. Normal Forms Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                      NORMAL FORMS HIERARCHY                          │
└─────────────────────────────────────────────────────────────────────┘

    Each normal form builds on the previous one:
    
    
         ┌──────────────────────────────────────────────┐
         │               5NF (Highest)                  │
         │  ┌────────────────────────────────────────┐  │
         │  │               4NF                      │  │
         │  │  ┌──────────────────────────────────┐  │  │
         │  │  │              BCNF                │  │  │
         │  │  │  ┌────────────────────────────┐  │  │  │
         │  │  │  │           3NF              │  │  │  │
         │  │  │  │  ┌──────────────────────┐  │  │  │  │
         │  │  │  │  │        2NF           │  │  │  │  │
         │  │  │  │  │  ┌────────────────┐  │  │  │  │  │
         │  │  │  │  │  │      1NF       │  │  │  │  │  │
         │  │  │  │  │  │   (Innermost)  │  │  │  │  │  │
         │  │  │  │  │  └────────────────┘  │  │  │  │  │
         │  │  │  │  └──────────────────────┘  │  │  │  │
         │  │  │  └────────────────────────────┘  │  │  │
         │  │  └──────────────────────────────────┘  │  │
         │  └────────────────────────────────────────┘  │
         └──────────────────────────────────────────────┘
    
    
    SUMMARY:
    ─────────
    
    ┌──────┬────────────────────────────────────────────────────────┐
    │  NF  │                    Requirement                         │
    ├──────┼────────────────────────────────────────────────────────┤
    │ 1NF  │ Atomic values (no multivalued attributes)             │
    │ 2NF  │ 1NF + No partial dependencies                         │
    │ 3NF  │ 2NF + No transitive dependencies                      │
    │ BCNF │ 3NF + Every determinant is a candidate key            │
    │ 4NF  │ BCNF + No multi-valued dependencies                   │
    │ 5NF  │ 4NF + No join dependencies                            │
    └──────┴────────────────────────────────────────────────────────┘
    
    Most practical: Aim for BCNF (sometimes 3NF is acceptable)
```

---

## 3. First Normal Form (1NF)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FIRST NORMAL FORM (1NF)                           │
└─────────────────────────────────────────────────────────────────────┘

    RULE: All attributes must contain only ATOMIC (indivisible) values.
    No repeating groups, no multi-valued attributes, no nested tables.
    

    NOT IN 1NF (Violations):
    ─────────────────────────
    
    Example 1: Multi-valued attribute
    ┌─────────┬───────┬──────────────────────┐
    │ Stud_ID │ Name  │       Phones         │
    ├─────────┼───────┼──────────────────────┤
    │  S101   │ Alice │ 111-1111, 222-2222   │  ← Multiple values!
    │  S102   │  Bob  │ 333-3333             │
    └─────────┴───────┴──────────────────────┘
    
    Example 2: Repeating groups
    ┌─────────┬───────┬─────────┬─────────┬─────────┐
    │ Stud_ID │ Name  │ Course1 │ Course2 │ Course3 │
    ├─────────┼───────┼─────────┼─────────┼─────────┤
    │  S101   │ Alice │  CS101  │  MA101  │  NULL   │
    │  S102   │  Bob  │  CS101  │  NULL   │  NULL   │
    └─────────┴───────┴─────────┴─────────┴─────────┘
    

    CONVERTING TO 1NF:
    ───────────────────
    
    Solution 1: Separate rows
    ┌─────────┬───────┬──────────┐
    │ Stud_ID │ Name  │  Phone   │
    ├─────────┼───────┼──────────┤
    │  S101   │ Alice │ 111-1111 │
    │  S101   │ Alice │ 222-2222 │
    │  S102   │  Bob  │ 333-3333 │
    └─────────┴───────┴──────────┘
    
    Solution 2: Separate table (better)
    
    Student                    Student_Phone
    ┌─────────┬───────┐        ┌─────────┬──────────┐
    │ Stud_ID │ Name  │        │ Stud_ID │  Phone   │
    ├─────────┼───────┤        ├─────────┼──────────┤
    │  S101   │ Alice │        │  S101   │ 111-1111 │
    │  S102   │  Bob  │        │  S101   │ 222-2222 │
    └─────────┴───────┘        │  S102   │ 333-3333 │
                               └─────────┴──────────┘


    1NF CHECKLIST:
    ───────────────
    
    ✓ Each column contains atomic values
    ✓ No arrays, lists, or sets in columns
    ✓ No repeating column groups (Course1, Course2, Course3)
    ✓ Each row is unique (has a primary key)
    ✓ Order of rows/columns doesn't matter
```

---

## 4. Second Normal Form (2NF)

```
┌─────────────────────────────────────────────────────────────────────┐
│                   SECOND NORMAL FORM (2NF)                           │
└─────────────────────────────────────────────────────────────────────┘

    RULE: Must be in 1NF + No PARTIAL DEPENDENCIES
    
    A partial dependency exists when a non-key attribute depends on
    only PART of a composite primary key.
    

    NOT IN 2NF (Violation):
    ────────────────────────
    
    Enrollment(Stud_ID, Course_ID, Stud_Name, Course_Title, Grade)
    Primary Key: {Stud_ID, Course_ID}
    
    FDs: {Stud_ID, Course_ID} → Grade         (Full dependency ✓)
         Stud_ID → Stud_Name                   (Partial dependency ✗)
         Course_ID → Course_Title              (Partial dependency ✗)
    
    ┌─────────┬───────────┬───────────┬──────────────┬───────┐
    │ Stud_ID │ Course_ID │ Stud_Name │ Course_Title │ Grade │
    ├─────────┼───────────┼───────────┼──────────────┼───────┤
    │  S101   │   CS101   │   Alice   │   Database   │   A   │
    │  S101   │   MA101   │   Alice   │   Calculus   │   B   │ ← Alice repeated
    │  S102   │   CS101   │    Bob    │   Database   │   A   │ ← Database repeated
    └─────────┴───────────┴───────────┴──────────────┴───────┘
    
    
    VISUALIZATION OF PROBLEM:
    ──────────────────────────
    
                    Primary Key
                   ┌───────────────────┐
                   │{Stud_ID, Course_ID}│
                   └─────────┬─────────┘
                             │
           ┌─────────────────┼─────────────────┐
           ↓                 ↓                 ↓
       ┌──────────┐     ┌──────────┐     ┌──────────┐
       │Stud_Name │     │  Grade   │     │Course_   │
       │(Partial) │     │ (Full)   │     │Title     │
       │ depends  │     │ depends  │     │(Partial) │
       │on Stud_ID│     │on whole  │     │depends on│
       │ only     │     │key       │     │Course_ID │
       └──────────┘     └──────────┘     └──────────┘
            ✗                ✓                ✗


    CONVERTING TO 2NF:
    ───────────────────
    
    Decompose to remove partial dependencies:
    
    Student(Stud_ID, Stud_Name)
    ┌─────────┬───────────┐
    │ Stud_ID │ Stud_Name │
    ├─────────┼───────────┤
    │  S101   │   Alice   │
    │  S102   │    Bob    │
    └─────────┴───────────┘
    
    Course(Course_ID, Course_Title)
    ┌───────────┬──────────────┐
    │ Course_ID │ Course_Title │
    ├───────────┼──────────────┤
    │   CS101   │   Database   │
    │   MA101   │   Calculus   │
    └───────────┴──────────────┘
    
    Enrollment(Stud_ID, Course_ID, Grade)
    ┌─────────┬───────────┬───────┐
    │ Stud_ID │ Course_ID │ Grade │
    ├─────────┼───────────┼───────┤
    │  S101   │   CS101   │   A   │
    │  S101   │   MA101   │   B   │
    │  S102   │   CS101   │   A   │
    └─────────┴───────────┴───────┘
    
    No more partial dependencies!


    2NF NOTE:
    ──────────
    
    If primary key is a SINGLE attribute, table is automatically in 2NF
    (after being in 1NF). Partial dependencies only occur with
    COMPOSITE primary keys.
```

---

## 5. Third Normal Form (3NF)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THIRD NORMAL FORM (3NF)                           │
└─────────────────────────────────────────────────────────────────────┘

    RULE: Must be in 2NF + No TRANSITIVE DEPENDENCIES
    
    A transitive dependency: A → B → C (A determines C through B,
    where B is not a candidate key)
    

    NOT IN 3NF (Violation):
    ────────────────────────
    
    Student(Stud_ID, Name, Dept_ID, Dept_Name, Dept_Location)
    Primary Key: Stud_ID
    
    FDs: Stud_ID → Dept_ID                (Direct dependency ✓)
         Dept_ID → Dept_Name, Dept_Location (But Dept_ID is not a key!)
    
    Transitive: Stud_ID → Dept_ID → Dept_Name
    
    ┌─────────┬───────┬─────────┬───────────┬──────────────┐
    │ Stud_ID │ Name  │ Dept_ID │ Dept_Name │ Dept_Location│
    ├─────────┼───────┼─────────┼───────────┼──────────────┤
    │  S101   │ Alice │   D01   │    CS     │  Building A  │
    │  S102   │  Bob  │   D01   │    CS     │  Building A  │ ← Repeated!
    │  S103   │ Carol │   D02   │   Math    │  Building B  │
    └─────────┴───────┴─────────┴───────────┴──────────────┘
    
    
    VISUALIZATION:
    ───────────────
    
         Stud_ID ──────────────────────→ Dept_Name
             │                              ↑
             │                              │
             └──→ Dept_ID ─────────────────┘
                  (not a key!)
                  
         This is a TRANSITIVE dependency!


    CONVERTING TO 3NF:
    ───────────────────
    
    Decompose to remove transitive dependencies:
    
    Student(Stud_ID, Name, Dept_ID)
    ┌─────────┬───────┬─────────┐
    │ Stud_ID │ Name  │ Dept_ID │
    ├─────────┼───────┼─────────┤
    │  S101   │ Alice │   D01   │
    │  S102   │  Bob  │   D01   │
    │  S103   │ Carol │   D02   │
    └─────────┴───────┴─────────┘
    
    Department(Dept_ID, Dept_Name, Dept_Location)
    ┌─────────┬───────────┬──────────────┐
    │ Dept_ID │ Dept_Name │ Dept_Location│
    ├─────────┼───────────┼──────────────┤
    │   D01   │    CS     │  Building A  │
    │   D02   │   Math    │  Building B  │
    └─────────┴───────────┴──────────────┘
    
    Now: Stud_ID → Dept_ID (foreign key)
         Dept_ID → Dept_Name, Dept_Location (in Department table)
         
    No transitive dependencies within each table!


    3NF FORMAL DEFINITION:
    ───────────────────────
    
    For every non-trivial FD X → A:
    Either:
    - X is a superkey, OR
    - A is part of some candidate key (prime attribute)
    
    (More permissive than BCNF)
```

---

## 6. Boyce-Codd Normal Form (BCNF)

```
┌─────────────────────────────────────────────────────────────────────┐
│                   BOYCE-CODD NORMAL FORM (BCNF)                      │
└─────────────────────────────────────────────────────────────────────┘

    RULE: For EVERY non-trivial FD X → Y, X must be a superkey.
    
    Stricter than 3NF - every determinant must be a candidate key.
    

    3NF vs BCNF:
    ─────────────
    
    3NF allows: X → A where A is a prime attribute (part of key)
    BCNF: NO exceptions! X must always be a superkey.
    

    NOT IN BCNF (Violation):
    ─────────────────────────
    
    Teaching(Student, Subject, Teacher)
    Constraint: Each teacher teaches only one subject
    
    FDs: {Student, Subject} → Teacher
         Teacher → Subject            ← Teacher is NOT a superkey!
    
    Candidate Key: {Student, Subject}
    
    ┌─────────┬─────────┬─────────┐
    │ Student │ Subject │ Teacher │
    ├─────────┼─────────┼─────────┤
    │  Alice  │  Math   │  Smith  │
    │   Bob   │  Math   │  Smith  │  ← Smith → Math repeated
    │   Bob   │ Physics │  Jones  │
    │  Carol  │ Physics │  Jones  │  ← Jones → Physics repeated
    └─────────┴─────────┴─────────┘
    
    Teacher → Subject violates BCNF
    (Teacher is not a superkey of the table)


    CONVERTING TO BCNF:
    ────────────────────
    
    For each violating FD X → Y:
    1. Create new table with X and Y
    2. Remove Y from original table
    
    Original: Teaching(Student, Subject, Teacher)
    Violating FD: Teacher → Subject
    
    Decompose:
    
    Teacher_Subject(Teacher, Subject)
    ┌─────────┬─────────┐
    │ Teacher │ Subject │
    ├─────────┼─────────┤
    │  Smith  │  Math   │
    │  Jones  │ Physics │
    └─────────┴─────────┘
    
    Student_Teacher(Student, Teacher)
    ┌─────────┬─────────┐
    │ Student │ Teacher │
    ├─────────┼─────────┤
    │  Alice  │  Smith  │
    │   Bob   │  Smith  │
    │   Bob   │  Jones  │
    │  Carol  │  Jones  │
    └─────────┴─────────┘
    
    Both tables are now in BCNF!


    BCNF DECOMPOSITION ALGORITHM:
    ──────────────────────────────
    
    1. Find an FD X → Y that violates BCNF
    2. Decompose R into:
       - R1 = X ∪ Y (with key X)
       - R2 = R - Y (with key being original key or subset)
    3. Repeat for R1 and R2 until all in BCNF


    BCNF vs 3NF TRADE-OFF:
    ───────────────────────
    
    ┌────────────────┬─────────────────────┬─────────────────────┐
    │                │        3NF          │        BCNF         │
    ├────────────────┼─────────────────────┼─────────────────────┤
    │ Strictness     │ Less strict         │ More strict         │
    │ Redundancy     │ Some possible       │ No redundancy       │
    │ Dependency     │ Always preserved    │ May lose FDs        │
    │ Lossless       │ Always              │ Always              │
    │ Use when       │ Need all FDs        │ Minimize redundancy │
    └────────────────┴─────────────────────┴─────────────────────┘
```

---

## 7. Normalization Process Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                  NORMALIZATION STEP BY STEP                          │
└─────────────────────────────────────────────────────────────────────┘


    STEP 1: Achieve 1NF
    ────────────────────
    • Remove multi-valued attributes
    • Remove repeating groups
    • Identify primary key
    
    
    STEP 2: Achieve 2NF
    ────────────────────
    • Identify partial dependencies
    • Move partially dependent attributes to new table
    • Link with foreign key
    
    
    STEP 3: Achieve 3NF
    ────────────────────
    • Identify transitive dependencies
    • Move transitively dependent attributes to new table
    • Link with foreign key
    
    
    STEP 4: Achieve BCNF (optional but recommended)
    ────────────────────────────────────────────────
    • Find FDs where determinant is not a superkey
    • Decompose accordingly
    
    
    COMPLETE EXAMPLE:
    ──────────────────
    
    Original (Not normalized):
    ┌─────────┬───────┬────────────────┬─────────┬───────────┬───────────┬───────┐
    │ Stud_ID │ Name  │    Phones      │ Dept_ID │ Dept_Name │ Course_ID │ Grade │
    └─────────┴───────┴────────────────┴─────────┴───────────┴───────────┴───────┘
    
    ↓ 1NF: Remove multi-valued (Phones)
    
    ┌─────────┬───────┬─────────┬───────────┬───────────┬───────┐
    │ Stud_ID │ Name  │ Dept_ID │ Dept_Name │ Course_ID │ Grade │
    └─────────┴───────┴─────────┴───────────┴───────────┴───────┘
    + Student_Phone(Stud_ID, Phone)
    
    ↓ 2NF: Remove partial dependencies (Name, Dept_ID, Dept_Name on Stud_ID only)
    
    Student(Stud_ID, Name, Dept_ID, Dept_Name)
    Enrollment(Stud_ID, Course_ID, Grade)
    + Student_Phone
    
    ↓ 3NF: Remove transitive dependency (Dept_Name through Dept_ID)
    
    Student(Stud_ID, Name, Dept_ID)
    Department(Dept_ID, Dept_Name)
    Enrollment(Stud_ID, Course_ID, Grade)
    Student_Phone(Stud_ID, Phone)
    
    ↓ BCNF: Check all FDs - all determinants are superkeys ✓
    
    Final normalized schema!


    VISUALIZATION:
    ───────────────
    
    ┌─────────────┐          ┌─────────────┐
    │   Student   │          │  Department │
    │─────────────│          │─────────────│
    │ PK: Stud_ID │────────→│ PK: Dept_ID │
    │     Name    │          │   Dept_Name │
    │ FK: Dept_ID │          └─────────────┘
    └──────┬──────┘
           │
           │ 1:N
           ↓
    ┌─────────────┐          ┌───────────────┐
    │ Student_    │          │  Enrollment   │
    │   Phone     │          │───────────────│
    │─────────────│          │ PK: Stud_ID   │
    │PK: Stud_ID  │          │ PK: Course_ID │
    │PK: Phone    │          │     Grade     │
    └─────────────┘          └───────────────┘
```

---

## 8. Denormalization

```
┌─────────────────────────────────────────────────────────────────────┐
│                      DENORMALIZATION                                 │
└─────────────────────────────────────────────────────────────────────┘

    Sometimes we deliberately VIOLATE normal forms for PERFORMANCE.
    
    
    WHY DENORMALIZE?
    ─────────────────
    
    Normalization:                  Denormalization:
    • Many small tables             • Fewer, larger tables
    • Many joins needed             • Less joins needed
    • Slower reads                  • Faster reads
    • Faster writes                 • Slower writes
    • No redundancy                 • Controlled redundancy
    
    
    WHEN TO DENORMALIZE:
    ─────────────────────
    
    ✓ Read-heavy applications (reporting, analytics)
    ✓ Performance-critical queries
    ✓ Data that rarely changes
    ✓ When joins are too expensive
    
    ✗ Write-heavy applications
    ✗ Frequently changing data
    ✗ When data integrity is critical
    
    
    EXAMPLE:
    ─────────
    
    Normalized:
    Order(Order_ID, Customer_ID, Date)
    Customer(Customer_ID, Name, City)
    
    Query: "Get order with customer name" - requires JOIN
    
    Denormalized:
    Order(Order_ID, Customer_ID, Customer_Name, Date)
    
    Now: Customer_Name is redundant, but query is faster!
    
    
    TRADE-OFFS:
    ────────────
    
    ┌─────────────────┬──────────────────┬────────────────────┐
    │                 │   Normalized     │   Denormalized     │
    ├─────────────────┼──────────────────┼────────────────────┤
    │ Storage         │ Less             │ More               │
    │ Read speed      │ Slower (joins)   │ Faster             │
    │ Write speed     │ Faster           │ Slower (updates)   │
    │ Consistency     │ Guaranteed       │ Risk of anomalies  │
    │ Complexity      │ More tables      │ Fewer tables       │
    └─────────────────┴──────────────────┴────────────────────┘
```

---

## 📊 Summary Table

| Normal Form | Requirement | Eliminates |
|-------------|-------------|------------|
| **1NF** | Atomic values only | Multi-valued attributes |
| **2NF** | 1NF + No partial dependencies | Partial dependencies |
| **3NF** | 2NF + No transitive dependencies | Transitive dependencies |
| **BCNF** | Every determinant is a superkey | All anomalies |

| Problem | Cause | Solution |
|---------|-------|----------|
| **Insertion Anomaly** | Can't insert without related data | Normalize |
| **Update Anomaly** | Must update multiple places | Normalize |
| **Deletion Anomaly** | Lose unrelated data on delete | Normalize |
| **Redundancy** | Same data stored multiple times | Normalize |

| Dependency Type | Definition | Remove In |
|-----------------|------------|-----------|
| **Partial** | Non-key depends on part of composite key | 2NF |
| **Transitive** | Non-key depends through another non-key | 3NF |
| **Non-superkey determinant** | FD where LHS is not superkey | BCNF |

---

## ❓ Quick Revision Questions

1. **What is the purpose of normalization?**
   <details>
   <summary>Click for Answer</summary>
   To organize database tables to minimize redundancy and eliminate insertion, update, and deletion anomalies. It creates smaller, well-structured tables linked by relationships.
   </details>

2. **What is a partial dependency?**
   <details>
   <summary>Click for Answer</summary>
   When a non-key attribute depends on only PART of a composite primary key. Example: In Enrollment(Stud_ID, Course_ID, Stud_Name), Stud_Name depends only on Stud_ID, not the full key.
   </details>

3. **What is a transitive dependency?**
   <details>
   <summary>Click for Answer</summary>
   When A → B → C, meaning A determines C through B, where B is not a candidate key. Example: Stud_ID → Dept_ID → Dept_Name is transitive.
   </details>

4. **What is the difference between 3NF and BCNF?**
   <details>
   <summary>Click for Answer</summary>
   3NF allows X → A if A is a prime attribute (part of candidate key). BCNF requires X to be a superkey for ALL non-trivial FDs - no exceptions. BCNF is stricter.
   </details>

5. **When would you denormalize?**
   <details>
   <summary>Click for Answer</summary>
   When read performance is critical and data rarely changes. Common in reporting databases, data warehouses, and read-heavy applications where join overhead is too expensive.
   </details>

6. **How do you convert a table to 2NF?**
   <details>
   <summary>Click for Answer</summary>
   Identify partial dependencies (attributes depending on part of composite key). Create separate tables for each partial dependency, moving the partially dependent attributes there. Link with foreign keys.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← Functional Dependencies](01-functional-dependencies.md) | [📚 Table of Contents](../README.md) | [ER Model →](03-er-model.md) |

---

*Last Updated: January 2026*
