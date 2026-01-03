# 📘 Phase 5 – Transactions & Spring Integration (Deep + Simple)

**Phase 5 is what separates “it works on my machine” from production-safe systems**.
Many JPA bugs are **not JPA bugs at all**, but **transaction misunderstandings**.

I’ll explain everything in **simple language**, with **clear examples**, and **Spring-specific behavior**.

## 🎯 Goal of Phase 5

After this phase, you will:

* Know **exactly when transactions start & end**
* Predict **commit vs rollback**
* Avoid partial updates
* Use transactions for **performance**, not just correctness

---

# 1️⃣ ACID Principles (Foundation)

Transactions exist to protect **data consistency**.

---

## 1.1 Atomicity

> “All or nothing”

```java
createOrder();
updateInventory();
```

If one fails → **everything rolls back**

---

## 1.2 Consistency

> DB always moves from one valid state to another

* Constraints
* Foreign keys
* Business rules

---

## 1.3 Isolation

> Transactions don’t see half-done work

* Prevents dirty reads
* Controls concurrency

---

## 1.4 Durability

> Once committed, data survives crashes

Handled by DB (logs, WAL)

---

# 2️⃣ `@Transactional` (Most Important Spring Annotation)

## What It Does

`@Transactional`:

* Starts a DB transaction
* Binds a persistence context
* Commits or rolls back automatically

---

### Example

```java
@Transactional
public void createUser() {
    User user = new User();
    em.persist(user);
}
```

Flow:

1. Transaction starts
2. Persistence context created
3. SQL queued
4. Commit → flush → SQL executed

---

## Where You Can Put It

* Class level
* Method level

⚠️ **Public methods only** (Spring proxy limitation)

---

# 3️⃣ Propagation Types (🔥 VERY IMPORTANT)

Propagation defines:

> “What happens if a transaction already exists?”

---

## 3.1 REQUIRED (Default)

> Join existing TX, else create new

```java
@Transactional
public void parent() {
    child();
}
```

```java
@Transactional
public void child() { }
```

✔ Single transaction

---

## 3.2 REQUIRES_NEW

> Suspend current TX, start new

```java
@Transactional
public void parent() {
    child();
}
```

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void child() { }
```

✔ Two separate transactions

📌 Child commits even if parent rolls back

---

## 3.3 SUPPORTS

> Run with TX if exists, else non-TX

Rarely used.

---

## 3.4 NOT_SUPPORTED

> Suspend TX, run without TX

Used for:

* Reporting
* Read-only operations

---

## 3.5 MANDATORY

> Must have TX, else exception

---

## 3.6 NEVER

> Must NOT have TX, else exception

---

## 3.7 NESTED (Advanced)

> Savepoint inside same TX

```java
@Transactional(propagation = Propagation.NESTED)
```

✔ Roll back part without killing whole TX
⚠️ Requires DB savepoint support

---

# 4️⃣ Rollback Rules (CRITICAL)

---

## Default Spring Behavior

| Exception Type    | Rollback? |
| ----------------- | --------- |
| RuntimeException  | ✅         |
| Error             | ✅         |
| Checked Exception | ❌         |

---

## Example (Surprise!)

```java
@Transactional
public void test() throws Exception {
    em.persist(user);
    throw new Exception("checked");
}
```

❌ **Transaction commits!**

---

## Fixing Rollback Rules

```java
@Transactional(rollbackFor = Exception.class)
```

Now:
✔ Rollback on checked exceptions

---

## No Rollback For

```java
@Transactional(noRollbackFor = IllegalArgumentException.class)
```

---

# 5️⃣ Read-Only Transactions (🔥 PERFORMANCE BOOST)

---

## What It Means

```java
@Transactional(readOnly = true)
```

Tells:

* Spring
* Hibernate
* Database

That:

> “This transaction will not modify data”

---

## Benefits

* Disables dirty checking
* Optimizes SQL
* Prevents accidental writes

---

## Example

```java
@Transactional(readOnly = true)
public User getUser(Long id) {
    return em.find(User.class, id);
}
```

---

## What Happens If You Modify?

```java
user.setName("Oops");
```

❌ Change ignored or exception (DB-dependent)

---

# 6️⃣ Hands-On Scenarios (IMPORTANT)

---

## 6.1 Nested Transactions

```java
@Transactional
public void placeOrder() {
    saveOrder();
    saveAudit();
    throw new RuntimeException();
}
```

### Case 1: REQUIRED

Both rollback ❌

---

### Case 2: saveAudit → REQUIRES_NEW

```java
@Transactional(propagation = REQUIRES_NEW)
```

✔ Audit saved
❌ Order rolled back

---

## 6.2 Exception Rollback Scenarios

```java
@Transactional
public void test() {
    em.persist(user);
    throw new RuntimeException();
}
```

✔ Rollback

---

```java
@Transactional
public void test() throws Exception {
    em.persist(user);
    throw new Exception();
}
```

❌ Commit (unless rollbackFor specified)

---

## 6.3 Read-Only Optimization

```java
@Transactional(readOnly = true)
public List<User> findAll() {
    return userRepository.findAll();
}
```

Hibernate:

* Skips dirty checking
* Faster execution

---

# 7️⃣ Common Transaction Mistakes (READ THIS)

❌ Using `@Transactional` on private methods
❌ Expecting rollback on checked exceptions
❌ Using REQUIRES_NEW blindly
❌ Writing data inside read-only TX
❌ Multiple DB calls outside TX

---

# 🧠 Phase 5 Mental Model (MEMORIZE THIS)

> **Transactions define boundaries for consistency and performance**

If you control:

* Propagation
* Rollback rules
* Read-only flags

👉 You control your system

---

# ✅ Phase 5 Outcome (ACHIEVED)

You now:
✔ Understand ACID
✔ Control commit vs rollback
✔ Predict nested TX behavior
✔ Optimize read-only queries

---

## 🚀 Next Phase (Phase 6)

**JPQL, Criteria API & Native Queries**

* Fetch joins
* DTO projections
* Dynamic queries
* Native SQL mapping
