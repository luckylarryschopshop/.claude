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

---

### RICE Prioritisation

Use alongside MoSCoW for ranking features within the Must/Should tiers.

**RICE score = (Reach × Impact × Confidence) ÷ Effort**

| Factor | Meaning | How to estimate |
|--------|---------|----------------|
| **Reach** | How many users affected per quarter | From analytics or user research |
| **Impact** | How much it moves the metric | 3 = massive, 2 = significant, 1 = low, 0.5 = minimal |
| **Confidence** | How sure are you? | 100% = high evidence, 80% = some data, 50% = gut feel |
| **Effort** | Person-weeks to build | Engineering estimate |

```
Feature A: Reach=1000, Impact=2, Confidence=80%, Effort=2 → RICE = (1000 × 2 × 0.8) / 2 = 800
Feature B: Reach=5000, Impact=1, Confidence=50%, Effort=5 → RICE = (5000 × 1 × 0.5) / 5 = 500
```

Feature A scores higher despite lower reach — less effort and higher confidence.

Rules:
- Use RICE to rank items within a MoSCoW tier (don't use it to override Must/Won't decisions)
- Document the estimates — RICE is only as good as the inputs
- Revisit when any input changes significantly (new data, scope change)

---

### Definition of Ready (DoR)

A user story is only moved into implementation when it meets ALL of these:

- [ ] Story has a specific user type (not "user" or "admin")
- [ ] Acceptance criteria are written and testable
- [ ] All external dependencies are identified and available
- [ ] Designs / wireframes approved (if UI work)
- [ ] Non-functional requirements specified (performance targets, security requirements)
- [ ] Story is sized (effort estimated)
- [ ] "Won't do" boundary is explicit (what the story does NOT include)

A story that fails DoR goes back to PM/Analyst — it is not the engineer's job to define scope during implementation.

---

### A/B Test Design Checklist

Before running an experiment, these must be specified in the acceptance criteria:

- [ ] **Hypothesis**: "If we change [X], we expect [metric Y] to change by [Z%] because [reason]"
- [ ] **Primary metric**: one metric that determines success or failure (not a dashboard of 10)
- [ ] **Guardrail metrics**: metrics that must not degrade (e.g. revenue, error rate)
- [ ] **Minimum detectable effect (MDE)**: smallest change worth detecting (e.g. +2% conversion)
- [ ] **Sample size**: calculated from MDE, baseline rate, desired power (80%), significance (α=0.05)
  - Use a power calculator: `statsmodels.stats.proportion_power` or online tool
- [ ] **Test duration**: minimum 2 weeks to capture weekly patterns; do NOT stop early on positive results
- [ ] **Randomisation unit**: user, session, or device (must be consistent throughout)
- [ ] **Stopping rule**: test runs to planned end date regardless of interim results (no peeking)
- [ ] **Analysis method**: proportion test (conversions), t-test (continuous), Mann-Whitney (non-normal)

**Statistical significance ≠ practical significance.** A 0.1% lift that is "statistically significant"
may not be worth shipping. Report both p-value AND effect size.
