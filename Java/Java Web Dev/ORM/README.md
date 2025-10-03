# Object-Relational Mapping (ORM) and JPA

Object-Relational Mapping (ORM) is a programming technique that allows you to map objects in your application to tables in a relational database. This allows you to work with objects in your code, and the ORM framework will take care of translating those objects into SQL queries that can be executed against the database.

## Java Persistence API (JPA)

JPA is a Java specification that defines a standard way to perform ORM. It provides a set of interfaces and annotations that you can use to map your Java objects to database tables.

### Key Concepts in JPA

*   **Entity:** A Java class that is mapped to a database table. Each instance of an entity represents a row in the table.
*   **`@Entity` annotation:** Used to mark a class as an entity.
*   **`@Table` annotation:** Used to specify the name of the database table that an entity is mapped to.
*   **`@Id` annotation:** Used to specify the primary key of an entity.
*   **`@GeneratedValue` annotation:** Used to specify how the primary key is generated.
*   **`@Column` annotation:** Used to specify the mapping between an entity attribute and a database column.
*   **`EntityManager`:** The main interface for interacting with the persistence context. It is used to create, read, update, and delete entities.
*   **`persistence.xml`:** The configuration file for a JPA application. It defines the database connection details and other persistence-related settings.

## JPA Example

This example shows how to define a simple JPA entity and use the `EntityManager` to perform CRUD operations.

**`User.java` (the entity):**

```java
package com.example.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Getters and setters
}
```

**CRUD Operations with `EntityManager`:**

```java
// Create an EntityManagerFactory
EntityManagerFactory emf = Persistence.createEntityManagerFactory("my-persistence-unit");

// Create an EntityManager
EntityManager em = emf.createEntityManager();

// Create a new user
em.getTransaction().begin();
User user = new User();
user.setName("John Doe");
user.setEmail("john.doe@example.com");
em.persist(user);
em.getTransaction().commit();

// Find a user
User foundUser = em.find(User.class, 1L);

// Update a user
em.getTransaction().begin();
foundUser.setEmail("new.email@example.com");
em.getTransaction().commit();

// Delete a user
em.getTransaction().begin();
em.remove(foundUser);
em.getTransaction().commit();

// Close the EntityManager and EntityManagerFactory
em.close();
emf.close();
```

## Entity Relationships

JPA allows you to define relationships between entities, just like you can define relationships between tables in a database.

*   **`@OneToOne`:** A one-to-one relationship.
*   **`@OneToMany`:** A one-to-many relationship.
*   **`@ManyToOne`:** A many-to-one relationship.
*   **`@ManyToMany`:** A many-to-many relationship.

## JPQL (Java Persistence Query Language)

JPQL is an object-oriented query language that is used to query for entities in a database. It is similar to SQL, but it works with objects instead of tables.

**Example of a JPQL query:**

```java
TypedQuery<User> query = em.createQuery("SELECT u FROM User u WHERE u.name = :name", User.class);
query.setParameter("name", "John Doe");
List<User> users = query.getResultList();
```

## Benefits of using JPA

*   **Database Independence:** JPA provides a standard API for interacting with databases, so you can easily switch between different database vendors without changing your code.
*   **Reduced Boilerplate Code:** JPA handles the low-level details of JDBC, so you don't have to write as much boilerplate code.
*   **Object-Oriented:** JPA allows you to work with objects rather than SQL queries, which can make your code more readable and maintainable.

## JPA Providers

JPA is just a specification, so you need a JPA provider to implement it. Some popular JPA providers include:

*   **Hibernate:** The most popular JPA provider.
*   **EclipseLink:** The reference implementation of JPA.
*   **Apache OpenJPA:** Another open-source JPA provider.