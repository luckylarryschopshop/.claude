---
name: code-quality
description: Clean, performant, portable code standards. Three-layer architecture
  (domain/services/API), OOP vs functional pragmatic rules, type safety, immutability,
  naming conventions, performance rules. Load when writing any service, domain
  function, or non-trivial implementation.
---

# Code Quality Skill

## Three-Layer Architecture

Every feature maintains strict separation across three layers.
This is the primary portability mechanism — core logic must be re-implementable
in any language by reading the domain layer alone.

### Layer 1 — Domain (pure functions only)
- Zero framework imports. Zero database imports. Zero I/O.
- Takes plain data in, returns plain data out.
- Uses frozen dataclasses or equivalent immutable structures.
- Can be copied into Swift, Kotlin, or TypeScript and re-implemented line by line.
- Location: `[project]/domain/` or `[project]/core/`

### Layer 2 — Services (orchestration)
- Calls domain functions. Handles DB sessions, file I/O, external APIs.
- Contains all side effects.
- Uses OOP or functions based on the pragmatic rule below.
- Location: `[project]/services/`

### Layer 3 — API / Interface (thin boundary)
- HTTP handlers, CLI commands, or UI event handlers only.
- No business logic. No direct DB access.
- Route handlers: 5–15 lines maximum.
- Location: `[project]/api/` or `[project]/handlers/`

---

## OOP vs Functional — Pragmatic Rule

**Use a class when:**
- Managing state across multiple operations (e.g. a service that caches a lookup table per session)
- Representing an entity with identity (User, Order, Account)
- Dependency injection makes testing significantly cleaner

**Use a pure function when:**
- The operation is a transformation: input → output
- No state needs to survive between calls
- The logic could be described as a mathematical function

**Document the choice inline — always:**
```python
# Class: caches lookup table across processing session. Pure logic in domain/matching.py.
class RecordMatcher:
    ...

# Function: single transformation, no session state needed.
def compute_status(value: float, limit: float) -> StatusResult:
    ...
```

Never use OOP to wrap a single function.
Never use a function when shared mutable state is genuinely needed.

---

## Domain Function Specification Standard

All domain functions must:

1. Be pure — same inputs always produce same outputs, no side effects
2. Use typed, immutable data structures for all inputs and outputs
3. Have complete type annotations
4. Have a docstring written as a **language-agnostic specification**:

```python
def compute_hash(
    date: str,       # ISO format: YYYY-MM-DD
    key: str,        # normalised identifier string
    amount: float,   # signed float
) -> str:
    """
    Compute a stable hash for deduplication.

    Algorithm:
      1. Lowercase and strip the key string
      2. Format amount to 2 decimal places with explicit sign: "-15.99" or "+5.00"
      3. Concatenate as: "{date}|{key}|{amount}"
      4. Return SHA-256 hex digest of the UTF-8 encoded string

    This specification is language-agnostic. Implementations in Swift,
    Kotlin, or TypeScript must produce identical hashes for identical inputs.

    Examples:
      compute_hash("2025-01-15", "acme corp", -15.99) → deterministic 64-char hex
    """
```

5. Raise ValueError (not return None) for invalid inputs, with a clear message
6. Have tests for: happy path, invalid input, each documented edge case

---

## Type Safety

- Complete type annotations on all functions
- No bare `Any` types
- API request/response models: use a validation library (Pydantic v2 / Zod / Codable)
- Validation models are API-layer only — never pass them into domain functions
- Run type checker in strict mode on domain/core layer
- Type checker must pass before a phase is marked complete

---

## Immutability

- Domain data structures are frozen/immutable after creation
- Services create new instances rather than mutating existing ones
- The only mutable objects are ORM/DB model instances — they never leave the services layer
- Prefer `const` / `let` over `var`. Prefer value types over reference types where possible.

---

## Performance Rules

- All DB queries filter on indexed columns. Declare indexes explicitly in schema.
  Document which queries each index serves in a comment above the index definition.
- N+1 queries are forbidden. Use joins or eager loading.
  If a query could produce N+1, flag it with a comment and fix it before committing.
- Pagination required on all list endpoints:
  Response shape: `{ items, total, page, page_size, pages }`
- Bulk operations must be a single UPDATE/INSERT query, not a loop of individual operations.
- Stream large files row-by-row — do not load entire file into memory.
- Domain functions prefer generators over lists where the caller iterates.

---

## Naming Conventions

| What | Convention | Examples |
|---|---|---|
| Functions | verb_noun | `compute_hash`, `apply_rules`, `validate_record` |
| Classes | NounNoun | `RecordMatcher`, `DataPipeline`, `StatusCalculator` |
| Constants | UPPER_SNAKE | `MATCH_THRESHOLD`, `DEFAULT_CHUNK_SIZE`, `MAX_RETRIES` |
| Files | snake_case | `record_matcher.py`, `status_calculator.py` |
| Tests | test_[module]_[what]_[condition] | `test_hash_same_inputs_returns_same_value` |
| Swift types | UpperCamelCase | `RecordStore`, `StatusResult` |
| Swift functions | lowerCamelCase | `computeHash`, `applyRules` |

---

## Inline Comment Standards

Comment WHY, not WHAT. The code shows what — comments explain intent.

```python
# Good: explains a non-obvious decision
# Skip manual overrides — user decisions take priority over automated rules
if record.source == "manual":
    continue

# Bad: restates the code
# Loop through records
for record in records:
```

Required for:
- All threshold values and magic numbers
- All algorithm choices (why this one, not another)
- All regex patterns (what they match and why)
- Any non-obvious data transformation
- Any place where the obvious implementation was rejected

Constants always have an explanatory comment:
```python
# Minimum confidence score to apply a match automatically without user review.
# Set conservatively to avoid incorrect merges on short or ambiguous strings.
SILENT_MERGE_THRESHOLD = 85.0
```

---

## "Staff Engineer" Test

Before marking any task complete, ask:
- Would a staff engineer approve this without change?
- Is there a simpler way to achieve the same result?
- Does this introduce any new failure modes?
- Is the error handling complete and informative?
- Would someone unfamiliar with this codebase understand it in 5 minutes?

If any answer is no: fix it before committing.
