# 📘 Phase 1 – JPA Fundamentals (Complete Deep Dive)

## 🎯 Goal

**Understand what JPA is and how it works internally**, not just how to use annotations.

By the end, you should be able to explain:

> “How does JPA convert my Java object changes into SQL, and *when* does it do that?”

---

# 1️⃣ What Is JPA vs Hibernate

## What Is JPA?

**JPA (Java Persistence API)** is:

* A **specification**
* A set of **interfaces + annotations**
* Defines *how ORM should work*

It does **NOT**:

* Generate SQL itself
* Talk to DB directly

### JPA Provides:

* Annotations (`@Entity`, `@Id`)
* APIs (`EntityManager`)
* Rules (entity lifecycle, persistence context)

---

## What Is Hibernate?

**Hibernate is an implementation of JPA**

| JPA            | Hibernate                   |
| -------------- | --------------------------- |
| Spec (rules)   | Concrete ORM engine         |
| Interfaces     | Actual code                 |
| Vendor-neutral | Hibernate-specific features |

Spring Boot default:

```properties
spring.jpa.hibernate.ddl-auto=update
```

👉 You **code to JPA**, Hibernate **executes it**

---

## Analogy

Think of:

* **JPA** = JDBC API
* **Hibernate** = MySQL JDBC Driver

---

# 2️⃣ ORM Basics (Object–Relational Mapping)

## The Core Problem ORM Solves

### Database World

* Tables
* Rows
* Foreign keys

### Java World

* Objects
* References
* Inheritance

ORM bridges these two **very different models**.

---

## Mapping Concept

| Database | Java             |
| -------- | ---------------- |
| Table    | Entity class     |
| Row      | Object           |
| Column   | Field            |
| FK       | Object reference |

```sql
users
------
id
name
```

```java
@Entity
class User {
    @Id
    Long id;
    String name;
}
```

### What JPA Does

```java
user.setName("Ayush");
```

⬇️ internally becomes

```sql
UPDATE users SET name = 'Ayush' WHERE id = ?
```

👉 **You change objects, JPA generates SQL**

---

# 3️⃣ Entity (Core Building Block)

## What Is an Entity?

An **Entity**:

* Is a **persistent domain object**
* Represents **one row in DB**
* Is managed by JPA

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue
    private Long id;

    private String name;
}
```

---

## Entity Rules (Why They Exist)

| Rule               | Reason                    |
| ------------------ | ------------------------- |
| No-arg constructor | JPA uses reflection       |
| Must have `@Id`    | Identity tracking         |
| Not final          | Hibernate creates proxies |
| Fields not final   | Dirty checking            |

⚠️ Entities are **not DTOs**

---

# 4️⃣ Persistence Unit

## What Is a Persistence Unit?

A **Persistence Unit** defines:

* DB connection
* Entity classes
* JPA provider
* Configuration

Traditionally (`persistence.xml`):

```xml
<persistence-unit name="myPU">
    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
</persistence-unit>
```

### In Spring Boot

Persistence Unit is auto-configured using:

```properties
spring.datasource.*
spring.jpa.*
```

👉 One application = usually **one persistence unit**

---

# 5️⃣ JPA Architecture (Very Important)

## High-Level Architecture

```
Application
   ↓
JPA (EntityManager API)
   ↓
Hibernate (Provider)
   ↓
JDBC
   ↓
Database
```

<img width="431" height="491" alt="image" src="https://github.com/user-attachments/assets/68a5631b-c8e7-4ad6-8c2a-9ab3bc747351" />

### Internally:

```
EntityManager
   ↓
Persistence Context
   ↓
Hibernate Session
   ↓
SQL Generator
   ↓
DB
```

👉 **Everything revolves around Persistence Context**

---

# 6️⃣ Persistence Context (🔥 MOST IMPORTANT)

## What Is Persistence Context?

A **Persistence Context** is:

> A **memory space** that holds managed entities

Think of it as:

```java
Map<EntityKey, EntityInstance>
```

Where:

* `EntityKey = (EntityClass + PrimaryKey)`

---

## Why It Exists

1. First-level cache
2. Identity guarantee
3. Dirty checking
4. Write-behind optimization

---

## Example: Identity Guarantee

```java
User u1 = em.find(User.class, 1L);
User u2 = em.find(User.class, 1L);

System.out.println(u1 == u2); // true
```

Only **one SQL query** runs.

---

# 7️⃣ Entity Lifecycle

## Entity States

```
NEW → MANAGED → DETACHED → REMOVED
```

### 1. NEW (Transient)

```java
User user = new User();
```

* Not known to JPA
* No DB row

---

### 2. MANAGED

```java
em.persist(user);
```

* Stored in Persistence Context
* Changes tracked
* No SQL yet

---

### 3. DETACHED

```java
em.detach(user);
```

* Exists in Java
* Not tracked
* Changes ignored

---

### 4. REMOVED

```java
em.remove(user);
```

* Marked for deletion
* SQL executed at flush

---

## Lifecycle Diagram (Mental Model)

```
new User()
   ↓ persist
MANAGED (tracked)
   ↓ commit
DB sync
```

---

# 8️⃣ Dirty Checking (Automatic Change Detection)

## What Is Dirty Checking?

JPA:

1. Takes **snapshot** of entity
2. Tracks changes
3. Compares at flush
4. Generates SQL automatically

---

## Example

```java
@Transactional
public void updateUser() {
    User user = em.find(User.class, 1L);
    user.setName("Ayush");
}
```

No `save()` call.

### What Happens Internally

* Snapshot: `name = "Old"`
* Current: `name = "Ayush"`
* Change detected → UPDATE generated

```sql
UPDATE users SET name='Ayush' WHERE id=1;
```

👉 This works **only for MANAGED entities**

---

# 9️⃣ Flushing vs Committing (CRITICAL)

## Flush

* Synchronizes Persistence Context → DB
* SQL is executed
* Transaction still open

```java
em.flush();
```

---

## Commit

* Commits DB transaction
* Ends Persistence Context
* Flush happens automatically before commit

---

## Comparison

| Action | SQL Runs? | TX Ends? |
| ------ | --------- | -------- |
| flush  | ✅         | ❌        |
| commit | ✅         | ✅        |

---

## Example

```java
@Transactional
public void example() {
    User user = new User();
    em.persist(user);

    em.flush(); // INSERT happens here

    // still inside transaction
}
```

---

# 🔟 EntityManager vs Repository

## EntityManager

Low-level JPA API

```java
em.persist(entity);
em.find(User.class, id);
em.remove(entity);
```

### Pros

* Full control
* Precise behavior

### Cons

* Verbose
* Manual

---

## Repository (Spring Data JPA)

High-level abstraction

```java
userRepository.save(user);
userRepository.findById(1L);
```

### Internally

```java
save() →
  if (new) persist()
  else merge()
```

👉 Repositories **use EntityManager internally**

---

# 1️⃣1️⃣ First-Level Cache (Persistence Context Cache)

## Important Rules

* Scope: **Transaction**
* Cannot be disabled
* One per EntityManager

```java
@Transactional
public void testCache() {
    em.find(User.class, 1L); // SQL
    em.find(User.class, 1L); // No SQL
}
```

---

# 🧠 Phase 1 Master Mental Model

> **JPA is NOT a database tool**
> **JPA is an object state manager**

Key truths:

* SQL is delayed
* Objects are tracked
* Changes are automatic
* Persistence Context is king

---

# ✅ What You Should Be Able To Explain Now

✔ Difference between JPA & Hibernate
✔ Why `save()` is not always required
✔ When SQL executes
✔ Why same entity instance is reused
✔ How dirty checking works

---

## 🚀 Next Step (Phase 2)

**Entity Mapping Deep Dive**

* Columns
* Enums
* Dates
* Embedded objects
* LOBs
* Real-world mappings
