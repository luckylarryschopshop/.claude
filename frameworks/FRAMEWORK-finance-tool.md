---
name: finance-tool
description: Financial tool framework — agents, phases, and skill loading for finance, trading, and investment applications.
---

# Framework: Finance Tool

## Agent Roster
| Agent | Phase | Trigger |
|-------|-------|---------|
| PM | 1 | Define use cases, user types, compliance requirements |
| Finance | 1–2 | Model design and calculation logic |
| Trader | 1–2 | Strategy specification (if trading system) |
| Analyst | 1–2 | Data sources and reporting requirements |
| Architect | 2 | System design with compliance in mind |
| Database | 2 | Schema with audit trail and data classification |
| Backend | 3–4 | Calculation engine and API |
| Security | 3–4 | Financial data security, encryption, access control |
| Frontend | 3–4 | Reporting UI and dashboards |
| DevOps | 4–5 | Secure deployment, backup, compliance logging |
| Tester | All | After every phase — mandatory gate |

## Phase Structure

### Phase 1 — Requirements and Model Design
Goal: Financial requirements defined; calculation methodology agreed.
Agents: PM → Finance + Analyst (parallel) → Trader (if trading system)
Skills: SKILL-requirements, SKILL-financial-modelling, SKILL-trading-strategy (if applicable)
Key decisions:
- Data sources: market data vendor, internal data
- Calculation methodology: must be documented and auditable
- Regulatory constraints: which jurisdiction, which regulations apply?
- Data residency: where can financial data be stored?

### Phase 2 — Architecture with Compliance
Goal: System designed to meet regulatory and security requirements from the start.
Agents: Architect + Database + Security (parallel)
Skills: SKILL-system-design, SKILL-database, SKILL-security, SKILL-security-audit
Compliance considerations:
- Audit trail: every calculation must be traceable to its inputs
- Data retention: financial records must be retained per regulatory minimum (often 7 years)
- Access control: who can see what financial data? (need-to-know basis)
- Encryption: financial data at rest and in transit

### Phase 3 — Calculation Engine
Goal: Core financial calculations implemented, tested, and audited.
Agents: Backend (with Finance agent guidance)
Skills: SKILL-code-quality, SKILL-tdd, SKILL-financial-modelling, SKILL-logging
Critical: financial calculations must have property-based tests, not just unit tests
Precision: use decimal arithmetic (not floating point) for all financial calculations

### Phase 4 — UI and Complete
Goal: Reporting and dashboard complete; security review passed.
Agents: Frontend + Security → Tester
Skills: SKILL-frontend, SKILL-security-audit
Financial UI requirements:
- Numbers must clearly show currency, precision, and period
- Negative numbers must be clearly distinguished (colour alone is not accessible)
- Export functionality: CSV and PDF for all reports

### Phase 5 — Secure Deploy
Goal: Deployed with enhanced security posture for financial data.
Agents: DevOps → Security → Tester
Skills: SKILL-devops, SKILL-security
Additional requirements: encrypted backups, access logs, rate limiting on all financial endpoints

## Finance-Specific Non-Negotiables
- **Decimal arithmetic**: never use IEEE 754 floating point for monetary values
- **Audit log**: every financial transaction or calculation must be logged immutably
- **Reconciliation**: any system that moves or calculates money must have a reconciliation report
- **Data classification**: all financial data fields tagged at schema design time
- **Memory security**: agent-memory/finance/ and agent-memory/trader/ are gitignored

## Skill Loading Table
| Phase | Load these skills |
|-------|-------------------|
| 1 | SKILL-requirements, SKILL-financial-modelling |
| 2 | SKILL-system-design, SKILL-database, SKILL-security-audit |
| 3 | SKILL-code-quality, SKILL-tdd, SKILL-financial-modelling, SKILL-logging |
| 4 | SKILL-frontend, SKILL-security-audit |
| 5 | SKILL-devops, SKILL-security |
