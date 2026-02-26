# Indexing & Performance

## What is an Index?

An index is a **separate data structure** (like a book's table of contents) that helps the database find rows **without scanning the entire table**.

```
Without index:  Scan ALL 1,000,000 rows → find match     (SLOW)
With index:     Lookup in B-Tree → jump to exact row      (FAST)
```

## How It Works — B-Tree (Default Index)

```
                    [50]
                   /    \
             [20, 35]   [70, 85]
             /  |  \     /  |  \
          [10] [25] [40] [60] [75] [90]
                              ↓
                         Points to actual row on disk

Instead of scanning all rows, DB traverses the tree.
1,000,000 rows → ~20 comparisons (log2)
```

## Creating Indexes

```sql
-- Basic index
CREATE INDEX idx_users_email ON users(email);

-- Unique index (also enforces uniqueness)
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Composite index (multiple columns)
CREATE INDEX idx_users_name_city ON users(name, city);

-- Drop index
DROP INDEX idx_users_email;
```

## Clustered vs Non-Clustered Index

```
CLUSTERED INDEX
├── Table data is PHYSICALLY sorted by this index
├── Only ONE per table (like a phone book sorted by name)
├── Primary key = clustered index (in most DBs)
└── Searching by PK is fastest

NON-CLUSTERED INDEX
├── Separate structure that POINTS to table rows
├── Multiple allowed per table (like a book's index at the back)
├── Contains: indexed column value + pointer to row
└── Requires extra lookup to get full row
```

```
Non-clustered index lookup:

INDEX (email)              TABLE (on disk)
┌──────────────┬─────┐     ┌────┬───────┬──────────────┐
│ email        │ ptr │     │ id │ name  │ email        │
├──────────────┼─────┤     ├────┼───────┼──────────────┤
│ alice@m.com  │ ──────►   │ 3  │ Alice │ alice@m.com  │
│ bob@m.com    │ ──────►   │ 1  │ Bob   │ bob@m.com    │
│ zara@m.com   │ ──────►   │ 2  │ Zara  │ zara@m.com   │
└──────────────┴─────┘     └────┴───────┴──────────────┘
  (sorted by email)          (sorted by id / PK)
```

## Composite Index — Column Order Matters

```sql
CREATE INDEX idx_name_city ON users(name, city);
```

```
Works for:
  WHERE name = 'Alice'                    ✅ (leftmost column)
  WHERE name = 'Alice' AND city = 'NYC'   ✅ (both columns)

Does NOT work for:
  WHERE city = 'NYC'                      ❌ (skipped leftmost column)

Think of it like a phone book:
  Sorted by (last_name, first_name)
  You can search by last_name alone       ✅
  You CANNOT search by first_name alone   ❌
```

This is called the **leftmost prefix rule**.

## When to Use Indexes

```
✅ USE indexes on:
  - Columns in WHERE clauses
  - Columns in JOIN conditions (foreign keys)
  - Columns in ORDER BY / GROUP BY
  - Columns with high cardinality (many unique values — email, id)

❌ AVOID indexes on:
  - Small tables (full scan is faster)
  - Columns with low cardinality (boolean, gender — few unique values)
  - Columns that are frequently updated (index rebuild cost)
  - Tables with heavy INSERT/UPDATE/DELETE (indexes slow down writes)
```

## The Write Penalty

```
Every INSERT/UPDATE/DELETE must also update ALL indexes on that table.

Table with 0 indexes:   INSERT = fast
Table with 5 indexes:   INSERT = slower (5 index updates)

Indexes = faster reads, slower writes
          It's always a trade-off.
```

## EXPLAIN — Reading Query Plans

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@mail.com';
```

```
Without index:
  Seq Scan on users  (cost=0.00..25.00 rows=1)
  ^^^^^^^^
  Sequential scan = reading EVERY row = BAD for large tables

With index:
  Index Scan using idx_users_email on users  (cost=0.28..8.29 rows=1)
  ^^^^^^^^^^
  Index scan = jumped directly to the row = GOOD
```

### Key Things to Look For in EXPLAIN

```
Seq Scan         → No index used, full table scan (bad on large tables)
Index Scan       → Index used to find rows (good)
Index Only Scan  → Answer found entirely in index, no table lookup (best)
Bitmap Scan      → Index used, but many rows matched
Sort             → Sorting in memory (check if ORDER BY needs index)
Nested Loop      → Join strategy (watch for N+1 pattern)
cost=            → Estimated cost (lower is better)
rows=            → Estimated rows returned
actual time=     → Real execution time (with ANALYZE)
```

## N+1 Query Problem

```
BAD — N+1 queries:
  1 query:   SELECT * FROM users                     → returns 100 users
  100 queries: SELECT * FROM posts WHERE user_id = ?  → 1 per user
  Total: 101 queries ❌

GOOD — Single query with JOIN:
  SELECT u.*, p.*
  FROM users u
  LEFT JOIN posts p ON u.id = p.author_id;
  Total: 1 query ✅

In ORMs: use eager loading (JOIN / include)
  Drizzle:  db.query.users.findMany({ with: { posts: true } })
  Prisma:   prisma.user.findMany({ include: { posts: true } })
```

## Query Optimization Tips

```
1. SELECT only needed columns
   ❌  SELECT * FROM users
   ✅  SELECT id, name FROM users

2. Use LIMIT for pagination
   ✅  SELECT * FROM users LIMIT 20 OFFSET 40

3. Avoid functions on indexed columns
   ❌  WHERE LOWER(email) = 'alice@mail.com'   -- can't use index
   ✅  WHERE email = 'alice@mail.com'           -- uses index

4. Use EXISTS instead of IN for subqueries
   ❌  WHERE id IN (SELECT user_id FROM orders)
   ✅  WHERE EXISTS (SELECT 1 FROM orders WHERE orders.user_id = users.id)

5. Avoid SELECT DISTINCT unless necessary
   It forces sorting/hashing all results

6. Index foreign keys
   Columns used in JOINs should always be indexed

7. Use covering indexes
   If query only needs (name, email), index on (name, email)
   avoids going back to table — Index Only Scan
```

## Index Types (PostgreSQL)

| Type | Use Case |
|---|---|
| **B-Tree** (default) | Equality, range queries (`=, <, >, BETWEEN`) |
| **Hash** | Equality only (`=`), slightly faster than B-Tree for `=` |
| **GIN** | JSONB, arrays, full-text search |
| **GiST** | Geometric data, full-text search |
| **BRIN** | Very large tables with naturally ordered data (timestamps) |

```sql
-- GIN index for JSONB
CREATE INDEX idx_metadata ON users USING GIN (metadata);

-- Query JSONB with index
SELECT * FROM users WHERE metadata @> '{"role": "admin"}';
```

## Summary

```
Index        = separate lookup structure, speeds up reads
B-Tree       = default, works for =, <, >, BETWEEN
Clustered    = 1 per table, data physically sorted (PK)
Non-clustered = separate structure, points to rows
Composite    = multi-column, leftmost prefix rule
Trade-off    = faster reads, slower writes
EXPLAIN      = see how DB executes your query
N+1 problem  = too many queries, solve with JOINs/eager loading
```
