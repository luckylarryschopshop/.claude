# Retrospective — Agent Standards Upgrade — 2026-03-18

## Summary

Systematic review of all 15 agents and their skill files against established industry
standards. Primary finding: agents validated "does it pass?" but not "is the right
thing being tested/built/secured?". Applied ~25 targeted improvements.

---

## Pattern Analysis

| Pattern | Agents Affected | Severity |
|---------|----------------|----------|
| Tests validated pass/fail but not completeness | Tester | High |
| Backend had no domain-specific skill (agent: []) | Backend | High |
| Two incompatible error response formats in live skills | Backend, API Design | High |
| DORA metrics absent — no delivery health baseline | DevOps | High |
| OWASP LLM Top 10 absent from security skill | Security | High |
| DDD tactical vocabulary absent from architect skill | Architect | High |
| Pagination standard only documented offset strategy | Backend, API Design | Medium |
| Logging: text vs JSON not reconciled across skills | Backend, DevOps | Medium |
| Token storage / JWT security absent | All | Medium |
| Code review standards absent — no PR checklist anywhere | All | Medium |
| SLI/SLO/error budget absent | DevOps | Medium |
| Transaction isolation levels absent | Database | Medium |
| Cognitive load / error recovery absent from UX | UX | Low |
| Teacher had no pedagogical framework | Teacher | Low |

---

## Contradictions Fixed

| Issue | Resolution |
|-------|-----------|
| Error response format: api-design custom schema vs RFC 7807 in backend-patterns | Updated `SKILL-api-design.md` to RFC 7807 `application/problem+json` |
| Pagination: offset-only in api-design vs cursor+offset in backend-patterns | Updated api-design to document both with decision criteria |

---

## New Files Created

| File | Key content |
|------|------------|
| `agent-skills/SKILL-testing-strategy.md` | Testing pyramid (70/20/10), risk tiers, completeness matrix per domain, coverage thresholds, quality gates, resilience checklist, WCAG 2.2, property-based testing |
| `agent-skills/SKILL-backend-patterns.md` | RFC 7807 errors, idempotency keys, cursor vs offset pagination, API versioning + deprecation headers, background job retry/DLQ, rate limiting |

---

## Agent Skills Updated (by tier)

**Tier 1:**
- `SKILL-devops.md`: DORA metrics, deployment strategy matrix, GitOps, secret rotation, SLI/SLO/error budget
- `SKILL-security-audit.md`: OWASP LLM Top 10, SLSA supply chain levels, zero-trust checklist, Secure SDLC gates
- `SKILL-system-design.md`: DDD tactical patterns, CAP theorem + consistency models, event-driven (outbox/saga/event sourcing), strangler fig + ACL

**Tier 2:**
- `SKILL-database.md`: isolation levels, connection pool sizing formula, partitioning, MVCC
- `SKILL-infrastructure.md`: incident runbook template, CIS Benchmarks, capacity planning
- `SKILL-requirements.md`: RICE scoring, Definition of Ready, A/B test design checklist
- `SKILL-project-mgmt.md`: Definition of Done (15-gate), DORA as health signal

**Tier 3:**
- `SKILL-ux-research.md`: Kano model, cognitive load principles, error recovery patterns
- `SKILL-visual-design.md`: dark mode implementation, Gestalt principles critique checklist

**Global skills:**
- `SKILL-frontend.md`: Core Web Vitals (LCP/INP/CLS), state management matrix, bundle analysis
- `SKILL-logging.md`: text vs JSON reconciliation, structured JSON format, correlation IDs, PII redaction
- `SKILL-security.md`: JWT/token storage table, JWT security checklist, CORS configuration
- `SKILL-git.md`: PR size guidelines, PR description template, reviewer checklist, merge strategy
- `SKILL-tdd.md`: pyramid ratios, 80% coverage threshold, resilience test requirement

**Agent definitions:**
- `AGENT-tester.md`: completeness check as Step 2, extended report format (Coverage %, Category Matrix, Quality Gates)
- `AGENT-backend.md`: linked to SKILL-backend-patterns
- `AGENT-teacher.md`: Bloom's Taxonomy, Feynman Technique, spaced repetition escalation rule

---

## Proposed Future Work

| Topic | Value | Effort |
|-------|-------|--------|
| `SKILL-i18n.md` — locale, RTL, date/number formatting | Medium | Medium |
| `SKILL-webhooks.md` — event design, retry semantics, signature verification | Medium | Low |
| `SKILL-code-quality.md` — add SOLID principles explicitly | Low | Low |

---

## Key Insight

The most systemic gap: agents could approve work without asking whether the right
things were being done. The Tester, Backend, Security, and Architect agents were
most underspecified relative to available established standards. The error response
format contradiction was the highest-priority fix — it would have produced
inconsistent outputs across projects.
