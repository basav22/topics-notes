# ACID Properties

The **four guarantees** every relational database provides for transactions.

## What is a Transaction?

A transaction is a **group of operations** treated as a **single unit**.

```sql
-- Transfer $500 from Alice to Bob
BEGIN;
  UPDATE accounts SET balance = balance - 500 WHERE user = 'Alice';
  UPDATE accounts SET balance = balance + 500 WHERE user = 'Bob';
COMMIT;

-- Both succeed together, or both fail together
```

---

## A — Atomicity

**"All or Nothing"** — Either all operations in a transaction succeed, or none do.

```
Transfer $500: Alice → Bob

Step 1: Deduct $500 from Alice    ✅
Step 2: Add $500 to Bob           ❌ (server crashes)

Without Atomicity:
  Alice lost $500, Bob got nothing     ← money disappeared!

With Atomicity:
  Entire transaction ROLLED BACK
  Alice still has her $500             ← safe
```

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 500 WHERE user = 'Alice';
  -- something fails here
ROLLBACK;  -- everything undone
```

---

## C — Consistency

**"Valid state to valid state"** — A transaction can only bring the database from one valid state to another. All rules (constraints, triggers, cascades) must be satisfied.

```
Rule: balance >= 0

Alice has $300
Transfer $500 from Alice → Bob

Without Consistency:
  Alice.balance = -200     ← violates rule, invalid state

With Consistency:
  Transaction REJECTED
  DB stays in valid state  ← Alice still has $300
```

```sql
-- Enforced by constraints
ALTER TABLE accounts ADD CONSTRAINT positive_balance CHECK (balance >= 0);

-- This transaction will FAIL
BEGIN;
  UPDATE accounts SET balance = balance - 500 WHERE user = 'Alice';  -- $300 - $500 = -$200
  -- ERROR: violates check constraint "positive_balance"
ROLLBACK;  -- automatic
```

---

## I — Isolation

**"Transactions don't interfere"** — Concurrent transactions behave as if they run **one after another**, not simultaneously.

```
Transaction A: Read Alice's balance → $1000
Transaction B: Read Alice's balance → $1000

Transaction A: Deduct $500 → $500
Transaction B: Deduct $300 → $700    ← should be $200!

Without Isolation:
  Both read $1000, both deduct independently
  Final balance: $700 (lost $300 somewhere)

With Isolation:
  Transaction B waits until A finishes
  A: $1000 → $500
  B: $500 → $200  ← correct
```

### Isolation Levels (Low → High)

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|---|---|---|---|---|
| **Read Uncommitted** | Possible | Possible | Possible | Fastest |
| **Read Committed** | Prevented | Possible | Possible | Fast |
| **Repeatable Read** | Prevented | Prevented | Possible | Slower |
| **Serializable** | Prevented | Prevented | Prevented | Slowest |

### The Three Problems Explained

```
DIRTY READ
  Transaction A updates a row but hasn't committed yet
  Transaction B reads that uncommitted data
  Transaction A rolls back
  → B has data that NEVER existed

NON-REPEATABLE READ
  Transaction A reads a row → gets $1000
  Transaction B updates that row to $500 and commits
  Transaction A reads same row again → gets $500
  → Same query, different results within same transaction

PHANTOM READ
  Transaction A: SELECT * FROM users WHERE age > 25  → 10 rows
  Transaction B: INSERT INTO users (age=30)  → commits
  Transaction A: SELECT * FROM users WHERE age > 25  → 11 rows
  → New rows appeared like "phantoms"
```

```sql
-- Set isolation level
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

BEGIN;
  -- your queries here
COMMIT;
```

**PostgreSQL default:** Read Committed
**Most common for apps:** Read Committed or Repeatable Read

---

## D — Durability

**"Committed = Permanent"** — Once a transaction is committed, it survives **crashes, power failures, restarts**.

```
BEGIN;
  UPDATE accounts SET balance = 500 WHERE user = 'Alice';
COMMIT;  ← written to disk (WAL — Write-Ahead Log)

  ⚡ Server crashes 1ms later

  Server restarts → Alice's balance is still $500
  Because COMMIT wrote to disk, not just memory
```

```
How it works:
  1. Changes written to WAL (Write-Ahead Log) on disk
  2. COMMIT confirmed to client
  3. Even if crash happens, DB replays WAL on restart
```

---

## ACID Summary

| Property | Guarantee | Simple Analogy |
|---|---|---|
| **Atomicity** | All or nothing | A bank transfer either fully completes or fully reverses |
| **Consistency** | Rules always satisfied | Balance can never go negative if that's the rule |
| **Isolation** | Transactions don't interfere | Two ATM withdrawals don't corrupt the balance |
| **Durability** | Committed = permanent | Power outage won't lose committed data |

## Common Interview Questions

**Q: What happens if a system crashes mid-transaction?**
> Atomicity ensures the transaction is rolled back. Durability ensures previously committed transactions are safe.

**Q: Can you have ACID in NoSQL?**
> Most NoSQL databases sacrifice some ACID (especially Isolation) for performance. MongoDB added multi-document transactions in v4.0. DynamoDB has transactions but limited scope.

**Q: Which isolation level should I use?**
> Read Committed for most apps (good balance). Serializable for financial/critical systems. Higher isolation = more correctness but slower.

**Q: What is optimistic vs pessimistic locking?**

```
PESSIMISTIC LOCKING
  Lock the row BEFORE reading
  Other transactions wait
  SELECT * FROM accounts WHERE id = 1 FOR UPDATE;  ← row locked
  Safe but slow (blocks others)

OPTIMISTIC LOCKING
  No locks. Read freely.
  On update, check if data changed since you read it
  UPDATE accounts SET balance = 500
  WHERE id = 1 AND version = 3;  ← fails if someone else updated
  Fast but may need retries
```
