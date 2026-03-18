---
name: api-service
description: Standalone API service framework — agents, phases, and skill loading for backend services and microservices.
---

# Framework: API Service

## Agent Roster
| Agent | Phase | Trigger |
|-------|-------|---------|
| PM / Analyst | 1 | Define API consumers and use cases |
| Architect | 1–2 | Service design, API contract, data model |
| Database | 2 | Schema and migration strategy |
| Backend | 2–4 | API implementation |
| Security | 3–4 | Auth design, vulnerability review |
| DevOps | 4–5 | CI/CD, containerisation, deployment |
| Tester | All | After every phase — mandatory gate |

## Phase Structure

### Phase 1 — API Design
Goal: API contract defined; consumers agreed; data model sketched.
Agents: PM/Analyst → Architect
Skills: SKILL-requirements, SKILL-system-design, SKILL-api-design
Key outputs:
- OpenAPI spec (draft) — endpoints, request/response shapes
- Auth strategy: API key / OAuth2 / JWT / mTLS
- Versioning strategy: URL path (`/v1`) vs header
- Rate limiting and quotas defined
- Error response schema standardised

### Phase 2 — Schema and Security Baseline
Goal: Database schema finalised; auth implemented; secrets managed.
Agents: Database + Security (parallel) → Backend (scaffolding)
Skills: SKILL-database, SKILL-security-audit, SKILL-security, SKILL-code-quality
Done when: schema migrated; auth tested end-to-end; TESTER APPROVED

### Phase 3 — Core Endpoints
Goal: All CRUD and business logic endpoints implemented and tested.
Agents: Backend
Skills: SKILL-code-quality, SKILL-tdd, SKILL-api-design, SKILL-logging
Standards:
- All endpoints: input validation, auth check, structured log entry, OpenAPI annotation
- Pagination on all list endpoints (cursor-based by default)
- Idempotency keys on mutation endpoints where relevant

### Phase 4 — Observability and Security Review
Goal: Production-ready: fully instrumented and security-approved.
Agents: Security → Backend (fix findings) → Tester
Skills: SKILL-security-audit, SKILL-logging
Instrumentation checklist:
- Health endpoint: `GET /health` → `{"status":"ok","version":"x.y.z"}`
- Metrics endpoint (Prometheus format) or cloud metrics
- Request ID in every response header and log line
- Distributed trace context propagated (OpenTelemetry)

### Phase 5 — Deploy
Goal: Service running in production with CI/CD pipeline.
Agents: DevOps → Tester
Skills: SKILL-devops
Include: Dockerfile, Kubernetes manifests or compose file, Terraform for cloud resources

## API Design Non-Negotiables
- ISO 8601 for all timestamps (`2026-03-18T10:00:00Z`)
- `application/json` content type unless file upload required
- Consistent error schema: `{"error": {"code": "X", "message": "Y", "details": {...}}}`
- 4xx for client errors (don't use 500 for validation failures)
- API changelog maintained — breaking changes require version bump

## Skill Loading Table
| Phase | Load these skills |
|-------|-------------------|
| 1 | SKILL-requirements, SKILL-system-design, SKILL-api-design |
| 2 | SKILL-database, SKILL-security, SKILL-code-quality |
| 3 | SKILL-code-quality, SKILL-tdd, SKILL-api-design, SKILL-logging |
| 4 | SKILL-security-audit, SKILL-logging |
| 5 | SKILL-devops |
