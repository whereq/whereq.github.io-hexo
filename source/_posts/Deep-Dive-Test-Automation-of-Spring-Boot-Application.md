---
title: Deep Dive Test Automation of Spring Boot Application
date: 2024-10-27 17:57:50
categories:
- Spring Boot
- Test Automation
- Deep Dive
tags:
- Spring Boot
- Test Automation
- Deep Dive
---



Testing Spring Boot applications is crucial for validating functionality across the persistence, service, and controller layers, especially when deployed in containerized environments. This guide covers everything from configuring each layer’s tests to handling parallel endpoint testing and leveraging Docker for containerized environments.

---

- [1. Overview](#1-overview)
- [2. Testing the Persistence Layer](#2-testing-the-persistence-layer)
  - [Embedded Database for Testing](#embedded-database-for-testing)
  - [Integration Testing with Testcontainers](#integration-testing-with-testcontainers)
- [3. Testing the Service Layer](#3-testing-the-service-layer)
  - [Unit Tests with Mocks](#unit-tests-with-mocks)
  - [Integration Testing with Dependencies](#integration-testing-with-dependencies)
- [4. Testing the Controller Layer](#4-testing-the-controller-layer)
  - [MockMvc for Isolated Controller Testing](#mockmvc-for-isolated-controller-testing)
  - [Testing Real Endpoints in a Container](#testing-real-endpoints-in-a-container)
  - [Parallel Testing for API Endpoints](#parallel-testing-for-api-endpoints)
- [5. End-to-End Testing in Containerized Environments](#5-end-to-end-testing-in-containerized-environments)
  - [Using Docker Compose for Integration Testing](#using-docker-compose-for-integration-testing)
- [6. Sample Code and Configuration Summary](#6-sample-code-and-configuration-summary)
  - [Maven Dependencies](#maven-dependencies)
  - [Sample Application Properties](#sample-application-properties)
  - [Persistence Layer Test Example](#persistence-layer-test-example)
  - [Service Layer Test Example](#service-layer-test-example)
  - [Controller Layer Test Example](#controller-layer-test-example)
  - [Integration Testing with Testcontainers](#integration-testing-with-testcontainers-1)
  - [Docker Compose Configuration](#docker-compose-configuration)
  - [Running Tests in Docker](#running-tests-in-docker)

---

<a name="overview"></a>
## 1. Overview

Testing a Spring Boot application ensures each layer functions as expected and that the system performs cohesively. Tests should cover:
- **Persistence Layer**: Verifying data access and manipulation.
- **Service Layer**: Ensuring business logic functions correctly.
- **Controller Layer**: Testing API endpoints and interactions.

This article includes testing strategies for both development and containerized environments, plus configurations for parallel endpoint testing.

---

<a name="testing-the-persistence-layer"></a>
## 2. Testing the Persistence Layer

The persistence layer involves data storage and access. To test without connecting to production databases, use embedded databases for unit testing and Testcontainers for more complex integration tests.

<a name="embedded-database-for-testing"></a>
### Embedded Database for Testing

Embedded databases (like H2) are ideal for simple, fast tests:

1. **Add H2 dependency** in `pom.xml`:
    ```xml
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>
    ```

2. **Application configuration** for testing (`src/test/resources/application-test.yml`):
    ```yaml
    spring:
      datasource:
        url: jdbc:h2:mem:testdb
        driver-class-name: org.h2.Driver
        username: sa
        password:
      jpa:
        hibernate:
          ddl-auto: create-drop
        show-sql: true
    ```

3. **Persistence Layer Test**:
    ```java
    @SpringBootTest
    @Transactional
    public class UserRepositoryTest {
        @Autowired
        private UserRepository userRepository;

        @Test
        public void testSaveUser() {
            User user = new User("John Doe", "john.doe@example.com");
            User savedUser = userRepository.save(user);
            assertNotNull(savedUser.getId());
        }
    }
    ```

<a name="integration-testing-with-testcontainers"></a>
### Integration Testing with Testcontainers

For more realistic integration tests with databases (e.g., PostgreSQL), use **Testcontainers**:

1. **Add Testcontainers dependency**:
    ```xml
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <version>1.16.2</version>
        <scope>test</scope>
    </dependency>
    ```

2. **Set up Testcontainers for PostgreSQL**:
    ```java
    @SpringBootTest
    public class UserRepositoryTest {

        static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("user")
            .withPassword("password");

        @DynamicPropertySource
        static void configureProperties(DynamicPropertyRegistry registry) {
            postgres.start();
            registry.add("spring.datasource.url", postgres::getJdbcUrl);
            registry.add("spring.datasource.username", postgres::getUsername);
            registry.add("spring.datasource.password", postgres::getPassword);
        }

        @Autowired
        private UserRepository userRepository;

        @Test
        public void testFindUserById() {
            // Test logic
        }
    }
    ```

---

<a name="testing-the-service-layer"></a>
## 3. Testing the Service Layer

The service layer holds business logic. Use mocks for isolated unit tests and real dependencies for integration tests.

<a name="unit-tests-with-mocks"></a>
### Unit Tests with Mocks

1. **Define a test with `@MockBean`**:
    ```java
    @ExtendWith(MockitoExtension.class)
    public class UserServiceTest {

        @Mock
        private UserRepository userRepository;

        @InjectMocks
        private UserService userService;

        @Test
        public void testCreateUser() {
            User user = new User("Alice", "alice@example.com");
            when(userRepository.save(any(User.class))).thenReturn(user);

            User savedUser = userService.createUser(user);
            assertEquals("Alice", savedUser.getName());
        }
    }
    ```

<a name="integration-testing-with-dependencies"></a>
### Integration Testing with Dependencies

For full service layer integration:

```java
@SpringBootTest
@Transactional
public class UserServiceIntegrationTest {

    @Autowired
    private UserService userService;

    @Test
    public void testFindAllUsers() {
        List<User> users = userService.findAllUsers();
        assertNotNull(users);
    }
}
```

---

<a name="testing-the-controller-layer"></a>
## 4. Testing the Controller Layer

Controller tests validate API endpoints. `MockMvc` is used for isolated testing, while containerized setups allow testing real endpoints.

<a name="mockmvc-for-isolated-controller-testing"></a>
### MockMvc for Isolated Controller Testing

MockMvc allows API testing without starting a full web server.

```java
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    public void testGetUser() throws Exception {
        when(userService.getUserById(anyLong())).thenReturn(new User(1L, "Alice"));

        mockMvc.perform(get("/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

<a name="testing-real-endpoints-in-a-container"></a>
### Testing Real Endpoints in a Container

For real endpoint testing, use `RestTemplate` with a server running on a random port:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class UserControllerIntegrationTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void testGetUser() {
        ResponseEntity<User> response = restTemplate.getForEntity("http://localhost:" + port + "/users/1", User.class);
        assertEquals(HttpStatus.OK, response.getStatusCode());
    }
}
```

<a name="parallel-testing-for-api-endpoints"></a>
### Parallel Testing for API Endpoints

When testing multiple endpoints simultaneously, use `RANDOM_PORT` to prevent port conflicts.

---

<a name="end-to-end-testing-in-containerized-environments"></a>
## 5. End-to-End Testing in Containerized Environments

Using Docker Compose, spin up services to perform full integration tests in a controlled environment.

<a name="using-docker-compose-for-integration-testing"></a>
### Using Docker Compose for Integration Testing

1. **Docker Compose File**:
    ```yaml
    version: '3'
    services:
      app:
        image: my-spring-app:latest
        ports:
          - "8080:8080"
        environment:
          - SPRING_PROFILES_ACTIVE=test
      postgres:
        image: postgres:13
        environment:
          POSTGRES_USER: user
          POSTGRES_PASSWORD: password
          POSTGRES_DB: testdb
    ```

2. **Run and Tear Down Tests**:
    ```bash
    docker-compose up -d
    # Run integration tests
    docker-compose down
    ```

---

<a name="sample-code-and-configuration-summary"></a>
## 6. Sample Code and Configuration Summary

This section provides practical code snippets and configurations to help automate testing in Spring Boot applications. The examples cover the persistence layer, service layer, controller layer, and containerized testing.

### Maven Dependencies

Make sure to include the necessary dependencies in your `pom.xml` for testing:

```xml
<dependencies>
    <!-- Spring Boot Starter for Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- H2 Database for In-Memory Testing -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- Testcontainers for Integration Testing -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>1.16.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <version>1.16.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Sample Application Properties

Define properties for the test environment in `src/test/resources/application-test.yml`:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
```

### Persistence Layer Test Example

Example of testing a repository with an embedded H2 database:

```java
@SpringBootTest
@Transactional
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void testSaveUser() {
        User user = new User("John Doe", "john.doe@example.com");
        User savedUser = userRepository.save(user);
        assertNotNull(savedUser.getId());
    }
}
```

### Service Layer Test Example

Example of unit testing the service layer with Mockito:

```java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    public void testCreateUser() {
        User user = new User("Alice", "alice@example.com");
        when(userRepository.save(any(User.class))).thenReturn(user);

        User savedUser = userService.createUser(user);
        assertEquals("Alice", savedUser.getName());
    }
}
```

### Controller Layer Test Example

Example of testing a controller using `MockMvc`:

```java
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    public void testGetUser() throws Exception {
        when(userService.getUserById(anyLong())).thenReturn(new User(1L, "Alice"));

        mockMvc.perform(get("/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

### Integration Testing with Testcontainers

Example of a test that uses Testcontainers to spin up a PostgreSQL instance for integration tests:

```java
@SpringBootTest
public class UserRepositoryIntegrationTest {

    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("user")
            .withPassword("password");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        postgres.start();
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    public void testFindUserById() {
        User user = new User("Alice", "alice@example.com");
        userRepository.save(user);

        Optional<User> foundUser = userRepository.findById(user.getId());
        assertTrue(foundUser.isPresent());
        assertEquals("Alice", foundUser.get().getName());
    }
}
```

### Docker Compose Configuration

An example of a `docker-compose.yml` file for running your application and PostgreSQL for testing:

```yaml
version: '3.8'
services:
  app:
    image: my-spring-app:latest
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: test
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/testdb
      SPRING_DATASOURCE_USERNAME: user
      SPRING_DATASOURCE_PASSWORD: password

  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: testdb
```

### Running Tests in Docker

Run your integration tests with Docker Compose:

```bash
docker-compose up -d
# Run tests
mvn test
docker-compose down
```

This guide offers configurations and code samples to help set up automated testing for Spring Boot applications in both development and containerized environments, suitable for integration into CI/CD pipelines.