# Data Management Cheatsheet (candidate items)

## DBMS purpose
- Persistent storage
- Concurrent access (multiple users)
- Transactions (ACID)
- Queries via declarative language (SQL)
- Data integrity (constraints, keys)

## SQL basics
```sql
-- create
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE
);

-- insert
INSERT INTO users (id, name, email) VALUES (1, 'Alice', 'a@x.com');

-- select
SELECT name, email FROM users WHERE id = 1;
SELECT * FROM users ORDER BY name DESC LIMIT 10;

-- update
UPDATE users SET email = 'b@x.com' WHERE id = 1;

-- delete
DELETE FROM users WHERE id = 1;
```

## Joins
```sql
SELECT u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```
| Join | Returns |
|---|---|
| `INNER JOIN` | Only matching rows on both sides |
| `LEFT JOIN` | All from LEFT, matched from RIGHT (NULL if no match) |
| `RIGHT JOIN` | All from RIGHT, matched from LEFT |
| `FULL OUTER JOIN` | All rows, NULL where no match |
| `CROSS JOIN` | Cartesian product (every combo) |

## Aggregations
```sql
SELECT COUNT(*), COUNT(DISTINCT name), AVG(price), SUM(price), MAX(price), MIN(price)
FROM orders;

SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```
- `WHERE` filters rows BEFORE grouping
- `HAVING` filters AFTER grouping (used with aggregates)

## Relational algebra (mental model)
- **Selection** σ — filter rows (`WHERE`)
- **Projection** π — pick columns (`SELECT col1, col2`)
- **Join** ⋈ — combine tables on condition

## Keys
- **Primary key** — uniquely identifies a row (often `INT id`)
- **Foreign key** — references another table's primary key
- **Composite key** — multiple cols form the unique identifier

## Transactions
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;        -- or ROLLBACK on error
```

## ACID
- **A**tomicity — all-or-nothing (no partial commit)
- **C**onsistency — DB stays in valid state (constraints hold)
- **I**solation — concurrent txns appear sequential
- **D**urability — once committed, survives crashes

### Bank transfer example
- Atomicity: BOTH updates happen or NEITHER
- Consistency: total money unchanged
- Isolation: other queries don't see mid-transfer state
- Durability: after commit, $ stays moved even if power dies

## NoSQL
- **Document stores** — MongoDB (JSON-like docs)
- **Key-value** — Redis, DynamoDB
- **Column family** — Cassandra, HBase
- **Graph** — Neo4j
- Trade strict ACID for scale + schema flexibility
- Often offer **eventual consistency** (not immediate)

## CAP Theorem
In a distributed system, you can have at most TWO of three:
- **C**onsistency — every read gets the most recent write
- **A**vailability — every request gets a response (success or failure)
- **P**artition tolerance — system works despite network splits

In practice, P is unavoidable in distributed systems → must choose **CP** or **AP**.

| System | Trade-off | Examples |
|---|---|---|
| **CA** | Consistency + Availability (single node only) | Traditional RDBMS, single-node |
| **CP** | Consistency + Partition tolerance | MongoDB (default), HBase |
| **AP** | Availability + Partition tolerance | Cassandra, DynamoDB |

## ATM example (CAP applied)
- During network partition between branches:
  - **CP** choice: refuse withdrawals (consistent, unavailable)
  - **AP** choice: allow withdrawals (available, may double-spend) → reconcile later

## Eventual Consistency
- Updates propagate over time
- Reads might temporarily return stale data
- Eventually, all nodes converge to the same state

## Indexing (concept)
- B-tree index speeds up lookups on indexed columns
- Trade-off: slower writes, more storage

## Normalization (BCNF/3NF, brief)
- Eliminate redundancy
- Each fact stored once
- Update anomalies avoided

## Vulnerabilities — connect to security
- **SQL injection** → use prepared statements
- **Unencrypted at rest** → encrypt sensitive columns
- **Excessive privilege** → least privilege per service

## Quick mental model
| Question | Answer in SQL |
|---|---|
| How many users? | `SELECT COUNT(*) FROM users` |
| Average order? | `SELECT AVG(amount) FROM orders` |
| Top 5 customers by spend | `SELECT user_id, SUM(amount) FROM orders GROUP BY user_id ORDER BY SUM(amount) DESC LIMIT 5` |
| Users with no orders | `SELECT u.* FROM users u LEFT JOIN orders o ON u.id=o.user_id WHERE o.id IS NULL` |

## Additional items (potentially missing)

### Constraints
- `PRIMARY KEY` — unique + not null
- `FOREIGN KEY (col) REFERENCES other(col)`
- `UNIQUE` — value must be unique across rows
- `NOT NULL` — required
- `CHECK (col > 0)` — validation predicate
- `DEFAULT 0` — default value

### CREATE TABLE example
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    amount DECIMAL(10,2) CHECK (amount > 0),
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Subqueries
```sql
-- in WHERE
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE amount > 100);

-- correlated (refers to outer query)
SELECT name, (SELECT COUNT(*) FROM orders WHERE user_id = u.id) AS order_count
FROM users u;
```

### VIEW (virtual table)
```sql
CREATE VIEW active_users AS
SELECT * FROM users WHERE status = 'active';

SELECT * FROM active_users;     -- query like a table
```

### INDEX
```sql
CREATE INDEX idx_email ON users(email);
CREATE UNIQUE INDEX idx_username ON users(username);
DROP INDEX idx_email;
```
- Speeds up SELECT on indexed columns
- Slows down INSERT/UPDATE/DELETE
- Each index costs disk space

### UNION vs UNION ALL
```sql
SELECT name FROM employees
UNION                       -- removes duplicates
SELECT name FROM contractors;

SELECT name FROM employees
UNION ALL                   -- keeps duplicates (faster)
SELECT name FROM contractors;
```

### Window functions (advanced but common)
```sql
SELECT name, salary,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    AVG(salary) OVER (PARTITION BY dept) AS dept_avg
FROM employees;
```

### Aggregate vs window
- Aggregate (`GROUP BY`) collapses rows
- Window keeps all rows but adds a computed col

### COUNT variants
- `COUNT(*)` — all rows
- `COUNT(col)` — rows where col IS NOT NULL
- `COUNT(DISTINCT col)` — unique non-null values

### NULL behavior
- `NULL = NULL` is NULL (not true!)
- Use `IS NULL` / `IS NOT NULL`
- `NULL` propagates through arithmetic: `NULL + 5 = NULL`
- `COALESCE(a, b, c)` returns first non-null

### CASE
```sql
SELECT name,
    CASE
        WHEN age < 18 THEN 'minor'
        WHEN age < 65 THEN 'adult'
        ELSE 'senior'
    END AS category
FROM users;
```

### LIMIT / OFFSET
```sql
SELECT * FROM users LIMIT 10;
SELECT * FROM users LIMIT 10 OFFSET 20;     -- pagination
```

### Common interview-style queries
- Top N per group
- Duplicates: `GROUP BY x HAVING COUNT(*) > 1`
- Users with no orders: `LEFT JOIN ... WHERE other.id IS NULL`
- Self-join (e.g., employees + their manager)

### Database normalization (1NF, 2NF, 3NF)
- **1NF** — each cell holds atomic (indivisible) value, no repeating groups
- **2NF** — 1NF + no partial dependency on composite key
- **3NF** — 2NF + no transitive dependency
- **BCNF** — stricter 3NF
- Goal: eliminate redundancy, prevent update anomalies

### Denormalization
- Sometimes intentional for performance
- Trade integrity for read speed
- Common in analytics / OLAP

### OLTP vs OLAP
- **OLTP** (Online Transaction Processing) — many small txns, normalized DB
- **OLAP** (Online Analytical Processing) — complex queries on aggregates, often denormalized (data warehouse)

### Transactions in code (Python example)
```python
conn.begin()
try:
    cursor.execute("UPDATE accounts SET ...")
    cursor.execute("INSERT INTO logs ...")
    conn.commit()
except:
    conn.rollback()
    raise
```

### Isolation levels (for ACID)
- READ UNCOMMITTED (lowest, dirty reads possible)
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE (highest, most strict)
- Trade-off: stricter = more locking = slower

### NoSQL types reminder
| Type | Examples | Use case |
|---|---|---|
| Document | MongoDB, CouchDB | flexible schemas, JSON-like |
| Key-Value | Redis, DynamoDB | caches, sessions |
| Column-Family | Cassandra, HBase | huge scale, analytics |
| Graph | Neo4j, ArangoDB | social networks, paths |

### Indexes — when NOT to index
- Tiny tables (full scan is fine)
- Columns with low cardinality (gender: M/F)
- Write-heavy columns where index overhead matters
