# 📘 Phase 2 – Entity Mapping Deep Dive

In this phase we'll learn:

* **What each annotation really does**
* **Default behavior vs explicit mapping**
* **Common mistakes**
* **Real-world examples (User, Order, Product)**
* **How Hibernate interprets your mappings**
---

## 🎯 Goal of Phase 2

After this phase, you should be able to:

* Look at a DB schema and **confidently design entities**
* Control **column names, types, constraints**
* Avoid silent bugs caused by default mappings
* Model **real-world domains cleanly**

---

# 1️⃣ Basic Mapping

---

## 1.1 `@Entity` – Declaring a Persistent Class

### What It Means

`@Entity` tells JPA:

> “This class represents a database table and must be managed.”

```java
@Entity
public class User {
}
```

### What Happens Internally

* Hibernate registers this class in the **Persistence Unit**
* Metadata is created (fields, types, columns)
* Hibernate prepares SQL templates

⚠️ Without `@Entity`, the class is **ignored completely**

---

## 1.2 `@Table` – Controlling Table Mapping

### Default Behavior

If you don’t specify `@Table`:

```java
@Entity
public class User { }
```

Hibernate assumes:

```sql
table name = user
```

⚠️ This can break on:

* Reserved keywords (`user`, `order`)
* Naming mismatches

---

### Explicit Mapping

```java
@Entity
@Table(name = "users")
public class User {
}
```

### Advanced Options

```java
@Table(
  name = "users",
  uniqueConstraints = @UniqueConstraint(columnNames = "email")
)
```

---

## 1.3 `@Id` – Primary Key

### Why It’s Mandatory

JPA **must identify each entity uniquely** for:

* Persistence Context tracking
* Dirty checking
* Caching

```java
@Id
private Long id;
```

No `@Id` → **runtime exception**

---

## 1.4 `@GeneratedValue` – ID Generation Strategy

### Common Strategies

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

* DB generates ID (AUTO_INCREMENT)
* INSERT happens immediately
* No batch inserts

```java
@GeneratedValue(strategy = GenerationType.SEQUENCE)
```

* Uses DB sequence
* Best for PostgreSQL / Oracle

```java
@GeneratedValue(strategy = GenerationType.AUTO)
```

* Provider decides

⚠️ **IDENTITY disables JDBC batching** (important later)

---

## 1.5 `@Column` – Column Customization

### Default Mapping

```java
private String name;
```

Hibernate assumes:

```sql
column name = name
nullable = true
length = 255
```

---

### Explicit Column Mapping

```java
@Column(
  name = "user_name",
  nullable = false,
  length = 100,
  unique = true
)
private String name;
```

### Important Attributes

| Attribute  | Purpose           |
| ---------- | ----------------- |
| name       | DB column name    |
| nullable   | NOT NULL          |
| length     | VARCHAR length    |
| unique     | Unique constraint |
| updatable  | Prevent updates   |
| insertable | Prevent inserts   |

---

## 1.6 `@Transient` – Non-Persistent Field

### Meaning

```java
@Transient
private int age;
```

JPA ignores this field completely.

### Use Cases

* Calculated fields
* Temporary values
* Derived attributes

```java
@Transient
public String getDisplayName() {
    return firstName + " " + lastName;
}
```

---

# 2️⃣ Advanced Mapping

---

## 2.1 Enum Mapping (`@Enumerated`)

### The Problem

Enums can be stored as:

* ORDINAL (0,1,2)
* STRING ("ACTIVE")

### Default (Dangerous)

```java
@Enumerated
private Status status;
```

Defaults to:

```java
ORDINAL
```

❌ Adding a new enum breaks data.

---

### Correct Way (Always Use STRING)

```java
@Enumerated(EnumType.STRING)
private Status status;
```

### Example

```java
public enum Status {
    ACTIVE,
    INACTIVE,
    BLOCKED
}
```

DB:

```sql
status VARCHAR(20)
```

---

## 2.2 Date/Time Mapping (`LocalDateTime`)

### Old Way (Avoid)

```java
Date
```

### Modern Way (Recommended)

```java
private LocalDateTime createdAt;
```

Hibernate maps it to:

* TIMESTAMP (most DBs)

---

### Example

```java
@Column(name = "created_at", nullable = false)
private LocalDateTime createdAt;
```

✔ Timezone-safe
✔ Immutable
✔ Cleaner API

---

## 2.3 Large Objects (`@Lob`)

### What It Is

Used for:

* Large text
* Binary data

---

### Text Example

```java
@Lob
@Column(columnDefinition = "TEXT")
private String description;
```

Maps to:

* TEXT (MySQL/Postgres)
* CLOB (Oracle)

---

### Binary Example

```java
@Lob
private byte[] image;
```

Maps to:

* BLOB

⚠️ **Never fetch LOBs eagerly in production**

---

## 2.4 Embedded Objects (`@Embeddable`, `@Embedded`)

### The Problem It Solves

Avoid repeating same fields across entities.

Example:

* Address
* Audit info

---

### Step 1: Create Embeddable Class

```java
@Embeddable
public class Address {

    private String street;
    private String city;
    private String zipCode;
}
```

---

### Step 2: Embed It

```java
@Entity
public class User {

    @Embedded
    private Address address;
}
```

---

### DB Result

```sql
street
city
zip_code
```

All inside `users` table.

---

### Column Override

```java
@Embedded
@AttributeOverrides({
  @AttributeOverride(name = "city", column = @Column(name = "home_city"))
})
private Address address;
```

---

# 3️⃣ Hands-On: Real-World Entity Mapping

---

## 3.1 User Entity

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Enumerated(EnumType.STRING)
    private Status status;

    @Embedded
    private Address address;

    private LocalDateTime createdAt;
}
```

---

## 3.2 Product Entity

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    private Double price;

    @Lob
    private String description;
}
```

---

## 3.3 Order Entity

```java
@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}
```

---

# 4️⃣ Common Mistakes (Very Important)

❌ Using ORDINAL enum
❌ Using `@Transient` for business fields
❌ Relying on default column names
❌ Using `Date` instead of `LocalDateTime`
❌ Not specifying length for strings

---

# 🧠 Phase 2 Mental Model

> **Phase 1 = how JPA works**
> **Phase 2 = how data is represented**

If Phase 1 is the **engine**, Phase 2 is the **shape of data**.

---

# ✅ You Have Mastered Phase 2 If You Can

✔ Design entities without trial-and-error
✔ Predict DB schema from entity
✔ Avoid enum & column pitfalls
✔ Use embedded objects cleanly

---

## 🚀 Next Phase (Phase 3)

**Relationships & Associations**

* OneToMany, ManyToOne
* Owning side
* mappedBy
* Fetch strategies
* N+1 problem
