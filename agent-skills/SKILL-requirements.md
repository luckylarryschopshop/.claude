---
name: requirements
description: Requirements gathering and specification — user stories, acceptance criteria, scope management. Load when acting as Analyst or PM agent.
---

# Requirements Skill

## Core Methodology

### The Requirements Hierarchy
```
Business Goal → User Need → Feature → User Story → Acceptance Criteria
```
Work top-down. Never write acceptance criteria before you understand the business goal.

### User Story Writing
Format: "As a [specific user type], I want [specific capability], so that [measurable outcome]."

Rules:
- **Specific user type** — not "user" or "admin". Who specifically? A new user? A power user? A billing admin?
- **Specific capability** — verb + noun + scope. Not "manage orders" but "filter orders by status and date"
- **Measurable outcome** — what changes for the user? Not "find things faster" but "reach any order in under 3 clicks"

### Acceptance Criteria Standards
Each criterion must be:
- **Testable** — can be verified mechanically, not subjectively
- **Specific** — one clear condition, not "and also" compound clauses
- **Observable** — describes system behaviour the user or tester can observe
- **Scoped** — covers success, failure, and edge cases

Gherkin format (optional, improves clarity):
```
Given [initial state]
When [user action]
Then [observable outcome]
And [additional observable outcome]
```

### Scope Management
**MoSCoW prioritisation:**
- **Must have** — without this, the product cannot launch
- **Should have** — high value, significant effort, can launch without but shouldn't ship to v2
- **Could have** — nice to have, cut when under pressure
- **Won't have** — explicitly deferred; write it down to prevent scope creep arguments

Write the "Won't have" list explicitly — it prevents the same deferred items from being re-raised every sprint.

### Non-Functional Requirements Template
Capture for every project:
| NFR | Requirement | Measurement |
|-----|-------------|-------------|
| Performance | [e.g. p99 API response < 500ms] | Load test at 2× expected peak |
| Availability | [e.g. 99.9% uptime] | Monitoring alert |
| Scalability | [e.g. support 10k concurrent users] | Load test |
| Security | [e.g. PII encrypted at rest] | Security audit |
| Compliance | [e.g. GDPR data residency] | Legal review |
| Accessibility | [e.g. WCAG 2.1 AA] | Automated + manual audit |

### Requirements Review Checklist
Before handing off to Architect:
- [ ] Every must-have has at least one acceptance criterion
- [ ] Non-functional requirements are specified with measurable targets
- [ ] Conflicting requirements have been surfaced and resolved
- [ ] External dependencies are listed with owners and timelines
- [ ] Constraints (regulatory, technical, budget) are documented
- [ ] "Won't have" list is written and agreed
- [ ] Success metrics are defined (how will we know this worked?)
