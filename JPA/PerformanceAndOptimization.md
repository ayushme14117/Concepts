# 📘 Phase 8 – Performance & Optimization (Deep Dive)

Perfect — **Phase 8 is where JPA becomes production-grade**.
Most teams *use* JPA, very few **optimize** it. This phase teaches you how to **diagnose slow APIs and fix them confidently**.

## 🎯 Goal of Phase 8

After this phase, you should be able to:

* Look at slow APIs and **pinpoint the root cause**
* Count SQL queries and **explain why they happen**
* Choose the **right fetching strategy**
* Optimize **bulk operations**
* Use caching **without breaking consistency**

> If earlier phases taught *correctness*,
> **Phase 8 teaches efficiency**.

---

# 1️⃣ N+1 Problem (🔥 Deep Dive)

## What Is the N+1 Problem?

N+1 happens when:

1. One query loads parent entities
2. N additional queries load child entities lazily

---

## Example Scenario

```java
List<User> users = userRepository.findAll();

for (User user : users) {
    user.getOrders().size();
}
```

### SQL Executed

```sql
1 → select * from users;
N → select * from orders where user_id = ?;
```

If:

* 1,000 users
* Each has orders

👉 **1,001 SQL queries**

---

## Why It Happens (Internal Reason)

* `orders` is `LAZY`
* Each access triggers a query
* JPA cannot guess your intent

📌 **LAZY is correct — misuse causes N+1**

---

## How to Detect N+1

* Enable SQL logging
* Look for repeating similar queries
* Count queries per request

---

# 2️⃣ Fetch Join vs Entity Graph (🔥 VERY IMPORTANT)

---

## 2.1 Fetch Join (JPQL-Level)

### What Is Fetch Join?

A **fetch join** tells JPA:

> “Load related entities in the same SQL query”

---

### Example

```java
@Query("""
  select u from User u
  join fetch u.orders
""")
List<User> findUsersWithOrders();
```

### SQL

```sql
select u.*, o.*
from users u
join orders o on u.id = o.user_id;
```

✔ One query
✔ No N+1
❌ Hard-coded fetch strategy

---

### Problems with Fetch Join

* Cannot paginate safely
* Over-fetching
* Duplicates (needs `distinct`)

---

## 2.2 Entity Graph (🔥 Recommended)

### What Is EntityGraph?

EntityGraph lets you:

* Control fetching **outside JPQL**
* Keep repository methods reusable

---

### Example

```java
@EntityGraph(attributePaths = "orders")
List<User> findAll();
```

### Benefits

✔ Cleaner repositories
✔ Flexible fetching
✔ Works well with pagination (to some extent)

---

### Multiple Attributes

```java
@EntityGraph(attributePaths = {"orders", "profile"})
```

---

## Fetch Join vs Entity Graph

| Feature     | Fetch Join | Entity Graph |
| ----------- | ---------- | ------------ |
| Location    | JPQL       | Repository   |
| Flexibility | Low        | High         |
| Pagination  | ❌          | ⚠️           |
| Reusability | ❌          | ✅            |

📌 **Use EntityGraph by default, Fetch Join when necessary**

---

# 3️⃣ Batch Processing (Large Data Handling)

## Problem

Processing large datasets:

* Consumes memory
* Persistence context grows endlessly
* Performance degrades

---

## Bad Example

```java
for (User user : users) {
    em.persist(user);
}
```

Persistence Context keeps growing → 💥 memory

---

## Correct Batch Pattern

```java
int batchSize = 50;

for (int i = 0; i < users.size(); i++) {
    em.persist(users.get(i));

    if (i % batchSize == 0) {
        em.flush();
        em.clear();
    }
}
```

### Why This Works

* `flush()` → sends SQL
* `clear()` → frees memory

📌 **Mandatory for batch jobs**

---

# 4️⃣ JDBC Batch Inserts (🔥 BIG PERFORMANCE BOOST)

## Problem

Default behavior:

```sql
INSERT user1
INSERT user2
INSERT user3
```

Each insert = network round trip

---

## Enable JDBC Batching

```properties
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
```

---

## Result

Hibernate sends:

```sql
INSERT users VALUES (...)
INSERT users VALUES (...)
INSERT users VALUES (...)
```

As a **batch**

✔ Faster
✔ Fewer round trips

⚠️ Does NOT work with `GenerationType.IDENTITY`

---

# 5️⃣ Second-Level Cache (L2 Cache)

---

## What Is Second-Level Cache?

* Cache shared across sessions
* Lives beyond one transaction
* Stores entities by ID

---

## Cache Levels

| Level    | Scope               |
| -------- | ------------------- |
| L1 Cache | Persistence Context |
| L2 Cache | Application-wide    |

---

## Enable L2 Cache (Hibernate)

```properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```

---

## Mark Entity as Cacheable

```java
@Entity
@Cacheable
public class Product {
}
```

---

## When to Use L2 Cache

✔ Read-heavy data
✔ Rarely changing entities
✔ Reference data

❌ High-write entities
❌ Frequently updated rows

---

# 6️⃣ Query Caching

## What Is Query Cache?

* Caches **query results**
* Uses L2 cache internally
* Stores IDs, not entities

---

## Enable Query Cache

```properties
spring.jpa.properties.hibernate.cache.use_query_cache=true
```

---

## Usage

```java
@QueryHints(
  @QueryHint(name = "org.hibernate.cacheable", value = "true")
)
List<Product> findAllCached();
```

---

## Important Caveat

* Any update invalidates cache
* Use only for **static data**

---

# 7️⃣ Hands-On: Diagnose Slow API

---

## 7.1 Enable SQL Logging

```properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql=TRACE
```

---

## 7.2 Compare Query Counts

### Without Optimization

```
GET /users → 101 queries
```

### With EntityGraph

```
GET /users → 1–2 queries
```

✔ Huge improvement

---

## 7.3 Use `@EntityGraph`

```java
@EntityGraph(attributePaths = "orders")
List<User> findByStatus(Status status);
```

---

# 8️⃣ Common Performance Mistakes (READ THIS)

❌ Using EAGER everywhere
❌ Ignoring SQL logs
❌ Pagination with fetch joins
❌ No batching for bulk inserts
❌ Caching mutable entities

---

# 🧠 Phase 8 Mental Model (CRITICAL)

> **JPA performance is about controlling SQL, not annotations**

If you can:

* Count queries
* Explain why they run
* Reduce them intentionally

👉 You can handle production load

---

# ✅ Phase 8 Outcome (ACHIEVED)

You now:
✔ Diagnose N+1 issues
✔ Choose correct fetching strategy
✔ Optimize bulk operations
✔ Use caching safely
✔ Read SQL like a pro

---

## 🚀 Next Phase (Phase 9)

**Advanced JPA Concepts**

* Inheritance strategies
* Composite keys
* Optimistic vs pessimistic locking
* Auditing
* Soft deletes

