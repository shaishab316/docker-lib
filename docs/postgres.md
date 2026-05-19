# PostgreSQL Cheatsheet

---

## 1. Connect

```bash
# Enter CLI
docker exec -it postgres psql -U postgres

# Connect to specific database
docker exec -it postgres psql -U postgres -d mydb
```

---

## 2. Database

```sql
CREATE DATABASE mydb;
DROP DATABASE mydb;
\l                         -- list databases
\c mydb                    -- switch database
SELECT current_database();
```

---

## 3. Users & Permissions

```sql
CREATE USER john WITH PASSWORD 'secret';
ALTER USER postgres WITH PASSWORD 'newpass';
DROP USER john;

GRANT ALL PRIVILEGES ON DATABASE mydb TO john;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO john;
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM john;

\du                        -- list users
```

---

## 4. Tables

```sql
-- Create
CREATE TABLE users (
  id        SERIAL PRIMARY KEY,
  name      VARCHAR(100)        NOT NULL,
  email     VARCHAR(255) UNIQUE NOT NULL,
  status    VARCHAR(20)         DEFAULT 'active',
  created_at TIMESTAMP          DEFAULT NOW()
);

-- Modify
ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users RENAME COLUMN name TO full_name;
ALTER TABLE users ALTER COLUMN name SET NOT NULL;

-- Delete
DROP TABLE users;
TRUNCATE TABLE users;      -- wipe all rows, keep table

-- Inspect
\dt                        -- list tables
\d users                   -- describe table
```

---

## 5. CRUD

```sql
-- Insert
INSERT INTO users (name, email) VALUES ('John', 'john@example.com');

-- Insert multiple
INSERT INTO users (name, email) VALUES
  ('Alice', 'alice@example.com'),
  ('Bob',   'bob@example.com');

-- Select
SELECT * FROM users;
SELECT id, name FROM users WHERE id = 1;
SELECT * FROM users ORDER BY created_at DESC LIMIT 10 OFFSET 20;

-- Update
UPDATE users SET name = 'Jane' WHERE id = 1;

-- Delete
DELETE FROM users WHERE id = 1;
```

---

## 6. Filtering

```sql
WHERE age > 18
WHERE age BETWEEN 18 AND 30
WHERE status IN ('active', 'pending')
WHERE status NOT IN ('banned')
WHERE name LIKE '%john%'         -- contains (case sensitive)
WHERE name ILIKE '%john%'        -- contains (case insensitive)
WHERE deleted_at IS NULL
WHERE deleted_at IS NOT NULL
```

---

## 7. Joins

```sql
-- Inner Join — only matching rows
SELECT u.name, o.total
FROM users u
JOIN orders o ON o.user_id = u.id;

-- Left Join — all users, even with no orders
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;

-- Right Join — all orders, even with no user
SELECT u.name, o.total
FROM users u
RIGHT JOIN orders o ON o.user_id = u.id;
```

---

## 8. Aggregations

```sql
SELECT COUNT(*)                        FROM users;
SELECT COUNT(*) FILTER (WHERE status = 'active') FROM users;
SELECT AVG(price), MAX(price), MIN(price), SUM(price) FROM products;

-- Group
SELECT status, COUNT(*)
FROM users
GROUP BY status;

-- Group with condition
SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id
HAVING SUM(total) > 100;
```

---

## 9. Indexes

```sql
CREATE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);
DROP INDEX idx_users_email;

\di                        -- list indexes
```

---

## 10. Transactions

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- On error
ROLLBACK;
```

---

## 11. Run SQL File

```bash
# From host
docker exec -i postgres psql -U postgres -d mydb < file.sql

# From inside container
docker exec -it postgres bash
psql -U postgres -d mydb -f /path/to/file.sql
```

---

## 12. Backup & Restore

```bash
# Dump single database
docker exec postgres pg_dump -U postgres mydb > backup.sql

# Dump all databases
docker exec postgres pg_dumpall -U postgres > all.sql

# Restore
docker exec -i postgres psql -U postgres -d mydb < backup.sql
```

---

## 13. Useful Debug Queries

```sql
-- Active queries
SELECT pid, state, query FROM pg_stat_activity WHERE state = 'active';

-- Kill a query
SELECT pg_terminate_backend(1234);

-- Table size
SELECT pg_size_pretty(pg_total_relation_size('users'));

-- Database size
SELECT pg_size_pretty(pg_database_size('mydb'));

-- Check locks
SELECT * FROM pg_locks;
```

---

## 14. Meta Commands (inside psql)

| Command       | Description                     |
|---------------|---------------------------------|
| `\l`          | List databases                  |
| `\c mydb`     | Connect to database             |
| `\dt`         | List tables                     |
| `\d users`    | Describe table                  |
| `\di`         | List indexes                    |
| `\du`         | List users                      |
| `\x`          | Toggle expanded row display     |
| `\timing`     | Show query execution time       |
| `\e`          | Open query in editor            |
| `\i file.sql` | Run a SQL file                  |
| `\q`          | Quit                            |

---

## 15. Common Data Types

| Type           | Use For                        |
|----------------|--------------------------------|
| `SERIAL`       | Auto-increment integer (PK)    |
| `BIGSERIAL`    | Large auto-increment           |
| `INT`          | Integer                        |
| `BIGINT`       | Large integer                  |
| `VARCHAR(n)`   | String with max length         |
| `TEXT`         | Unlimited string               |
| `BOOLEAN`      | true / false                   |
| `NUMERIC(p,s)` | Exact decimal (money)          |
| `FLOAT`        | Approximate decimal            |
| `TIMESTAMP`    | Date + time (no timezone)      |
| `TIMESTAMPTZ`  | Date + time (with timezone)    |
| `JSONB`        | JSON (indexable, preferred)    |
| `UUID`         | UUID                           |