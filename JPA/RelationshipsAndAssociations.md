# 📘 Phase 3 – Relationships & Associations (DEEP DIVE)

Most **production bugs, performance issues, and “JPA is slow/weird” complaints come from Phase 3** being misunderstood.

Below is a **complete, deep, real-world explanation** of **Phase 3: Relationships & Associations**, written for a **senior backend developer**, with **mental models, examples, SQL impact, and common traps**.

## 🎯 Goal of Phase 3

After this phase, you should be able to:

* Design **correct entity relationships**
* Predict **generated SQL**
* Avoid **N+1 problems**
* Understand **why mappedBy exists**
* Control **performance, not fight JPA**

> If Phase 1 is *how JPA works*
> and Phase 2 is *how data is mapped*
> **Phase 3 is where real systems live or die**

---

# 1️⃣ Why Relationships Are Hard in JPA

### Root Problem

Databases and Java think differently:

| Database     | Java              |
| ------------ | ----------------- |
| Foreign keys | Object references |
| JOINs        | Navigation        |
| Tables       | Object graphs     |

JPA must:

* Convert **object graphs → SQL**
* Decide **who owns the relationship**
* Decide **when to fetch data**

---

# 2️⃣ Relationship Types

---

## 2.1 `@ManyToOne` (MOST IMPORTANT)

### Real-World Meaning

> Many entities refer to **one parent**

Examples:

* Many Orders → One User
* Many OrderItems → One Order

---

### Example: Order → User

```java
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

### Database Schema

```sql
orders
------
id
user_id (FK)
```

📌 **Foreign key always lives on the `@ManyToOne` side**

---

### Fetch Type (IMPORTANT)

```java
@ManyToOne(fetch = FetchType.LAZY)
```

⚠️ Default is **EAGER** → dangerous in production
**Always make it LAZY**

---

## 2.2 `@OneToMany`

### Concept

> One parent has **many children**

But…

🚨 **`@OneToMany` is NEVER the owner by default**

---

### Correct Mapping (Bidirectional)

```java
@Entity
public class User {

    @OneToMany(mappedBy = "user")
    private List<Order> orders = new ArrayList<>();
}
```

```java
@Entity
public class Order {

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

📌 **Order owns the relationship**

---

### Why `mappedBy` Exists

It tells JPA:

> “I do NOT manage the FK column”

Without it:

* Extra join table created ❌
* Duplicate updates ❌

---

## 2.3 `@OneToOne`

### Use Case

* User ↔ Profile
* Order ↔ Invoice

---

### Example

```java
@Entity
public class User {

    @OneToOne
    @JoinColumn(name = "profile_id")
    private Profile profile;
}
```

📌 One side owns the FK
📌 Same rules as `@ManyToOne`

---

## 2.4 `@ManyToMany` (USE CAREFULLY)

### Example

* Students ↔ Courses
* Users ↔ Roles

---

### Mapping

```java
@Entity
public class User {

    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles;
}
```

### DB

```sql
user_roles
-----------
user_id
role_id
```

⚠️ **Avoid `@ManyToMany` in complex domains**
Prefer **explicit join entity**

---

# 3️⃣ Ownership & Mapping (🔥 MOST CONFUSING PART)

---

## 3.1 Owning Side vs Inverse Side

### Rule (MEMORIZE THIS)

> **The side with the foreign key is the owning side**

---

### Example: User ↔ Order

| Side  | Annotation             | Owns FK? |
| ----- | ---------------------- | -------- |
| User  | `@OneToMany(mappedBy)` | ❌        |
| Order | `@ManyToOne`           | ✅        |

---

### Why Ownership Matters

Only **owning side updates DB**

```java
user.getOrders().add(order);
```

❌ Does nothing unless:

```java
order.setUser(user);
```

---

## 3.2 `mappedBy`

### What It Means

```java
@OneToMany(mappedBy = "user")
```

Means:

> “The field `user` in Order owns this relationship”

---

### Without `mappedBy` (BUG)

```java
@OneToMany
private List<Order> orders;
```

Result:

```sql
user_orders
-----------
user_id
order_id
```

🚨 **Unwanted join table**

---

## 3.3 Bidirectional vs Unidirectional

### Unidirectional

```java
Order → User
```

Pros:

* Simple
* Less memory

Cons:

* Harder navigation

---

### Bidirectional

```java
User ↔ Order
```

Pros:

* Easier navigation
* Common in real apps

Cons:

* Must sync both sides manually

---

### Helper Method (BEST PRACTICE)

```java
public void addOrder(Order order) {
    orders.add(order);
    order.setUser(this);
}
```

---

# 4️⃣ Join Tables

Used in:

* `@ManyToMany`
* `@OneToMany` (when misconfigured)

### Explicit Join Table

```java
@JoinTable(
  name = "user_roles",
  joinColumns = @JoinColumn(name = "user_id"),
  inverseJoinColumns = @JoinColumn(name = "role_id")
)
```

📌 Always name join tables explicitly

---

# 5️⃣ Fetching (🔥 PERFORMANCE KILLER ZONE)

---

## 5.1 FetchType.LAZY vs EAGER

### EAGER

* Loads immediately
* Causes JOIN explosion
* Default for `@ManyToOne` ❌

### LAZY

* Loads when accessed
* Proxy-based
* Default for `@OneToMany` ✅

---

### Rule of Thumb

> **Always use LAZY**
> Fetch explicitly when needed

---

## 5.2 N+1 Problem (MOST COMMON BUG)

### Scenario

```java
List<User> users = userRepository.findAll();

for (User user : users) {
    user.getOrders().size();
}
```

### SQL Executed

```
1 query → users
N queries → orders per user
```

💥 Performance disaster

---

### Solution 1: Fetch Join

```java
@Query("""
    select u from User u
    join fetch u.orders
""")
List<User> findAllWithOrders();
```

---

### Solution 2: EntityGraph

```java
@EntityGraph(attributePaths = "orders")
List<User> findAll();
```

---

# 6️⃣ Hands-On: Real-World Model

---

## 6.1 User → Orders

```java
@Entity
public class User {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
}
```

---

## 6.2 Order → OrderItems

```java
@Entity
public class Order {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private User user;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();
}
```

---

## 6.3 OrderItem

```java
@Entity
public class OrderItem {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Order order;

    private String productName;
    private int quantity;
}
```

---

# 7️⃣ Experiments (DO THESE)

---

### Experiment 1: Remove `mappedBy`

Result:

* Extra join table
* Duplicate SQL
* Broken logic

---

### Experiment 2: Switch LAZY → EAGER

Result:

* Huge JOIN queries
* Slow APIs
* Memory spikes

---

# 8️⃣ Common Mistakes (READ CAREFULLY)

❌ Forgetting `mappedBy`
❌ Updating inverse side only
❌ Using EAGER everywhere
❌ Ignoring N+1
❌ Using `@ManyToMany` blindly

---

# 🧠 Phase 3 Mental Model (CRITICAL)

> **JPA manages relationships via foreign keys, not collections**

Collections:

* Do NOT own relationships
* Are just navigation helpers

Foreign keys:

* Decide ownership
* Control SQL

---

# ✅ You Have Mastered Phase 3 If You Can

✔ Explain owning side without notes
✔ Predict when JOINs happen
✔ Identify N+1 instantly
✔ Design relationships without trial-and-error

---

## 🚀 Next Phase (Phase 4)

**Persistence Context & Lifecycle (Advanced)**

* Entity states deep dive
* Merge vs persist
* Detach, clear
* Flush modes
* Real transaction scenarios

