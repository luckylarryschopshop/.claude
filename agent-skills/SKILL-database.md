---
name: database
description: Database engineering methodology — schema design, indexing, migrations, query optimisation. Load when acting as Database agent.
---

# Database Skill

## Core Methodology

### Schema Design Principles
1. **Query-driven design** — start with the queries you must serve efficiently; work backward to schema
2. **Normalise first, denormalise when measured** — not the reverse
3. **Constraints at the database level** — do not rely on application code to maintain invariants
4. **Explicit nullability** — every column should be NOT NULL unless null is a meaningful value
5. **Immutable primary keys** — never use a business key (email, username) as a primary key

### Primary Key Strategy
| Type | Use when | Example |
|------|----------|---------|
| `UUID v4` (random) | Distributed systems, no ordering needed | `id UUID DEFAULT gen_random_uuid()` |
| `UUID v7` (time-ordered) | Need ordering by creation time | `id UUID DEFAULT gen_uuid_v7()` |
| `BIGSERIAL` | Single-node, need compact int FK, high insert rate | `id BIGSERIAL PRIMARY KEY` |
| Natural key | Truly immutable external identifier | ISO country code, IATA airport code |

### Indexing Strategy
- **Index foreign keys** — always; missing FK indexes cause table scans on joins
- **Index query predicates** — columns in WHERE, JOIN ON, and ORDER BY that appear in slow queries
- **Partial indexes** — when most rows satisfy a filter (`WHERE deleted_at IS NULL`)
- **Composite index column order**: most selective column first; columns in WHERE before ORDER BY
- **EXPLAIN ANALYZE before and after** — measure, don't guess

Index bloat warning: indexes slow writes. Add only indexes with a measured query justification.

### Migration Standards
```sql
-- Migration: V{N}__{description}.sql (Flyway) or {timestamp}_{description} (Alembic)
-- Always write both UP and DOWN

-- UP
ALTER TABLE orders ADD COLUMN coupon_code VARCHAR(50);
CREATE INDEX idx_orders_coupon_code ON orders (coupon_code) WHERE coupon_code IS NOT NULL;

-- DOWN
DROP INDEX idx_orders_coupon_code;
ALTER TABLE orders DROP COLUMN coupon_code;
```

Rules:
- Every migration is reversible — write the DOWN migration at the same time as UP
- Never modify a migration once it has been applied to any environment
- Test migration on a production-sized dataset before applying (table size matters)
- For large tables: use `ADD COLUMN DEFAULT NULL` first (instant), then backfill, then add constraint

### Query Optimisation Process
1. `EXPLAIN ANALYZE [query]` — identify the expensive node
2. Check for Seq Scan on large tables — usually needs an index
3. Check for high row estimates vs actual — stale statistics (`ANALYZE [table]`)
4. Check for nested loop with high iterations — may need an index or query rewrite
5. Check for sort operations — if sorting is required, index on the sort key

### Data Classification and Retention
Classify every column at schema design time:
| Class | Examples | Storage | Retention |
|-------|----------|---------|-----------|
| PII | name, email, phone, IP address | Encrypted at rest | Delete on request + legal min |
| Financial | amounts, account numbers | Encrypted at rest | 7 years (typical legal) |
| Operational | order status, timestamps | Standard | Per product requirement |
| Public | product names, categories | Standard | Indefinite |

### PostgreSQL Defaults
```sql
-- Always set on new PostgreSQL databases:
SET default_transaction_isolation = 'read committed'; -- default, but explicit
SET timezone = 'UTC'; -- always store in UTC
GRANT CONNECT ON DATABASE [db] TO [app_user];
REVOKE ALL ON DATABASE [db] FROM PUBLIC;
```

---

### Transaction Isolation Levels

Choose the correct isolation level — wrong choice causes phantom reads, lost updates, or
unnecessary lock contention.

| Level | Prevents | Allows | Use when |
|-------|----------|--------|----------|
| **Read Uncommitted** | — | Dirty reads, phantom reads | Almost never — don't use |
| **Read Committed** (PostgreSQL default) | Dirty reads | Non-repeatable reads, phantom reads | Most application reads; safe default |
| **Repeatable Read** | Dirty reads, non-repeatable reads | Phantom reads (in most DBs; PG prevents them too) | Reports that read the same row multiple times in one transaction |
| **Serializable** | All anomalies | Nothing — full isolation | Financial calculations, inventory deductions, anything where concurrent writes produce incorrect results |

**Decision guide:**
- Single-row reads/writes → Read Committed (default)
- Multi-row read then write (e.g. check balance, then deduct) → Serializable or explicit `SELECT ... FOR UPDATE`
- Long-running report reading many rows → Repeatable Read
- Never use Read Uncommitted in production

```sql
-- Explicit serializable transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  SELECT balance FROM accounts WHERE id = $1 FOR UPDATE;
  UPDATE accounts SET balance = balance - $2 WHERE id = $1;
COMMIT;
```

**Handle serialization failures:** at SERIALIZABLE level, PostgreSQL may abort a transaction
with `ERROR 40001: could not serialize access`. Application must retry these.

---

### Connection Pooling

**Never open a DB connection per request without a pool.** Raw connections are expensive
(~10MB RAM each, ~50ms setup cost).

**Pool sizing formula (PgBouncer / SQLAlchemy):**
```
pool_size = (num_cpu_cores × 2) + num_disk_spindles
# For most cloud VMs (SSD storage): num_disk_spindles = 1
# 4-core machine → pool_size = 9
```

**PostgreSQL hard limit:** set `max_connections` in `postgresql.conf` to 100–200 for most apps.
Never exceed 500 — connection overhead degrades performance above this.

**PgBouncer modes:**
| Mode | Use when |
|------|----------|
| **Session pooling** | Persistent connections required (prepared statements, `SET` vars) |
| **Transaction pooling** (recommended) | Stateless queries; highest connection reuse |
| **Statement pooling** | Only single-statement transactions; most restrictive |

**Connection pool config checklist:**
- [ ] Pool size calculated from CPU/disk formula
- [ ] `pool_timeout` set (don't wait indefinitely for a connection)
- [ ] `pool_recycle` set (recycle connections after N seconds to prevent stale conn errors)
- [ ] `pool_pre_ping=True` (SQLAlchemy) — test connection before use, handle server-side timeouts

---

### Table Partitioning

Use when a single table exceeds ~50M rows or when you need fast bulk drops of old data.

| Strategy | Partition by | Use when |
|----------|-------------|----------|
| **Range** | Date/time column | Time-series data; drop old partitions instead of DELETE |
| **List** | Enum/category column | Known set of values; queries always filter by that column |
| **Hash** | Any column | Even distribution needed; no natural range or list |

```sql
-- Range partitioning by month (PostgreSQL declarative)
CREATE TABLE events (
  id         BIGSERIAL,
  event_type VARCHAR(50),
  created_at TIMESTAMPTZ NOT NULL
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2024_01 PARTITION OF events
  FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE events_2024_02 PARTITION OF events
  FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Drop a whole month instantly (no DELETE scan needed)
DROP TABLE events_2024_01;
```

**Partitioning rules:**
- The partition key must be included in every query's WHERE clause — otherwise PostgreSQL
  scans all partitions (partition pruning doesn't help)
- Create indexes on each partition, not just the parent table
- Use `pg_partman` extension to automate partition creation and retention
- Don't partition tables under 10M rows — overhead outweighs benefit

**MVCC (how PostgreSQL avoids read locks):**
PostgreSQL uses Multi-Version Concurrency Control: readers never block writers, writers never
block readers. Each transaction sees a snapshot of the DB at its start time. Old row versions
are cleaned up by VACUUM. If VACUUM can't run (long-running transactions, autovacuum disabled),
table bloat accumulates. Monitor: `pg_stat_user_tables.n_dead_tup`.
