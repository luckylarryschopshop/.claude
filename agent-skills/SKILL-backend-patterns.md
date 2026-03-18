---
name: backend-patterns
description: Backend engineering patterns — error response standards, idempotency,
  pagination, API versioning, background jobs, rate limiting. Load when acting as
  Backend agent or implementing any server-side API.
---

# Backend Patterns Skill

## 1. Error Response Standard — RFC 7807 Problem Details

Every API error response must follow RFC 7807 `application/problem+json`. No ad-hoc error schemas.

```json
{
  "type": "https://example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "The 'amount' field must be a positive number.",
  "instance": "/api/orders/",
  "errors": [
    { "field": "amount", "message": "Must be > 0", "code": "range_error" }
  ]
}
```

Rules:
- `type`: URI identifying the error class (stable, documentable)
- `title`: human-readable, does not change per-instance
- `status`: HTTP status code (matches the actual HTTP status)
- `detail`: instance-specific explanation — what went wrong THIS time
- `instance`: the request URI that caused the error
- `errors[]`: field-level detail for validation errors (optional extension)

**Do not** return `{"error": "something went wrong"}` — untypeable, unsearchable, unsupportable.

Status code conventions:
| Situation | Status |
|-----------|--------|
| Input validation failure | 422 |
| Resource not found | 404 |
| Auth not provided | 401 |
| Auth provided but insufficient permissions | 403 |
| Conflict with existing state | 409 |
| Rate limit exceeded | 429 |
| Upstream dependency failed | 502 |
| Server error (unexpected) | 500 |

---

## 2. Idempotency

### When idempotency is required
Any operation that: creates a resource, charges money, sends a message, or triggers an
irreversible side effect. Safe methods (GET, HEAD) are inherently idempotent.

### Idempotency key pattern
```
POST /charges
Idempotency-Key: <client-generated UUID>
```

Server behaviour:
1. Hash the key → look up in `idempotency_keys` table
2. If found and request fingerprint matches → return cached response (no re-execution)
3. If found but fingerprint differs → return 422 (key reuse with different payload)
4. If not found → execute, store `{key, request_fingerprint, response, created_at}`
5. Expire keys after 24 hours (configurable)

```sql
CREATE TABLE idempotency_keys (
  key         VARCHAR(255) PRIMARY KEY,
  fingerprint VARCHAR(64)  NOT NULL,   -- SHA-256 of method+path+body
  response    JSONB        NOT NULL,
  status_code SMALLINT     NOT NULL,
  created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
  expires_at  TIMESTAMPTZ  NOT NULL
);
CREATE INDEX idx_idempotency_expires ON idempotency_keys (expires_at);
```

### At-least-once vs exactly-once
- **At-least-once** (message queues): consumers must be idempotent — deduplicate by message ID
- **Exactly-once** (transactions): use DB transactions with idempotency key check inside the transaction

---

## 3. Pagination

### Cursor pagination (preferred for feeds, infinite scroll, large datasets)

```json
GET /orders?after=eyJpZCI6MTIzfQ==&limit=20

{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTQzfQ==",
    "has_more": true,
    "limit": 20
  }
}
```

Cursor is an opaque base64-encoded pointer (e.g. `{"id": 123, "created_at": "2024-01-15T10:00:00Z"}`).
Never expose raw DB IDs in cursors — encode them.

Rules:
- Cursors are stable: inserting new rows doesn't shift pages
- Never accept page numbers from clients for large datasets (skip/offset scans degrade at scale)
- Cursor encodes the sort key(s) — if sorting by `created_at DESC`, cursor must contain `created_at`

### Offset pagination (acceptable for admin UIs, small bounded datasets)

```json
GET /admin/users?page=3&per_page=25

{
  "data": [...],
  "pagination": {
    "page": 3,
    "per_page": 25,
    "total": 847,
    "total_pages": 34
  }
}
```

Use when: UI requires "jump to page N", dataset is bounded (<10k rows), consistency between
pages is not critical. Avoid for feeds or any table that changes frequently.

### Decision matrix
| Criteria | Cursor | Offset |
|----------|--------|--------|
| Large dataset (>10k rows) | ✓ | ✗ (slow at high offsets) |
| Stable results under inserts | ✓ | ✗ |
| "Jump to page N" UI | ✗ | ✓ |
| Total count needed | Manual count query | ✓ |

---

## 4. API Versioning

### URL path versioning (recommended default)
```
/api/v1/orders
/api/v2/orders   ← breaking changes go here
```

Rules:
- Increment major version ONLY for breaking changes (removed fields, changed types, new required fields)
- Additive changes (new optional fields, new endpoints) are non-breaking — no version bump
- Maintain v(N-1) for minimum 6 months after v(N) launches
- Add `Deprecation` and `Sunset` headers to deprecated version responses:
  ```
  Deprecation: true
  Sunset: Sat, 31 Dec 2025 23:59:59 GMT
  Link: <https://api.example.com/v2/orders>; rel="successor-version"
  ```

### Header versioning (alternative — cleaner URLs, harder to test in browser)
```
Accept: application/vnd.example.v2+json
```

### What counts as a breaking change
| Breaking | Not breaking |
|----------|-------------|
| Removing a field | Adding an optional field |
| Renaming a field | Adding a new endpoint |
| Changing a field type | Adding new enum values (if clients handle unknown values) |
| Making an optional field required | Changing error message text |
| Changing authentication scheme | Adding new optional query params |

---

## 5. Background Job Patterns

### Retry with exponential backoff
```python
def calculate_delay(attempt: int, base: float = 1.0, max_delay: float = 300.0) -> float:
    """Exponential backoff with jitter."""
    delay = min(base * (2 ** attempt), max_delay)
    jitter = delay * 0.1 * random.random()
    return delay + jitter

# attempt 0 → ~1s, 1 → ~2s, 2 → ~4s, 3 → ~8s ... max 300s
```

Standard retry policy:
- Max attempts: 5 (configurable per job type)
- Backoff: exponential with jitter (prevents thundering herd)
- Non-retryable errors: 4xx (bad input — retrying won't help), mark as `failed` immediately
- Retryable errors: 5xx, timeouts, transient network errors

### Dead letter queue (DLQ)
After max retries exhausted → move to DLQ (not deleted).
DLQ jobs: alert ops, inspect payload, replay manually when root cause fixed.

```
job_queue → worker → [success: done] | [retryable failure: requeue with delay] | [fatal: DLQ]
```

### Job visibility and observability
Every job must log:
```json
{
  "job_id": "uuid",
  "job_type": "send_invoice",
  "attempt": 2,
  "status": "retrying",
  "error": "ConnectionTimeout",
  "next_retry_at": "2024-01-15T10:05:00Z",
  "payload_summary": {"order_id": 456}   // ← no PII/secrets in logs
}
```

### Idempotency in consumers
Every job handler must be idempotent — it may run more than once (at-least-once delivery).
Deduplicate using job ID stored in a processed-jobs table or Redis set.

---

## 6. Rate Limiting

### Token bucket (recommended — handles bursts gracefully)
Each client gets a bucket of N tokens. Each request consumes 1 token. Tokens refill at R/second.

```python
# Redis-backed token bucket
def check_rate_limit(client_id: str, capacity: int = 100, refill_rate: float = 10.0) -> bool:
    """Returns True if request is allowed."""
    # Use Redis EVALSHA with atomic Lua script for correctness
    ...
```

### Sliding window log (precise, high memory cost)
Track timestamp of each request. Count requests in the last N seconds.
Use for: low-traffic critical APIs where precision matters.

### Response headers (always include)
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1705315200    ← Unix timestamp when bucket resets
Retry-After: 42                  ← only on 429 response
```

### Rate limit dimensions (apply the narrowest first)
1. Per IP (unauthenticated endpoints)
2. Per user/token (authenticated endpoints)
3. Per endpoint (expensive operations get tighter limits)
4. Global (circuit breaker for entire service)

### 429 response body
```json
{
  "type": "https://example.com/errors/rate-limit-exceeded",
  "title": "Rate Limit Exceeded",
  "status": 429,
  "detail": "You have exceeded 100 requests per minute. Retry after 42 seconds.",
  "retry_after": 42
}
```
