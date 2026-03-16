---
name: tdd
description: Test-driven development workflow, test naming conventions, fixture
  standards, test ID format, and the test-first discipline. Load when writing
  tests or implementing against a test specification.
---

# TDD Skill

## The Discipline

Tests are written BEFORE implementation. Every time. No exceptions.

The sequence is always:
1. Write the test (it fails — that is correct and expected)
2. Commit the failing test: `test(phaseN): add [module] test skeletons`
3. Implement the minimum code to make it pass
4. Refactor if needed — tests must still pass
5. Commit: `feat(phaseN): implement [module] — [TEST-ID] through [TEST-ID] pass`

If you find yourself writing implementation before tests: stop, write the test first.
If a test is hard to write, that is a signal the design needs simplifying.

---

## Test Naming Convention

```
test_[module]_[what]_[condition]
```

Examples:
```python
test_dedup_same_row_twice_returns_single_record
test_dedup_different_merchant_same_amount_creates_two_records
test_provision_amount_at_tolerance_boundary_is_prefunded
test_provision_amount_over_tolerance_is_not_prefunded
test_budget_status_under_global_over_category_headline_is_green
```

The name must be readable as a sentence describing the behaviour.
Abbreviations are allowed for well-known domain terms (tx, dedup, provision).

---

## Test ID Format

Test IDs allow tracing between spec and implementation.

Format: `[MODULE]-[TYPE]-[NN]`

Where TYPE is:
- `DOMAIN` — tests of pure domain functions
- (no type) — integration/service/pipeline tests

Examples: `DEDUP-DOMAIN-01`, `PARSE-RH-03`, `BUDG-01`, `FORE-DOMAIN-02`

Every test function must reference its ID in a comment:
```python
def test_dedup_same_row_twice_returns_single_record():
    # DEDUP-01
    ...
```

---

## Fixture Standards

- All fixtures use fake data only — never real financial data, never real names
- Fixture files live in `tests/fixtures/`
- CSV fixtures must match the real column structure of their institution
- Use realistic-looking but obviously fake values:
  - Merchants: "FAKE COFFEE SHOP", "TEST GROCERY STORE", "SAMPLE STREAMING CO"
  - Amounts: round numbers or simple decimals (-12.99, -4.50, +1000.00)
  - Dates: use a consistent fake date range (2024-01-01 through 2024-03-31)

```
tests/
  fixtures/
    robinhood_gold_sample.csv     ← realistic column structure, fake data
    bank_of_america_sample.csv    ← realistic column structure, fake data
    category_rules_sample.yaml    ← sample rules for categorisation tests
```

CONTRIBUTING.md must state: "Test fixtures must never contain real financial data."

---

## Domain Function Test Requirements

Every domain function must be tested for:

1. **Happy path** — correct output for valid, typical input
2. **Boundary values** — 0, negative numbers, maximum reasonable values
3. **Invalid input** — wrong types, None, empty string → raises ValueError with clear message
4. **Determinism** — same inputs called twice → identical output (critical for hash functions)
5. **Edge cases** — every edge case documented in the function's spec

```python
class TestComputeDedupHash:
    def test_happy_path_returns_hex_string(self):
        # DEDUP-DOMAIN-01
        result = compute_dedup_hash("2024-01-15", "netflix", -15.99)
        assert len(result) == 64  # SHA-256 hex digest
        assert all(c in "0123456789abcdef" for c in result)

    def test_deterministic_same_inputs_same_output(self):
        # DEDUP-DOMAIN-01 (determinism)
        a = compute_dedup_hash("2024-01-15", "netflix", -15.99)
        b = compute_dedup_hash("2024-01-15", "netflix", -15.99)
        assert a == b

    def test_different_merchant_produces_different_hash(self):
        # DEDUP-DOMAIN-02
        a = compute_dedup_hash("2024-01-15", "netflix", -15.99)
        b = compute_dedup_hash("2024-01-15", "spotify", -15.99)
        assert a != b

    def test_amount_sign_matters(self):
        # DEDUP-DOMAIN-03
        debit = compute_dedup_hash("2024-01-15", "netflix", -15.99)
        credit = compute_dedup_hash("2024-01-15", "netflix", +15.99)
        assert debit != credit

    def test_empty_merchant_raises_value_error(self):
        with pytest.raises(ValueError, match="merchant"):
            compute_dedup_hash("2024-01-15", "", -15.99)
```

---

## Integration Test Requirements

For service/pipeline tests:

1. **Single responsibility** — each test tests one behaviour
2. **Isolated** — tests do not depend on each other's state
3. **Use real DB** (in-memory SQLite for speed) — not mocks for DB layer
4. **Mock external services** (Claude API, file system beyond fixtures)
5. **Arrange-Act-Assert** structure — clearly delineated

```python
def test_dedup_same_row_twice_returns_single_record(db_session, sample_csv):
    # DEDUP-01
    # Arrange
    pipeline = IngestPipeline(session=db_session)

    # Act
    pipeline.process(sample_csv)
    pipeline.process(sample_csv)  # same file again

    # Assert
    count = db_session.query(Transaction).count()
    assert count == len(expected_rows_in_sample)
```

---

## What Not to Test

- Framework boilerplate (FastAPI dependency injection, SQLAlchemy session management)
- Third-party library behaviour (rapidfuzz scoring, SHA-256 correctness)
- Trivial getters/setters with no logic
- Generated code (OpenAPI client, migration files)

Test the logic you wrote. Trust the libraries you use.

---

## CI Integration

Add to project CI (GitHub Actions or equivalent):
```yaml
- run: pytest --tb=short -q
- run: mypy --strict backend/app/domain/   # or equivalent type checker
- run: python scripts/generate_openapi.py  # validates all routes have operation_id
```

All three must pass before a phase is marked complete.
All three run on every push to main.
