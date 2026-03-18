---
name: testing-strategy
description: Risk-based test strategy, completeness matrix, coverage thresholds, and
  quality gates. Load when validating whether the RIGHT tests exist, not just
  whether tests pass. Used by Tester agent to assess test completeness.
---

# Testing Strategy Skill

## Core Principle

A passing test suite proves what was tested works. It says nothing about what was not tested.
This skill answers: **are the right things tested?**

---

## 1. Testing Pyramid

Target ratios for every project. Enforce before approving any phase.

| Layer | Target % | What it covers | Speed |
|-------|----------|----------------|-------|
| Unit | 70% | Pure functions, domain logic, isolated components | <1s per test |
| Integration | 20% | Service boundaries, DB queries, API contracts | 1–10s |
| E2E | 10% | Critical user journeys end-to-end | 10–60s |

**Pyramid violations to flag:**
- More E2E tests than integration tests → inverted pyramid (brittle, slow)
- Zero integration tests → over-reliance on mocks, integration gaps invisible
- Zero unit tests → no fast feedback loop, slow CI
- Unit tests that mock every dependency → unit tests in name only, no domain coverage

---

## 2. Risk-Based Scoping

Not all code needs equal coverage depth. Classify before testing.

| Risk tier | Criteria | Required depth |
|-----------|----------|----------------|
| **Critical** | Data mutation, payments, auth, core domain calculations, migration scripts | All 5 categories (see matrix below) |
| **Supporting** | Service orchestration, API handlers, background jobs | Happy path + error paths + boundary |
| **Utility** | Config loaders, formatters, simple helpers | Happy path + one error case |

Skip deep testing for: generated code, framework boilerplate, trivial getters/setters with no branching logic.

---

## 3. Test Completeness Matrix

Before marking a domain APPROVED, verify each required category is present.
Missing categories → status is PARTIAL, not PASS.

### Backend (API / service layer)
| Category | Required | Example |
|----------|----------|---------|
| Happy path | MUST | `POST /orders` with valid payload → 201 + persisted |
| Error paths | MUST | Invalid input → 422, auth failure → 401, DB error → 500 |
| Auth/permissions | MUST | Unauthenticated → 401; wrong role → 403 |
| Boundary values | MUST | Empty body, max-length string, 0 and negative numbers |
| Resilience (external calls) | MUST | Timeout → handled; service down → fallback or error response |

### Frontend (UI components)
| Category | Required | Example |
|----------|----------|---------|
| Interaction | MUST | Button click, form submit, keyboard navigation |
| Empty/loading state | MUST | No data → empty state message shown |
| Error state | MUST | API failure → error message rendered, not blank |
| Accessibility | MUST | See accessibility checklist below |
| Responsive layout | SHOULD | Renders correctly at mobile breakpoint |

### Database (migrations, queries, constraints)
| Category | Required | Example |
|----------|----------|---------|
| Migration up | MUST | Apply migration → schema matches expected |
| Migration down | MUST | Rollback migration → previous schema restored |
| Constraints | MUST | Duplicate key → error; nullable column → null allowed |
| Index performance | SHOULD | Query against indexed column uses index |
| Cascade behaviour | MUST (if FK exists) | Delete parent → child behaviour matches FK definition |

### Domain / pure functions
| Category | Required | Example |
|----------|----------|---------|
| Happy path | MUST | Correct output for typical valid input |
| Boundary values | MUST | 0, negative, maximum, empty collection |
| Invalid input | MUST | Wrong type / None / empty string → raises with clear message |
| Determinism | MUST (if stateful output possible) | Same inputs → same output always |
| Edge cases | MUST | Every edge case in the function spec |

---

## 4. Coverage Thresholds

Minimum targets. Below these → flag in report, do not auto-approve.

| Metric | Threshold | Tool reference |
|--------|-----------|----------------|
| Line coverage (new code) | 80% | SonarQube quality gate standard |
| Branch coverage (conditionals) | 75% | Measure every `if`/`else`/`switch` arm |
| Function coverage | 90% | Every exported function must be called in tests |

**How to read coverage reports:**
- `pytest-cov --json-report`: check `totals.percent_covered`
- `nyc/Istanbul` (JS): check `lines.pct`, `branches.pct` in `coverage-summary.json`
- `go test -cover`: check `coverage: X%` in output

Report format: include actual % next to threshold. Example: `lines: 73% (threshold: 80%) ⚠`

---

## 5. Quality Gates

Flag any of these in the report. They block APPROVED status.

| Gate | Rule |
|------|------|
| Cyclomatic complexity | No new function with complexity > 15 (SonarSource threshold) |
| Empty assertions | No test file with zero assertions |
| TODO in tests | No `TODO`, `FIXME`, or `pass` (Python) / `// TODO` (JS) left in test code |
| Skipped tests | All skipped tests must have a reason comment; mass-skipping blocks approval |
| Test isolation | No test that reads state written by another test (order-dependent failures) |
| Magic numbers | No unexplained numeric literals in assertions — use named constants or comments |

---

## 6. Resilience Testing Checklist

For every external dependency call (HTTP client, database, message queue, AI API):

- [ ] **Timeout**: what happens when the call takes too long? (assert: error handled, not hung)
- [ ] **Service unavailable** (500 / connection refused): assert fallback or propagated error
- [ ] **Partial response**: truncated or malformed JSON/body → assert error, not silent corruption
- [ ] **Invalid data**: response schema mismatch → assert validation error, not silent wrong state
- [ ] **Retry behaviour**: if retried, assert idempotency (no double-writes)

If a function calls an external service and has zero error-path tests: mark **PARTIAL**.

---

## 7. Accessibility Checklist (WCAG 2.2 AA, code-review level)

Check in rendered output or component test assertions. No external tooling required.

| Criterion | Check |
|-----------|-------|
| Images | Every `<img>` has non-empty `alt` (decorative images: `alt=""`) |
| Form labels | Every input has an associated `<label>` or `aria-label` |
| Heading hierarchy | `<h1>` → `<h2>` → `<h3>` — no skipping levels |
| Colour contrast | Text on background meets 4.5:1 ratio (AA); large text: 3:1 |
| Touch targets | Interactive elements ≥ 44×44 CSS pixels |
| Focus indicators | All focusable elements have visible `:focus` style |
| ARIA roles | `role` attributes match element semantics; no redundant `role="button"` on `<button>` |
| Error messages | Form validation errors are associated with their input via `aria-describedby` |

Flag missing items as PARTIAL. Do not block on SHOULD-level items (colour contrast advisory).

---

## 8. Property-Based Testing Guidance

Use when: pure functions, parsers, serialisers, financial calculations, sorting/ranking.
Skip when: functions with side effects, UI components, integration paths.

**Pattern (Hypothesis / Python):**
```python
from hypothesis import given, strategies as st

@given(st.floats(min_value=0.01, max_value=1e6), st.floats(min_value=0.01, max_value=1e6))
def test_tax_calculation_never_exceeds_gross(gross, rate):
    assert calculate_tax(gross, rate) <= gross
```

**Pattern (fast-check / TypeScript):**
```typescript
import fc from 'fast-check';
fc.assert(
  fc.property(fc.string(), s => parse(serialise(s)) === s)
);
```

Rule: if a function has a mathematical invariant (commutativity, idempotency, round-trip),
write a property test. Example invariants: `parse(serialise(x)) == x`, `sort(sort(x)) == sort(x)`.

---

## 9. Performance Baseline Template

Load testing is out of CI scope, but targets must be defined before approval.
Add to `docs/performance-targets.md` (or equivalent) for every external-facing endpoint.

```markdown
## [Endpoint / Feature Name]

| Metric | Target | Measured (date) |
|--------|--------|-----------------|
| p50 latency | < 100ms | — |
| p99 latency | < 500ms | — |
| Error rate (steady state) | < 0.1% | — |
| Throughput (peak) | X req/s | — |

Baseline measured with: [tool — k6, locust, wrk]
Last measured: [date]
```

Flag if a new endpoint has no entry in performance targets: add advisory note, do not block.

---

## Advisory Notes (Out of Scope for Gates)

- **Mutation testing** (Mutmut/Stryker): run periodically to validate test quality; not a CI gate due to execution cost
- **Contract testing** (Pact): add when multiple services share an API contract; requires Pact broker
- **Chaos engineering**: resilience checklist covers code-review layer; fault injection (Chaos Monkey) is advisory for production
- **LLM evals**: for AI-calling code, add PromptFoo or equivalent eval suite; see future SKILL-llm-evals.md
