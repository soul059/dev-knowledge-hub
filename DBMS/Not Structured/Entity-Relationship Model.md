---
tags:
  - database
  - data-modeling
  - er-model
  - database-design
  - conceptual-model
---

# Entity-Relationship Model (E-R Model)

The Entity-Relationship (E-R) model is a high-level **conceptual data model** used during the initial stages of database design. Its purpose is to represent the structure of the data as perceived by the users, focusing on real-world entities, their attributes, and the relationships between them. It provides a simple and intuitive way to describe the data requirements of an enterprise.

The E-R model is typically represented visually using **Entity-Relationship Diagrams (ERDs)**, which are powerful tools for communicating the database design.

## Core Concepts of the E-R Model

The E-R model is based on three core concepts:

### 1. Entities

* **Definition:** An entity represents a distinguishable real-world object or concept about which the organization wants to store data. It can be a person, place, object, event, or concept.
* **Entity Set:** A collection of entities of the same type. For example, the entity set "Student" contains all student entities, the entity set "Course" contains all course entities.
* **Notation in ERDs:** Represented by a **rectangle**.

* **Types of Entities:**
    * **Strong Entity:** An entity that can exist independently. It has its own unique identifier (primary key). Most entities are strong entities.
        * *Example:* Student, Course, Department.
        * *Notation in ERDs:* Standard rectangle.
    * **Weak Entity:** An entity that cannot be uniquely identified by its own attributes alone. Its existence depends on the existence of a strong entity (called the *identifying owner* or *parent entity*). The weak entity's identifier includes the identifier of the identifying owner.
        * *Example:* A "Dependent" entity in an employee database might be weak because its unique identification might require knowing the employee it depends on (e.g., Dependent Name + Employee ID). A "Section" of a course might be weak if its identification depends on the specific course (e.g., Section Number + Course ID).
        * *Notation in ERDs:* Represented by a **double rectangle**.

### 2. Attributes

* **Definition:** An attribute is a property or characteristic that describes an entity or a relationship.
* **Attribute Set:** A set of attributes describing a particular entity set or relationship set.
* **Notation in ERDs:** Represented by an **oval** connected to the entity or relationship set.

* **Types of Attributes:**
    * **Simple vs. Composite Attributes:**
        * **Simple Attribute:** An attribute that cannot be further divided into smaller components with independent meaning.
            * *Example:* `Age`, `Gender`, `CourseID`.
            * *Notation:* Single oval.
        * **Composite Attribute:** An attribute that can be divided into smaller sub-attributes, each with its own meaning.
            * *Example:* `Address` can be composed of `Street`, `City`, `State`, `Zip Code`. `Full Name` can be composed of `First Name`, `Middle Initial`, `Last Name`.
            * *Notation:* Oval connected to the entity, with lines branching to other ovals representing the component attributes.
    * **Single-valued vs. Multi-valued Attributes:**
        * **Single-valued Attribute:** An attribute that holds only one value for each entity instance.
            * *Example:* `StudentID`, `Date of Birth`.
            * *Notation:* Single oval.
        * **Multi-valued Attribute:** An attribute that can hold multiple values for a single entity instance.
            * *Example:* `Phone Number` (a person might have multiple phone numbers), `Degrees` (a person might have multiple degrees).
            * *Notation:* Represented by a **double oval**.
    * **Stored vs. Derived Attributes:**
        * **Stored Attribute:** An attribute whose value is stored directly in the database.
            * *Example:* `Date of Birth`.
        * **Derived Attribute:** An attribute whose value can be calculated or derived from other stored attributes. Derived attributes are typically not stored explicitly to avoid redundancy and potential inconsistencies.
            * *Example:* `Age` can be derived from `Date of Birth`. `Total Price` can be derived from `Quantity` and `Unit Price`.
            * *Notation:* Represented by a **dashed oval**.
    * **Key Attributes:**
        * **Key Attribute:** An attribute or a set of attributes whose values uniquely identify an entity within an entity set. In the E-R model, this corresponds to the concept of a primary key in the relational model.
        * **Notation:* The name of the key attribute is **underlined**.

### 3. Relationships

* **Definition:** A relationship represents an association or connection between two or more entities.
* **Relationship Set:** A collection of relationships of the same type. It is an association between entity sets. For example, the relationship set "Takes" is an association between the "Student" entity set and the "Course" entity set.
* **Degree of a Relationship:** The number of entity sets participating in a relationship.
    * **Binary Relationship:** Involves two entity sets (most common).
    * **Ternary Relationship:** Involves three entity sets.
    * **N-ary Relationship:** Involves n entity sets.
* **Notation in ERDs:** Represented by a **diamond** connected to the participating entity sets by lines.

* **Constraints on Relationships:** Define the rules governing how entities are related.
    * **Cardinality (Mapping Cardinality):** Specifies the number of instances of one entity that can be associated with an instance of another entity in the relationship. Common types for binary relationships:
        * **One-to-One (1:1):** Each entity in one set is related to at most one entity in the other set, and vice versa. (e.g., A Person is related to at most one Social Security Card, and a Social Security Card is related to at most one Person).
        * **One-to-Many (1:N):** An entity in one set can be related to multiple entities in the other set, but each entity in the second set is related to at most one entity in the first set. (e.g., A Department has many Employees, but an Employee belongs to at most one Department).
        * **Many-to-One (N:1):** An entity in one set can be related to at most one entity in the other set, but each entity in the second set can be related to multiple entities in the first set. (This is the inverse of 1:N, e.g., Many Employees belong to one Department).
        * **Many-to-Many (M:N):** An entity in one set can be related to multiple entities in the other set, and vice versa. (e.g., A Student takes many Courses, and a Course is taken by many Students).
        * *Notation in ERDs:* Shown as numbers (1 or *) or letters (N, M) near the connection lines between the relationship diamond and the entity rectangles. Crow's Foot notation is also widely used, using symbols to represent cardinality.
    * **Participation Constraint (Participation Role):** Specifies whether the existence of an entity depends on its being related to another entity in the relationship.
        * **Total Participation:** Every entity instance in the entity set *must* participate in the relationship.
            * *Notation in ERDs:* Represented by a **double line** connecting the entity rectangle to the relationship diamond.
            * *Example:* If every Employee *must* work for a Department, the line between Employee and Works-For relationship would be double.
        * **Partial Participation:** Entity instances in the entity set *may or may not* participate in the relationship.
            * *Notation in ERDs:* Represented by a **single line** connecting the entity rectangle to the relationship diamond.
            * *Example:* An Employee *may or may not* have a Dependent.

* **Weak Entity and Identifying Relationship:**
    * A weak entity set is existence-dependent on its identifying owner (a strong entity).
    * The relationship connecting a weak entity set to its identifying owner is called the **identifying relationship**. This relationship *must* have total participation for the weak entity.
    * *Notation in ERDs:* Represented by a **double diamond**.

## E-R Diagrams (ERDs)

ERDs are the graphical representation of the E-R model. They use specific symbols to depict entities, attributes, relationships, and their constraints.

* **Common Symbols:**
    * Rectangle: Entity Set
    * Double Rectangle: Weak Entity Set
    * Oval: Attribute
    * Underlined Oval: Key Attribute
    * Double Oval: Multi-valued Attribute
    * Dashed Oval: Derived Attribute
    * Diamond: Relationship Set
    * Double Diamond: Identifying Relationship Set
    * Line: Links attributes to entity sets or entity sets to relationship sets.
    * Double Line: Represents total participation of an entity set in a relationship set.
    * Cardinality Notation (1, N, M, * or Crow's Foot symbols): Specifies the cardinality ratio of relationship sets.

## Role of E-R Model in Database Design

The E-R model plays a vital role in the **conceptual design** phase of the database development lifecycle.

1.  **Requirements Gathering:** Helps in understanding and documenting the data requirements of the users and the enterprise.
2.  **Conceptual Design:** Provides a high-level blueprint of the database structure, independent of a specific DBMS. It clarifies entities, attributes, and relationships.
3.  **Communication:** ERDs serve as an excellent communication tool between database designers and users, ensuring that the design accurately reflects the real world.
4.  **Foundation for Logical Design:** The E-R model is then typically translated or mapped to a logical data model, most commonly the [[Relational Model]]. Rules exist for converting entities, attributes, and relationships into tables, columns, and foreign keys.

In summary, the E-R model is a powerful tool for modeling the structure of data in the real world at a conceptual level, providing the necessary foundation for building a robust and accurate database design.