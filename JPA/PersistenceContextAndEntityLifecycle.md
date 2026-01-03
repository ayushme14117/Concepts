# 📘 Phase 4 – Persistence Context & Entity Lifecycle (DEEP + SIMPLE)

**Phase 4 is where JPA stops feeling “magical” and starts feeling predictable**.
Once this phase is clear, you will **always know when SQL runs, why it runs, or why it doesn’t**.

I’ll explain everything in **simple language**, with **step-by-step flow**, **examples**, and **what Hibernate actually does internally**.

## 🎯 Goal of Phase 4

After this phase, you should confidently answer:

> “I changed an entity… why did / didn’t JPA hit the database?”

And:

* Why `save()` is sometimes unnecessary
* Why changes are sometimes ignored
* Why SQL appears at commit, not immediately

---

# 1️⃣ Persistence Context (Quick Refresher)

A **Persistence Context** is:

> A **memory area** where JPA tracks entities

Think of it as:

```java
Map<EntityId, ManagedEntity>
```

* One persistence context per transaction
* Managed by `EntityManager`
* Enables dirty checking & caching

📌 **Everything in Phase 4 depends on this**

---

# 2️⃣ Entity States (Lifecycle)

Every entity is **always in exactly ONE state**.

```
Transient → Managed → Detached → Removed
```

---

## 2.1 Transient (NEW)

### What It Means

* Just a normal Java object
* JPA does not know it exists
* No DB row

### Example

```java
User user = new User();
user.setName("Ayush");
```

### Characteristics

❌ No persistence
❌ No tracking
❌ No SQL

---

## 2.2 Managed (🔥 MOST IMPORTANT)

### What It Means

* JPA is tracking this object
* Stored in persistence context
* Changes are automatically detected

### How an Entity Becomes Managed

```java
em.persist(user);           // new entity
em.find(User.class, 1L);    // existing entity
```

---

### Example

```java
@Transactional
public void example() {
    User user = em.find(User.class, 1L);
    user.setName("Ayush");
}
```

### What Happens

* No `save()` call
* JPA detects change
* UPDATE SQL generated at commit

📌 **Managed = dirty checking enabled**

---

## 2.3 Detached

### What It Means

* Entity exists
* But JPA is no longer tracking it
* Changes are ignored

### How It Happens

* Transaction ends
* `em.detach(entity)`
* `em.clear()`

---

### Example

```java
User user = em.find(User.class, 1L);
em.detach(user);
user.setName("Ayush"); // ignored
```

### Result

❌ No SQL
❌ No update

---

## 2.4 Removed

### What It Means

* Entity marked for deletion
* Not deleted immediately

### Example

```java
User user = em.find(User.class, 1L);
em.remove(user);
```

### SQL Runs At

```java
transaction commit or flush
```

---

# 3️⃣ Dirty Checking (Automatic Change Detection)

## What Is Dirty Checking?

JPA:

1. Takes a **snapshot** of entity state
2. Compares before flush
3. Generates UPDATE SQL if changed

---

### Example

```java
@Transactional
public void update() {
    User user = em.find(User.class, 1L);
    user.setName("New Name");
}
```

### Internally

```
Original: name = "Old"
Current:  name = "New"
```

➡️ UPDATE generated automatically

---

### Key Rule

> **Dirty checking only works for MANAGED entities**

---

# 4️⃣ Write-Behind (Delayed SQL Execution)

## What Is Write-Behind?

JPA does **not execute SQL immediately**.

Instead:

* SQL is queued
* Executed later at flush

---

### Example

```java
em.persist(user1);
em.persist(user2);
em.persist(user3);
```

No SQL yet.

At flush:

```sql
INSERT user1
INSERT user2
INSERT user3
```

📌 Enables:

* Batch inserts
* Optimized SQL execution

---

# 5️⃣ Flush vs Commit (Very Important)

## Flush

* Sync persistence context → DB
* SQL is executed
* Transaction still open

```java
em.flush();
```

---

## Commit

* Commits DB transaction
* Flush happens automatically before commit
* Persistence context ends

---

### Comparison

| Action | SQL runs? | TX ends? |
| ------ | --------- | -------- |
| flush  | ✅         | ❌        |
| commit | ✅         | ✅        |

---

# 6️⃣ Persist vs Merge (🔥 VERY CONFUSING)

---

## 6.1 `persist()` (NEW → MANAGED)

### Use When

* Entity is NEW
* No DB row exists

```java
User user = new User();
em.persist(user);
```

### Behavior

* Entity becomes managed
* INSERT at flush

❌ Calling persist on detached entity → exception

---

## 6.2 `merge()` (DETACHED → MANAGED COPY)

### What Merge Actually Does

1. Takes detached entity
2. Creates **new managed instance**
3. Copies values
4. Returns managed instance

---

### Example

```java
User detachedUser = new User();
detachedUser.setId(1L);
detachedUser.setName("Updated");

User managed = em.merge(detachedUser);
```

⚠️ `detachedUser` is still detached
⚠️ `managed` is tracked

---

### Common Bug

```java
em.merge(user);
user.setName("Wrong"); // ignored
```

Correct:

```java
User managed = em.merge(user);
managed.setName("Correct");
```

---

## Persist vs Merge Summary

| Persist                       | Merge                 |
| ----------------------------- | --------------------- |
| NEW entities                  | Detached entities     |
| Same instance becomes managed | New instance returned |
| INSERT                        | UPDATE                |
| Safer                         | Error-prone           |

📌 **Prefer persist when possible**

---

# 7️⃣ Clear & Detach

---

## 7.1 `detach(entity)`

Removes **one entity** from persistence context.

```java
em.detach(user);
```

Changes ignored.

---

## 7.2 `clear()`

Removes **ALL entities**.

```java
em.clear();
```

Used when:

* Large batch jobs
* Memory optimization

---

# 8️⃣ Hands-On Experiments (DO THESE)

---

## Experiment 1: Modify Without Save()

```java
@Transactional
public void test() {
    User user = em.find(User.class, 1L);
    user.setName("Ayush");
}
```

✔ SQL runs at commit

---

## Experiment 2: Outside Transaction

```java
public void test() {
    User user = em.find(User.class, 1L);
    user.setName("Ayush");
}
```

❌ No transaction
❌ No dirty checking
❌ No SQL

---

## Experiment 3: Detach

```java
@Transactional
public void test() {
    User user = em.find(User.class, 1L);
    em.detach(user);
    user.setName("Ayush");
}
```

❌ No SQL

---

# 🧠 Phase 4 Mental Model (MEMORIZE THIS)

> **SQL runs when persistence context is flushed**
> **Persistence context flushes at commit**

If:

* Entity is managed ✔
* Transaction exists ✔

➡️ SQL will run

---

# ✅ Phase 4 Outcome (ACHIEVED)

You now know:
✔ Entity states
✔ Why save() is optional
✔ Why SQL is delayed
✔ How merge really works
✔ When updates are ignored

---

## 🚀 Next Phase (Phase 5)

**Transactions & Spring Integration**

* `@Transactional` deep dive
* Propagation
* Rollback rules
* Read-only optimization

