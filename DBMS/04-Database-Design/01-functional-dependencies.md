# Chapter 4.1: Functional Dependencies

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**Functional Dependencies (FDs)** are constraints that describe relationships between attributes in a relation. They are the foundation for database normalization and ensure data integrity by identifying how attributes depend on each other.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand what functional dependencies are                  │
│  • Identify FDs from data and requirements                      │
│  • Learn Armstrong's Axioms                                     │
│  • Compute attribute closure                                    │
│  • Find candidate keys using FDs                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. What is a Functional Dependency?

```
┌─────────────────────────────────────────────────────────────────────┐
│                  FUNCTIONAL DEPENDENCY BASICS                        │
└─────────────────────────────────────────────────────────────────────┘

    A functional dependency X → Y means:
    
    "The value of X uniquely determines the value of Y"
    
    If two tuples have the same X value, they MUST have the same Y value.
    
    
    NOTATION:
    ──────────
    
    X → Y    (X determines Y, or Y is functionally dependent on X)
    
    X = Determinant (left side)
    Y = Dependent (right side)
    
    
    EXAMPLE:
    ─────────
    
    Student(Stud_ID, Name, Age, Dept_ID)
    
    Stud_ID → Name       "Student ID determines Name"
    Stud_ID → Age        "Student ID determines Age"
    Stud_ID → Dept_ID    "Student ID determines Department"
    
    Combined: Stud_ID → Name, Age, Dept_ID
    
    
    VISUALIZATION:
    ───────────────
    
    ┌─────────┬─────────┬─────┬─────────┐
    │ Stud_ID │  Name   │ Age │ Dept_ID │
    ├─────────┼─────────┼─────┼─────────┤
    │  S101   │  Alice  │ 20  │   D01   │
    │  S102   │   Bob   │ 22  │   D02   │
    │  S101   │  Alice  │ 20  │   D01   │  ← Same Stud_ID = Same values
    └─────────┴─────────┴─────┴─────────┘
    
    Every time Stud_ID is 'S101', Name MUST be 'Alice'
    (Cannot have S101 → Alice in one row and S101 → Charlie in another)
    
    
    REAL-WORLD ANALOGY:
    ────────────────────
    
    Employee_ID → Salary
    
    Like a function in math: f(Employee_ID) = Salary
    Each employee has exactly ONE salary.
    You cannot have the same employee with different salaries.
```

---

## 2. Types of Functional Dependencies

```
┌─────────────────────────────────────────────────────────────────────┐
│              TYPES OF FUNCTIONAL DEPENDENCIES                        │
└─────────────────────────────────────────────────────────────────────┘


    1. TRIVIAL FD
    ──────────────
    
    An FD where Y is a subset of X (always true).
    
    {A, B} → A          Trivial (A is part of {A, B})
    {A, B} → B          Trivial
    {A, B} → {A, B}     Trivial
    
    NOT useful for normalization (always holds).
    
    
    2. NON-TRIVIAL FD
    ──────────────────
    
    An FD where Y is NOT a subset of X (meaningful).
    
    {A, B} → C          Non-trivial (C is not in {A, B})
    Stud_ID → Name      Non-trivial
    
    
    3. COMPLETELY NON-TRIVIAL FD
    ─────────────────────────────
    
    An FD where X and Y have no common attributes.
    
    {A, B} → C          Completely non-trivial (no overlap)
    {A, B} → {C, D}     Completely non-trivial
    {A, B} → {B, C}     Non-trivial but not completely (B overlaps)
    
    
    4. PARTIAL DEPENDENCY
    ──────────────────────
    
    Y depends on a PROPER SUBSET of X (not all of X).
    
    If {A, B} → C  but also  A → C  (C depends only on A)
    Then C is partially dependent on {A, B}
    
    Example:
    {Stud_ID, Course_ID} → Grade    Full dependency
    {Stud_ID, Course_ID} → Name     Partial (only Stud_ID → Name)
    
    
    5. TRANSITIVE DEPENDENCY
    ─────────────────────────
    
    X → Y and Y → Z, therefore X → Z (through Y)
    
    Example:
    Stud_ID → Dept_ID   (Student belongs to a department)
    Dept_ID → Dept_Name (Department has a name)
    
    Therefore: Stud_ID → Dept_Name (transitive)
    
    ┌─────────────────────────────────────────────────────────────┐
    │                                                              │
    │   Stud_ID ────→ Dept_ID ────→ Dept_Name                     │
    │       │                            ↑                        │
    │       └────────────────────────────┘                        │
    │              Transitive FD                                   │
    │                                                              │
    └─────────────────────────────────────────────────────────────┘
```

---

## 3. Identifying Functional Dependencies

```
┌─────────────────────────────────────────────────────────────────────┐
│             IDENTIFYING FUNCTIONAL DEPENDENCIES                      │
└─────────────────────────────────────────────────────────────────────┘


    FROM DATA (Limited - can only DISPROVE):
    ──────────────────────────────────────────
    
    Employee Table:
    ┌────────┬────────┬──────────┬────────┐
    │ Emp_ID │  Name  │   Dept   │ Salary │
    ├────────┼────────┼──────────┼────────┤
    │  E01   │  John  │  Sales   │  5000  │
    │  E02   │  Mary  │    IT    │  6000  │
    │  E03   │  John  │    HR    │  5500  │
    │  E04   │  Lisa  │  Sales   │  5000  │
    └────────┴────────┴──────────┴────────┘
    
    Can we conclude?
    
    Emp_ID → Name     ✓ Yes (unique Emp_IDs have unique names)
    Name → Emp_ID     ✗ No (John appears twice with E01 and E03)
    Dept → Salary     ✗ No (Sales: 5000, IT: 6000, HR: 5500 - but need more data)
    Salary → Dept     ✗ No (5000 for Sales, 5000 for Sales - but could change)
    
    ⚠️ Data can DISPROVE an FD, but cannot PROVE one (need semantics)
    
    
    FROM SEMANTICS (Domain Knowledge):
    ────────────────────────────────────
    
    FDs should be identified from business rules:
    
    "Each student has one student ID, one name, one address"
    → Stud_ID → Name, Address
    
    "Each course is offered by one department"
    → Course_ID → Dept_ID
    
    "A student can have only one grade per course"
    → {Stud_ID, Course_ID} → Grade
    
    
    PROCESS:
    ─────────
    
    1. Understand the domain/business rules
    2. Identify what attributes uniquely determine others
    3. Look for:
       - ID fields (usually determine other attributes)
       - Natural keys (unique identifiers)
       - Composite keys (combinations that are unique)
    4. Verify with sample data (to disprove, not prove)
```

---

## 4. Armstrong's Axioms

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ARMSTRONG'S AXIOMS                                │
└─────────────────────────────────────────────────────────────────────┘

    Three rules (axioms) for inferring new FDs from existing ones.
    These are SOUND (produce only valid FDs) and COMPLETE (can derive all FDs).
    

    1. REFLEXIVITY (Trivial Rule)
    ───────────────────────────────
    
    If Y ⊆ X, then X → Y
    
    Any set of attributes determines its subset.
    
    Examples:
    {A, B, C} → {A}       ✓
    {A, B, C} → {A, B}    ✓
    {A, B, C} → {A, B, C} ✓
    
    
    2. AUGMENTATION (Additivity)
    ─────────────────────────────
    
    If X → Y, then XZ → YZ (for any Z)
    
    Can add same attributes to both sides.
    
    Given: A → B
    Derive: AC → BC
           AD → BD
           ABC → BCD
    
    
    3. TRANSITIVITY
    ─────────────────
    
    If X → Y and Y → Z, then X → Z
    
    Chaining dependencies.
    
    Given: A → B and B → C
    Derive: A → C


    DERIVED RULES (from axioms):
    ──────────────────────────────
    
    4. UNION
    ─────────
    If X → Y and X → Z, then X → YZ
    
    Given: A → B and A → C
    Derive: A → BC
    
    
    5. DECOMPOSITION
    ─────────────────
    If X → YZ, then X → Y and X → Z
    
    Given: A → BC
    Derive: A → B and A → C
    
    
    6. PSEUDO-TRANSITIVITY
    ───────────────────────
    If X → Y and WY → Z, then WX → Z
    
    Given: A → B and CB → D
    Derive: CA → D


    VISUALIZATION:
    ───────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   REFLEXIVITY          AUGMENTATION          TRANSITIVITY       │
    │                                                                  │
    │   {A,B} → {A}          A → B                 A → B              │
    │   (subset)             ─────────             B → C              │
    │                        AC → BC               ─────              │
    │                        (add C)               A → C              │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 5. Attribute Closure

```
┌─────────────────────────────────────────────────────────────────────┐
│                     ATTRIBUTE CLOSURE                                │
└─────────────────────────────────────────────────────────────────────┘

    The closure of X, denoted X⁺, is the set of ALL attributes
    that can be determined by X using the given FDs.
    

    ALGORITHM:
    ───────────
    
    1. Start with X⁺ = X
    2. For each FD Y → Z:
       If Y ⊆ X⁺, then add Z to X⁺
    3. Repeat step 2 until no new attributes added
    
    
    EXAMPLE:
    ─────────
    
    Relation: R(A, B, C, D, E)
    FDs: A → B
         B → C
         C → D
         D → E
    
    Find A⁺:
    
    Step 1: A⁺ = {A}
    
    Step 2: A → B, A ⊆ {A}?  Yes → A⁺ = {A, B}
    
    Step 3: B → C, B ⊆ {A, B}?  Yes → A⁺ = {A, B, C}
    
    Step 4: C → D, C ⊆ {A, B, C}?  Yes → A⁺ = {A, B, C, D}
    
    Step 5: D → E, D ⊆ {A, B, C, D}?  Yes → A⁺ = {A, B, C, D, E}
    
    Step 6: No more FDs to apply
    
    Result: A⁺ = {A, B, C, D, E} = all attributes
    Therefore: A is a candidate key!
    
    
    VISUALIZATION:
    ───────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   Finding A⁺:                                                   │
    │                                                                  │
    │   A ────→ B ────→ C ────→ D ────→ E                            │
    │   │       ↑       ↑       ↑       ↑                            │
    │   │       │       │       │       │                            │
    │   └───────┴───────┴───────┴───────┘                            │
    │                                                                  │
    │   A⁺ = {A, B, C, D, E}                                          │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    ANOTHER EXAMPLE:
    ─────────────────
    
    Relation: R(A, B, C, D)
    FDs: A → B
         BC → D
    
    Find A⁺:
    A⁺ = {A}
    Apply A → B: A⁺ = {A, B}
    Apply BC → D: BC ⊆ {A, B}? No (C missing)
    
    Result: A⁺ = {A, B}
    
    Find {A, C}⁺:
    {A, C}⁺ = {A, C}
    Apply A → B: {A, C}⁺ = {A, B, C}
    Apply BC → D: BC ⊆ {A, B, C}? Yes → {A, C}⁺ = {A, B, C, D}
    
    Result: {A, C}⁺ = {A, B, C, D} = all attributes
    Therefore: {A, C} is a candidate key!
```

---

## 6. Finding Candidate Keys

```
┌─────────────────────────────────────────────────────────────────────┐
│                   FINDING CANDIDATE KEYS                             │
└─────────────────────────────────────────────────────────────────────┘

    A candidate key K is a minimal set of attributes where:
    1. K⁺ = all attributes (K determines everything)
    2. No proper subset of K has this property (minimal)
    

    STRATEGY:
    ──────────
    
    1. Identify attributes that appear:
       - Only on LEFT side of FDs → MUST be in every key
       - Only on RIGHT side of FDs → NEVER in any key
       - On BOTH sides → May or may not be in key
       - On NEITHER side → MUST be in every key
    
    2. Start with must-have attributes
    3. Add additional attributes until closure = all attributes
    4. Verify minimality
    
    
    EXAMPLE:
    ─────────
    
    Relation: R(A, B, C, D, E)
    FDs: A → BC
         CD → E
         B → D
    
    Step 1: Categorize attributes
    
    ┌───────────┬──────────────────────────────┐
    │ Attribute │          Appears             │
    ├───────────┼──────────────────────────────┤
    │    A      │ Left only → MUST be in key  │
    │    B      │ Both sides                   │
    │    C      │ Both sides                   │
    │    D      │ Both sides                   │
    │    E      │ Right only → NOT in key     │
    └───────────┴──────────────────────────────┘
    
    Step 2: Start with A (must be in key)
    
    Find A⁺:
    A⁺ = {A}
    Apply A → BC: A⁺ = {A, B, C}
    Apply B → D: A⁺ = {A, B, C, D}
    Apply CD → E: CD ⊆ {A, B, C, D}? Yes → A⁺ = {A, B, C, D, E}
    
    A⁺ = all attributes!
    
    Step 3: Check minimality
    A has no proper subsets (it's a single attribute)
    
    Result: A is the candidate key!


    EXAMPLE WITH MULTIPLE KEYS:
    ────────────────────────────
    
    Relation: R(A, B, C, D)
    FDs: A → B
         C → D
    
    Categorize:
    - A: Left only → Must be in key
    - C: Left only → Must be in key
    - B: Right only → Not in key
    - D: Right only → Not in key
    
    Candidate key must contain: {A, C}
    
    Find {A, C}⁺:
    {A, C}⁺ = {A, C}
    Apply A → B: {A, C}⁺ = {A, B, C}
    Apply C → D: {A, C}⁺ = {A, B, C, D}
    
    {A, C}⁺ = all attributes
    {A, C} is minimal (neither A nor C alone works)
    
    Candidate Key: {A, C}
```

---

## 7. Canonical Cover (Minimal Cover)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CANONICAL COVER                                   │
└─────────────────────────────────────────────────────────────────────┘

    A minimal/canonical cover Fc of F is a simplified set of FDs where:
    1. Fc is equivalent to F (same closure)
    2. All FDs have single attribute on right side
    3. No extraneous attributes in any FD
    4. No redundant FDs
    

    ALGORITHM:
    ───────────
    
    1. Decompose: Make RHS single attribute
       A → BC becomes A → B and A → C
    
    2. Remove extraneous attributes from LHS
       If AB → C and A → C (without B), remove B
    
    3. Remove redundant FDs
       If A → B can be derived from other FDs, remove it


    EXAMPLE:
    ─────────
    
    Given FDs:
    A → BC
    B → C
    AB → C
    
    Step 1: Decompose RHS
    A → B
    A → C
    B → C
    AB → C
    
    Step 2: Remove extraneous LHS attributes
    
    Check AB → C:
    Can we remove A? Check B → C. Yes! (already have it)
    Can we remove B? Check A → C. Yes! (already have it)
    → AB → C is redundant with simpler FDs
    Remove AB → C
    
    Remaining:
    A → B
    A → C
    B → C
    
    Step 3: Remove redundant FDs
    
    Is A → C redundant?
    Without A → C, can we derive A → C?
    A → B and B → C gives A → C by transitivity
    Yes! Remove A → C
    
    Final Canonical Cover:
    A → B
    B → C


    VISUALIZATION:
    ───────────────
    
    Original:               After Simplification:
    
    A → BC                  A → B
    B → C                   B → C
    AB → C
    
    ┌─────────────┐         ┌─────────────┐
    │  A ──→ B,C  │         │  A ──→ B    │
    │  B ──→ C    │    →    │  B ──→ C    │
    │  AB ──→ C   │         │             │
    └─────────────┘         └─────────────┘
    
    Equivalent but simpler!
```

---

## 8. Equivalence of FD Sets

```
┌─────────────────────────────────────────────────────────────────────┐
│                  EQUIVALENCE OF FD SETS                              │
└─────────────────────────────────────────────────────────────────────┘

    Two sets of FDs F and G are equivalent if:
    - Every FD in F can be derived from G
    - Every FD in G can be derived from F
    
    (They have the same closure: F⁺ = G⁺)
    

    TO CHECK EQUIVALENCE:
    ──────────────────────
    
    1. For each FD X → Y in F:
       Check if X → Y can be derived from G
       (Check if Y ⊆ X⁺ using G)
    
    2. For each FD X → Y in G:
       Check if X → Y can be derived from F
       (Check if Y ⊆ X⁺ using F)
    
    If both pass, F ≡ G
    
    
    EXAMPLE:
    ─────────
    
    F = {A → B, B → C}
    G = {A → B, A → C, B → C}
    
    Check F ⊆ G⁺:
    A → B: Is B ⊆ A⁺ using G?
           A⁺ = {A, B, C} using G. B ⊆ {A,B,C}? Yes ✓
    B → C: Is C ⊆ B⁺ using G?
           B⁺ = {B, C} using G. C ⊆ {B,C}? Yes ✓
    
    Check G ⊆ F⁺:
    A → B: Is B ⊆ A⁺ using F?
           A⁺ = {A, B, C} using F. Yes ✓
    A → C: Is C ⊆ A⁺ using F?
           A⁺ = {A, B, C} using F. Yes ✓
    B → C: Is C ⊆ B⁺ using F?
           B⁺ = {B, C} using F. Yes ✓
    
    All checks pass → F ≡ G
```

---

## 📊 Summary Table

| Term | Definition |
|------|------------|
| **Functional Dependency** | X → Y: X uniquely determines Y |
| **Trivial FD** | Y ⊆ X (always true) |
| **Partial Dependency** | Depends on subset of key |
| **Transitive Dependency** | X → Y → Z implies X → Z |
| **Attribute Closure (X⁺)** | All attributes determined by X |
| **Candidate Key** | Minimal set with closure = all attributes |
| **Canonical Cover** | Minimal equivalent set of FDs |

| Armstrong's Axiom | Rule | Example |
|-------------------|------|---------|
| **Reflexivity** | Y ⊆ X → X → Y | AB → A |
| **Augmentation** | X → Y → XZ → YZ | A → B then AC → BC |
| **Transitivity** | X → Y, Y → Z → X → Z | A → B, B → C then A → C |

| Derived Rule | Formula |
|--------------|---------|
| **Union** | X → Y, X → Z → X → YZ |
| **Decomposition** | X → YZ → X → Y, X → Z |

---

## ❓ Quick Revision Questions

1. **What does X → Y mean?**
   <details>
   <summary>Click for Answer</summary>
   X → Y means X functionally determines Y. If two tuples have the same value for X, they must have the same value for Y. Example: Stud_ID → Name means each student ID has exactly one name.
   </details>

2. **What is the difference between partial and transitive dependency?**
   <details>
   <summary>Click for Answer</summary>
   Partial: Attribute depends on only part of a composite key (AB → C but A → C). Transitive: Attribute depends through another non-key attribute (A → B → C, so A → C transitively).
   </details>

3. **What are Armstrong's three axioms?**
   <details>
   <summary>Click for Answer</summary>
   (1) Reflexivity: If Y ⊆ X then X → Y. (2) Augmentation: If X → Y then XZ → YZ. (3) Transitivity: If X → Y and Y → Z then X → Z.
   </details>

4. **How do you find if X is a candidate key?**
   <details>
   <summary>Click for Answer</summary>
   Calculate X⁺ (closure of X). If X⁺ contains all attributes in the relation, X is a superkey. If no proper subset of X also has this property, then X is a candidate key.
   </details>

5. **Given R(A,B,C) with FDs {A→B, B→C}, find A⁺.**
   <details>
   <summary>Click for Answer</summary>
   Start: A⁺ = {A}. Apply A→B: A⁺ = {A,B}. Apply B→C: A⁺ = {A,B,C}. Therefore A⁺ = {A,B,C} = all attributes, so A is a candidate key.
   </details>

6. **What is a canonical cover and why is it useful?**
   <details>
   <summary>Click for Answer</summary>
   A canonical cover is a minimal set of FDs equivalent to the original. It has single RHS attributes, no extraneous LHS attributes, and no redundant FDs. Useful for normalization and efficient constraint checking.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← DCL and TCL](../03-SQL/06-dcl-tcl.md) | [📚 Table of Contents](../README.md) | [Normalization →](02-normalization.md) |

---

*Last Updated: January 2026*
