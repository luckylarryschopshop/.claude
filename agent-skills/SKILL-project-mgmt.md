---
name: project-mgmt
description: Project management — phase planning, risk management, dependencies, and progress tracking. Load when acting as PM agent.
---

# Project Management Skill

## Core Methodology

### Phase Planning Principles
- **Phases must produce something testable** — a phase ending with "design complete" has no gate; "design approved by stakeholder" does
- **Each phase has a single owner** — who is accountable for it being done?
- **Phases should be 1–2 weeks** — longer phases hide risk until it's too late
- **The last phase is not "testing"** — testing happens within each phase

### tasks/todo.md Structure
```markdown
# [Project] — Work Plan

## Phase [N]: [Name]
Goal: [one sentence — what's true when this phase is done]
Owner: [agent or human]
Done when: [specific, verifiable condition]

Tasks:
- [ ] [task — atomic, achievable in one session]
- [ ] [task]

Acceptance criteria:
- [ ] [criterion]

## Dependencies
| This phase needs | From | Status |
|-----------------|------|--------|
| [item] | [source] | [status] |

## Risks
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [risk] | H/M/L | H/M/L | [mitigation] |
```

### Risk Management
Risks must be assessed on two axes: likelihood AND impact.
- High likelihood + high impact → mitigate before the phase starts, not during
- Low likelihood + high impact → have a contingency plan written
- High likelihood + low impact → accept, document, monitor

**Common project risks by category:**
- **Dependency risk**: external API, third-party team, regulatory approval
- **Scope risk**: requirements that expand during implementation
- **Technical risk**: unproven technology, missing expertise
- **Timeline risk**: hard deadlines with variable scope

### Progress Tracking
Update `tasks/todo.md` at the end of every session:
- Mark completed items `[x]`
- Add discovered tasks immediately (don't hold them for the next planning session)
- Update the "Current State" section in project CLAUDE.md
- If a phase will slip: update Done When date + note reason

### Escalation Thresholds
Stop and notify when:
- A phase has been "in progress" for 2× its estimated duration
- A dependency is blocked for more than 3 days
- A risk has materialised that changes the project's feasibility
- Scope has grown by more than 20% without explicit approval

### Handoff Checklist (Phase to Phase)
Before marking a phase complete:
- [ ] All acceptance criteria met (verified, not assumed)
- [ ] Tester agent run and APPROVED
- [ ] Any discovered technical debt logged in project CLAUDE.md
- [ ] Next phase owner has been briefed on current state
- [ ] Blockers for the next phase have been resolved or have mitigation plans

---

### Definition of Done (DoD)

A phase or story is DONE only when ALL of these are true. Not "mostly done". Not "done except tests".

**Code:**
- [ ] All acceptance criteria verified (not assumed)
- [ ] Tester agent run → APPROVED (not PARTIAL, not BLOCKED)
- [ ] No regressions in previously passing tests
- [ ] Build/lint/typecheck passes with zero errors
- [ ] No TODO, FIXME, or commented-out code left in the diff

**Review and quality:**
- [ ] Code reviewed and approved (or peer-reviewed by another agent in the chain)
- [ ] Cyclomatic complexity within limits (no function > 15)
- [ ] Security review passed (input validation, auth checks, no secrets in logs)

**Documentation and observability:**
- [ ] New env vars added to `.env.example`
- [ ] Structured logging added at decision points
- [ ] API changes reflected in OpenAPI spec or equivalent

**Deployment readiness:**
- [ ] Migrations tested (up and down)
- [ ] Feature works in staging environment
- [ ] Performance target not regressed (check p99 if measurable)

**Anything not meeting DoD is not done — it is in progress.** Partial credit is not accepted
for the purpose of marking a phase complete.

---

### DORA as a Project Health Signal

Track these four signals as leading indicators of project health. Poor scores predict incidents
before they happen.

| Signal | What poor score means | Action |
|--------|----------------------|--------|
| Long lead time (>1 week commit → prod) | Pipeline bottleneck or large batch sizes | Break work into smaller shippable units |
| Low deployment frequency (<1/week) | Fear of deploying, manual gates, large batches | Automate deployment, add feature flags |
| High change failure rate (>15%) | Insufficient testing, too-large changes | Enforce test completeness, reduce batch size |
| Slow MTTR (>1 day) | Poor observability, unclear runbooks | Add alerting, write incident runbooks |
