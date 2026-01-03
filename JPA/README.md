# 📘 JPA Mastery – From Fundamentals to Internals

This repository provides a **phase-wise, in-depth guide to mastering JPA (Java Persistence API)** with a strong focus on **real-world usage, performance, and internals**.

It is designed for **backend developers** who want to go beyond basic CRUD and truly understand how JPA and Hibernate work under the hood.

---

## 🧭 Learning Roadmap

Each phase builds on the previous one. Follow them **in order** for best results.

---

## 🔹 Phase 1: JPA Fundamentals  
**Goal:** Understand what JPA is and how it works internally

📄 File:  
- [JPAFundamentals.md](JPAFundamentals.md)

Topics covered:
- JPA vs Hibernate  
- ORM basics  
- Entity lifecycle  
- Persistence Context  
- Dirty checking  
- Flush vs Commit  

---

## 🔹 Phase 2: Entity Mapping Deep Dive  
**Goal:** Master entity annotations and table mappings

📄 File:  
- [EntityMappings.md](EntityMappings.md)

Topics covered:
- `@Entity`, `@Table`, `@Id`, `@GeneratedValue`  
- `@Column`, `@Transient`  
- Enum, Date/Time, LOB mapping  
- Embedded objects (`@Embeddable`, `@Embedded`)  

---

## 🔹 Phase 3: Relationships & Associations  
**Goal:** Avoid the #1 cause of JPA bugs

📄 File:  
- [RelationshipsAndAssociations.md](RelationshipsAndAssociations.md)

Topics covered:
- OneToOne, OneToMany, ManyToOne, ManyToMany  
- Owning side vs inverse side  
- `mappedBy`, join tables  
- FetchType (LAZY vs EAGER)  
- N+1 problem  

---

## 🔹 Phase 4: Persistence Context & Entity Lifecycle  
**Goal:** Understand JPA’s internal magic

📄 File:  
- [PersistenceContextAndEntityLifecycle.md](PersistenceContextAndEntityLifecycle.md)

Topics covered:
- Entity states (Transient, Managed, Detached, Removed)  
- Dirty checking  
- Write-behind  
- Persist vs Merge  
- Clear & Detach  

---

## 🔹 Phase 5: Transactions & Spring Integration  
**Goal:** Control data consistency correctly

📄 File:  
- [TransactionsAndSpringIntegration.md](TransactionsAndSpringIntegration.md)

Topics covered:
- ACID principles  
- `@Transactional`  
- Propagation types  
- Rollback rules  
- Read-only transactions  

---

## 🔹 Phase 6: JPQL, Criteria API & Native Queries  
**Goal:** Query data efficiently

📄 File:  
- [JPQLAndCriteriaAPIAndNativeQueries.md](JPQLAndCriteriaAPIAndNativeQueries.md)

Topics covered:
- JPQL select, join, fetch join  
- Named queries  
- DTO projections  
- Criteria API & dynamic queries  
- Native queries  

---

## 🔹 Phase 7: Spring Data JPA Mastery  
**Goal:** Use Spring Data the right way

📄 File:  
- [SpringDataJPAMastery.md](SpringDataJPAMastery.md)

Topics covered:
- JpaRepository internals  
- Query derivation  
- `@Query`  
- Projections  
- Pagination & sorting  
- Specifications  

---

## 🔹 Phase 8: Performance & Optimization  
**Goal:** Write production-ready JPA code

📄 File:  
- [PerformanceAndOptimization.md](PerformanceAndOptimization.md)

Topics covered:
- N+1 problem (deep dive)  
- Fetch Join vs Entity Graph  
- Batch processing  
- JDBC batch inserts  
- Second-level cache  
- Query caching  

---

## 🔹 Phase 9: Advanced JPA Concepts  
**Goal:** Handle complex real-world scenarios

📄 File:  
- [Advanced JPA Concepts – included in internals & optimization phases]

Topics covered:
- Inheritance strategies  
- Composite keys  
- Optimistic & pessimistic locking  
- Auditing  
- Soft deletes  

---

## 🔹 Phase 10: Testing & Best Practices  
**Goal:** Write clean, testable architecture

📄 File:  
- [Testing&BestPractices.md](Testing&BestPractices.md)

Topics covered:
- Repository testing  
- `@DataJpaTest`  
- Testcontainers  
- Schema validation  
- DTO vs Entity exposure  

---

## 🔹 Phase 11: JPA & Hibernate Internals  
**Goal:** Truly master JPA

📄 File:  
- [JPAHibernateInternals.md](JPAHibernateInternals.md)

Topics covered:
- Hibernate internals  
- SQL generation process  
- Session vs EntityManager  
- Flush modes  

---
