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
