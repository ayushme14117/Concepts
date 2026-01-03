# 🎯 Advanced JPA Interview Preparation Guide

---

## 1️⃣ Core Mental Models (Interviewers LOVE these)

### Q: What is JPA in one sentence?

✅ **Best Answer**

> JPA is an object state management specification that synchronizes Java object graphs with relational databases using a persistence context and deferred SQL execution.

❌ Weak answer

> JPA is used to connect Java with database.

---

### Q: What is the most important concept in JPA?

✅ **Persistence Context**

Follow-up explanation:

* First-level cache
* Identity guarantee
* Dirty checking
* Write-behind

If you can’t explain persistence context clearly → **red flag**

---

## 2️⃣ Entity Lifecycle & Persistence Context (High-Frequency)

### Q: Why does JPA update data without calling `save()`?

**Answer**

* Entity is **managed**
* Persistence context tracks changes
* Dirty checking runs at flush

```java
@Transactional
void update() {
    User u = em.find(User.class, 1L);
    u.setName("New");
}
```

➡️ UPDATE happens at commit

---

### Q: When does SQL actually run in JPA?

✅ Correct answer:

* At **flush**
* Flush happens:

  * Before commit
  * Before JPQL query (AUTO flush mode)

---

### Q: Difference between `persist()` and `merge()`?

| Persist       | Merge                   |
| ------------- | ----------------------- |
| NEW → MANAGED | DETACHED → MANAGED copy |
| Same instance | New instance returned   |
| Safer         | Error-prone             |

🚨 Interview trap:

```java
em.merge(entity);
entity.setName("X"); // ignored
```

---

## 3️⃣ Relationships & Mapping (🔥 Most Asked)

### Q: What is the owning side of a relationship?

✅ **The side with the foreign key**

Example:

```java
@ManyToOne
@JoinColumn(name="user_id")
private User user;
```

Even if `@OneToMany` exists — it is **never** the owner.

---

### Q: What happens if you remove `mappedBy`?

Answer:

* Hibernate creates a **join table**
* Extra SQL
* Broken domain model

---

### Q: Why is `@ManyToOne` default EAGER dangerous?

Answer:

* Hidden joins
* Accidental N+1
* Large object graphs
* Memory & performance issues

✅ Always override to `LAZY`

---

## 4️⃣ N+1 & Performance (🔥 Critical)

### Q: Explain N+1 problem in detail

Good answer structure:

1. One query loads parents
2. Lazy collection access triggers N queries
3. Happens due to object navigation
4. Not a bug — a usage problem

---

### Q: How do you fix N+1?

Best answers (in order):

1. **Fetch Join**
2. **EntityGraph**
3. Batch fetching (secondary option)

---

### Q: Fetch Join vs EntityGraph?

| Fetch Join        | EntityGraph      |
| ----------------- | ---------------- |
| JPQL-based        | Annotation-based |
| Hard-coded        | Flexible         |
| Breaks pagination | Safer            |
| Less reusable     | More reusable    |

---

## 5️⃣ Transactions & Spring (Very Common)

### Q: Default rollback behavior of `@Transactional`?

✅ Rolls back on:

* `RuntimeException`
* `Error`

❌ Does NOT rollback on checked exceptions

---

### Q: How to rollback on checked exception?

```java
@Transactional(rollbackFor = Exception.class)
```

---

### Q: Difference between REQUIRED and REQUIRES_NEW?

| REQUIRED          | REQUIRES_NEW       |
| ----------------- | ------------------ |
| Joins existing TX | Suspends existing  |
| One commit        | Independent commit |
| Default           | Use carefully      |

Use case:

* Auditing
* Logging
* Notifications

---

## 6️⃣ JPQL, Criteria & Native Queries

### Q: JPQL vs SQL?

JPQL:

* Entity-based
* Portable
* Object-oriented

SQL:

* Table-based
* DB-specific
* Full control

---

### Q: When should you use native queries?

Correct scenarios:

* Reporting queries
* DB-specific features
* Performance-critical SQL
* Legacy schemas

🚫 Not for normal CRUD

---

### Q: Why DTO projections are important?

Answer:

* Avoid over-fetching
* Avoid lazy loading issues
* Reduce persistence context size
* Clean API boundaries

---

## 7️⃣ Spring Data JPA (Advanced)

### Q: What does JpaRepository actually do?

Answer:

* It’s a Spring proxy
* Delegates to EntityManager
* Adds CRUD + pagination + sorting

---

### Q: When NOT to use query derivation?

Answer:

* Too many conditions
* Joins
* Performance-critical queries
* Readability issues

Use:

* `@Query`
* Specifications

---

### Q: How do Specifications work internally?

Answer:

* Built on Criteria API
* Composable predicates
* Type-safe dynamic queries

---

## 8️⃣ Caching (Senior-Level Questions)

### Q: Difference between L1 and L2 cache?

| L1 Cache            | L2 Cache         |
| ------------------- | ---------------- |
| Persistence Context | Application-wide |
| Mandatory           | Optional         |
| Per transaction     | Shared           |

---

### Q: When should you use L2 cache?

✔ Read-heavy
✔ Rarely updated
✔ Reference data

❌ High-write tables

---

### Q: Why query cache is dangerous?

Answer:

* Any update invalidates cache
* Hard to reason about consistency
* Only safe for static data

---

## 9️⃣ Locking & Concurrency (🔥 Very Important)

### Q: Optimistic vs Pessimistic Locking?

| Optimistic     | Pessimistic |
| -------------- | ----------- |
| Version-based  | DB lock     |
| No blocking    | Blocking    |
| Scalable       | Risky       |
| Default choice | Rare        |

---

### Q: How does `@Version` work?

Answer:

* Adds version column
* Update checks version
* Prevents lost updates
* Throws `OptimisticLockException`

---

## 🔟 Testing & Best Practices

### Q: Why is H2 dangerous for tests?

Answer:

* Different SQL dialect
* Different constraints
* Different behavior

Use:

* H2 for fast tests
* Testcontainers for integration tests

---

### Q: Should entities be exposed via REST?

❌ No

Reasons:

* Lazy loading issues
* Security leaks
* Tight coupling
* Serialization problems

Use DTOs.

---

## 1️⃣1️⃣ Hibernate Internals (Lead-Level)

### Q: EntityManager vs Session?

Answer:

* EntityManager = JPA abstraction
* Session = Hibernate implementation
* EntityManager wraps Session

---

### Q: What is Hibernate ActionQueue?

Answer:

* Queue of pending SQL actions
* Enables write-behind
* Executes at flush

---

### Q: Flush modes?

| Mode   | Behavior             |
| ------ | -------------------- |
| AUTO   | Default              |
| COMMIT | Flush only at commit |
| MANUAL | Explicit flush       |
| ALWAYS | Rare, expensive      |

---

## 🔥 Common Interview Traps (Watch Out)

❌ “Hibernate saves immediately”
❌ “save() always hits DB”
❌ “mappedBy is optional”
❌ “EAGER is convenient”
❌ “merge updates same instance”

---

## 🧠 Final Interview Mindset

Interviewers are checking:

* Can you **predict SQL**?
* Can you **reason about performance**?
* Can you **debug production issues**?
* Can you **design clean persistence layers**?

If you answer with:

* Persistence context
* Flush timing
* Ownership
* Fetch strategies

👉 You stand out immediately.

---

Next:

* Create **mock interview Q&A (50+ questions)**
* Provide **system design JPA scenarios**
* Build **debugging exercises**
* Share **real production war stories**
