# 📘 Phase 10 – Testing & Best Practices (Deep & Practical)

**Phase 10 is what turns “working JPA code” into “maintainable, production-safe JPA code.”**
This is the phase most teams skip — and then struggle with flaky tests, broken mappings, and leaking entities.

## 🎯 Goal of Phase 10

After this phase, you will:

* Test repositories **confidently**
* Catch mapping issues **before production**
* Use the **right database for tests**
* Design a **clean, stable data layer**
* Avoid common architectural mistakes

---

# 1️⃣ Testing Repositories (Why It Matters)

## Why Repository Tests Are Important

Repository tests verify:

* Entity mappings
* JPQL queries
* Relationships
* Constraints

They **do NOT test business logic** — only persistence correctness.

📌 If a repository test fails, your **DB interaction is broken**.

---

## What to Test

✔ Save & retrieve
✔ Queries
✔ Constraints
✔ Relationships
✔ Lazy loading behavior

---

# 2️⃣ `@DataJpaTest` (Core Tool)

## What Is `@DataJpaTest`?

A **slice test** that:

* Loads only JPA components
* Uses transactional rollback
* Configures an embedded DB by default

```java
@DataJpaTest
class UserRepositoryTest {
}
```

---

## What It Loads

✔ Entities
✔ Repositories
✔ EntityManager
❌ Controllers
❌ Services

---

## Example Repository Test

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    UserRepository userRepository;

    @Test
    void shouldSaveAndFindUser() {
        User user = new User("Ayush", "ayush@test.com");

        userRepository.save(user);

        Optional<User> found =
            userRepository.findByEmail("ayush@test.com");

        assertTrue(found.isPresent());
    }
}
```

✔ Fast
✔ Isolated
✔ Reliable

---

# 3️⃣ Testcontainers (🔥 REAL DATABASE TESTING)

## Problem with In-Memory DBs

H2 ≠ PostgreSQL ≠ MySQL

Differences:

* SQL dialect
* Index behavior
* Constraints
* JSON support
* Case sensitivity

📌 **H2 passing does not guarantee production works**

---

## What Is Testcontainers?

Testcontainers:

* Runs real DB in Docker
* Spins up per test suite
* Same DB as production

---

## Example: PostgreSQL Testcontainer

```java
@Testcontainers
@DataJpaTest
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:15");

    @DynamicPropertySource
    static void configure(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}
```

✔ Same DB behavior
✔ Catches real issues
❌ Slightly slower

---

## When to Use Which

| DB Type        | Use Case          |
| -------------- | ----------------- |
| H2             | Fast unit tests   |
| Testcontainers | Integration tests |

---

# 4️⃣ Schema Validation (Prevent Silent Breakage)

## Problem

* Entity changed
* DB schema not updated
* App starts… and breaks later

---

## Solution: Schema Validation

```properties
spring.jpa.hibernate.ddl-auto=validate
```

At startup:

* Hibernate compares entity ↔ DB schema
* Fails fast if mismatch

---

## DDL Modes Summary

| Mode     | Behavior        |
| -------- | --------------- |
| create   | Drop & recreate |
| update   | Try to alter    |
| validate | Validate only   |
| none     | Do nothing      |

📌 **Production: `validate` only**

---

# 5️⃣ Naming Strategies (Clean & Predictable DB)

## Problem

Default naming is often ugly:

```sql
user_address_street_name
```

---

## Solution: Naming Strategy

```properties
spring.jpa.hibernate.naming.physical-strategy=
org.hibernate.boot.model.naming.CamelCaseToUnderscoresNamingStrategy
```

---

## Result

| Java Field | DB Column   |
| ---------- | ----------- |
| createdAt  | created_at  |
| orderItems | order_items |

✔ Predictable
✔ Clean
✔ Standard

---

# 6️⃣ DTO vs Entity Exposure (🔥 VERY IMPORTANT)

## Problem: Exposing Entities Directly

```java
@RestController
public class UserController {

    @GetMapping("/users")
    public List<User> users() {
        return userRepository.findAll();
    }
}
```

### Why This Is Bad

❌ Lazy loading issues
❌ Infinite recursion
❌ Security risks
❌ Tight coupling

---

## Correct Approach: Use DTOs

```java
public record UserDto(Long id, String name) {}
```

```java
@GetMapping("/users")
public List<UserDto> users() {
    return userRepository.findAllUsersDto();
}
```

✔ Stable API
✔ No persistence leakage

---

## Rule of Thumb

> **Entities are internal, DTOs are external**

---

# 7️⃣ Hands-On Examples

---

## 7.1 Write a Repository Test

```java
@DataJpaTest
class OrderRepositoryTest {

    @Autowired
    OrderRepository orderRepository;

    @Test
    void shouldFindOrdersByUser() {
        Order order = new Order();
        orderRepository.save(order);

        List<Order> orders =
            orderRepository.findByUserId(1L);

        assertEquals(1, orders.size());
    }
}
```

---

## 7.2 In-Memory vs Real DB

### H2 Test (Fast)

```properties
spring.datasource.url=jdbc:h2:mem:testdb
```

### Real DB Test (Safe)

```java
@Testcontainers
class RepositoryIT { }
```

---

# 8️⃣ Best Practices Checklist (SAVE THIS)

✔ Use `@DataJpaTest`
✔ Use Testcontainers for DB-specific logic
✔ Validate schema in prod
✔ Apply naming strategy
✔ Never expose entities
✔ Test queries & mappings
✔ Keep repositories thin

---

# 🧠 Phase 10 Mental Model

> **Persistence layer should be boring, predictable, and tested**

If:

* Tests are fast
* Mappings are validated
* DTOs are used correctly

👉 Your JPA layer will not break at scale

---

# ✅ Phase 10 Outcome (ACHIEVED)

You now:
✔ Write reliable repository tests
✔ Use real DBs when needed
✔ Catch schema issues early
✔ Design clean data boundaries

---
