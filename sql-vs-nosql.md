# SQL vs NoSQL

## The Core Difference

```
SQL  (Relational)          NoSQL (Non-Relational)
─────────────────          ─────────────────────
Structured tables          Flexible formats
Fixed schema               Schema-less / dynamic
Rows & Columns             Documents, Key-Value, Graphs, Columns
ACID transactions          BASE (eventual consistency)
Vertical scaling           Horizontal scaling
```

## SQL Databases

Data stored in **tables with fixed schema**.

```
Users Table
┌────┬─────────┬──────────────┬─────┐
│ id │ name    │ email        │ age │
├────┼─────────┼──────────────┼─────┤
│ 1  │ Alice   │ alice@m.com  │ 28  │
│ 2  │ Bob     │ bob@m.com    │ 32  │
└────┴─────────┴──────────────┴─────┘

Every row MUST have the same columns.
Adding a column = ALTER TABLE (affects all rows).
```

**Popular:** PostgreSQL, MySQL, SQLite, MS SQL Server, Oracle

## NoSQL Databases — 4 Types

### 1. Document Store (Most Common)

Data stored as **JSON-like documents**. Each document can have different fields.

```json
// MongoDB collection: "users"
{
  "_id": "abc123",
  "name": "Alice",
  "email": "alice@m.com",
  "age": 28,
  "address": {                  // nested object — no separate table needed
    "city": "Mumbai",
    "zip": "400001"
  },
  "hobbies": ["reading", "coding"]   // arrays — no join table needed
}

{
  "_id": "def456",
  "name": "Bob",
  "email": "bob@m.com"
  // no age, no address — that's fine, schema is flexible
}
```

**Popular:** MongoDB, CouchDB, Firestore

**Best for:** Content management, user profiles, catalogs, any data with varying structure

### 2. Key-Value Store

Simplest. Like a **giant hash map**. No queries by value, only by key.

```
KEY                  VALUE
─────────────────    ──────────────────
"user:1"          →  "{name: 'Alice', email: 'alice@m.com'}"
"session:abc123"  →  "{userId: 1, expires: '2026-03-01'}"
"cart:user:1"     →  "[{item: 'Laptop', qty: 1}]"
```

**Popular:** Redis, DynamoDB, Memcached

**Best for:** Caching, sessions, real-time leaderboards, rate limiting

### 3. Column-Family Store

Data stored in **columns instead of rows**. Optimized for reading specific columns across millions of rows.

```
Row-oriented (SQL):          Column-oriented:
Read entire row at once      Read entire column at once

Row: [Alice, alice@m, 28]    Names:  [Alice, Bob, Charlie, ...]
Row: [Bob, bob@m, 32]        Emails: [alice@m, bob@m, ...]
Row: [Charlie, c@m, 25]      Ages:   [28, 32, 25, ...]

"Get average age" →          "Get average age" →
  Read ALL columns             Read ONLY age column
  for every row (slow)         skip name, email (fast)
```

**Popular:** Cassandra, HBase, ScyllaDB

**Best for:** Analytics, time-series data, IoT, write-heavy workloads at massive scale

### 4. Graph Database

Data stored as **nodes and relationships**. Queries traverse connections.

```
    (Alice)──FRIENDS_WITH──►(Bob)
       │                      │
   WORKS_AT              WORKS_AT
       │                      │
       ▼                      ▼
  (Google)◄──WORKS_AT──(Charlie)
```

```
Query: "Find friends of Alice who work at Google"
  Traverse: Alice → FRIENDS_WITH → Bob → WORKS_AT → Google ✅
```

**Popular:** Neo4j, Amazon Neptune, ArangoDB

**Best for:** Social networks, recommendation engines, fraud detection, knowledge graphs

---

## Side-by-Side Comparison

| Feature | SQL | NoSQL |
|---|---|---|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scaling** | Vertical (bigger server) | Horizontal (more servers) |
| **Relationships** | JOINs, foreign keys | Embedded/denormalized |
| **Transactions** | ACID (strong) | BASE (eventual consistency) |
| **Query Language** | SQL (standardized) | Varies per database |
| **Data Integrity** | Strong (constraints, FKs) | Application-level |
| **Best for** | Complex queries, relationships | High volume, flexible data |

## ACID vs BASE

```
ACID (SQL)                          BASE (NoSQL)
───────────                         ─────────────
Atomicity                           Basically Available
Consistency                         Soft state
Isolation                           Eventually consistent
Durability

Strong consistency:                 Eventual consistency:
  Read always returns               Read MIGHT return stale
  latest write                      data for a brief period

Bank transfer: MUST use ACID        Social media feed: BASE is fine
```

## Scaling

```
VERTICAL SCALING (SQL)              HORIZONTAL SCALING (NoSQL)
──────────────────────              ────────────────────────
Add more power to                   Add more machines
ONE server                          (distribute data)

┌──────────┐                        ┌────┐ ┌────┐ ┌────┐
│          │                        │ DB │ │ DB │ │ DB │
│  BIG DB  │                        │  1 │ │  2 │ │  3 │
│  SERVER  │                        └────┘ └────┘ └────┘
│          │                        Data split across nodes
└──────────┘
Has a ceiling                       Nearly unlimited
(hardware limits)                   (add more nodes)
```

## When to Use SQL

```
✅ Use SQL when:
  - Data has clear relationships (users → orders → products)
  - You need complex queries with JOINs
  - Data integrity is critical (banking, healthcare)
  - ACID transactions are required
  - Schema is well-defined and stable
  - Reporting and analytics with complex aggregations

Examples:
  - E-commerce (orders, payments, inventory)
  - Banking systems
  - ERP / CRM systems
  - Any app with structured, relational data
```

## When to Use NoSQL

```
✅ Use NoSQL when:
  - Data structure varies or evolves frequently
  - Massive scale (millions of reads/writes per second)
  - Data is naturally document-shaped (JSON)
  - Low latency is critical
  - Horizontal scaling is needed
  - Simple key-based lookups

Examples:
  - Real-time analytics (Cassandra)
  - Caching / sessions (Redis)
  - Content management / catalogs (MongoDB)
  - Social networks / recommendations (Neo4j)
  - IoT sensor data (Cassandra / TimescaleDB)
```

## Can You Use Both? (Polyglot Persistence)

```
Most real-world apps use MULTIPLE databases:

┌─────────────────────────────────────────────┐
│                  Your App                    │
├──────────┬──────────┬───────────┬───────────┤
│ PostgreSQL│ Redis    │ MongoDB   │ Elastic   │
│ Users     │ Cache    │ Product   │ Search    │
│ Orders    │ Sessions │ Catalog   │ Logs      │
│ Payments  │ Rate     │ CMS       │ Full-text │
│           │ Limiting │ Content   │           │
└──────────┴──────────┴───────────┴───────────┘
```

## Common Interview Questions

**Q: Can MongoDB do JOINs?**
> Yes, using `$lookup` (aggregation pipeline), but it's slower than SQL JOINs. MongoDB encourages **embedding** related data instead.

**Q: Is NoSQL always faster?**
> No. For complex queries with multiple JOINs, SQL is often faster. NoSQL is faster for simple key-based lookups and write-heavy workloads.

**Q: Can SQL scale horizontally?**
> Yes, with sharding (e.g., Citus for PostgreSQL, Vitess for MySQL), but it's harder than NoSQL which is designed for it.

**Q: When would you migrate from SQL to NoSQL?**
> When schema changes are too frequent, when you need massive horizontal scale, or when your data is naturally unstructured. Don't migrate just for hype.

## Summary

```
SQL    = structured, relationships, ACID, complex queries
NoSQL  = flexible, scalable, fast writes, simple lookups

Document  (MongoDB)   → flexible JSON documents
Key-Value (Redis)     → caching, sessions
Column    (Cassandra) → analytics, time-series
Graph     (Neo4j)     → relationships, social networks

Most apps: Start with SQL, add NoSQL where needed.
```
