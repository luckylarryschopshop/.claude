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
5. Commit: `feat(phaseN): implement [module] — [TEST-IDs] pass`

If you find yourself writing implementation before tests: stop, write the test first.
If a test is hard to write, that is a signal the design needs simplifying.

---

## Test Naming Convention

```
test_[module]_[what]_[condition]
```

Examples:
```python
test_hash_same_inputs_returns_same_value
test_hash_different_key_produces_different_result
test_status_at_threshold_boundary_returns_amber
test_status_over_limit_returns_red
test_import_duplicate_row_skipped
test_import_chunk_failure_rolls_back
```

The name must be readable as a sentence describing the behaviour.

---

## Test ID Format

Test IDs allow tracing between spec and implementation.

Format: `[MODULE]-[TYPE]-[NN]`

Where TYPE is:
- `DOMAIN` — tests of pure domain functions
- (no type suffix) — integration/service/pipeline tests

Examples: `HASH-DOMAIN-01`, `IMPORT-03`, `STATUS-DOMAIN-02`

Every test function references its ID in a comment:
```python
def test_hash_same_inputs_returns_same_value():
    # HASH-DOMAIN-01
    ...
```

---

## Fixture Standards

- All fixtures use fake data only — never real user data, real names, or real credentials
- Fixture files live in `tests/fixtures/`
- Data files must match the real column structure of their source
- Use realistic-looking but obviously fake values:
  - Names: "FAKE COMPANY A", "TEST VENDOR B", "SAMPLE SERVICE CO"
  - Amounts: round numbers or simple decimals
  - Dates: use a consistent fake date range

```
tests/
  fixtures/
    source_a_sample.csv     ← realistic column structure, fake data
    source_b_sample.csv     ← realistic column structure, fake data
    sample_rules.yaml       ← sample config for rule-based tests
```

CONTRIBUTING.md must state: "Test fixtures must never contain real user data."

---

## Domain Function Test Requirements

Every domain function must be tested for:

1. **Happy path** — correct output for valid, typical input
2. **Boundary values** — 0, negative numbers, maximum reasonable values
3. **Invalid input** — wrong types, None, empty string → raises ValueError with clear message
4. **Determinism** — same inputs called twice → identical output (critical for hash functions)
5. **Edge cases** — every edge case documented in the function's spec

```python
class TestComputeHash:
    def test_returns_hex_string(self):
        # HASH-DOMAIN-01
        result = compute_hash("2024-01-15", "acme corp", -15.99)
        assert len(result) == 64
        assert all(c in "0123456789abcdef" for c in result)

    def test_deterministic(self):
        # HASH-DOMAIN-01 (determinism)
        a = compute_hash("2024-01-15", "acme corp", -15.99)
        b = compute_hash("2024-01-15", "acme corp", -15.99)
        assert a == b

    def test_different_key_different_hash(self):
        # HASH-DOMAIN-02
        a = compute_hash("2024-01-15", "acme corp", -15.99)
        b = compute_hash("2024-01-15", "other corp", -15.99)
        assert a != b

    def test_empty_key_raises(self):
        with pytest.raises(ValueError, match="key"):
            compute_hash("2024-01-15", "", -15.99)
```

---

## Integration Test Requirements

For service/pipeline tests:

1. **Single responsibility** — each test tests one behaviour
2. **Isolated** — tests do not depend on each other's state
3. **Use real DB** (in-memory for speed) — not mocks for DB layer
4. **Mock external services** (AI APIs, remote calls)
5. **Arrange-Act-Assert** structure — clearly delineated

```python
def test_import_duplicate_row_skipped(db_session, sample_file):
    # IMPORT-01
    # Arrange
    pipeline = ImportPipeline(session=db_session)

    # Act
    pipeline.process(sample_file)
    pipeline.process(sample_file)  # same file again

    # Assert
    count = db_session.query(Record).count()
    assert count == EXPECTED_ROWS_IN_SAMPLE
```

---

## What Not to Test

- Framework boilerplate (routing, session management)
- Third-party library behaviour (hashing correctness, fuzzy match scoring)
- Trivial getters/setters with no logic
- Generated code (client libraries, migration files)

Test the logic you wrote. Trust the libraries you use.

---

## CI Integration

Add to project CI:
```yaml
- run: pytest --tb=short -q
- run: mypy --strict [domain_directory]/   # or equivalent type checker
- run: python scripts/generate_openapi.py  # if project uses OpenAPI
```

All must pass before a phase is marked complete.
All run on every push to main.
