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
