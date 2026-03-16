---
name: api-design
description: REST API design standards, OpenAPI spec requirements, pagination,
  error response format, route handler discipline, and iOS portability contract.
  Load when writing API routes, generating OpenAPI specs, or designing endpoints.
---

# API Design Skill

## Route Handler Discipline

Route handlers are thin wrappers. 5–15 lines maximum.
No business logic. No direct DB access.
Pattern: validate input → call service → serialise response.

The example below is Python/FastAPI. The principle applies to any framework —
Express, Hono, Vapor (Swift), Ktor (Kotlin) — the shape should be the same.

```python
# Python/FastAPI example — principle is framework-agnostic
@router.get("/transactions", response_model=TransactionListResponse,
            operation_id="get_transaction_list",
            summary="List transactions with optional filters")
async def get_transaction_list(
    month: str | None = Query(None, description="Filter by month (YYYY-MM)"),
    category: str | None = Query(None),
    page: int = Query(1, ge=1),
    page_size: int = Query(50, ge=1, le=200),
    session: Session = Depends(get_session),  # FastAPI dependency injection
) -> TransactionListResponse:
    return transaction_service.list_transactions(
        session, month=month, category=category, page=page, page_size=page_size
    )
```

---

## OpenAPI Requirements

Every route must have:
- `operation_id`: unique, verb-noun format — `get_transaction_list`, `patch_transaction_category`
- `summary`: one plain-English sentence
- `response_model`: explicit Pydantic/schema model — never `dict`, never `Any`
- All error responses documented

Every Pydantic response model field must have:
- `description`: what this field represents
- `example`: illustrative value where helpful

```python
class TransactionResponse(BaseModel):
    id: str = Field(..., description="UUID of the transaction", example="abc-123")
    date: str = Field(..., description="Transaction date (YYYY-MM-DD)", example="2025-01-15")
    amount: float = Field(..., description="Signed amount. Negative=debit, positive=credit",
                          example=-15.99)
    category: str = Field(..., description="Assigned category", example="Subscriptions")
```

---

## OpenAPI Spec Generation

Generate and commit `docs/openapi.json` at the end of the API phase:

```python
# scripts/generate_openapi.py
"""
Generates docs/openapi.json from the live app.
Validates that all routes have operation_id and response_model.
Exits with error if any route is missing either — CI enforcer.
"""
import json, sys
from app.main import app

missing = []
for route in app.routes:
    if not hasattr(route, "methods"):
        continue  # skip non-HTTP routes (websockets, mounts)
    op_id = getattr(route, "operation_id", None)
    response_model = getattr(route, "response_model", None)
    if not op_id:
        missing.append(f"Missing operation_id: {route.path} {list(route.methods)}")
    if not response_model:
        missing.append(f"Missing response_model: {route.path} {list(route.methods)}")

if missing:
    for m in missing: print(m)
    sys.exit(1)

with open("docs/openapi.json", "w") as f:
    json.dump(app.openapi(), f, indent=2)
print(f"openapi.json generated — {len(app.routes)} routes documented")
```

This script runs in CI and blocks merges if any route is undocumented.

---

## Pagination

Required on all list endpoints. Standard response envelope:

```python
class PaginatedResponse(BaseModel, Generic[T]):
    items: list[T]
    total: int = Field(..., description="Total matching records")
    page: int = Field(..., description="Current page (1-indexed)")
    page_size: int = Field(..., description="Items per page")
    pages: int = Field(..., description="Total pages")
```

Query params: `page` (default 1), `page_size` (default 50, max 200).
DB query must use LIMIT/OFFSET — never load all records.

---

## Error Response Format

All errors return a consistent shape:

```python
class ErrorResponse(BaseModel):
    error: str = Field(..., description="Machine-readable error code",
                        example="duplicate_import")
    message: str = Field(..., description="Human-readable description",
                          example="This file has already been imported")
    detail: dict | None = Field(None, description="Additional context")
```

Standard status codes:
- `400` — Invalid request (malformed input, validation failure)
- `404` — Resource not found
- `409` — Conflict (duplicate import, already exists)
- `422` — Unprocessable entity (input validation failure)
- `501` — Not implemented (stub endpoints)
- `500` — Unexpected server error (log it, return generic message)

Never return raw exception messages to the client.
Internal error details go to the log, not the response.

---

## iOS Portability Contract

The committed `docs/openapi.json` is the contract between the Python backend
and any future Swift/Kotlin client.

Requirements for iOS compatibility:
- All UUIDs as strings (not binary)
- All dates as ISO strings (YYYY-MM-DD), never timestamps
- All amounts as floats with 2 decimal precision
- No Python-specific types in responses
- Multipart upload endpoints documented with explicit content-type

Documenting for Swift client generation in `docs/ios-client.md`:
```markdown
## Generating the Swift Client

1. Regenerate the spec: `python scripts/generate_openapi.py`
2. Run the generator: `swift-openapi-generator generate --input docs/openapi.json`
3. The generated client is type-safe and matches all endpoint signatures.

### Special handling required
- POST /api/v1/import/upload: multipart/form-data — see swift-openapi multipart docs
- Passphrase header: add X-Passphrase header to all requests (not in spec — local only)
```

---

## Versioning

All routes under `/api/v1/`. Version in URL, not header.
When a breaking change is required: add `/api/v2/` routes alongside v1.
Never remove or modify v1 routes — iOS clients may not have updated.
Document version differences in `docs/api-reference.md`.

---

## Prompt Caching (Anthropic API)

For AI features that send the same system prompt on every call, use
`cache_control` to avoid re-billing the static portion each time.

```python
# services/advice_builder.py
import anthropic

def call_advice_api(system_prompt: str, user_payload: str) -> str:
    # Lazy init — only instantiate when called, so missing API key at startup
    # (e.g. when FEATURE_ADVICE_ENABLED=false) does not crash the application.
    client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from environment
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=4096,  # advice + forecast JSON can be substantial; 1000 is too tight
        system=[
            {
                "type": "text",
                "text": system_prompt,
                "cache_control": {"type": "ephemeral"},  # cache the static system prompt
            }
        ],
        messages=[{"role": "user", "content": user_payload}],
    )
    return response.content[0].text
```

Rules:
- Only cache content that does not change between calls (system prompt, persona, rules)
- Never cache the user payload — it changes every call
- `"ephemeral"` is currently the only supported cache type
- Caching requires the system prompt to be at least 1024 tokens to be effective
- ANTHROPIC_API_KEY must be in `.env`, never hardcoded — the client reads it automatically
