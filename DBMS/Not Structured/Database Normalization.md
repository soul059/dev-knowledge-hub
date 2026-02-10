---
tags:
  - database
  - database-design
  - normalization
  - relational-model
  - data-integrity
  - data-redundancy
---

# Database Normalization

**Database Normalization** is a systematic process used in [[Database Design|database design]] to organize the columns (attributes) and tables (relations) of a [[Relational Model|relational database]] to reduce [[Data Redundancy]] and improve [[Data Integrity]]. Its primary goals are to eliminate update, insertion, and deletion anomalies.

Normalization was developed by E.F. Codd (the inventor of the relational model) and further refined over time.

## Purpose and Goals

* **Reduce Data Redundancy:** Avoid storing the same piece of data in multiple places, which wastes storage and leads to [[Data Inconsistency]].
* **Improve Data Integrity:** Ensure that data is consistent and accurate by preventing anomalies.
* **Eliminate Update Anomalies:** Changes to redundant data only need to be made in one place.
* **Eliminate Insertion Anomalies:** Allow adding new data without requiring the existence of unrelated data.
* **Eliminate Deletion Anomalies:** Prevent the accidental loss of non-redundant data when deleting a tuple.

Normalization involves decomposing (breaking down) a relation (table) into smaller relations until it satisfies certain criteria called **Normal Forms (NF)**.

## Functional Dependency ($A \rightarrow B$)

A core concept in normalization is the **Functional Dependency**.

* **Definition:** An attribute or set of attributes A **functionally determines** another attribute or set of attributes B if, for any two tuples in the relation, if the values of A are the same, then the values of B must also be the same.
* **Notation:** $A \rightarrow B$
* **Determinant:** The attribute(s) on the left side of the arrow (A).
* **Dependent:** The attribute(s) on the right side of the arrow (B).

* **Example:** In a `Student` table with attributes `StudentID`, `StudentName`, `Major`, `MajorHead`. If each student has only one name, major, and major head, and each major has only one major head, then:
    * `StudentID` $\rightarrow$ `StudentName` (StudentID determines StudentName)
    * `StudentID` $\rightarrow$ `Major`
    * `StudentID` $\rightarrow$ `MajorHead`
    * `Major` $\rightarrow$ `MajorHead` (Major determines MajorHead)

Understanding functional dependencies is crucial for determining the appropriate normal form of a relation and guiding the decomposition process.

## Normal Forms (NF)

Database normalization is typically achieved through a series of steps, each corresponding to a normal form. Higher normal forms have stricter requirements and reduce more types of anomalies.

### First Normal Form (1NF)

* **Definition:** A relation is in 1NF if all attribute values are **atomic** (indivisible). This means:
    * Each cell contains a single value.
    * There are no multi-valued attributes (attributes that can hold multiple values for a single entity).
    * There are no composite attributes (attributes that can be divided into smaller parts) that are treated as single attributes in the table.
    * Each column has a unique name.
    * The order of data does not matter. (These last two are inherent in the [[Relational Model]] formal definition).
* **How to Achieve:**
    * Convert composite attributes into separate simple attributes.
    * For multi-valued attributes, create a separate table for the multi-valued attribute, including the primary key of the original entity (See [[ER to Relational Mapping, Relational Model Structure, and Query Languages#Rule 3: Strong Entity Set with Multi-Valued Attributes|ER to Relational Conversion]]).

* **Example (Not in 1NF):**
    `Student(StudentID, StudentName, PhoneNumbers, Address(Street, City))`
    * `PhoneNumbers` is multi-valued.
    * `Address` is composite.

* **Example (In 1NF):**
    `Student(StudentID, StudentName)`
    `StudentPhones(StudentID, PhoneNumber)`
    `StudentAddresses(StudentID, Street, City)`
    *(Or flatten address: `Student(StudentID, StudentName, Street, City)`) *

### Second Normal Form (2NF)

* **Definition:** A relation is in 2NF if it is in 1NF **AND** every non-prime attribute is **fully functionally dependent** on the [[Relational Model#Keys|Primary Key]].
    * **Prime Attribute:** An attribute that is part of *any* [[Relational Model#Keys|Candidate Key]].
    * **Non-Prime Attribute:** An attribute that is *not* part of any candidate key.
    * **Full Functional Dependency:** An attribute B is fully functionally dependent on a composite key A if B is functionally dependent on A, but not on any proper subset of A.
* **How to Achieve:** Identify and remove **partial dependencies** (where a non-prime attribute is dependent on only a *part* of the composite primary key). Create a new table for the partially dependent attributes and the part of the key they depend on.

* **Example (In 1NF, but Not 2NF):** Consider a table `OrderDetails` with composite PK (`OrderID`, `ProductID`).
    `OrderDetails(OrderID, ProductID, ProductName, OrderAmount)`
    Functional Dependencies:
    * (`OrderID`, `ProductID`) $\rightarrow$ `ProductName` (This is likely incorrect if ProductName is only dependent on ProductID)
    * (`OrderID`, `ProductID`) $\rightarrow$ `OrderAmount` (Full dependency)
    * `ProductID` $\rightarrow$ `ProductName` (Partial dependency - ProductName depends only on ProductID, which is *part* of the PK)

* **Example (In 2NF):** Decompose `OrderDetails` into two tables:
    `OrderDetails(OrderID, ProductID, OrderAmount)` (PK: (`OrderID`, `ProductID`))
    `Products(ProductID, ProductName)` (PK: `ProductID`)
    * `ProductName` is now fully dependent on its PK `ProductID`.

### Third Normal Form (3NF)

* **Definition:** A relation is in 3NF if it is in 2NF **AND** there is **no transitive dependency** for non-prime attributes on the Primary Key.
    * **Transitive Dependency:** An indirect dependency. If $A \rightarrow B$ and $B \rightarrow C$, then $A \rightarrow C$ is a transitive dependency (provided $B$ is not a super key and $B$ is not a prime attribute). In simpler terms, a non-prime attribute (C) is dependent on another non-prime attribute (B), which is dependent on the primary key (A).
* **How to Achieve:** Identify and remove transitive dependencies. Create a new table for the transitively dependent attributes and the determinant attribute that causes the transitive dependency.

* **Example (In 2NF, but Not 3NF):** Consider a `StudentDetails` table with PK `StudentID`.
    `StudentDetails(StudentID, StudentName, Major, MajorHead)`
    Functional Dependencies:
    * `StudentID` $\rightarrow$ `StudentName`
    * `StudentID` $\rightarrow$ `Major`
    * `StudentID` $\rightarrow$ `MajorHead`
    * `Major` $\rightarrow$ `MajorHead`
    * Here, `StudentID` $\rightarrow$ `MajorHead` is a transitive dependency because `StudentID` $\rightarrow$ `Major` and `Major` $\rightarrow$ `MajorHead`, and `Major` is a non-prime attribute.

* **Example (In 3NF):** Decompose `StudentDetails` into two tables:
    `Students(StudentID, StudentName, Major)` (PK: `StudentID`)
    `Majors(Major, MajorHead)` (PK: `Major`)
    * `MajorHead` is now directly dependent on `Major`, and `Major` is directly dependent on `StudentID`. There is no transitive dependency for a non-prime attribute on the PK in either table.

### Boyce-Codd Normal Form (BCNF)

* **Definition:** A relation is in BCNF if for every non-trivial functional dependency $A \rightarrow B$, $A$ is a **[[Relational Model#Keys|Super Key]]**.
    * **Non-trivial FD:** Where B is not a subset of A.
* **Comparison with 3NF:** BCNF is a slightly stricter form than 3NF. If a relation has only one candidate key, 3NF and BCNF are equivalent. BCNF addresses a specific situation not covered by 3NF where a non-prime attribute determines a prime attribute, or where there are multiple overlapping candidate keys.
* **How to Achieve:** If a relation is in 3NF but not BCNF, it means there is a functional dependency $A \rightarrow B$ where $A$ is *not* a super key. Decompose the relation by creating a new table for A and B.

* **Example (In 3NF, but Not BCNF):** Consider a table `Enrollment` for students enrolling in courses taught by specific instructors. Assume:
    * A student enrolls in a course, which is taught by only one instructor *for that student*.
    * An instructor teaches multiple courses.
    * A course can be taught by multiple instructors (but only one for a given student enrollment instance).
    * The combination of (StudentID, CourseID) uniquely identifies an enrollment.
    * The combination of (StudentID, InstructorID) uniquely identifies an enrollment.
    * The combination of (InstructorID, CourseID) does *not* uniquely identify an enrollment (An instructor teaches many courses).
    * An instructor teaches a course, and *that* course has a specific credit amount. (Let's slightly modify the dependency based on common BCNF examples, or use the classic example: `Teach(StudentID, Course, Instructor)` where (StudentID, Course) is PK, (StudentID, Instructor) is a CK, but Instructor -%3E Course). Let's use a simpler classic: `R(A, B, C, D)` with FDs: $AB \rightarrow C$, $C \rightarrow D$, $B \rightarrow C$. (BC is not PK). $B \rightarrow C$ is the BCNF violation as B is not a Super Key.

    Let's use the standard example from many texts: `Enroll(StudentID, Course, Instructor)`
    FDs:
    * (StudentID, Course) $\rightarrow$ Instructor (Composite PK)
    * (StudentID, Instructor) $\rightarrow$ Course (Candidate Key)
    * Instructor $\rightarrow$ Course (Incorrect - an instructor teaches many courses)
    Let's use a common university example: `StudentAdvisor(StudentID, AdvisorID, Dept)` where (StudentID, AdvisorID) is CK, and StudentID -> Dept, AdvisorID -> Dept. (StudentID, AdvisorID) is PK. StudentID is CK. AdvisorID is CK. But StudentID -> Dept and AdvisorID -> Dept. Here, StudentID determines Dept, but StudentID is *not* a Super Key (it's *part* of a CK). AdvisorID determines Dept, but AdvisorID is *not* a Super Key (it's *part* of a CK). This might be 3NF but not BCNF.

    A clearer BCNF violation example: `Teach(InstructorID, Course, StudentID)`
    Assume:
    * An instructor can teach the same course multiple times to different students.
    * A student takes a course from only one instructor at a time.
    * A course has only one instructor teaching a particular student.
    * An instructor teaches many students in a course.
    FDs:
    * (InstructorID, Course) $\rightarrow$ StudentID (Assuming a unique student enrollment per instructor/course instance - seems wrong)
    * (StudentID, Course) $\rightarrow$ InstructorID (Assuming a student takes a course from one instructor) - Let's make this the PK.
    * InstructorID $\rightarrow$ Course (Incorrect - an instructor teaches multiple courses)
    Let's rethink the BCNF example provided earlier: `R(A, B, C, D)` with FDs: $AB \rightarrow C$, $C \rightarrow D$, $B \rightarrow C$. PK is $AB$. $B \rightarrow C$ is a BCNF violation because B is not a superkey.

    Let's use the `StudentDetails(StudentID, StudentName, Major, MajorHead)` example again, but add a subtlety. PK: StudentID.
    FDs: StudentID -> StudentName, StudentID -> Major, StudentID -> MajorHead, Major -> MajorHead.
    This was not in 3NF due to StudentID -> Major -> MajorHead. Decomposed to `Students(StudentID, StudentName, Major)` and `Majors(Major, MajorHead)`. Both are in 3NF.

    Now, let's consider the edge case BCNF handles: Relation `R(A, B, C)` with FDs $AB \rightarrow C$ and $C \rightarrow B$. Let $AB$ be the PK.
    - It's in 1NF (atomic).
    - It's in 2NF (no partial dependency on AB).
    - It's in 3NF (no transitive dependency on AB through a non-prime attribute; C is non-prime, depends on A or B? No. AB -> C. C -> B. This doesn't look like A -> X -> Non-Prime directly from the PK definition).
    The BCNF definition is simpler: for every non-trivial FD $A \rightarrow B$, A must be a super key.
    In R(A, B, C), with $AB \rightarrow C$ (PK) and $C \rightarrow B$:
    * $AB \rightarrow C$: $AB$ is a super key. OK for BCNF.
    * $C \rightarrow B$: $C$ is the determinant. Is $C$ a super key? No, because $C$ alone doesn't determine $A$.
    * Therefore, this relation is in 3NF but *not* in BCNF because of the FD $C \rightarrow B$ where C is not a super key.

* **Example (In BCNF):** Decompose `R(A, B, C)` into two tables:
    `S1(A, C)` (Derived from the PK and the determinant C)
    `S2(C, B)` (Derived from the violating FD $C \rightarrow B$)
    * `S1` (PK: `A`, `C`) and `S2` (PK: `C`). Now $C \rightarrow B$ is handled in `S2`, and in `S1` the only determinant is `A`, `C`.
    * This decomposition is lossless. It might *not* be dependency-preserving if the original dependency $AB \rightarrow C$ cannot be checked solely within `S1` or `S2`. (Need to join S1 and S2 to check $AB \rightarrow C$).

### Higher Normal Forms

* **Fourth Normal Form (4NF):** Addresses multi-valued dependencies.
* **Fifth Normal Form (5NF):** Addresses join dependencies.

BCNF is often considered sufficient for most business database applications. Higher normal forms are less commonly encountered in practice during initial design.

## Decomposition Properties

When normalizing a relation by breaking it into smaller relations, it's important that the decomposition has certain properties:

* **Lossless-Join Decomposition:** It must be possible to reconstruct the original relation by joining the decomposed relations. No spurious (incorrect) tuples should be generated. This is crucial for preserving the information content.
* **Dependency-Preserving Decomposition:** All functional dependencies from the original relation should be enforceable by checking constraints within the decomposed relations, without needing to perform joins. This is important for efficiently maintaining [[Data Integrity]].

Sometimes, achieving BCNF might result in a decomposition that is not dependency-preserving. In such cases, a designer might choose to remain in 3NF to ensure dependency preservation, accepting the minimal redundancy that 3NF allows but BCNF eliminates.

## Denormalization

While normalization is generally the goal during logical design, **Denormalization** is the process of intentionally introducing redundancy into a database schema, often by combining tables that were split during normalization.

* **Purpose:** Primarily done to improve query performance, especially for read-heavy workloads or complex reporting, by reducing the number of joins required to retrieve data.
* **Trade-offs:** Denormalization sacrifices some degree of [[Data Integrity]] and increases the risk of [[Data Inconsistency]] due to the re-introduced redundancy. It requires careful management, often with compensating mechanisms like triggers or controlled procedures for updates.
* **When Used:** Typically applied after normalization, based on performance testing and specific application requirements, not during the initial logical design phase.

Normalization is a fundamental technique in relational database design, balancing the goals of minimizing redundancy and ensuring data integrity against potential performance considerations, sometimes leading to a strategic denormalization step.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Relational Model]]
* [[Database Design]]
* [[Data Redundancy]]
* [[Data Integrity]]
* [[Data Inconsistency]]
* [[Functional Dependency]] (Create if needed, or use section link)
* [[Lossless Join Decomposition]] (Create if needed, or use section link)
* [[Dependency Preservation]] (Create if needed, or use section link)
* [[Primary Key]] (Link to [[Relational Model#Keys|Relational Model - Keys]])
* [[Candidate Key]] (Link to [[Relational Model#Keys|Relational Model - Keys]])
* [[Super Key]] (Link to [[Relational Model#Keys|Relational Model - Keys]])
* [[SQL Constraints]]