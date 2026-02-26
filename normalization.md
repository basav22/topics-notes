# Normalization

The process of **organizing database tables** to reduce **data redundancy** and prevent **data anomalies**.

## Why Normalize? — The Problem

```
UNNORMALIZED TABLE — orders
┌────┬──────────┬───────┬──────────────┬─────────┬──────────┬───────┐
│ id │ customer │ phone │ email        │ product │ category │ price │
├────┼──────────┼───────┼──────────────┼─────────┼──────────┼───────┤
│ 1  │ Alice    │ 9876  │ alice@m.com  │ Laptop  │ Electronics│ 1000 │
│ 2  │ Alice    │ 9876  │ alice@m.com  │ Mouse   │ Electronics│ 25   │
│ 3  │ Bob      │ 1234  │ bob@m.com    │ Laptop  │ Electronics│ 1000 │
│ 4  │ Alice    │ 9876  │ alice@m.com  │ Shirt   │ Clothing  │ 40   │
└────┴──────────┴───────┴──────────────┴─────────┴──────────┴───────┘
```

### Three Problems (Anomalies)

```
1. UPDATE ANOMALY
   Alice changes phone number → must update 3 rows
   Miss one? Now Alice has TWO phone numbers. Inconsistent.

2. INSERT ANOMALY
   New customer "Charlie" — but hasn't ordered anything yet.
   Can't insert Charlie because there's no product/order to fill.

3. DELETE ANOMALY
   Delete Bob's only order (id=3).
   Bob's customer info (phone, email) is LOST entirely.
```

Normalization solves all three.

---

## 1NF — First Normal Form

### Rules:
- Each column has **atomic (single) values** — no lists, no sets
- Each row is **unique** (has a primary key)

### Violation:

```
┌────┬──────────┬─────────────────┐
│ id │ name     │ phones          │
├────┼──────────┼─────────────────┤
│ 1  │ Alice    │ 9876, 5555      │  ← multiple values in one cell
│ 2  │ Bob      │ 1234            │
└────┴──────────┴─────────────────┘
```

### Fixed (1NF):

```
users                    user_phones
┌────┬──────────┐        ┌─────────┬───────┐
│ id │ name     │        │ user_id │ phone │
├────┼──────────┤        ├─────────┼───────┤
│ 1  │ Alice    │        │ 1       │ 9876  │
│ 2  │ Bob      │        │ 1       │ 5555  │
└────┴──────────┘        │ 2       │ 1234  │
                         └─────────┴───────┘
```

**1NF = No multi-valued columns, every row is unique.**

---

## 2NF — Second Normal Form

### Rules:
- Must be in **1NF**
- No **partial dependency** — every non-key column must depend on the **entire** primary key (not just part of it)

> Only applies when you have a **composite primary key** (2+ columns).

### Violation:

```
Primary Key = (student_id, course_id)

┌────────────┬───────────┬──────────────┬───────┐
│ student_id │ course_id │ student_name │ grade │
├────────────┼───────────┼──────────────┼───────┤
│ 1          │ CS101     │ Alice        │ A     │
│ 1          │ CS102     │ Alice        │ B     │
│ 2          │ CS101     │ Bob          │ A     │
└────────────┴───────────┴──────────────┴───────┘

student_name depends ONLY on student_id, NOT on (student_id + course_id)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
             This is a PARTIAL dependency — violates 2NF
```

### Fixed (2NF):

```
students                    enrollments
┌────────────┬──────────┐   ┌────────────┬───────────┬───────┐
│ student_id │ name     │   │ student_id │ course_id │ grade │
├────────────┼──────────┤   ├────────────┼───────────┼───────┤
│ 1          │ Alice    │   │ 1          │ CS101     │ A     │
│ 2          │ Bob      │   │ 1          │ CS102     │ B     │
└────────────┴──────────┘   │ 2          │ CS101     │ A     │
                            └────────────┴───────────┴───────┘

Now student_name is in its own table, depending on student_id (full key).
grade depends on BOTH student_id AND course_id (full composite key). ✅
```

**2NF = 1NF + no partial dependencies on composite keys.**

---

## 3NF — Third Normal Form

### Rules:
- Must be in **2NF**
- No **transitive dependency** — non-key columns must not depend on OTHER non-key columns

### Violation:

```
┌────┬──────────┬─────────────┬──────────┐
│ id │ name     │ department  │ dept_head│
├────┼──────────┼─────────────┼──────────┤
│ 1  │ Alice    │ Engineering │ Dr. Shah │
│ 2  │ Bob      │ Marketing   │ Mr. Roy  │
│ 3  │ Charlie  │ Engineering │ Dr. Shah │  ← repeated
└────┴──────────┴─────────────┴──────────┘

dept_head depends on department, NOT on id (the primary key)

id → department → dept_head
                ^^^^^^^^^^^^
          transitive dependency — violates 3NF
```

### Fixed (3NF):

```
employees                         departments
┌────┬──────────┬────────────┐    ┌─────────────┬──────────┐
│ id │ name     │ dept_id    │    │ id          │ head     │
├────┼──────────┼────────────┤    ├─────────────┼──────────┤
│ 1  │ Alice    │ 1          │    │ 1           │ Dr. Shah │
│ 2  │ Bob      │ 2          │    │ 2           │ Mr. Roy  │
│ 3  │ Charlie  │ 1          │    └─────────────┴──────────┘
└────┴──────────┴────────────┘

dept_head is now in departments table — depends directly on dept id. ✅
No transitive dependency.
```

**3NF = 2NF + no transitive dependencies.**

---

## BCNF — Boyce-Codd Normal Form

A stricter version of 3NF. **Every determinant must be a candidate key.**

### Violation:

```
A professor can teach only ONE subject.
A subject can be taught by MULTIPLE professors.

┌─────────┬───────────┬────────────┐
│ student │ subject   │ professor  │
├─────────┼───────────┼────────────┤
│ Alice   │ Math      │ Dr. Shah   │
│ Bob     │ Math      │ Dr. Shah   │
│ Alice   │ Physics   │ Dr. Roy    │
│ Charlie │ Math      │ Dr. Gupta  │
└─────────┴───────────┴────────────┘

professor → subject  (professor determines subject)
But professor is NOT a candidate key.
Violates BCNF.
```

### Fixed (BCNF):

```
professor_subjects              student_professors
┌────────────┬───────────┐      ┌─────────┬────────────┐
│ professor  │ subject   │      │ student │ professor  │
├────────────┼───────────┤      ├─────────┼────────────┤
│ Dr. Shah   │ Math      │      │ Alice   │ Dr. Shah   │
│ Dr. Roy    │ Physics   │      │ Bob     │ Dr. Shah   │
│ Dr. Gupta  │ Math      │      │ Alice   │ Dr. Roy    │
└────────────┴───────────┘      │ Charlie │ Dr. Gupta  │
                                └─────────┴────────────┘
```

**BCNF = Every determinant is a candidate key.**

---

## Quick Comparison

| Form | Rule | Depends on |
|---|---|---|
| **1NF** | Atomic values, unique rows | Table structure |
| **2NF** | No partial dependency | Composite keys |
| **3NF** | No transitive dependency | Non-key columns |
| **BCNF** | Every determinant is a candidate key | All determinants |

## Easy Memory Trick

```
1NF:  "One value per cell"
2NF:  "Depend on the WHOLE key"
3NF:  "Depend on NOTHING BUT the key"

Or the oath:
"The key, the whole key, and nothing but the key — so help me Codd."
       1NF      2NF                 3NF
```

---

## Denormalization — When to Break the Rules

```
Normalization  = split tables → reduce redundancy → more JOINs
Denormalization = merge tables → add redundancy → fewer JOINs, faster reads

✅ Denormalize when:
  - Read-heavy application (blogs, dashboards)
  - JOINs are slowing down queries
  - Caching derived/computed values
  - Reporting / analytics tables

❌ Don't denormalize when:
  - Data integrity is critical
  - Write-heavy application
  - Data changes frequently (updates become painful)
```

### Example

```
NORMALIZED (3NF):
  orders JOIN customers JOIN products → 3 table JOIN (slow for reports)

DENORMALIZED:
  order_reports table with customer_name, product_name baked in
  → single table scan (fast for reports, but redundant data)
```

---

## The Full Journey — From Unnormalized to 3NF

```
UNNORMALIZED
┌────┬────────┬───────┬─────────┬──────────┬───────┐
│ id │ student│ phone │ course  │ professor│ grade │
├────┼────────┼───────┼─────────┼──────────┼───────┤
│ 1  │ Alice  │ 9876  │ CS101   │ Dr.Shah  │ A     │
│ 1  │ Alice  │ 9876  │ CS102   │ Dr.Roy   │ B     │
│ 2  │ Bob    │ 1234  │ CS101   │ Dr.Shah  │ A     │
└────┴────────┴───────┴─────────┴──────────┴───────┘
Problems: redundancy, update/insert/delete anomalies

        ↓ 1NF (already atomic, add proper PK)

        ↓ 2NF (remove partial dependencies)

students              enrollments
┌────┬───────┬──────┐ ┌────────┬────────┬───────┐
│ id │ name  │phone │ │stu_id  │course  │ grade │
└────┴───────┴──────┘ └────────┴────────┴───────┘

        ↓ 3NF (remove transitive dependencies)

students              courses                enrollments
┌────┬───────┬──────┐ ┌────────┬──────────┐  ┌────────┬──────────┬───────┐
│ id │ name  │phone │ │ id     │professor │  │stu_id  │course_id │ grade │
└────┴───────┴──────┘ └────────┴──────────┘  └────────┴──────────┴───────┘

Clean. No redundancy. No anomalies.
```

## Common Interview Questions

**Q: What normal form should I aim for?**
> 3NF is the standard for most applications. BCNF for stricter requirements. Beyond that is rarely needed.

**Q: Is higher normalization always better?**
> No. Over-normalization leads to too many JOINs and slower queries. Balance normalization with practical query performance.

**Q: How does normalization relate to indexing?**
> Normalized tables are smaller, so indexes are more effective. But more JOINs mean more index lookups. Trade-off.
