# SQL Joins — Complete Guide

## Setup — Tables We'll Use

```sql
CREATE TABLE departments (
    id   SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE users (
    id            SERIAL PRIMARY KEY,
    name          VARCHAR(100) NOT NULL,
    department_id INT REFERENCES departments(id)
);

CREATE TABLE posts (
    id        SERIAL PRIMARY KEY,
    title     VARCHAR(255) NOT NULL,
    author_id INT REFERENCES users(id)
);
```

### Sample Data

```
USERS                          DEPARTMENTS
id │ name    │ department_id   id │ name
───┼─────────┼───────────────  ───┼──────────
1  │ Alice   │ 1               1  │ Engineering
2  │ Bob     │ 2               2  │ Marketing
3  │ Charlie │ NULL            3  │ Finance
4  │ Diana   │ 1

POSTS
id │ title          │ author_id
───┼────────────────┼──────────
1  │ Learn Node     │ 1
2  │ Learn React    │ 1
3  │ Marketing 101  │ 2
```

---

## 1. INNER JOIN

Returns **only matching rows** from both tables.

```sql
SELECT
    u.name AS user_name,
    d.name AS dept_name
FROM users u
INNER JOIN departments d ON u.department_id = d.id;
```

```
user_name │ dept_name
──────────┼────────────
Alice     │ Engineering
Bob       │ Marketing
Diana     │ Engineering

-- Charlie excluded (department_id is NULL — no match)
-- Finance excluded (no user belongs to it)
```

```
  USERS          DEPARTMENTS
┌─────────┐    ┌─────────────┐
│         │████│             │
│         │████│             │
│         │████│             │
└─────────┘    └─────────────┘
         ████
      INNER JOIN
   (only overlap)
```

---

## 2. LEFT JOIN

Returns **all rows from left table** + matching from right. No match = `NULL`.

```sql
SELECT
    u.name AS user_name,
    d.name AS dept_name
FROM users u
LEFT JOIN departments d ON u.department_id = d.id;
```

```
user_name │ dept_name
──────────┼────────────
Alice     │ Engineering
Bob       │ Marketing
Charlie   │ NULL          -- included, but no department
Diana     │ Engineering

-- All users shown
-- Finance still excluded (no user references it)
```

```
  USERS          DEPARTMENTS
┌─────────┐    ┌─────────────┐
│█████████│████│             │
│█████████│████│             │
│█████████│████│             │
└─────────┘    └─────────────┘
██████████████
    LEFT JOIN
(all left + matches)
```

---

## 3. RIGHT JOIN

Returns **all rows from right table** + matching from left.

```sql
SELECT
    u.name AS user_name,
    d.name AS dept_name
FROM users u
RIGHT JOIN departments d ON u.department_id = d.id;
```

```
user_name │ dept_name
──────────┼────────────
Alice     │ Engineering
Diana     │ Engineering
Bob       │ Marketing
NULL      │ Finance       -- included, but no user in it

-- Charlie excluded (has no department match)
-- All departments shown
```

```
  USERS          DEPARTMENTS
┌─────────┐    ┌─────────────┐
│         │████│█████████████│
│         │████│█████████████│
│         │████│█████████████│
└─────────┘    └─────────────┘
              ████████████████
              RIGHT JOIN
        (all right + matches)
```

---

## 4. FULL JOIN

Returns **all rows from both tables**. No match = `NULL` on either side.

```sql
SELECT
    u.name AS user_name,
    d.name AS dept_name
FROM users u
FULL JOIN departments d ON u.department_id = d.id;
```

```
user_name │ dept_name
──────────┼────────────
Alice     │ Engineering
Diana     │ Engineering
Bob       │ Marketing
Charlie   │ NULL          -- user with no department
NULL      │ Finance       -- department with no user

-- Everyone included
```

```
  USERS          DEPARTMENTS
┌─────────┐    ┌─────────────┐
│█████████│████│█████████████│
│█████████│████│█████████████│
│█████████│████│█████████████│
└─────────┘    └─────────────┘
█████████████████████████████
         FULL JOIN
       (everything)
```

---

## 5. Joining Multiple Tables

```sql
SELECT
    u.name AS user_name,
    d.name AS dept_name,
    p.title AS post_title
FROM users u
LEFT JOIN departments d ON u.department_id = d.id
LEFT JOIN posts p ON u.id = p.author_id;
```

```
user_name │ dept_name    │ post_title
──────────┼──────────────┼──────────────
Alice     │ Engineering  │ Learn Node
Alice     │ Engineering  │ Learn React      -- Alice appears twice (2 posts)
Bob       │ Marketing    │ Marketing 101
Charlie   │ NULL         │ NULL
Diana     │ Engineering  │ NULL
```

---

## 6. Join with Filters

```sql
-- Users in Engineering who have posts
SELECT
    u.name AS user_name,
    p.title AS post_title
FROM users u
INNER JOIN posts p ON u.id = p.author_id
INNER JOIN departments d ON u.department_id = d.id
WHERE d.name = 'Engineering';
```

```
user_name │ post_title
──────────┼────────────
Alice     │ Learn Node
Alice     │ Learn React
```

---

## 7. Self Join

A self join is when a table is joined **with itself**. You treat the same table as two separate tables using **aliases**.

### Syntax

```sql
SELECT a.column, b.column
FROM table_name a        -- alias "a" (first copy)
JOIN table_name b        -- alias "b" (second copy)
ON a.some_column = b.some_column;
```

### Mental Model

```
Self Join = pretend the table is TWO separate tables

    employees (as "e")     employees (as "m")
    ──────────────────     ──────────────────
    id │ name │ mgr_id     id │ name │ mgr_id
    ───┼──────┼────────    ───┼──────┼────────
    ...                    ...

    Then join them like any normal two-table join.
```

---

### Use Case 1: Employee — Manager Hierarchy

The most classic example.

```sql
CREATE TABLE employees (
    id         INT PRIMARY KEY,
    name       VARCHAR(100),
    manager_id INT REFERENCES employees(id)  -- points to SAME table
);

INSERT INTO employees VALUES
(1, 'CEO',     NULL),
(2, 'CTO',     1),
(3, 'CFO',     1),
(4, 'Alice',   2),
(5, 'Bob',     2),
(6, 'Charlie', 3);
```

```
Hierarchy:
        CEO (1)
       /       \
    CTO (2)    CFO (3)
    /   \         \
Alice  Bob     Charlie
```

```sql
-- Find each employee's manager name
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

```
employee │ manager
─────────┼─────────
CEO      │ NULL       -- no manager
CTO      │ CEO
CFO      │ CEO
Alice    │ CTO
Bob      │ CTO
Charlie  │ CFO
```

```sql
-- Find employees who report to 'CTO'
SELECT e.name
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE m.name = 'CTO';

-- Result: Alice, Bob
```

```sql
-- Find managers who have more than 1 report
SELECT
    m.name AS manager,
    COUNT(e.id) AS report_count
FROM employees e
JOIN employees m ON e.manager_id = m.id
GROUP BY m.name
HAVING COUNT(e.id) > 1;

-- Result: CEO (2 reports), CTO (2 reports)
```

---

### Use Case 2: Find Pairs / Comparisons Within Same Table

```sql
CREATE TABLE students (
    id   INT PRIMARY KEY,
    name VARCHAR(100),
    city VARCHAR(100)
);

INSERT INTO students VALUES
(1, 'Alice',   'Mumbai'),
(2, 'Bob',     'Mumbai'),
(3, 'Charlie', 'Delhi'),
(4, 'Diana',   'Mumbai'),
(5, 'Eve',     'Delhi');
```

```sql
-- Find all pairs of students from the same city
SELECT
    a.name AS student_1,
    b.name AS student_2,
    a.city
FROM students a
JOIN students b ON a.city = b.city
WHERE a.id < b.id;    -- avoids duplicates & self-pairing
```

```
student_1 │ student_2 │ city
──────────┼───────────┼────────
Alice     │ Bob       │ Mumbai
Alice     │ Diana     │ Mumbai
Bob       │ Diana     │ Mumbai
Charlie   │ Eve       │ Delhi
```

#### Why `a.id < b.id`?

```
Without it you get:
  Alice-Bob   AND  Bob-Alice    -- duplicate pair
  Alice-Alice                   -- self-pair (meaningless)

a.id < b.id ensures each pair appears only ONCE
```

---

### Use Case 3: Sequential / Previous Row Comparison

```sql
CREATE TABLE daily_sales (
    id     INT PRIMARY KEY,
    day    DATE,
    amount DECIMAL(10,2)
);

INSERT INTO daily_sales VALUES
(1, '2026-01-01', 500),
(2, '2026-01-02', 700),
(3, '2026-01-03', 600),
(4, '2026-01-04', 900);
```

```sql
-- Find days where sales increased compared to previous day
SELECT
    curr.day AS today,
    prev.amount AS yesterday_sales,
    curr.amount AS today_sales,
    curr.amount - prev.amount AS increase
FROM daily_sales curr
JOIN daily_sales prev ON curr.id = prev.id + 1
WHERE curr.amount > prev.amount;
```

```
today      │ yesterday_sales │ today_sales │ increase
───────────┼─────────────────┼─────────────┼─────────
2026-01-02 │ 500             │ 700         │ 200
2026-01-04 │ 600             │ 900         │ 300

-- 2026-01-03 excluded (600 < 700, sales dropped)
```

> Note: In modern SQL, `LAG()` window function is preferred for this, but self join is commonly asked in interviews.

---

### Use Case 4: Find Duplicates

```sql
CREATE TABLE customers (
    id    INT PRIMARY KEY,
    email VARCHAR(255),
    name  VARCHAR(100)
);

INSERT INTO customers VALUES
(1, 'john@mail.com', 'John'),
(2, 'jane@mail.com', 'Jane'),
(3, 'john@mail.com', 'John D'),   -- duplicate email
(4, 'bob@mail.com',  'Bob');
```

```sql
-- Find rows with duplicate emails
SELECT
    a.id,
    a.name,
    a.email
FROM customers a
JOIN customers b ON a.email = b.email
WHERE a.id <> b.id;    -- same email, different row
```

```
id │ name   │ email
───┼────────┼──────────────
1  │ John   │ john@mail.com
3  │ John D │ john@mail.com
```

---

### Use Case 5: Referral / Friendship System

```sql
CREATE TABLE users_ref (
    id          INT PRIMARY KEY,
    name        VARCHAR(100),
    referred_by INT REFERENCES users_ref(id)
);

INSERT INTO users_ref VALUES
(1, 'Alice',   NULL),
(2, 'Bob',     1),
(3, 'Charlie', 1),
(4, 'Diana',   2);
```

```sql
-- Who referred whom?
SELECT
    u.name AS user,
    r.name AS referred_by
FROM users_ref u
LEFT JOIN users_ref r ON u.referred_by = r.id;
```

```
user    │ referred_by
────────┼────────────
Alice   │ NULL
Bob     │ Alice
Charlie │ Alice
Diana   │ Bob
```

```sql
-- Count referrals per user
SELECT
    r.name AS referrer,
    COUNT(u.id) AS total_referrals
FROM users_ref u
JOIN users_ref r ON u.referred_by = r.id
GROUP BY r.name
ORDER BY total_referrals DESC;
```

```
referrer │ total_referrals
─────────┼────────────────
Alice    │ 2
Bob      │ 1
```

---

## Quick Reference

| Join | Left Table | Right Table | No Match |
|---|---|---|---|
| **INNER** | only matched | only matched | row excluded |
| **LEFT** | all rows | only matched | right = NULL |
| **RIGHT** | only matched | all rows | left = NULL |
| **FULL** | all rows | all rows | either = NULL |

## When to Use Which?

| Join | Use When |
|---|---|
| **INNER JOIN** | "Give me users WITH departments" (both must exist) |
| **LEFT JOIN** | "Give me ALL users, and their department IF they have one" (most common) |
| **RIGHT JOIN** | "Give me ALL departments, and their users IF any" (rare — rewrite as LEFT JOIN) |
| **FULL JOIN** | "Give me everything, matched or not" (rare — data reconciliation) |

## Self Join Use Cases Summary

| Use Case | Join Condition | Key Trick |
|---|---|---|
| Employee-Manager | `e.manager_id = m.id` | FK to same table |
| Find pairs (same city) | `a.city = b.city` | `a.id < b.id` to avoid duplicates |
| Compare consecutive rows | `curr.id = prev.id + 1` | Treat as "current" and "previous" |
| Find duplicates | `a.email = b.email` | `a.id <> b.id` to exclude self |
| Referral chain | `u.referred_by = r.id` | Same pattern as employee-manager |
