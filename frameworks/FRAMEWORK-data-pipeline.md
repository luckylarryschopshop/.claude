---
name: data-pipeline
description: Data pipeline framework — agents, phases, and skill loading for ETL/ELT, data warehouses, and analytics infrastructure.
---

# Framework: Data Pipeline

## Agent Roster
| Agent | Phase | Trigger |
|-------|-------|---------|
| Analyst | 1 | Define data sources, metrics, and reporting needs |
| PM | 1 | Stakeholder requirements and SLAs |
| Architect | 2 | Pipeline architecture, warehouse design, orchestration |
| Database | 2 | Source schema analysis, warehouse schema, transformations |
| Backend | 3–4 | Ingestion code, transformation logic |
| Security | 3–4 | PII handling, data access control |
| DevOps | 4–5 | Orchestration deployment, monitoring |
| Tester | All | After every phase — mandatory gate |

## Phase Structure

### Phase 1 — Data Discovery and Requirements
Goal: Source systems understood; reporting requirements defined; SLAs agreed.
Agents: Analyst → PM
Skills: SKILL-requirements, SKILL-database (for source schema analysis)
Key questions:
- What decisions should this pipeline enable?
- What are the data sources? (databases, APIs, files, streams)
- What is the required freshness? (real-time, hourly, daily)
- What is the data volume? (rows/day, GB/day)
- Who are the consumers? (analysts, dashboards, downstream systems)

### Phase 2 — Architecture and Schema Design
Goal: Pipeline architecture decided; warehouse schema designed; orchestration tool chosen.
Agents: Architect + Database (parallel)
Skills: SKILL-system-design, SKILL-database
Key decisions:
- ETL vs ELT (transform in warehouse = ELT is usually better with modern warehouses)
- Orchestration: dbt + Airflow / Prefect / Dagster
- Warehouse: BigQuery / Snowflake / Redshift / DuckDB (choose for scale and cost)
- Medallion architecture: Bronze (raw) → Silver (cleaned) → Gold (business-ready)

### Phase 3 — Ingestion and Bronze Layer
Goal: All sources ingested to raw layer reliably.
Agents: Backend
Skills: SKILL-code-quality, SKILL-logging, SKILL-database
Requirements:
- Idempotent ingestion (safe to re-run)
- Schema evolution handling (new columns don't break pipeline)
- Failed run alerting and dead-letter storage for failed records
- Full row counts logged at each stage

### Phase 4 — Transformations (Silver + Gold)
Goal: Clean, tested, documented datasets ready for consumers.
Agents: Backend + Analyst (for metric definitions)
Skills: SKILL-code-quality, SKILL-tdd, SKILL-database
Standards:
- Every dbt model has: description, column docs, at least one test (not_null + unique on PK)
- Metric definitions are code (dbt metrics) not spreadsheets
- Data quality tests run on every pipeline execution

### Phase 5 — PII, Security, and Access Control
Goal: PII identified, handled, and access-controlled.
Agents: Security + Database
Skills: SKILL-security, SKILL-security-audit
- PII columns masked or hashed in Silver layer
- Access policies by role (analysts, ML engineers, downstream systems)
- Data lineage documented for GDPR right-to-erasure compliance

### Phase 6 — Orchestration and Monitoring
Goal: Pipeline runs reliably on schedule with alerting on failures.
Agents: DevOps → Tester
Skills: SKILL-devops, SKILL-logging
Monitoring:
- Row count anomaly detection (alert if today's ingestion is <50% or >200% of 7-day average)
- Freshness alerts (SLA breach if data older than agreed threshold)
- Cost monitoring (warehouse query costs)

## Data Pipeline Non-Negotiables
- **Idempotency**: every step is safe to re-run without duplicate data
- **Data lineage**: every Gold metric traceable to its Bronze source
- **PII classification**: done at schema design, not retrofitted
- **Tests in CI**: dbt tests run on every PR — no untested transformations

## Skill Loading Table
| Phase | Load these skills |
|-------|-------------------|
| 1 | SKILL-requirements, SKILL-database |
| 2 | SKILL-system-design, SKILL-database |
| 3–4 | SKILL-code-quality, SKILL-tdd, SKILL-logging, SKILL-database |
| 5 | SKILL-security, SKILL-security-audit |
| 6 | SKILL-devops, SKILL-logging |
