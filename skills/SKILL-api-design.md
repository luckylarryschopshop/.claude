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
Generate docs/openapi.json. Validates operationId + 200 response schema on every route.
Exits 1 if any route fails either check.

Add to sys.path so the script is runnable directly (python backend/scripts/generate_openapi.py)
without needing PYTHONPATH set externally.
"""
import json, sys
from pathlib import Path

_project_root = Path(__file__).parent.parent.parent
if str(_project_root) not in sys.path:
    sys.path.insert(0, str(_project_root))

from backend.app.main import app  # noqa: E402

schema = app.openapi()
errors = []
for path, methods in schema.get("paths", {}).items():
    for method, op in methods.items():
        if method not in ("get", "post", "put", "patch", "delete"):
            continue
        if "operationId" not in op:
            errors.append(f"Missing operationId: {method.upper()} {path}")
        content = op.get("responses", {}).get("200", {}).get("content", {})
        if not content:
            errors.append(f"Missing response_model: {method.upper()} {path}")

if errors:
    for e in errors: print(e)
    sys.exit(1)

out = Path(__file__).parent.parent.parent / "docs" / "openapi.json"
out.parent.mkdir(parents=True, exist_ok=True)
out.write_text(json.dumps(schema, indent=2))
print(f"openapi.json written — {len(schema.get('paths', {}))} paths")
```

Key points:
- Use `schema["paths"]` (JSON introspection) not `app.routes` attribute access. FastAPI
  may rewrite operation metadata during schema generation — the JSON result is authoritative.
- Set `sys.path` explicitly so the script is runnable directly without `PYTHONPATH`.

This script runs in CI and blocks merges if any route is undocumented.

### Pydantic v2: aliasing reserved words in JSON output

When a response field must be named a Python keyword (e.g. `global`):
- Use `Field(serialization_alias="global")` on the field
- Add `model_config = {"populate_by_name": True}` on the model
- Add `response_model_by_alias=True` on the route decorator

```python
class BudgetStatusOut(BaseModel):
    global_status: BudgetStatusItem = Field(serialization_alias="global")
    model_config = {"populate_by_name": True}

@router.get("/budget/status", response_model=BudgetStatusOut,
            response_model_by_alias=True, ...)
```

---

## Pagination

Required on all list endpoints. Standard response envelope:

```python
from typing import Generic, TypeVar
T = TypeVar("T")

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
