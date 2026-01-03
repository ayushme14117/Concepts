# 📘 Phase 7 – Spring Data JPA Mastery

**Phase 7 is where JPA becomes clean, expressive, and maintainable**.
Most bad JPA code comes from **abusing repositories** instead of understanding them.

Below is a **clear, deep, practical explanation** of **Spring Data JPA Mastery**, with **examples, internals, and best practices**.

## 🎯 Goal of Phase 7

After this phase, you will:

* Know **what JpaRepository really does**
* Write **clean repository interfaces**
* Avoid unreadable method names
* Implement **dynamic filtering properly**
* Use pagination **without breaking performance**

---

# 1️⃣ JpaRepository Internals (How It Really Works)

## What Is JpaRepository?

`JpaRepository<T, ID>` is:

* A **Spring-generated proxy**
* Built on top of `EntityManager`
* Provides CRUD + paging + sorting

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

---

## What Happens Under the Hood

```
Controller
   ↓
Service
   ↓
Repository Proxy
   ↓
EntityManager
   ↓
Hibernate
```

Every method eventually calls:

```java
entityManager.persist / find / merge / remove
```

📌 **Repositories are convenience layers, not magic**

---

# 2️⃣ Query Derivation (Method Name Queries)

## What Is Query Derivation?

Spring parses method names and **generates JPQL** automatically.

---

## Simple Example

```java
List<User> findByStatus(Status status);
```

Generated JPQL:

```jpql
select u from User u where u.status = :status
```

---

## Supported Keywords

```java
findByName
findByNameAndStatus
findByAgeGreaterThan
findByEmailContaining
findByCreatedAtBetween
```

---

## Complex Example (Readable Limit)

```java
List<User> findByStatusAndEmailContainingAndCreatedAtAfter(
    Status status,
    String email,
    LocalDateTime date
);
```

⚠️ **Stop here — beyond this, use @Query or Specifications**

---

# 3️⃣ `@Query` (Explicit Queries)

## When To Use `@Query`

✔ Complex joins
✔ Fetch joins
✔ DTO projections
✔ Performance-critical queries

---

## JPQL Example

```java
@Query("""
  select u from User u
  join fetch u.orders
  where u.status = :status
""")
List<User> findActiveUsersWithOrders(Status status);
```

✔ Clear
✔ Predictable
✔ Optimizable

---

## Native Query Example

```java
@Query(
  value = "select * from users where status = ?1",
  nativeQuery = true
)
List<User> findByStatusNative(String status);
```

---

# 4️⃣ Projections (🔥 VERY IMPORTANT)

## Why Projections Matter

Returning full entities:

* Loads unnecessary data
* Triggers lazy loading
* Pollutes persistence context

---

## 4.1 Interface-Based Projections (Recommended)

```java
public interface UserView {
    Long getId();
    String getName();
}
```

```java
List<UserView> findByStatus(Status status);
```

✔ Fast
✔ Clean
✔ No entity tracking

---

## 4.2 Class-Based Projections (DTO)

```java
public record UserDto(Long id, String name) {}
```

```java
@Query("""
  select new com.example.UserDto(u.id, u.name)
  from User u
""")
List<UserDto> findUserDtos();
```

✔ Explicit
✔ Safe
⚠️ More code

---

# 5️⃣ Pagination & Sorting

## Why Pagination Is Tricky

* Pagination happens at DB level
* Fetch joins can break pagination

---

## Basic Pagination

```java
Page<User> findByStatus(Status status, Pageable pageable);
```

Usage:

```java
PageRequest.of(0, 10, Sort.by("createdAt").descending());
```

---

## Pagination with Joins (⚠️ Common Trap)

❌ This breaks:

```java
@Query("""
  select u from User u
  join fetch u.orders
""")
Page<User> findAll(Pageable pageable);
```

Reason:

* Fetch join multiplies rows

---

## Correct Approach

### Step 1: Page IDs

```java
@Query("""
  select u.id from User u
""")
Page<Long> findUserIds(Pageable pageable);
```

### Step 2: Fetch by IDs

```java
@Query("""
  select u from User u
  join fetch u.orders
  where u.id in :ids
""")
List<User> findUsersWithOrders(List<Long> ids);
```

---

# 6️⃣ Specifications (🔥 DYNAMIC FILTERING)

## What Are Specifications?

Specifications are:

* Type-safe
* Composable
* Based on Criteria API

---

## 6.1 Basic Specification

```java
public class UserSpecs {

    public static Specification<User> hasStatus(Status status) {
        return (root, query, cb) ->
            cb.equal(root.get("status"), status);
    }
}
```

---

## 6.2 Combine Specifications

```java
Specification<User> spec =
    where(hasStatus(ACTIVE))
    .and(emailContains("gmail"));
```

---

## Repository Setup

```java
public interface UserRepository
    extends JpaRepository<User, Long>,
            JpaSpecificationExecutor<User> {
}
```

---

## Usage

```java
List<User> users = userRepo.findAll(spec);
```

✔ Clean
✔ Reusable
✔ Dynamic

---

# 7️⃣ Hands-On Scenarios

---

## 7.1 Dynamic Filtering Using Specifications

```java
Specification<User> spec = Specification.where(null);

if (status != null) {
    spec = spec.and(UserSpecs.hasStatus(status));
}

if (email != null) {
    spec = spec.and(UserSpecs.emailContains(email));
}

List<User> result = userRepo.findAll(spec);
```

---

## 7.2 Pagination with Specifications

```java
Page<User> page = userRepo.findAll(
    spec,
    PageRequest.of(0, 20)
);
```

---

# 8️⃣ Common Mistakes (READ THIS)

❌ Huge derived method names
❌ Returning entities everywhere
❌ Pagination with fetch join
❌ Mixing business logic into repositories
❌ Ignoring Specifications

---

# 🧠 Phase 7 Mental Model

> **Repositories should describe intent, not implementation**

If:

* Method names are readable
* Queries are explicit
* DTOs are used for reads

👉 Your data layer is clean

---

# ✅ Phase 7 Outcome (ACHIEVED)

You now:
✔ Understand repository internals
✔ Write clean queries
✔ Use projections properly
✔ Implement dynamic filtering
✔ Paginate safely

---

## 🚀 Next Phase (Phase 8)

**Performance & Optimization**

* N+1 deep dive
* Batch processing
* Entity graphs
* Second-level cache
* Query caching
