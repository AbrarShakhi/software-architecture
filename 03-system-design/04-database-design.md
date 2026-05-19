# Database Design

> How to model, structure, and query data so that the system is correct, fast, and maintainable at scale.

---

## Normalization

Normalization reduces data duplication and keeps the database consistent. The forms (1NF, 2NF, 3NF) are milestones on the path from chaotic to well-structured data.

### Before Normalization (Problems)

```sql
-- UNNORMALIZED: Order table with repeated data
orders:
┌────┬──────────────┬───────────────┬───────────────┬───────────┬───────────┐
│ id │ customer_name│ customer_email│ product_names │  prices   │  qty      │
├────┼──────────────┼───────────────┼───────────────┼───────────┼───────────┤
│  1 │ Alice        │ alice@b.com   │ Pen, Notebook │ 1.0, 5.0  │ 3, 1      │
│  2 │ Alice        │ alice@b.com   │ Stapler       │ 12.0      │ 2         │
└────┴──────────────┴───────────────┴───────────────┴───────────┴───────────┘

Problems:
- Alice's email appears in every order she places (duplication)
- Products and prices are in comma-separated strings (can't query efficiently)
- Update Alice's email → must update EVERY row
- Delete an order → lose product price history
```

### After Normalization (3NF)

```sql
-- NORMALIZED: Each fact in one place
CREATE TABLE customers (
    id    SERIAL PRIMARY KEY,
    name  TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

CREATE TABLE products (
    id    SERIAL PRIMARY KEY,
    name  TEXT NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE orders (
    id          SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    created_at  TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
    order_id   INT REFERENCES orders(id),
    product_id INT REFERENCES products(id),
    quantity   INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,  -- snapshot at time of order
    PRIMARY KEY (order_id, product_id)
);
```

Now Alice's email is in one row. Changing it takes one UPDATE. No comma-separated hell.

---

## Indexing

Indexes are the single biggest performance lever for read-heavy systems. Without them, every query scans the entire table.

```sql
-- Without index: full table scan — O(n) for n rows
-- With index:    B-tree lookup   — O(log n)
EXPLAIN SELECT * FROM orders WHERE customer_id = 42;

-- Without index:
-- Seq Scan on orders  (cost=0.00..10000.00 rows=5 width=40)
--   Filter: (customer_id = 42)

-- After: CREATE INDEX ON orders(customer_id);
-- Index Scan on orders (cost=0.12..12.14 rows=5 width=40)
--   Index Cond: (customer_id = 42)
```

### Index Types

```sql
-- Single column — most common
CREATE INDEX idx_orders_customer ON orders(customer_id);

-- Composite — covers queries that filter on both columns
-- Order matters: (customer_id, created_at) covers:
--   WHERE customer_id = 42
--   WHERE customer_id = 42 AND created_at > '2024-01-01'
-- But NOT: WHERE created_at > '2024-01-01' alone
CREATE INDEX idx_orders_customer_date ON orders(customer_id, created_at DESC);

-- Partial index — only index rows that match a condition
-- Much smaller than full index
CREATE INDEX idx_pending_orders ON orders(customer_id) WHERE status = 'PENDING';

-- Full-text search index
CREATE INDEX idx_products_search ON products USING gin(to_tsvector('english', name || ' ' || description));

-- JSON field index (PostgreSQL)
CREATE INDEX idx_metadata_type ON events((data->>'event_type'));
```

### Index Anti-Patterns

```sql
-- DON'T: Index every column — indexes slow down writes and use disk space
-- Index only columns you query/sort/join on frequently

-- DON'T: Negate the index with a function on the left side
SELECT * FROM users WHERE UPPER(email) = 'ALICE@EXAMPLE.COM';
-- ↑ PostgreSQL won't use the index on email
-- FIX: Store email lowercase, or create a functional index:
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

-- DO: Check your query plan regularly
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 42;
```

---

## The N+1 Query Problem

The most common performance bug in ORM-based applications.

```python
# BAD: N+1 queries (1 for orders list + 1 per order for customer)
orders = Order.query.all()                    # 1 query → returns 100 orders
for order in orders:
    print(order.customer.name)               # 1 query per order → 100 queries!
# Total: 101 queries for 100 orders

# GOOD: Eager loading (JOIN in one query)
orders = Order.query.options(
    joinedload(Order.customer)               # JOIN customers in the same query
).all()
# Total: 1 query for 100 orders + their customers
```

```sql
-- The actual SQL difference:

-- N+1 (bad):
SELECT * FROM orders;
SELECT * FROM customers WHERE id = 1;
SELECT * FROM customers WHERE id = 2;
-- ... 98 more queries

-- Eager load (good):
SELECT o.*, c.name, c.email
FROM orders o
JOIN customers c ON c.id = o.customer_id;
-- 1 query
```

---

## Transactions and ACID

**ACID** properties guarantee database correctness:

| Property | Meaning | Example |
|----------|---------|---------|
| **Atomicity** | All steps succeed or all fail | Transfer money: debit + credit both succeed, or neither |
| **Consistency** | Data always satisfies constraints | Can't have negative balance if constraint says so |
| **Isolation** | Concurrent transactions don't see each other's partial state | Two transfers don't read the same balance simultaneously |
| **Durability** | Committed data survives crashes | Confirmed transfer survives power loss |

```python
def transfer_money(from_id: int, to_id: int, amount: float) -> None:
    with db.transaction():   # ACID guarantee
        sender = db.execute("SELECT balance FROM accounts WHERE id=%s FOR UPDATE", from_id).fetchone()
        if sender["balance"] < amount:
            raise InsufficientFundsError()
        db.execute("UPDATE accounts SET balance = balance - %s WHERE id=%s", amount, from_id)
        db.execute("UPDATE accounts SET balance = balance + %s WHERE id=%s", amount, to_id)
        # If any exception: transaction rolls back automatically
        # If committed: both updates persist atomically
```

---

## SQL vs. NoSQL Trade-offs

Don't treat this as "SQL is old, NoSQL is modern." Each solves different problems.

```mermaid
flowchart TD
    Q[What is the data model?] --> R{Highly relational\n(joins, foreign keys)?}
    R -- Yes --> SQL[Use SQL\n(PostgreSQL, MySQL)]
    R -- No --> S{Document-like\n(nested, variable schema)?}
    S -- Yes --> DOC[MongoDB, Firestore]
    S -- No --> T{Key-Value\nor cache?}
    T -- Yes --> KV[Redis, DynamoDB]
    T -- No --> U{Time-series data?}
    U -- Yes --> TS[InfluxDB, TimescaleDB]
    U -- No --> GR{Graph relationships?}
    GR -- Yes --> GRAPH[Neo4j, Amazon Neptune]
```

| Database | Model | Best For | Weakness |
|----------|-------|----------|----------|
| PostgreSQL | Relational | Complex queries, ACID | Horizontal write scaling |
| MongoDB | Document | Flexible schema, nested data | No joins, eventual consistency |
| Redis | Key-Value | Cache, sessions, pub/sub | Data size limited by RAM |
| Cassandra | Wide Column | High write throughput, time-series | Complex queries, no joins |
| Elasticsearch | Inverted Index | Full-text search, logs | Not a primary database |
| Neo4j | Graph | Highly connected data (social, recommendations) | Not for tabular data |

---

## Schema Evolution

**Never break existing clients when changing a schema.**

```sql
-- SAFE: Add a nullable column (existing rows get NULL)
ALTER TABLE users ADD COLUMN preferred_language VARCHAR(10);

-- SAFE: Add a column with a default (existing rows get the default)
ALTER TABLE users ADD COLUMN is_verified BOOLEAN NOT NULL DEFAULT FALSE;

-- DANGEROUS: Rename a column (breaks every query using the old name)
ALTER TABLE users RENAME COLUMN email TO email_address;

-- SAFE migration path for renaming:
-- 1. Add new column
ALTER TABLE users ADD COLUMN email_address TEXT;
-- 2. Backfill data
UPDATE users SET email_address = email;
-- 3. Update application to write to both columns
-- 4. Switch reads to new column
-- 5. (Later) Drop old column when all code is updated
ALTER TABLE users DROP COLUMN email;
```

---

## Connection Pooling

Opening a database connection is expensive (~20–50ms). Connection pools reuse existing connections.

```python
from sqlalchemy import create_engine

# PostgreSQL with connection pool
engine = create_engine(
    "postgresql://user:pass@localhost/db",
    pool_size=20,        # maintain 20 connections open
    max_overflow=10,     # allow up to 10 extra connections under load
    pool_timeout=30,     # wait up to 30s for a connection
    pool_recycle=3600,   # recycle connections every hour (avoid stale connections)
)
```

---

## Key Takeaways

- Normalize data to avoid duplication; selectively denormalize for performance.
- Index columns you filter, sort, or join on — but not every column.
- The N+1 problem is the most common ORM performance bug — use eager loading or batch queries.
- ACID transactions ensure correctness; use them for multi-step operations that must be atomic.
- Choose SQL for relational data and complex queries; NoSQL when the data model or scale demands it.
- Schema migrations require a backward-compatible strategy — never break existing queries in one atomic step.
