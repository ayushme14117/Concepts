# 📘 Phase 9 – Advanced JPA Concepts (Deep & Practical)

**Phase 9 is where JPA moves from CRUD apps to real enterprise systems**.
These concepts appear in **banking, e-commerce, SaaS, and high-concurrency systems**.

## 🎯 Goal of Phase 9

After this phase, you should be able to:

* Model **complex domain hierarchies**
* Handle **concurrent updates safely**
* Track **who created/updated data**
* Implement **soft deletes correctly**
* Avoid subtle data corruption bugs

---

# 1️⃣ Inheritance Strategies (Mapping Class Hierarchies)

## The Problem

Java supports inheritance.
Databases **do not**.

JPA gives **3 strategies** to map inheritance to tables.

---

## Example Domain

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public abstract class Payment {

    @Id @GeneratedValue
    private Long id;

    private Double amount;
}
```

```java
@Entity
public class CardPayment extends Payment {
    private String cardNumber;
}
```

```java
@Entity
public class UpiPayment extends Payment {
    private String upiId;
}
```

---

## 1.1 SINGLE_TABLE (🔥 Most Common)

### How It Works

* One table for all subclasses
* Discriminator column identifies type

```java
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type")
```

---

### Table Structure

```sql
payment
-------
id
amount
payment_type
card_number
upi_id
```

### Pros

✔ Fast queries
✔ Simple joins
✔ Easy to maintain

### Cons

❌ Nullable columns
❌ Table grows wide

📌 **Recommended default strategy**

---

## 1.2 JOINED (Normalized)

### How It Works

* Parent table + subclass tables
* JOIN at runtime

```java
@Inheritance(strategy = InheritanceType.JOINED)
```

---

### Tables

```sql
payment
-------
id
amount

card_payment
------------
id (FK)
card_number
```

---

### Pros

✔ Normalized schema
✔ No null columns

### Cons

❌ Slower (JOINs)
❌ Complex queries

📌 Use when schema clarity > performance

---

## 1.3 TABLE_PER_CLASS (Avoid)

### How It Works

* Each subclass has its own table
* No parent table

```java
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
```

---

### Tables

```sql
card_payment
-------------
id
amount
card_number
```

```sql
upi_payment
------------
id
amount
upi_id
```

---

### Pros

✔ Simple tables

### Cons

❌ UNION queries
❌ No shared IDs
❌ Poor performance

🚫 **Avoid in production**

---

## Inheritance Strategy Summary

| Strategy        | Performance | Schema | Recommendation     |
| --------------- | ----------- | ------ | ------------------ |
| SINGLE_TABLE    | ⭐⭐⭐         | Simple | ✅ Default          |
| JOINED          | ⭐⭐          | Clean  | ⚠️ Use selectively |
| TABLE_PER_CLASS | ⭐           | Messy  | ❌ Avoid            |

---

# 2️⃣ Composite Keys (Multiple Column Primary Keys)

## When Needed

* Legacy DB
* Join tables with extra columns
* Natural keys

---

## 2.1 `@Embeddable` Composite Key (Recommended)

### Key Class

```java
@Embeddable
public class OrderItemId implements Serializable {

    private Long orderId;
    private Long productId;
}
```

---

### Entity

```java
@Entity
public class OrderItem {

    @EmbeddedId
    private OrderItemId id;

    private int quantity;
}
```

✔ Clean
✔ Reusable

---

## 2.2 `@IdClass` (Older Style)

```java
@IdClass(OrderItemId.class)
@Entity
public class OrderItem {

    @Id private Long orderId;
    @Id private Long productId;
}
```

❌ More error-prone
❌ Less readable

📌 Prefer `@EmbeddedId`

---

# 3️⃣ Optimistic Locking (🔥 VERY IMPORTANT)

## Problem

Two users update the same row simultaneously.

---

## Solution: Optimistic Locking

```java
@Version
private Long version;
```

---

## How It Works

1. Entity read with version = 1
2. Update attempt checks version
3. If version mismatch → exception

---

### Example

```java
@Transactional
public void update(User user) {
    user.setName("New Name");
}
```

Generated SQL:

```sql
update users
set name=?, version=version+1
where id=? and version=?
```

---

### If Conflict Occurs

```java
OptimisticLockException
```

✔ Prevents lost updates
✔ No DB locks

📌 **Use by default**

---

# 4️⃣ Pessimistic Locking (Use Carefully)

## What It Does

* Locks DB row
* Other transactions wait

---

### Example

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
User findById(Long id);
```

---

### SQL

```sql
select ... for update
```

---

### Pros

✔ Strong consistency

### Cons

❌ Blocking
❌ Deadlocks

📌 Use only when conflicts are frequent

---

# 5️⃣ Auditing (Tracking Creation & Updates)

## Goal

Automatically track:

* Created date
* Last modified date
* Created by
* Modified by

---

## Step 1: Enable Auditing

```java
@EnableJpaAuditing
```

---

## Step 2: Entity Setup

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

---

## Result

* No manual timestamps
* Automatically filled

✔ Clean
✔ Reliable

---

# 6️⃣ Soft Deletes (🔥 VERY COMMON)

## Problem

Deleting rows causes:

* Data loss
* Broken audit trails

---

## Solution: Soft Delete

Instead of:

```sql
DELETE FROM users
```

Use:

```sql
UPDATE users SET deleted=true
```

---

## Implementation

```java
@Entity
@SQLDelete(sql = "UPDATE users SET deleted = true WHERE id=?")
@Where(clause = "deleted = false")
public class User {

    private boolean deleted = false;
}
```

---

### Behavior

* `delete()` → UPDATE
* All queries auto-filter deleted rows

✔ Transparent
✔ Safe

---

# 7️⃣ Hands-On Implementation

---

## 7.1 Implement Optimistic Locking

```java
@Entity
public class Product {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @Version
    private Long version;
}
```

Test:

* Two threads update same entity
* One fails with `OptimisticLockException`

---

## 7.2 Add Auditing

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Order {

    @CreatedDate
    private LocalDateTime createdAt;
}
```

---

# 8️⃣ Common Advanced Mistakes

❌ No versioning
❌ Using pessimistic locks everywhere
❌ Hard deletes in production
❌ Using TABLE_PER_CLASS
❌ Forgetting equals/hashCode in composite keys

---

# 🧠 Phase 9 Mental Model

> **Advanced JPA is about protecting data under load**

If you:

* Version entities
* Track changes
* Avoid hard deletes

👉 Your system is safe at scale

---

# ✅ Phase 9 Outcome (ACHIEVED)

You now:
✔ Model inheritance correctly
✔ Handle concurrent updates
✔ Implement auditing
✔ Use soft deletes safely

---

## 🚀 Next Phase (Phase 10)

**Testing & Best Practices**

* Repository testing
* Testcontainers
* DTO vs Entity rules
* Schema validation
