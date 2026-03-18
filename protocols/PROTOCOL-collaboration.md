---
name: collaboration
description: Multi-agent session orchestration — agent selection, phase sequencing, mandatory Tester gate, Teacher invocation.
---

# Multi-Agent Collaboration Protocol

## Purpose
This protocol governs how multiple agents coordinate within a project. It defines: how to select agents for a project type, the mandatory sequencing rules, when Tester auto-runs, and when Teacher is invoked.

## Agent Selection by Project Type
Load the relevant framework file for opinionated defaults:
- Web app: `~/.claude/frameworks/FRAMEWORK-webapp.md`
- Mobile: `~/.claude/frameworks/FRAMEWORK-mobile.md`
- Game: `~/.claude/frameworks/FRAMEWORK-game.md`
- CLI: `~/.claude/frameworks/FRAMEWORK-cli.md`
- API service: `~/.claude/frameworks/FRAMEWORK-api-service.md`
- Finance tool: `~/.claude/frameworks/FRAMEWORK-finance-tool.md`
- Data pipeline: `~/.claude/frameworks/FRAMEWORK-data-pipeline.md`

## Standard Phase Sequence

### Discovery Phase (always first)
Agents: **PM** (requirements) → **Analyst** (data/research) → optional: **UX** (user journeys)
Output: User stories, acceptance criteria, non-functional requirements

### Architecture Phase
Agents: **Architect** (system design) → **Database** (schema) → **Security** (threat model)
Output: ADRs, system diagram, service contracts, schema, threat model

### Implementation Phase(s)
Agents: **Backend** + **Frontend** (parallel if independent) → **Database** (migrations)
Output: Working implementation passing unit tests

### **MANDATORY GATE: Tester Agent**
After EVERY phase produces deliverables:
1. Load `~/.claude/agents/AGENT-tester.md`
2. Run all tests + validate acceptance criteria
3. If BLOCKED: return to the relevant implementation agent; do not advance
4. If APPROVED: write handoff file and advance to next phase

**This gate cannot be skipped or deferred. No phase advances without Tester approval.**

### Review Phase
Agents: **Security** (audit) → **DevOps** (deployment readiness) → **SysAdmin** (if applicable)
→ Tester Gate (smoke tests and security baseline verification)

### Launch / Deployment
Agents: **DevOps** (deployment) → **Tester** (production smoke test)

## Parallel Agent Work
Agents may work in parallel when their outputs are independent:
- Backend + Frontend can run in parallel after architecture is settled
- Database + DevOps can run in parallel after architecture is settled
- Designer + UX can run in parallel at the start of a project

**Never run in parallel:**
- PM and Architect (Architect depends on PM's requirements)
- Database and Backend on the same table (race condition on schema)
- Two agents modifying the same file

## Teacher Agent Invocation
The Teacher agent is invoked:
1. **At project completion** — always
2. **After 3+ consecutive Tester failures** — systemic issue to diagnose
3. **Manually**: "Run teacher retrospective" command

Teacher invocation is NOT automatic per phase — it is a heavier process run at reflection points.

## Agent Selection — Automatic vs Manual

**Default: agents are selected automatically.** CLAUDE.md contains a task classifier
that routes to `AGENT-orchestrator.md` for new-project and add-feature tasks. No per-project
declaration is required. Do not ask the user to declare agents.

**Manual override** (advanced): if a project needs to pin specific agents or override
the orchestrator's selection, declare them in project CLAUDE.md:

```markdown
## Agents — Manual Override (optional)
| Agent | Load when | Phase |
|---|---|---|
| ~/.claude/agents/AGENT-architect.md | System design | 1 |
| ~/.claude/agents/AGENT-backend.md   | API implementation | 2–3 |
| ~/.claude/agents/AGENT-tester.md    | After every phase | All |
```

When a manual override is present, the orchestrator is skipped for that project.

## Session Start Checklist
At the start of every session:
1. Classify the task (CLAUDE.md Auto-Orchestration table)
2. For new-project / add-feature: read `AGENT-orchestrator.md`; let it produce the session plan
3. Read the current phase's agent file(s)
4. Read agent-memory/[agent]/memory.md + lessons.md for each active agent
5. Read any pending handoff files in agent-notes/
6. Read tasks/todo.md if it exists

**Total context load ≤ 8 files at any one time. Load skills on demand within agent sessions.**

## Conflict Resolution
When two agents disagree on a decision:
1. The agent earlier in the phase sequence wins (Architect overrides Backend on structure)
2. Security constraints override all other agents (security wins without discussion)
3. Escalate to user only when both agents have valid points and the tradeoff affects user-facing behaviour or the data model
