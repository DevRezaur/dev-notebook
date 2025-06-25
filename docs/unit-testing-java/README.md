# Unit Testing in Java – A Practical Guide

## Table of Contents

- [Unit Testing in Java – A Practical Guide](#unit-testing-in-java--a-practical-guide)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction to Unit Testing](#1-introduction-to-unit-testing)
    - [What is Unit Testing?](#what-is-unit-testing)
    - [Benefits of Unit Testing](#benefits-of-unit-testing)
    - [Unit vs Integration vs E2E Tests](#unit-vs-integration-vs-e2e-tests)
    - [Role in CI/CD \& TDD](#role-in-cicd--tdd)
    - [Common Java Frameworks](#common-java-frameworks)
  - [2. JUnit 5 Basics](#2-junit-5-basics)
    - [What is JUnit?](#what-is-junit)
    - [Installing JUnit (Maven)](#installing-junit-maven)
    - [Writing a Basic Test Case](#writing-a-basic-test-case)
    - [Core Annotations \& Lifecycle](#core-annotations--lifecycle)
  - [3. Parameterized Tests](#3-parameterized-tests)
    - [Using @ValueSource](#using-valuesource)
    - [Using @CsvSource](#using-csvsource)
    - [Using @MethodSource](#using-methodsource)
    - [Custom Argument Providers](#custom-argument-providers)
  - [4. Assertions in Depth](#4-assertions-in-depth)
    - [Standard Assertions](#standard-assertions)
    - [Grouped Assertions](#grouped-assertions)
    - [Exception Testing](#exception-testing)
    - [Timeout Testing](#timeout-testing)
    - [Hamcrest Assertions](#hamcrest-assertions)
    - [AssertJ (Fluent API)](#assertj-fluent-api)
  - [5. Test Organization \& Best Practices](#5-test-organization--best-practices)
  - [6. Testing Exceptions \& Edge Cases](#6-testing-exceptions--edge-cases)
  - [7. Mocking \& Dependency Isolation](#7-mocking--dependency-isolation)
    - [Why Mock?](#why-mock)
    - [Mockito Setup](#mockito-setup)
    - [Creating \& Using Mocks](#creating--using-mocks)
  - [8. Mocking External Dependencies](#8-mocking-external-dependencies)
    - [Mocking External API Calls](#mocking-external-api-calls)
    - [Mocking Database Calls](#mocking-database-calls)
    - [Mocking File System Calls](#mocking-file-system-calls)
    - [General Tips](#general-tips)
  - [9. Spring Boot Test Utilities](#9-spring-boot-test-utilities)
  - [10. Advanced Testing Techniques](#10-advanced-testing-techniques)
  - [11. Test Coverage Tools](#11-test-coverage-tools)
    - [JaCoCo](#jacoco)
    - [Pitest (Mutation Testing)](#pitest-mutation-testing)
  - [12. Integrating with Build Tools](#12-integrating-with-build-tools)
  - [13. Common Testing Pitfalls](#13-common-testing-pitfalls)
  - [14. Test Reporting \& CI Integration](#14-test-reporting--ci-integration)
  - [15. Additional Resources](#15-additional-resources)

---

## 1. Introduction to Unit Testing

### What is Unit Testing?
A **unit test** verifies the smallest testable part of an application—often a single class or method—in isolation from the rest of the system.

### Benefits of Unit Testing
* **Regression safety** – catch bugs early.
* **Design feedback** – forces smaller, loosely-coupled classes.
* **Documentation** – tests show expected behaviour.
* **Enables refactoring** – fearless code changes.

### Unit vs Integration vs E2E Tests
| Type | Scope | Speed | Tools |
|------|-------|-------|-------|
| Unit | Single class/method | ⚡ milliseconds | JUnit, TestNG |
| Integration | Component interaction (DB, REST) | 🏃 seconds-minutes | SpringBootTest, Testcontainers |
| E2E | Full system incl. UI | 🐢 minutes-hours | Selenium, Cypress |

### Role in CI/CD & TDD
Unit tests are the **first gate** in a CI pipeline (`mvn test`). In **TDD** the cycle is *Red ➜ Green ➜ Refactor*: write a failing test, implement code, make tests pass, then clean up.

### Common Java Frameworks
* **JUnit 5 (Jupiter)** – de-facto standard.
* **TestNG** – alternative with flexible configuration.
* **Spock** (Groovy) – BDD style.

---

## 2. JUnit 5 Basics

### What is JUnit?
JUnit is an open-source testing framework that provides annotations, assertions, and a runner engine.

### Installing JUnit (Maven)
```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.10.0</version>
  <scope>test</scope>
</dependency>
```
For **Gradle**:
```groovy
testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
```
Enable the JUnit platform:
```groovy
test {
  useJUnitPlatform()
}
```

### Writing a Basic Test Case
```java
class Calculator {
  int add(int a, int b) { return a + b; }
}

import org.junit.jupiter.api.*;

class CalculatorTest {
  @Test
  void addsTwoNumbers() {
    Calculator calc = new Calculator();
    Assertions.assertEquals(4, calc.add(2, 2));
  }
}
```

### Core Annotations & Lifecycle
| Annotation | Purpose |
|------------|---------|
| `@Test` | Marks a test method |
| `@BeforeEach` / `@AfterEach` | Runs before/after every `@Test` |
| `@BeforeAll` / `@AfterAll` | Runs once per class (static) |
| `@Disabled` | Skip a test with an optional reason |
| `@Nested` | Group related tests in inner classes |
| `@Tag` | Mark tests for selective execution |

Example lifecycle:
```java
@BeforeAll static void initAll() {}
@BeforeEach void init() {}
@Test void testSomething() {}
@AfterEach void tearDown() {}
@AfterAll static void tearDownAll() {}
```

---

## 3. Parameterized Tests

Parameterized tests run the **same logic** with different inputs.

### Using @ValueSource
```java
@ParameterizedTest
@ValueSource(strings = {"", "   ", "\n"})
void isBlankShouldReturnTrueForBlankStrings(String input) {
  assertTrue(input.isBlank());
}
```
### Using @CsvSource
```java
@ParameterizedTest
@CsvSource({"2,3,5", "0,4,4"})
void add(int a, int b, int expected) {
  assertEquals(expected, new Calculator().add(a,b));
}
```
### Using @MethodSource
```java
static Stream<Arguments> provideNumbers() {
  return Stream.of(arguments(1,2,3), arguments(2,2,4));
}

@ParameterizedTest
@MethodSource("provideNumbers")
void addWithMethodSource(int a,int b,int expected){
  assertEquals(expected,new Calculator().add(a,b));
}
```
### Custom Argument Providers
Implement `ArgumentsProvider` to generate complex data (e.g., JSON, files).

---

## 4. Assertions in Depth

### Standard Assertions
`assertEquals`, `assertTrue`, `assertFalse`, `assertNull`, `assertNotNull`, etc.

### Grouped Assertions
```java
assertAll("address",
  () -> assertEquals("Main St", addr.street()),
  () -> assertEquals("NY", addr.city())
);
```
### Exception Testing
```java
assertThrows(IllegalArgumentException.class, () -> service.process(null));
```
### Timeout Testing
```java
assertTimeout(Duration.ofMillis(500),() -> slowService.run());
```
### Hamcrest Assertions
```java
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;

assertThat(list, hasSize(3));
```
### AssertJ (Fluent API)
```java
import static org.assertj.core.api.Assertions.*;

assertThat(actual).isNotNull().startsWith("foo");
```

---

## 5. Test Organization & Best Practices

* **Naming** – `ClassUnderTestTest` or Behaviour style `shouldDoX_whenY`.
* **Structure** – Keep one test class per production class.
* **AAA pattern** – *Arrange, Act, Assert* comment blocks.
* **No shared mutable state** between tests.
* **Coverage vs Quality** – 80% lines is useless if critical paths lack tests.

---

## 6. Testing Exceptions & Edge Cases

Use `assertThrows` and verify message:
```java
IllegalArgumentException ex = assertThrows(IllegalArgumentException.class,
    () -> userService.load(-1));
assertEquals("id must be positive", ex.getMessage());
```
Always test boundary conditions: empty lists, `null`, max/min values.

---

## 7. Mocking & Dependency Isolation

### Why Mock?
Isolate the unit under test and control side-effects.

### Mockito Setup
Maven:
```xml
<dependency>
  <groupId>org.mockito</groupId>
  <artifactId>mockito-junit-jupiter</artifactId>
  <version>5.11.0</version>
  <scope>test</scope>
</dependency>
```

### Creating & Using Mocks
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
  @Mock EmailSender sender;
  @InjectMocks UserService service;

  @Test
  void sendsWelcomeEmail() {
    when(sender.send(any())).thenReturn(true);
    service.register("bob@example.com");
    verify(sender).send(argThat(mail -> mail.to().equals("bob@example.com")));
  }
}
```
* **Spies** wrap real objects: `spy(new ArrayList<>())`.
* **Mocking void / exceptions**: `doThrow(new IOException()).when(client).connect();`
* **Static methods** (Java 11+) – add `mockito-inline` dependency and use `mockStatic()`.

---

## 8. Mocking External Dependencies

Unit tests should run in isolation, without relying on real external systems. Here's how to mock common external dependencies in Java:

### Mocking External API Calls

**Typical scenario:** Your code calls a REST API using an HTTP client (e.g., `RestTemplate`, `HttpClient`).

**Approach:**
- Use Mockito to mock the HTTP client and stub responses.
- For more complex scenarios, use [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) (from OkHttp) to simulate a real HTTP server.

**Example (Mockito):**
```java
RestTemplate restTemplate = mock(RestTemplate.class);
when(restTemplate.getForObject(anyString(), eq(String.class))).thenReturn("mocked response");

String result = myService.callExternalApi();
assertEquals("mocked response", result);
```

**Example (MockWebServer):**
```java
MockWebServer server = new MockWebServer();
server.enqueue(new MockResponse().setBody("Hello").setResponseCode(200));
server.start();

String baseUrl = server.url("/api").toString();
// Configure your HTTP client to use baseUrl

// ... run test ...

server.shutdown();
```

### Mocking Database Calls

**Typical scenario:** Your code uses JDBC, JPA, or Spring Data repositories.

**Approach:**
- Use Mockito to mock repository interfaces or DAOs.
- For integration tests, use in-memory databases (H2, HSQLDB, Testcontainers).

**Example (Mockito):**
```java
UserRepository repo = mock(UserRepository.class);
when(repo.findById(1L)).thenReturn(Optional.of(new User("bob")));

assertEquals("bob", repo.findById(1L).get().getName());
```

**Example (In-memory DB):**
```java
@DataJpaTest
class UserRepositoryTest {
  @Autowired UserRepository repo;
  // Uses H2 by default
}
```

### Mocking File System Calls

**Typical scenario:** Your code reads/writes files using `File`, `Files`, or streams.

**Approach:**
- Refactor code to accept an abstraction (e.g., `InputStream`, `Path`, or a service interface).
- Use Mockito to mock the abstraction.
- Use [Jimfs](https://github.com/google/jimfs) for an in-memory file system.

**Example (Mockito):**
```java
FileService fileService = mock(FileService.class);
when(fileService.readFile("foo.txt")).thenReturn("mocked content");

assertEquals("mocked content", fileService.readFile("foo.txt"));
```

**Example (Jimfs):**
```java
FileSystem fs = Jimfs.newFileSystem(Configuration.unix());
Path path = fs.getPath("/test.txt");
Files.write(path, "hello".getBytes());
String content = Files.readString(path);
assertEquals("hello", content);
```

### General Tips
- Always mock at the boundary of your system, not inside your business logic.
- Prefer constructor or setter injection for dependencies to make mocking easier.
- For legacy code, consider using tools like PowerMock for static/final methods, but refactor toward testable design when possible.

---

## 9. Spring Boot Test Utilities

| Annotation | Scope |
|------------|-------|
| `@SpringBootTest` | Load full application context |
| `@WebMvcTest` | MVC layer only |
| `@DataJpaTest` | Repository + H2 |

Example controller test:
```java
@WebMvcTest(controllers = GreetingController.class)
class GreetingControllerTest {
  @Autowired MockMvc mvc;

  @Test
  void returnsGreeting() throws Exception {
    mvc.perform(get("/greet")).andExpect(status().isOk())
       .andExpect(content().string(containsString("Hello")));
  }
}
```
Test DB with **Testcontainers**:
```java
@Testcontainers
class RepoTest {
  @Container static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:15");
  // Spring will pick up datasource props from container
}
```

---

## 10. Advanced Testing Techniques

* **TDD** – red/green/refactor loops.
* **BDD** – Cucumber scenarios or AssertJ fluent assertions.
* **Legacy code** – use characterization tests before refactor.
* **Data-driven tests** – external CSV/JSON feeding `@MethodSource`.
* **Custom Assertions** – extend AssertJ `AbstractAssert` for domain-specific checks.

---

## 11. Test Coverage Tools

### JaCoCo
Add Maven plugin:
```xml
<plugin>
 <groupId>org.jacoco</groupId>
 <artifactId>jacoco-maven-plugin</artifactId>
 <version>0.8.11</version>
 <executions>
  <execution><goals><goal>prepare-agent</goal></goals></execution>
  <execution><id>report</id><phase>verify</phase><goals><goal>report</goal></goals></execution>
 </executions>
</plugin>
```
View HTML in `target/site/jacoco/index.html`.

### Pitest (Mutation Testing)
Detects weak assertions by mutating bytecode.

---

## 12. Integrating with Build Tools

* **Maven**: `mvn test` or `mvn verify` (includes coverage).
* **Gradle**: `./gradlew test`.
* Fail build on test failure by default; use `-DskipTests` only in emergencies.
* HTML/XML reports generated in `target/surefire-reports` or `build/reports/tests`.

---

## 13. Common Testing Pitfalls

Unit testing is powerful, but common mistakes can undermine its value. Here's a detailed look at frequent pitfalls and how to avoid them:

### 1. Over-mocking
**Problem:** Excessive use of mocks can make tests brittle and less meaningful, as they may not reflect real-world interactions.

**Example:**
```java
// Over-mocked: testing a simple value object with mocks
MyValueObject obj = mock(MyValueObject.class);
when(obj.getValue()).thenReturn(42);
assertEquals(42, obj.getValue()); // This test is pointless
```
**Advice:**
- Only mock external dependencies (e.g., databases, HTTP clients, message brokers).
- Prefer real objects for value types and simple collaborators.

### 2. Flaky Tests
**Problem:** Tests that pass or fail unpredictably (due to timing, randomness, or shared state) erode trust in the test suite.

**Example:**
```java
@Test
void testWithSleep() throws InterruptedException {
    Thread.sleep(100); // May fail if system is slow
    assertTrue(service.isReady());
}
```
**Advice:**
- Avoid `Thread.sleep()`; use proper synchronization or test doubles.
- Isolate tests from external systems (network, filesystem).
- Reset static/shared state between tests.

### 3. Ignoring Edge Cases
**Problem:** Only testing "happy path" scenarios leaves code vulnerable to bugs in boundary or error conditions.

**Example:**
```java
@Test
void testAdd() {
    assertEquals(5, calc.add(2, 3)); // What about 0, negatives, nulls?
}
```
**Advice:**
- Always test with empty, null, min/max, and invalid inputs.
- Use parameterized tests to cover a range of cases.

### 4. Mixing Integration Logic in Unit Tests
**Problem:** Unit tests that hit real databases, web services, or file systems are slow and fragile.

**Example:**
```java
@Test
void testSavesToDatabase() {
    repo.save(new User("bob")); // Actually writes to DB
}
```
**Advice:**
- Use mocks or in-memory fakes for external dependencies.
- Reserve integration tests for a separate suite.

### 5. Poor Naming and Unclear Test Failures
**Problem:** Vague or misleading test names make it hard to diagnose failures.

**Example:**
```java
@Test
void test1() { ... }
```
**Advice:**
- Use descriptive names: `shouldThrowWhenInputIsNull`, `returnsZeroForEmptyList`.
- Include the expected outcome and scenario in the name.
- Add failure messages to assertions for clarity.

### 6. Shared Mutable State Between Tests
**Problem:** Tests that share mutable objects or static fields can interfere with each other, causing order-dependent failures.

**Example:**
```java
static List<String> sharedList = new ArrayList<>();

@Test
void testA() { sharedList.add("A"); }
@Test
void testB() { assertTrue(sharedList.isEmpty()); } // Fails if testA runs first
```
**Advice:**
- Avoid static/shared fields unless they are immutable.
- Use `@BeforeEach` to reset state before every test.

### 7. Asserting Too Much or Too Little
**Problem:**
- Too many assertions: tests become hard to read and diagnose.
- Too few: tests may pass even if the code is broken.

**Advice:**
- Focus each test on a single behavior or outcome.
- Use grouped assertions (`assertAll`) for related checks.

### 8. Not Running Tests Frequently
**Problem:** Tests that are only run before release may miss regressions introduced during development.

**Advice:**
- Run tests on every commit (locally and in CI).
- Use pre-commit hooks or IDE integration to automate.

### 9. Skipping Tests or Silencing Failures
**Problem:** Marking tests as `@Disabled` or catching and ignoring exceptions hides real issues.

**Advice:**
- Only disable tests with a clear, documented reason.
- Never catch exceptions just to make tests pass.

### 10. Relying on Test Coverage Alone
**Problem:** High coverage does not guarantee meaningful tests; code can be "covered" but not truly verified.

**Advice:**
- Focus on testing important logic and edge cases, not just lines.
- Use mutation testing (e.g., Pitest) to check test effectiveness.

---

**Summary:**
A robust test suite is readable, reliable, and focused. Avoid these pitfalls to ensure your tests provide real value and confidence in your Java codebase.

---

## 14. Test Reporting & CI Integration

* Generate **JUnit XML** via Maven Surefire (default) or Gradle.
* CI examples:
  * **GitHub Actions** – `actions/upload-artifact` + `Maven Surefire` XML.
  * **Jenkins** – **JUnit** plugin to publish results.
  * **GitLab CI** – `junit_report` artifact.
* Coverage badges: shields.io linking to Codecov or SonarCloud.

---

## 15. Additional Resources
* "JUnit 5 User Guide" – official docs.
* "Practical Unit Testing with JUnit" – Tomek Kaczanowski.
* Mockito documentation & examples.
* **Clean Code** – Chapter on Testing, Robert C. Martin. 