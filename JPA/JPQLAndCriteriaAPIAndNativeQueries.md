# 📘 Phase 6 – JPQL, Criteria API & Native Queries

**Phase 6 is where you stop “guessing repository methods” and start writing intentional, efficient queries**.
Most performance and correctness issues in JPA come from **poor querying**, not mappings.

I’ll explain everything **step-by-step**, in **simple language**, with **real examples**, and **why/when to use each approach**.

## 🎯 Goal of Phase 6

After this phase, you should be able to:

* Decide **JPQL vs Criteria vs Native SQL**
* Write **joins and fetch joins confidently**
* Avoid **N+1 issues**
* Return **DTOs instead of entities**
* Write **dynamic queries cleanly**

---

# 1️⃣ JPQL (Java Persistence Query Language)

## What Is JPQL?

JPQL is:

* Object-oriented query language
* Works on **entities**, not tables
* Database-independent

Think of it as:

> **SQL for objects**

---

## JPQL vs SQL

| JPQL              | SQL              |
| ----------------- | ---------------- |
| Uses entity names | Uses table names |
| Uses fields       | Uses columns     |
| DB-independent    | DB-specific      |

---

## 1.1 Basic Select

```java
@Query("select u from User u")
List<User> findAllUsers();
```

Generated SQL (example):

```sql
select * from users;
```

---

## 1.2 Where Clause

```java
@Query("select u from User u where u.email = :email")
User findByEmail(@Param("email") String email);
```

---

## 1.3 Join (Normal Join)

Used when:

* You want to filter
* You don’t need to fetch child entities

```java
@Query("""
  select o from Order o
  join o.user u
  where u.id = :userId
""")
List<Order> findOrdersByUser(Long userId);
```

⚠️ **JOIN ≠ FETCH JOIN**

---

## 1.4 Fetch Join (🔥 MOST IMPORTANT)

### Problem: N+1

```java
List<User> users = userRepo.findAll();
users.forEach(u -> u.getOrders().size());
```

SQL:

```
1 → users
N → orders
```

---

### Solution: Fetch Join

```java
@Query("""
  select u from User u
  join fetch u.orders
""")
List<User> findUsersWithOrders();
```

SQL:

```sql
select u.*, o.*
from users u
join orders o on u.id = o.user_id;
```

✔ Orders loaded in one query
✔ No N+1

---

## Important Rules for Fetch Join

* Use only when needed
* Avoid fetch join with pagination (dangerous)
* Fetch joins load **entire object graph**

---

# 2️⃣ Named Queries

## What Are Named Queries?

JPQL queries:

* Defined once
* Reused
* Validated at startup

---

### Example

```java
@Entity
@NamedQuery(
  name = "User.findByStatus",
  query = "select u from User u where u.status = :status"
)
public class User { }
```

Usage:

```java
em.createNamedQuery("User.findByStatus", User.class)
  .setParameter("status", Status.ACTIVE)
  .getResultList();
```

📌 Useful for:

* Large systems
* Shared queries

---

# 3️⃣ DTO Projections (🔥 VERY IMPORTANT)

## Problem: Returning Entities

* Loads unnecessary data
* Can cause lazy loading issues
* Leaks domain model

---

## 3.1 Constructor Projection

```java
@Query("""
  select new com.example.UserDto(u.id, u.name)
  from User u
""")
List<UserDto> findUserDtos();
```

DTO:

```java
public record UserDto(Long id, String name) {}
```

✔ Efficient
✔ No persistence context pollution

---

## 3.2 Interface-Based Projection (Spring)

```java
public interface UserView {
    Long getId();
    String getName();
}
```

```java
List<UserView> findByStatus(Status status);
```

✔ Clean
✔ Fast
⚠️ Spring-specific

---

# 4️⃣ Criteria API

## What Is Criteria API?

* Programmatic query builder
* Type-safe
* Good for **dynamic queries**

---

## 4.1 Basic Criteria Query

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> user = cq.from(User.class);

cq.select(user)
  .where(cb.equal(user.get("status"), Status.ACTIVE));

List<User> result = em.createQuery(cq).getResultList();
```

---

## 4.2 Type-Safe Criteria (Metamodel)

```java
cb.equal(user.get(User_.status), Status.ACTIVE);
```

✔ Compile-time safety
❌ Verbose

---

## 4.3 Dynamic Queries Example

```java
List<Predicate> predicates = new ArrayList<>();

if (status != null) {
    predicates.add(cb.equal(user.get("status"), status));
}
if (email != null) {
    predicates.add(cb.like(user.get("email"), "%" + email + "%"));
}

cq.where(predicates.toArray(new Predicate[0]));
```

✔ Dynamic filtering
✔ No string concatenation

---

# 5️⃣ Native Queries

## What Are Native Queries?

* Raw SQL
* DB-specific
* Full control

---

## When To Use Native Queries

✔ Complex reporting
✔ Vendor-specific features
✔ Performance-critical SQL
✔ Legacy DB

---

## 5.1 Basic Native Query

```java
@Query(
  value = "select * from users where status = ?1",
  nativeQuery = true
)
List<User> findByStatus(String status);
```

---

## 5.2 Mapping Native Query to DTO

```java
@Query(
  value = """
    select u.id as id, u.name as name
    from users u
  """,
  nativeQuery = true
)
List<UserProjection> findUsers();
```

```java
public interface UserProjection {
    Long getId();
    String getName();
}
```

---

## 5.3 SqlResultSetMapping (Advanced)

```java
@SqlResultSetMapping(
  name = "UserDtoMapping",
  classes = @ConstructorResult(
    targetClass = UserDto.class,
    columns = {
      @ColumnResult(name = "id", type = Long.class),
      @ColumnResult(name = "name", type = String.class)
    }
  )
)
```

---

# 6️⃣ Hands-On Scenarios (DO THESE)

---

## 6.1 Complex Join Query

```java
@Query("""
  select o from Order o
  join o.items i
  join o.user u
  where u.status = 'ACTIVE'
    and i.quantity > 1
""")
List<Order> findComplex();
```

---

## 6.2 Fetch Join to Fix N+1

```java
@Query("""
  select distinct o from Order o
  join fetch o.items
""")
List<Order> findOrdersWithItems();
```

⚠️ Use `distinct` to avoid duplicates

---

## 6.3 Native Query Mapping

```java
List<Object[]> rows = em.createNativeQuery(
    "select id, name from users"
).getResultList();
```

Better:
✔ Use projections instead

---

# 7️⃣ Choosing the Right Tool

| Scenario        | Best Choice        |
| --------------- | ------------------ |
| Simple queries  | Repository methods |
| Joins & fetch   | JPQL               |
| Dynamic filters | Criteria API       |
| DB-specific     | Native SQL         |
| Read-only views | DTO projection     |

---

# 🧠 Phase 6 Mental Model

> **JPQL is your default weapon**
> **Criteria API is for dynamic logic**
> **Native SQL is the escape hatch**

If you master fetch joins and DTOs:
👉 You avoid 90% of JPA problems

---

# ✅ Phase 6 Outcome (ACHIEVED)

You can now:
✔ Write efficient queries
✔ Solve N+1 properly
✔ Avoid entity over-fetching
✔ Choose correct query style

---

## 🚀 Next Phase (Phase 7)

**Spring Data JPA Mastery**

* Specifications
* Pagination + joins
* Projections
* Query derivation internals

