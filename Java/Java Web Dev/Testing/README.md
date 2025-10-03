# Testing

Testing is a critical part of the software development process. It helps to ensure that your application is working correctly, that it meets the requirements of the users, and that it is free of bugs.

## The Testing Pyramid

The testing pyramid is a model that helps to guide the testing strategy for a software project. It is made up of three layers:

*   **Unit Tests:** These are the foundation of the pyramid. They are small, fast tests that test individual units of code in isolation.
*   **Integration Tests:** These tests sit in the middle of the pyramid. They test how different parts of your application work together.
*   **End-to-End (E2E) Tests:** These tests are at the top of the pyramid. They test the entire application, from the user interface to the database.

The testing pyramid suggests that you should have a lot of unit tests, a smaller number of integration tests, and a very small number of E2E tests.

## Unit Testing with JUnit

JUnit is the most popular unit testing framework for Java.

### JUnit Annotations

*   `@Test`: Marks a method as a test method.
*   `@BeforeEach`: Executed before each test method.
*   `@AfterEach`: Executed after each test method.
*   `@BeforeAll`: Executed once, before any of the test methods in the class.
*   `@AfterAll`: Executed once, after all of the test methods in the class.

### JUnit Assertions

*   `assertEquals(expected, actual)`: Asserts that two values are equal.
*   `assertNotEquals(unexpected, actual)`: Asserts that two values are not equal.
*   `assertTrue(condition)`: Asserts that a condition is true.
*   `assertFalse(condition)`: Asserts that a condition is false.
*   `assertNull(object)`: Asserts that an object is null.
*   `assertNotNull(object)`: Asserts that an object is not null.

### JUnit Example

```java
package com.example;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class CalculatorTest {

    @Test
    public void testAdd() {
        Calculator calculator = new Calculator();
        int result = calculator.add(2, 3);
        assertEquals(5, result);
    }
}
```

## Mocking with Mockito

Mockito is a popular mocking framework for Java. It allows you to create mock objects that you can use to isolate the code that you are testing.

### Stubbing and Verification

*   **Stubbing:** Telling a mock object how to behave when a certain method is called.
*   **Verification:** Checking that a certain method was called on a mock object.

### Mockito Example

```java
package com.example;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
public class LoginServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private LoginService loginService;

    @Test
    public void testLogin() {
        // Stubbing
        User user = new User("testuser", "password");
        when(userRepository.findByUsername("testuser")).thenReturn(user);

        // Calling the method to be tested
        boolean result = loginService.login("testuser", "password");

        // Verification
        assertTrue(result);
    }
}
```

## Integration Testing with Spring Boot

Spring Boot provides excellent support for integration testing.

### Testing Slices

Spring Boot provides a number of testing slices that you can use to test different parts of your application in isolation.

*   `@WebMvcTest`: For testing the web layer (controllers).
*   `@DataJpaTest`: For testing the persistence layer (repositories).
*   `@RestClientTest`: For testing REST clients.

### `MockMvc`

`MockMvc` is a class that allows you to test your controllers without needing to start a full HTTP server.

### Spring Boot Test Example

```java
package com.example;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void helloShouldReturnDefaultMessage() throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("Hello, World!"));
    }
}
```