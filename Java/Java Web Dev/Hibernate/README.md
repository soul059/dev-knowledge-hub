# Hibernate

Hibernate is a popular open-source object-relational mapping (ORM) framework for Java. It simplifies database interactions in Java applications by mapping Java classes to database tables. This allows developers to work with database records as if they were regular Java objects, eliminating much of the boilerplate code associated with JDBC.

### Key Features of Hibernate:

*   **Object-Relational Mapping (ORM):** Hibernate automatically maps your Java objects to tables in a relational database.
*   **JPA Provider:** Hibernate is a provider of the Java Persistence API (JPA), which is a standard specification for ORM.
*   **Database Independence:** Hibernate abstracts away the specifics of the underlying database, allowing you to switch databases with minimal code changes.
*   **Hibernate Query Language (HQL):** HQL is an object-oriented query language, similar to SQL, but it works with your Java objects instead of database tables.
*   **Caching:** Hibernate uses a multi-level cache to improve performance by reducing the number of database queries.

### Getting Started with Hibernate:

To start with Hibernate, you'll typically need the following:

1.  **Hibernate Configuration:** This can be done through a `hibernate.cfg.xml` file or a `hibernate.properties` file. This file contains database connection details and other Hibernate settings.
2.  **Entity Classes:** These are your Plain Old Java Objects (POJOs) that are annotated to map to database tables. You use annotations like `@Entity` to mark a class as a persistent entity and `@Id` to specify the primary key.
3.  **Session Factory:** The `SessionFactory` is responsible for creating Hibernate `Session` objects. It's a thread-safe object that is typically created once at application startup.
4.  **Session:** A `Session` is a lightweight, single-threaded object that represents a single unit of work with the database. You use the `Session` to save, update, delete, and retrieve objects.

### Example of a Basic Hibernate Application:

1.  **Add Dependencies:** You'll need to add the `hibernate-core` dependency to your project.

    ```xml
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>5.6.0.Final</version>
    </dependency>
    ```

2.  **Create an Entity Class:**

    ```java
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.Table;

    @Entity
    @Table(name = "employees")
    public class Employee {
        @Id
        private int id;
        private String firstName;
        // getters and setters
    }
    ```

3.  **Configure Hibernate (`hibernate.cfg.xml`):**

    ```xml
    <hibernate-configuration>
        <session-factory>
            <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
            <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/test</property>
            <property name="hibernate.connection.username">root</property>
            <property name="hibernate.connection.password">password</property>
            <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
            <property name="show_sql">true</property>
            <mapping class="com.example.Employee"/>
        </session-factory>
    </hibernate-configuration>
    ```

4.  **Create a Main Class:**

    ```java
    import org.hibernate.Session;
    import org.hibernate.SessionFactory;
    import org.hibernate.cfg.Configuration;

    public class Main {
        public static void main(String[] args) {
            SessionFactory sessionFactory = new Configuration().configure().buildSessionFactory();
            Session session = sessionFactory.openSession();
            session.beginTransaction();

            Employee emp = new Employee();
            emp.setId(1);
            emp.setFirstName("John");
            session.persist(emp);

            session.getTransaction().commit();
            session.close();
            sessionFactory.close();
        }
    }
    ```
